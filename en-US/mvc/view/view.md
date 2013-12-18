---
name: Template Parsing
sort: 1
---

# Template Parsing

Beego uses Go's buildin package `html/template` as the template parser.  It will compile and cache the templates into map while Beego applicaion starts running.

## Template Directory

The default template directory of Beego is `views`. Templates file can be put in this directory. Beego will parse and cache these files automatically. In development mode it will parse everytime, no cache. The template directory can be changed by (it can only have one template directory):

	beego.ViewsPath = "myviewpath"

## Auto Rendering

You don't need to render and output template manually. Beego will call Render automatically after finishing the method. You can disable auto rendering in configuration file or in main.go if you don't need it.

In configuration file:

	autorender = false

In main.go:

	beego.AutoRender = false

## Template Tags

Go uses `{{` and `}}` as the default template tags. But sometime it will confilict with other template such AngularJS, to solve this we can use other tags. In configuration file:

	beego.TemplateLeft = "<<<"
	beego.TemplateRight = ">>>"

## Template Data

Template get its data from `this.Data` in Controller. So if you need `{{.Content}}` in template, you need to assign it in Controller first:

	this.Data["Content"] = "value"

Different rendering types:

- struct
	
  Struct variable:

		type A struct{
			Name string
			Age  int
		}
	
  Assign value in Controller:
			
		this.Data["a"]=&A{Name:"astaxie",Age:25}
		
  Render it in template:
	
		the username is {{.a.Name}} 
		the age is {{.a.Age}}
			
- map
	
  Assign value in Controller:
	
		mp["name"]="astaxie"
		mp["nickname"] = "haha"
		this.Data["m"]=mp

  Render it in template:
	
		the username is {{.m.name}}
		the username is {{.m.nickname}}
		
- slice

  Assign value in Controller:
	
		ss :=[]string{"a","b","c"}
		this.Data["s"]=ss
	
  Render it in template:
	
		{{range $key, $val := .s}}
		{{$key}}
		{{$val}}
	    {{end}}	

## Template Name

Beego uses Go's buildin template engine, so the syntax is as same as Go.  To learn more about template see [Templates](https://github.com/Unknwon/build-web-application-with-golang_EN/blob/master/ebook/07.4.md)。

You can set template name in Controller and Beego will find the template file under viewpath and render it automatically. In the config below, Beego will find add.tpl under admin and render it.

	this.TplNames = "admin/add.tpl"

Beego support `.tpl` and `.html` files by default. You need to set it in configuration if you are using other file extensions.

	beego.AddTemplateExt("file_extension_you_needed")

If TplNames is not set in Controller while auto render is enabled, Beego will use TplNames below by default:

	c.TplNames = c.ChildName + "/" + c.Ctx.Request.Method + "." + c.TplExt

It is Controller name + "/" + request method name + "." + template extension. So if the Controller name is `AddController`, request method is `POST` and the default template extension is `tpl`, Beego will render `/viewpath/AddController/post.tpl` template file.

## Layout Design

Beego supports layout design. E.g.: in you application the main navigation and footer are same and only the content part is different, you can use layout like this:

	this.Layout = "admin/layout.html"
	this.TplNames = "admin/add.tpl" 

In `layout.html` you must set variable like this:

	{{.LayoutContent}}
 
Beego will parse file of TplNames and assign it to `LayoutContent` then render `layout.html`.

Beego will cache all the template fiels. You can also implement layout by this way:

	{{template "header.html"}}
	处理逻辑
	{{template "footer.html"}}
	
## renderform

Define struct:

	type User struct {
		Id    int         `form:"-"`
		Name  interface{} `form:"username"`
		Age   int         `form:"age,text,age:"`
		Sex   string
		Intro string `form:",textarea"`
	}

* StructTag definition use `form` as tag. It uses the same tagas [Parse Form](controller/params.md#parse-to-struct). There are three optional params separated by ','.
  The first param is `name` attribute of the form field. If empty will use `struct field name` as the value.
  The second param is form filed type. If empty will use `text`.
  The third param is the tag of form field. If empty will use `struct field name` as the tag name.
* If `form` tag only has one value, it is the `name` attribute of form field. Except last value can be ignore all the other place must be separated by ','. E.g.: `form:",,username:"`
* 如果要忽略一个字段，有两种办法，一是：字段名小写开头，二是：`form` 标签的值设置为 `-`
* To ignore a field there are two ways:
  The first one is use lower case for field name.
  The second one is set `-` as the value of `form` tag.

controller：

	func (this *AddController) Get() {
	    this.Data["Form"] = &User{}
	    this.TplNames = "index.tpl"
	}

The param of Form must be a pointor of struct.

template:

	<form action="" method="post">
	{{.Form | renderform}}
	</form>

The code above will generate form below:
	
```
	Name: <input name="username" type="text" value="test"></br>
	Age: <input name="age" type="text" value="0"></br>
	Gender: <input name="Sex" type="text" value=""></br>
	Intro: <input name="Intro" type="textarea" value="">
```	
