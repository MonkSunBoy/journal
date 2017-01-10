# beego note

## program entry

### 常量

程序里经常会用到一些常量，比如说版本号等。我们需要把这些常量值定义在全局的常量列表中，就像beego.go中：

```
const (
	// VERSION represent beego web framework version.
	VERSION = "1.7.2"

	// DEV is for develop
	DEV = "dev"
	// PROD is for production
	PROD = "prod"
)
```

### 全局变量

程序里经常会用到一些全局的变量和操作全局变量的函数，我们也会把它定义到入口处：

```
//hook function to run
type hookfunc func() error

var (
	hooks = make([]hookfunc, 0) //hook function slice to store the hookfunc
)

// AddAPPStartHook is used to register the hookfunc
// The hookfuncs will run in beego.Run()
// such as sessionInit, middlerware start, buildtemplate, admin start
func AddAPPStartHook(hf hookfunc) {
	hooks = append(hooks, hf)
}
```

### 初始化全局变量

我们在程序刚开始执行的时候需要先初始化全局变量，完成初始化操作之后再启动整个程序

```
// Run beego application.
// beego.Run() default run on HttpPort
// beego.Run("localhost")
// beego.Run(":8089")
// beego.Run("127.0.0.1:8089")
func Run(params ...string) {

	initBeforeHTTPRun()

	if len(params) > 0 && params[0] != "" {
		strs := strings.Split(params[0], ":")
		if len(strs) > 0 && strs[0] != "" {
			BConfig.Listen.HTTPAddr = strs[0]
		}
		if len(strs) > 1 && strs[1] != "" {
			BConfig.Listen.HTTPPort, _ = strconv.Atoi(strs[1])
		}
	}

	BeeApp.Run()
}

func initBeforeHTTPRun() {
	//init hooks
	AddAPPStartHook(registerMime)
	AddAPPStartHook(registerDefaultErrorHandler)
	AddAPPStartHook(registerSession)
	AddAPPStartHook(registerTemplate)
	AddAPPStartHook(registerAdmin)
	AddAPPStartHook(registerGzip)

	for _, hk := range hooks {
		if err := hk(); err != nil {
			panic(err)
		}
	}
}
```

### registerMime

我们使用存储在mimemaps中的mime键值对初始化mime包
```
import (
	"mime"
)

//
func registerMime() error {
	for k, v := range mimemaps {
		mime.AddExtensionType(k, v)
	}
	return nil
}
```

注意到，mimemaps使用的是var声明的全局变量，但是这些值也不会改变，为什么不声明成常量呢？经过查询文档发现：

> There are boolean constants, rune constants, integer constants, floating-point constants, complex constants, and string constants. Rune, integer, floating-point, and complex constants are collectively called numeric constants.

常量只能是布尔/rune/整数/浮点数/复数/字符串。

```
### registerDefaultErrorHandler
// register default error http handlers, 404,401,403,500 and 503.
func registerDefaultErrorHandler() error {
	m := map[string]func(http.ResponseWriter, *http.Request){
		"401": unauthorized,
		"402": paymentRequired,
		"403": forbidden,
		"404": notFound,
		"405": methodNotAllowed,
		"500": internalServerError,
		"501": notImplemented,
		"502": badGateway,
		"503": serviceUnavailable,
		"504": gatewayTimeout,
	}
	for e, h := range m {
		if _, ok := ErrorMaps[e]; !ok {
			ErrorHandler(e, h)
		}
	}
	return nil
}

// ErrorHandler registers http.HandlerFunc to each http err code string.
// usage:
// 	beego.ErrorHandler("404",NotFound)
//	beego.ErrorHandler("500",InternalServerError)
func ErrorHandler(code string, h http.HandlerFunc) *App {
	ErrorMaps[code] = &errorInfo{
		errorType: errorTypeHandler,
		handler:   h,
		method:    code,
	}
	return BeeApp
}
```

