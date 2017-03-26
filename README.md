# DotWeb
简约大方的go Web微型框架

## 安装：

```
go get -u github.com/devfeel/dotweb
```

## 快速开始：

```go
func StartServer() error {
	//初始化DotServer
	dotserver := dotweb.New()
	//设置dotserver日志目录
	dotserver.SetLogPath("/home/logs/wwwroot/")
	//设置路由
	dotserver.HttpServer.Router().GET("/index", func(ctx *dotweb.HttpContext) {
		ctx.WriteString("welcome to my first web!")
	})
	//开始服务
	err := dotserver.StartServer(80)
	return err
}

```
## 特性
* 支持静态路由、参数路由
* 路由支持文件/目录服务，支持设置是否允许目录浏览
* 中间件支持
* 支持JSON/JSONP/HTML格式输出
* 统一的HTTP错误处理
* 统一的日志处理
* 支持Hijack与websocket
* 内建Cache支持
* 支持接入第三方模板引擎（需实现dotweb.Renderer接口）
* 支持维护配置，可设置维护欢迎语或维护跳转页

## 路由
特殊说明：集成github.com/julienschmidt/httprouter
#### 常规路由
* 支持GET\POST\HEAD\OPTIONS\PUT\PATCH\DELETE 这几类请求方法
* 支持HiJack\WebSocket\ServerFile三类特殊应用
* 支持Any注册方式，默认兼容GET\POST\HEAD\OPTIONS\PUT\PATCH\DELETE方式
* 支持通过配置开启默认添加HEAD方式
* 支持注册Handler，以启用配置化
```go
1、Router.GET(path string, handle HttpHandle)
2、Router.POST(path string, handle HttpHandle)
3、Router.HEAD(path string, handle HttpHandle)
4、Router.OPTIONS(path string, handle HttpHandle)
5、Router.PUT(path string, handle HttpHandle)
6、Router.PATCH(path string, handle HttpHandle)
7、Router.DELETE(path string, handle HttpHandle)
8、Router.HiJack(path string, handle HttpHandle)
9、Router.WebSocket(path string, handle HttpHandle)
10、Router.Any(path string, handle HttpHandle)
11、Router.RegisterRoute(routeMethod string, path string, handle HttpHandle)
12、Router.RegisterHandler(name string, handler HttpHandle)
13、Router.GetHandler(name string) (HttpHandle, bool)
```
接受两个参数，一个是URI路径，另一个是 HttpHandle 类型，设定匹配到该路径时执行的方法；
#### 静态路由
静态路由语法就是没有任何参数变量，pattern是一个固定的字符串。
```go
package main

import (
    "github.com/devfeel/dotweb"
)

func main() {
    dotserver := dotweb.New()
    dotserver.HttpServer.Router().Get("/hello", func(ctx *dotweb.HttpContext) {
        ctx.WriteString("hello world!")
    })
    dotserver.StartServer(80)
}
```
测试：
curl http://127.0.0.1/hello
#### 参数路由
参数路由以冒号 : 后面跟一个字符串作为参数名称，可以通过 HttpContext的 GetRouterName 方法获取路由参数的值。
```go
package main

import (
    "github.com/devfeel/dotweb"
)

func main() {
    dotserver := dotweb.New()
    dotserver.HttpServer.Router().Get("/hello/:name", func(ctx *dotweb.HttpContext) {
        ctx.WriteString("hello " + ctx.GetRouterName("name"))
    })
    dotserver.HttpServer.Router().Get("/news/:category/:newsid", func(ctx *dotweb.HttpContext) {
    	category := ctx.GetRouterName("category")
	    newsid := ctx.GetRouterName("newsid")
        ctx.WriteString("news info: category=" + category + " newsid=" + newsid)
    })
    dotserver.StartServer(80)
}
```
测试：
<br>curl http://127.0.0.1/hello/devfeel
<br>curl http://127.0.0.1/hello/category1/1


## 绑定
* HttpContext.Bind(interface{})
* 支持json、xml、Form数据
* 集成echo的bind实现模块
```go
type UserInfo struct {
		UserName string `form:"user"`
		Sex      int    `form:"sex"`
}

func(ctx *dotweb.HttpContext) TestBind{
        user := new(UserInfo)
        if err := ctx.Bind(user); err != nil {
        	 ctx.WriteString("err => " + err.Error())
        }else{
             ctx.WriteString("TestBind " + fmt.Sprint(user))
        }
}

```

## 中间件(拦截器)
#### RegisterModule
* 支持OnBeginRequest、OnEndRequest两类中间件
* 通过实现HttpModule.OnBeginRequest、HttpModule.OnEndRequest接口实现自定义中间件
* 通过设置HttpContext.End()提前终止请求

## 异常
#### 500错误
* 默认设置: 当发生未处理异常时，会根据DebugMode向页面输出默认错误信息或者具体异常信息，并返回 500 错误头
* 自定义: 通过DotServer.SetExceptionHandle(handler *ExceptionHandle)实现自定义异常处理逻辑
```go
type ExceptionHandle func(*HttpContext, interface{})
```

## Session
#### 支持runtime、redis两种
* 默认不开启Session支持
* runtime:基于内存存储实现session模块
* redis:基于Redis存储实现session模块,其中redis key以dotweb:session:xxxxxxxxxxxx组成
```go
//设置session支持
dotserver.SetEnabledSession(true)
//使用runtime模式
dotserver.SetSessionConfig(session.NewDefaultRuntimeConfig())
//使用redis模式
dotserver.SetSessionConfig(session.NewDefaultRedisConfig("127.0.0.1:6379"))
//HttpContext使用
ctx.Session().Set(key, value)
```

## Server Config
目前支持三个选项：Debug、Session、Gzip
* SetEnabledDebug 设置是否开启debug模式，会输出server端的debug日志，默认不开启
* SetEnabledSession 设置是否开启Session支持，目前支持runtime、redis两种模式，默认不开启
* SetEnabledGzip 设置是否开启Gzip支持，默认不开启
* EnabledListDir 设置是否启用目录浏览，仅对Router.ServerFile有效，若设置该项，则可以浏览目录文件，默认不开启
* EnabledAutoHEAD 设置是否自动启用Head路由，若设置该项，则会为除Websocket\HEAD外所有路由方式默认添加HEAD路由，默认不开启

## 外部依赖
websocket - golang.org/x/net/websocket
<br>
redis - github.com/garyburd/redigo/redis


## 相关项目
#### <a href="https://github.com/devfeel/tokenserver" target="_blank">TokenServer</a>
项目简介：token服务，提供token一致性服务以及相关的全局ID生成服务等

## 贡献名单
目前已经有几位朋友在为框架一起做努力，我们将在合适的时间向大家展现，谢谢他们的支持！

## 如何联系
QQ群：193409346
