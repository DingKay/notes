# Golang基础

## Go简介

Go又称Golang，是Google开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言 -- 维基百科

Go语言的特点：语法简洁，开发效率高，执行性能好（天生支持并发）

## 安装环境

 Go官网下载地址：https://golang.org/dl/

 Go官方镜像站（推荐）：https://golang.google.cn/dl/

```
go语言迭代更新比较快，推荐使用较新的版本，体验新特性。
```

`go version` 查看Go当前版本号

### 配置环境变量

#### GOPATH、GOROOT

`GOPATH` 和 `GOROOT` 都是环境变量，其中，`GOROOT` 是安装Go的路径，而从Go 1.8 版本开始，Go安装包在安装完成后会为 `GOPATH` 设置一个默认目录，参见下表：

| 平台    | GOPATH默认值     | 举例               |
| ------- | ---------------- | ------------------ |
| Windows | %USERPROFILE%/go | C:\Users\用户名\go |
| Unix    | $HOME/go         | /home/用户名/go    |

 我们只需要记住默认的GOPATH路径在哪里就可以了，并且默认情况下 `GOROOT`下的bin目录及`GOPATH`下的bin目录都已经添加到环境变量中了，我们也不需要额外配置了。 

并且在Go 1.11 版本开始，`go mod` 的出现 `GOPATH` 的配置显得不那么重要了。

`GOPATH`路径下

`src`：用来存放源码文件， 任何使用 `go get` 命令安装的包也将驻留在这里 (*及其依赖包*)。 

`pkg`：用来存放编译后生成的归档文件，他们以 `.a` 作为文件扩展名 (**.a** 代表 ***存档***)。

`bin`：用来存放编译后生成的可执行文件（可以将此目录添加到`PATH`中）

#### GOPROXY

 Go 1.14版本之后，都推荐使用`go mod`模式来管理依赖环境了，也不再强制我们把代码必须写在`GOPATH`下面的src目录了，你可以在你电脑的任意位置编写go代码。（网上有些教程适用于1.11版本之前） 

 默认 `GOPROXY` 配置是：`GOPROXY=https://proxy.golang.org,direct`，由于国内访问不到`https://proxy.golang.org`，所以我们需要换一个PROXY，这里推荐使用`https://goproxy.io`或`https://goproxy.cn`。 

 可以执行下面的命令修改GOPROXY： 

```go
go env -w GOPROXY=https://goproxy.cn,direct
```

## Go项目结构

在进行Go语言开发的时候，我们的代码总是会保存在 `$GOPATH/src` 路径下。在工程经过 `go build` 、 `go install` 或 `go get` 等指令后，会将下载的第三方包源码文件放在 `$GOPATH/src` 目录下，产生的二进制可执行文件放在 `$GOPATH/bin` 目录下，生成的中间缓存文件会被保存在 `$GOPATH/pkg` 下。

如果我们使用版本管理工具（Version Control System，VCS，如：Git）来管理我们的项目代码时，我们只需要添加 `$GOPATH/src` 目录的源代码即可。`bin` 和 `pkg` 目录的内容无须版本控制。

### 个人开发

在 `$GOPATH/src` 路径下，创建项目一、项目二、项目三...

### 目前流行的项目架构

Go语言中也是通过包来组织代码文件，我们可以引用别人的包，也可以发布自己的包，但是为了防止不同包的项目名冲突，我们通常使用 `顶级域名` 来作为包名的前缀，这样就不用担心项目名冲突的问题了。

例如：`$GOPATH/src` 路径下创建 `github.com/dingkay/goProject`

```go
import "github.com/dingkay/goProject"
```

从GitHub上下载别人包的时候，执行：

```go
go get github.com/dingkay/goProject
```

那么，这个包会下载到我们本地 `$GOPATH/src/github.com/dingkay/goProject` 目录

### 企业开发

`$GOPATH/src/{公司域名}/{公司部门、组}/{项目名}/{module}`

## IDE

Go采用的是 `UTF-8` 编码的文本文件存放源代码，理论上使用任何一款文本编辑器都可以做Go语言的开发，推荐使用 `VS Code` 和 `Goland`

## 第一个Go程序

路径 `$GOPATH/src/github.com/dingkay/study/day01/helloworld` 下创建 `main.go` 文件，代码如下：

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello World!")
}
```

在项目路径下执行 `go build` 编译成二进制可执行文件

问题：执行build命令时提示：`go: cannot find main module; see 'go help modules'`

解决方案：由于设置了 `GO111MODULE=on`，所以需要初始化 go mod

执行：`go mod init {项目名}` 例如：`go mod init hello` 生成了一个 go.mod 文件，然后再执行编译命令，编译成功。

原因：为了提高下载依赖包的速度，配置了 `GOPROXY`，设置了配置如下：

```go
// 开启go mod依赖管理 on/off/auto
go env -w GO111MODULE=on
// 配置依赖下载代理
go env -w GOPROXY=https://goproxy.cn,direct
```

这使得Go默认管理依赖的方式变成了 go module 模式，即依赖一个 go.mod 文件，其中描述了项目依赖的包和版本（类似于npm package.json，Java的maven pom.xml）

而目录中没有 go.mod 文件，所以Go不知道主模块是什么，所以无法编译

go语言在诞生之时，没有提供随之的包管理工具，而是使用 `go get` 来下载依赖包，并放在`$GOPATH/src`下，并且没有使用版本控制，每次都会拉取master分支的代码，软件包的代码放在src/github.com/xx/xx下面 

而Go 1.11 版本及之后的版本引入了Go模块（Go Modules），Go Modules使用 go.mod 中标记的软件包的版本，软件包的代码放在 `pkg/mod` 下面

### 使用Go Modules 还是 GOPATH

 Go使用一个环境变量 `GO111MODULE` 来决定使用Go Modules还是GOPATH，该变量有三个值，并在不同版本下有不同的语义

| 配置               | 1.11版本&1.12版本                                            | 1.13版本及以后                                               |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| GO111MODULE = on   | 不管在GOPATH 中还是外，都强制使用go.mod                      | ~                                                            |
| GO111MODULE = off  | 强制 Go 表现出 GOPATH 方式，即使在 GOPATH 之外               | ~                                                            |
| GO111MODULE = auto | 在GOPATH外时，GO111MODULE = on，在GOPATH内时，GO111MODULE = off | 当有go.mod或者在GOPATH 之外，GO111MODULE = on，当处于 GOPATH 内且没有 go.mod 文件时，GO111MODULE = off |

### 程序编译

在项目路径下执行 `go build`

编译成功后，会在当前路径下生成一个可执行的二进制文件

Window下默认编译成 `.exe`，如果需要在Windows中编译Linux程序，可以修改 `GOOS`

#### 非项目路径下编译程序

在其他路径下执行`go build`，需要在命令后加上项目的路径（`$GOPATH/src` 路径可以省略）

例如，项目路径为：`$GOPATH/src/github.com/dingkay/study/day01/helloworld`

则打包命令为：`go build github.com/dingkay/study/day01/helloworld`

打包成功后，二进制可执行文件生成与执行命令的路径下

![非项目路径下编译程序](images/非项目路径下编译程序.png)

```
Tips:在非项目路径下执行 go build，需要将 GO111MODULE 修改为 off 或者 auto
```

#### 跨平台编译

在Windows平台下，编译Linux平台的执行程序，或者MacOS平台下的执行程序

将配置 `CGO_ENABLED=1` 修改为 `CGO_ENABLED=0` 不禁用CGO可能会导致一些问题

修改 `GOOS=linux` （MacOS为 `GOOS=darwin`）

修改处理器架构 `GOARCH=amd64`

执行编译命令，则编译出来的二进制文件为Linux平台下的程序

#### 编译时指定生成的文件名称

执行命令：`go build -o {文件名}`

### 常用命令

直接执行 go 程序：`go run {文件名}`，此命令不常用，推荐使用 `go build` 编译后再启动。

编译并将二进制程序复制到 `$GOPATH/bin` 路径中： `go install`，同样 `go install` 命令在非项目路径下执行，在命令后加上目路径即可。由于我们将 `$GOPATH/bin` 添加到了 `$PATH` 环境变量中，我们可以在任何路径下执行我们编译后的二进制可执行文件。

## 基本语法

### 包（package）

每一段 Go 程序都 **必须** 属于一个包。一个标准的可执行的 Go 程序必须有 `package main` 的声明。如果一段程序是属于 main 包的，那么当执行 `go install` 的时候就会将其生成二进制文件，当执行这个文件时，就会调用 main 函数。如果一段程序是属于 main 以外的其他包，那么当执行 `go install` 的时候，就会创建一个 `包管理` 文件。

```
包的声明并非一定要与包名同名。因此，你可能会发现很多包的包名（文件夹名）与其内部声明的名字是不同的。当引用包的时候，需要使用包的声明来作为引用变量。
```

执行 `go install` 后，首先会在当前目录寻找 ` package main` 包声明的文件，Go就会将其编译成可执行的二进制文件。并将其复制到 `$GOPATH/src/bin` 路径下，一个包里面有很多的文件，但只能有一个 `main` 函数，其标志着一个程序的入口。

```go
package main

import "fmt"

func main() {
	fmt.Println("pkg/main/main.go => print")
}
```

![编译main包为可执行程序](images/go_install_编译main包为可执行程序.png)

如果没有 `main` 包声明的文件，那么Go就会在 `$GOPATH/pkg` 路径下创建 `包管理` （后缀 `.a`）文件。

```go
package app

import "fmt"

func main() {
	fmt.Println("pkg/appPrint/app.go => print")
}
```

则 `$GOPATH/pkg/windows_amd64/github.com/dingkay/study/day01/pkg` 

![编译非main包为包管理文件](images/go_install_编译非main包为包管理文件.png)

#### 包的命名规范

关于包的命名，Go 团队建议以简单，扁平为原则。例如， `strutils` 是 `string utility` 函数的名字， `http` 是 `HTTP` 请求的名字。

包的名字应避免使用下划线，中划线或掺杂大写字母。

#### 创建包

包分为两种：

可执行的包（main包）：可执行的包可以看作是主应用，编译后为可执行的程序。

工具包（非main包）：工具包自身是不可执行的，但是它会给主应用（可执行的包）增加一些功能，从而起到扩展主应用的作用。

#### 引用包

使用 `import`  语法引用包。Go 首先会在 `$GOROOT/src` 目录中搜索指定的包，如果找不到，再去 `$GOPATH/src` 目录中找。 `fmt` 包来自于Go的标准库，因此可以在 `$GOROOT/src` 路径中找到它。而不属于标准库中的包，在 `$GOROOT/src` 路径中找不到它，就会去 `$GOPATH/src` 中找

**greet**包

```go
package greet

var morning = "Good Morning"
var Morning = "Hey!" + morning
```

在main包中引入**greet**包

```go
package main

import (
	"fmt"

	"github.com/dingkay/study/day01/greet"
)

func main() {
	fmt.Println(greet.Morning)
}
```

引用包的方式主要有两种：

单独引包： `import "{packageName}"`

组合引包： `import ( "{packageName1}" "{packageName2}" "...")`

#### 包的访问权限

可以使用 `.` 符号来获取包的输出项，当一个包被引用的时候，Go会使用 `包的声明` 把这个包创建为全局变量。上述例子中，`greet` 就是全局变量，`greet.Morning` 可以直接访问输出项，但由于 `morning` 变量是 `greet` 包私有的， `greet.moring` 编译会报错，无法在包外访问私有的输出项。

##### 包的scope

![包的scope](images/包的scope.png)

main.go

```go
package main

import "fmt"

func main() {
	fmt.Print("version ==> " + version)
}
```

app.go

```go
package main

var version = "1.0.0"
```

 **scope 是指代码块中可以访问已定义变量的区域**。包的 scope 是指在一个包中 (包括包里的所有文件) 可以访问已定义变量的区域。这个区域是包中所有文件的顶层块。 

![同时执行多个go文件](images/go_run同时执行多个go文件.png) Go能够识别出应用的入口文件，因为它里面有 `main` 函数

尽管 `app.go` 文件中的 `version` 没有开头大写，但是我们依然可以在包内的任何地方访问它，因为它是在包的 scope 中声明的。如果 `version` 变量是在一个函数中声明的，那么它就不属于包的 scope 范畴，上面的程序就会出错。

**不可以在一个包中重复声明全局变量**。因此，一旦 `version` 声明了，在包的 scope 内，就不可以再次声明。但是你可以在 scope 以外的任何地方声明。

#### 包的嵌套

 可以在一个包中嵌套另一个包。因为在 Go 中，一个包就是一个目录，所谓嵌套，就相当于是在一个包中创建一个子目录。我们需要做的，就是指明其路径关系。 

```go
package main

import (
	"fmt"

	"github.com/dingkay/study/day01/pkgtest/pkg1"
	"github.com/dingkay/study/day01/pkgtest/pkg1/pkg2"
)

func main() {
	fmt.Println(pkg1.PkgMsg)
	fmt.Println(pkg2.PkgMsg)
}
```

输出结果：![包的嵌套](images/包的嵌套.png)

#### 包的别名

当引用包的时候，Go会根据包的声明来创建包的全局变量。如果引用多个重名包，就会导致冲突。

因此，我们需要使用 **包的别名**。在关键字 `import` 和包名之间，声明一个变量名，用以表示这个包。

```go
// pkgalias/greet
package greet

var Message = "i am pkgalias greet."
// pkgalias/greet/greet
package greet

var Message = "i am pkgalias/greet greet."
```

main.go

```go
package main

import (
	"fmt"

	_ "github.com/dingkay/study/day01/pkgalias/greet"
	subgreet "github.com/dingkay/study/day01/pkgalias/greet/greet"
)

func main() {
	fmt.Println(subgreet.Message)
}
```

在 Go 中，下划线是一个特殊的符号，表示 null 容器。如果我们引用了 greet 包，但是暂时没有用到它，Go 在编译的时候就会抛出这个问题。为了避免这个问题，我们就可以把暂时用不到的引用放在 _ 中，这样编译器就会忽略它。

给一个包加上 **下划线** 别名看似没有意义，但其实在某些情况下还是很有用的，比如在包初始化时用不到的变量。

#### 安装第三方包

例如将包发布到 `github.com` 上，本地下载的命令：

```
go get -u github.com/dingkay/study-go
```

上面这个命令的意思是，从 `https://github.com/dingkay/study-go` URL中引用文件，并保存到 `%GOPATH/src/github.com/dingkay/study-go` 目录中。

### 变量初始化

如果变量 `a` 依赖 `b` ，那么就要先初始化变量 `b`，否则无法编译。Go在函数内部会遵循这个原则

```go
package main

func main() {
	var a int = b
	var b int = c
	var c int = 6
}
```

执行失败：![变量初始化](images/变量初始化1.png)

但是在包的 scope 中定义的变量，它们在初始化周期中就已经被声明了。

```go
package main

import "fmt"

var a int = b
var b int = c
var c int = 9

func main() {
	fmt.Print(a, b, c)
}
```

执行结果：![变量初始化2](images/变量初始化2.png)

在上面的例子中，首先是变量 `c` 被赋值了。在下一个初始化周期时，变量 `b` 被赋值，因为它依赖于变量 `c`，而 `c` 已经被赋值了。在最后一次初始化周期中，变量 `a` 被赋予变量 `b` 的值。Go 可以处理复杂的初始化周期。

```go
package main

import "fmt"

var a int = b
var b int = f()
var c int = 2

func f() int {
	return c + 1
}

func main() {
	fmt.Print(a, b, c)
}
```

执行结果：![](images/变量初始化3.png)

注意，要避免像下面这样递归式的初始化循环

```go
package main

import "fmt"

var a int = b
var b int = c
var c int = a

func main() {
	fmt.Print(a, b, c)
}
```

执行结果：![变量初始化4](images/变量初始化4.png)

### Init 函数

跟 main 函数一样，当初始化包的时候，init 函数也会被执行。它没有参数，也没有返回值。init 函数是由 Go 来声明的，你无法引用这个函数（ 也不能用类似 *init()* 的方式调用它）。你可以在一个文件或一个包中定义多个 init 函数。在同一个文件中，init 函数是按照它们被定义的先后顺序被执行的。

```go
package main

import "fmt"

func init() {
	fmt.Println("main.go ==> init() [1]")
}

func init() {
	fmt.Println("main.go ==> init() [2]")
}

func init() {
	fmt.Println("main.go ==> init() [3]")
}

func main() {
	fmt.Println("main.go ==> main()")
}
```

执行结果：![init函数执行顺序](images/init函数执行顺序.png)

 你可以在包中的任何位置定义 `init` 函数。这些 `init` 函数以词法文件名的顺序（字母顺序）被调用。 

![](images/在包scope任何地方init.png)

a.go

```go
package subpkg

import "fmt"

var Test string = ""

func init() {
	fmt.Println("init ==> a.go")
}
```

f.go/z.go 略

init.go  在main.go中引用，触发初始化包 scope

```go
package subpkg

var InitMsg = "started init"
```

main.go

```go
package main

import (
	"fmt"

	"github.com/dingkay/study/day01/initfunc/pkgscope/subpkg"
)

func main() {
	fmt.Print(subpkg.InitMsg)
}
```

执行结果：![](images/init在包scope执行顺序.png)

 当所有的 `init` 函数被执行以后，`main` 函数才会被执行。因此， `init` 函数 的主要工作就是，初始化无法在全局范围内初始化的**全局变量** 。例如，初始化数组。 

```go
package main

import "fmt"

var arr [10]int

func init() {
	fmt.Println("main init arr")
	for i := 0; i < 10; i++ {
		arr[i] = i
	}
}

func main() {
	fmt.Println("main.go ==> start")
	fmt.Println(arr)
}
```

![初始化数组](images/init初始化数组.png)

特别强调一点，**被引用的包只能被初始化一次**。因此，如果你在一个包中，要引入很多包，每个被引用的包都只能在 main 包的生命周期中被初始化一次。

### 程序执行顺序

```
go run *.go
├── 执行 Main 包
├── 初始化所有引用的包
|  ├── 初始化所有引用的包 (recursive definition)
|  ├── 初始化全局变量
|  └── 以词法文件名的顺序调用 init 函数
└── 初始化 Main 包
   ├── 初始化全局变量
   └── 以词法文件名的顺序调用 init 函数
```

下面是一小例子：

```go
//declaration/version/version.go
package version

import "fmt"

func init() {
	fmt.Println("version/version.go ==> init")
}

var Version = "v1.2.1"

//declaration/version/fetch.go
package version

import "fmt"

var FerchVersion = fetchVersion()

func init() {
	fmt.Println("version/ferch.go ==> init")
}

func fetchVersion() string {
	fmt.Println("version/fetch.go fetchVersion")
	return Version
}
**************************************************
//declaration/entry.go
package main

import (
	"fmt"

	"github.com/dingkay/study/day01/declaration/version"
)

func init() {
	fmt.Println("declartion/entry.go ==> init")
}

func getVersion() string {
	fmt.Println("declartion/entry.go getVersion")
	return version.Version
}
//declaration/main.go
package main

import "fmt"

func init() {
	fmt.Println("declartion/main.go ==> init")
}

var myVersion = getVersion()

func main() {
	fmt.Println("declartion/main.go myVersion => " + myVersion)
}
```

执行结果：![初始化执行顺序](images/执行顺序.png)

`go run main.go entry.go` 首先根据 `package main` 知道编译成可执行文件（` go run` 编译的可执行文件存放于临时目录中），再寻找 `main` 方法的入口，递归查找依赖的 **包**，初始化 `version` 包，`fetch.go` 以及 `fetch.go` 中的全局变量首先初始化，其次是根据**文件名**的字母顺序，依此执行 `init` 初始化方法，依此规则，向上初始化 `main` 包，由于 `entry.go` 文件中没有全局变量需要初始化，但 `main.go` 文件中的全局变量依赖 `entry.go` 中的方法，故打印的输出是 `entry.go` 中的 **getVersion**，变量初始化结束后，则依据规则初始化 `init`，最后执行 `main` 方法的输出。

### 注释

Go 的注释规则跟 `JavaScript` 或 `c++` 一样。单行注释用 `//comment`，多行注释用 `/*comment*/`。

```
// 单行注释
// 有一条单行注释

/*
    这里是多行注释。
*/
```

### 分号

Go 的语法也是用分号来标记一个语句的结局，但与 C 不同的是，分号不需要写在源码里。因为 Go 的 Lexer 程序会在扫描程序的时候，根据一定的规则自动插入分号。

插入分号的规则是，如果换行符之前的最后一个标记是标识符（ 包括像 *int* 和 *float64* ），普通的字符像 数字 或 字符串，或者下面标记中的任意一个

```
break continue fallthrough return ++ -- ) }
```

**Lexer** 就会在标记后面插入分号。一句话总结， “**如果换行符在可能结束语句的标记之后，则插入分号**”.

所以，Go 是不需要写分号的。如果你不小心写了分号，VSCode 插件也会在保存时把分号去掉。你也可以使用 [**gofmt**](https://golang.org/cmd/gofmt/) 命令来格式化代码。唯一需要用到分号的地方就是 switch 语句和 for 循环。因为必须用分号来明确指出在哪里结束语句。

例如：

```
package main

import (
	_ "fmt" _ "math"
)

func main() {}
```

`go build` 时，会提示缺少分号；

![Go语言分号](images/分号.png)

当 `_ "fmt" _ "math"` 分两行写时，或者在中间添加 `;` 标识，则可以通过编译

```go
package main

import(
    "fmt"                           // ;
    "math"                          // ;
)                                   // ;

func main() {
    fmt.Println(math.Sqrt(16))      // ;
}                                   // ;
```

Go 是一门非常注重简洁和系统性的语言。所以你的代码必须遵守社区准则和 Go 的规则。 [gofmt](https://golang.org/cmd/gofmt/) 是一个可以自动格式化代码的工具，可以帮助你书写更加规范的 Go 代码。

### 变量&数据类型

#### 什么是变量

 一个变量就是一个指定类型的存储单元。你可以给这个存储单元命名。变量定义时必须指定数据类型。例如， `string` 就是用于存储字符串的数据类型。 

#### 数据类型

Go包含很多种数据类型：

| 类型       | 说明                                                         | 默认值        |
| ---------- | ------------------------------------------------------------ | ------------- |
| bool       | 布尔类型。值为 `true` 或 `false`                             | false         |
| string     | 字符串类型。值为 `UTF-8` 编码的字符串，Go中所有的字符串的默认编码都是 `UTF-8`。与 `JavaScript` 不同，只有字符串可以用双引号。 | `''` 空字符串 |
| int        | 整型。值为 `32-bit` 或 `64-bit` 的整数。系统会为 `32-bit` 分配 `32` 个子节的内存空间。为 `64-bit` 分配 `64` 个子节的内存空间。因此， `32-bit` 可以存储 `-2147483648 ~ 2147483647` （-2的31次方 ~ 2的31次方 - 1）的数字，而 `64-bit` 可以存储 `-9223372036854775808 ~ 9223372036854775807` （-2的63次方 ~ 2的63次方-1）的数字；分配 `32-bit` 或者 `64-bit` 是根据系统而定的； | 0             |
| uint       | 整型。与 `int` 一样， `uint` 也能存储 32 个字节或 64 个字节，但是是**非负值** 的整数。 | 0             |
| int8       | 整型。 系统会分配 8 个字节的内存来存储整数。因此，能够存储 -128 到 127 的数字。 | 0             |
| uint8      | 整型。与 `int8` 一样，`uint8` 能够存储 8 个字节的 **非负值** 整数。 因此，它能够存储的数字是 0 到 255。 | 0             |
| int16      | 整型。系统会为其分配 16 个字节的存储空间用于存储整数。因此，它能够存储 -32768 到 32767 的数字。 | 0             |
| uint16     | 整型。 与 `int16` 一样，`uint16` 能够存储 8 个字节的 **非负值** 整数。 因此，它能够存储的数字是 0 到 65535。 | 0             |
| int32      | 整型。 系统会分配 32 个字节的内存来存储整数。因此，能够存储 -2147483648 到 2147483647 的数字。 | 0             |
| uint32     | 整型。 与 `int32` 一样，`uint32` 能够存储 32 个字节的 **非负值** 整数。 因此，它能够存储的数字是 0 到 4294967295 的数字。 | 0             |
| int64      | 整型。 系统会分配 64 个字节的内存来存储整数。因此，能够存储 -9223372036854775808 到 922337203685477580 的数字。 | 0             |
| uint64     | 整型。 与 `int64` 一样，`uint64` 能够存储 64 个字节的 **非负值** 整数。 因此，它能够存储的数字是 0 到 18446744073709551615。 | 0             |
| float32    | 浮点型。系统会为其分配 32 个字节的内存空间用来存储浮点数。因此，它能够存储的数值是 -3.4E+38 到 +3.4E+38。 | 0             |
| float64    | 浮点型。系统会为其分配 64 个字节的内存空间用来存储浮点数。因此，它能够存储的数值是 -1.7E+308 到 +1.7E+308。 | 0             |
| complex64  | Go 还支持复合数据类型。 `complex64` 由 `float32` 的实数部分和 `float32` 的虚数部分构成。 | 0+0i          |
| complex128 | 与 `complex64` 类似，`complex128` 由 `float64` 的实数部分和 `float64` 的虚数部分构成。 | 0+0i          |
| byte       | `uint8` 的别名。                                             | 0             |
| rune       | `int32` 的别名。它代表一个 Unicode 字符。                    | 0             |

#### 特殊整型

`uint`、`int`、在不同的操作系统上可存储的大小是不同的，在32位操作系统上是**int32**，在64位操作系统上是**int64**

`uintptr`无符号整型，用于存放一个指针

注意：在使用`int`或者`uint`时，不能假定它为32位或者64位，需要考虑在不同平台上面的差异性。

获取对象的长度的内建`len()`函数返回的长度可以根据不同平台的字节长度进行变化。实际使用中，切片或 map 的元素数量等都可以用`int`来表示。在涉及到二进制传输、读写文件的结构描述时，为了保持文件的结构不会受到不同编译目标平台字节长度的影响，不要使用`int`和 `uint`。 

#### 零值

在其他编程语言中，对于未初始化的变量，系统会赋予其 `null` 或者 `undefined`，在 go 中， 则是赋给这些变量符合其数据类型的零值。参照上表，布尔类型的变量就会被赋值为 `false`，整形变量会被赋值为 `0`。

#### 数字字面量语法（Number literals syntax）

Go1.13版本之后引入了数字字面量语法，这样便于开发者以二进制、八进制或十六进制浮点数的格式定义数字，例如：

`v := 0b00101101`， 代表二进制的 101101，相当于十进制的 45。 `v := 0o377`，代表八进制的 377，相当于十进制的 255。 `v := 0x1p-2`，代表十六进制的 1 除以 2²，也就是 0.25。

而且还允许我们用 `_` 来分隔数字，比如说： `v := 123_456` 表示 v 的值等于 123456。

我们可以借助fmt函数来将一个整数以不同进制形式展示。

```go
package main

import "fmt"

func main() {
	// 十进制
	var a int = 10
	fmt.Printf("%d \n", a)
	fmt.Printf("%b \n", a)

	// 八进制
	var b int = 077
	fmt.Printf("%o \n", b)
	fmt.Printf("%d \n", b)

	// 十六进制
	var c int = 0xff
	fmt.Printf("%x \n", c)
	fmt.Printf("%X \n", c)
}
```

#### 标识符

Go语言中标识符由字母数字和`_`(下划线）组成，并且只能以字母和`_`开头。 举几个例子：`abc`, `_`, `_123`, `a123`。

Go语言中有25个关键字：

```go
    break        default      func         interface    select
    case         defer        go           map          struct
    chan         else         goto         package      switch
    const        fallthrough  if           range        type
    continue     for          import       return       var
```

此外，Go语言中还有37个保留字：

```
    Constants:    true  false  iota  nil

       Types:    int  int8  int16  int32  int64  
               uint  uint8  uint16  uint32  uint64  uintptr
               float32  float64  complex128  complex64
               bool  byte  rune  string  error

    Functions:   make  len  cap  new  append  copy  close  delete
                 complex  real  imag
                 panic  recover
```

#### 变量

Go语言中的变量需要声明后才能使用，同一作用域内不支持重复声明。 并且Go语言的变量 `声明后局部变量必须使用`。

```go
package main

var name = "Ding"

var (
	age    int  = 23
	gender byte = 1
)

func main() {
	var a = 10
}
```

![声明的局部变量必须使用](images/声明的局部变量必须使用.png)

全局变量可以声明但不使用，局部变量声明了后必须有地方引用，不然编译时会提示错误信息。

##### 变量声明

标准的声明格式如下：

`var 变量名 数据类型 = 值` 使用关键字 `var` 声明变量

在声明时也可以不带数据类型，Go语言会根据赋予的初始值默认类型推导出相应的类型。

```go
package main

import "fmt"

// 在一条语句中，同时声明多个同类型的变量同时赋值
var var1, var2, var3 int = 3, 4, 5
// 去掉数据类型，让 go 自动推导出变量的类型
var var4, var5, var6 = 1, 2.2, false

func main() {
	// 标准声明
	var bol bool = true
	// 类型推导（根据值判断该变量是什么类型）
	var str = "string type"
	// 简短声明
	a := "a word"
	// 批量声明
	var (
		test1        = "test1"
		test2 string = "test2"
	)
	var7, var8, var9 := 1, 2, 3
	fmt.Println(bol)
	fmt.Println(str)
	fmt.Println(a)
	fmt.Println(test1 + test2)
	fmt.Println(var7 + var8 + var9)
}
```

其中， `简短声明 :=` 只能在函数里面使用。 `:=` 符号左侧的变量中，必须至少有一个变量是第一次声明的变量。 

 通过这种简写语法，可以非常方便地实现在运行时给新变量赋值。比如像下面这样，把一个函数的返回值直接赋值给新变量。 

```go
package main

import (
	"fmt"
	"math"
)

func main() {
	a, b := 145.8, 543.8
	c := math.Min(a, b)
	fmt.Println("minimum value is ", c)
}
```



##### 变量名称的约定

 关于变量的命名，Go 建议用 `单个` 单词或者 `小驼峰` 的形式。用 `下划线` 不会产生语法错误，但是不推荐。 

##### 类型转换

 Go 不支持不同类型变量之间的加减法等任何算数运算。要想进行算数运算，必须先转成同样的数据类型。Go 支持这种数据类型转换， 语法格式为 `type(var)`. 

```go
package main

import "fmt"

func main() {
	var1 := 10   // int
	var2 := 10.5 // float

	// 非法的
	// var3 := var1 + var2

	// 合法的
	var3 := var1 + int(var2) // var3 == 20

	fmt.Println(var3)
}
```



##### 类型别名

 只要按照下面的语法格式，就可以对任何系统自带的或用户自定义的类型声明别名

` type aliasName aliasTo `

在上面的语法格式中，`aliasName` 是你想要自定义的类型别名。 `aliasTo` 是定义新类型的参照类型。

例如，你想要自定义一个跟 `float64` 功能一样的 `float` 类型，就可以这样声明

` type float float64 `

在上面的程序中，Go 会创建一个 `float64` 的副本，并命名为 `float` 类型。你可以像使用普通的 Go 类型一样，使用这个类型来声明变量。由于 Go 自身没有 `float` 类型名，而我又很讨厌每次都要写 `float64`，所以我通常都会这么自定义一下。

```go
package main

import "fmt"

type float float64

func main() {
	var f float = 52.2

	fmt.Printf("f has value %v and type %T", f, f)
}
```

在上面的代码中，我们把 `float` 定义为 `float64` 类型的别名，并创建了一个名为 `f`，类型为 `float`，初始值为 `52.2` 的变量。

输出：![类型别名](images/类型别名.png)

如果自定义的类型别名的首字母是小写字母，那么这个类型就不能被其他包所引用。

而且，尽管上面定义的 `float` 和 `float64` 有着同样的属性，但是两个类型的值也是不能做比较运算的，因为它们是不同的类型。

```go
package main

import "fmt"

type float float64

func main() {
	var f float = 52.2
	var g float64 = 52.2

    // fmt.Println("f == g", float64(f) == g)
	fmt.Println("f == g", f == g)
}
```

编译输出：![别名类型比较](images/别名类型比较.png)



#### 常量

 在 Go 中，常量就是有固定值的变量。任何试图改变常量值的行为都会引起运行出错。常量必须用 `const` 关键字声明并赋值。 

`  const const_name [data_type] = constant_value `

```
在 Go 中，必须在开始编译之前给常量赋值。不能用函数的返回值给常量赋值。 const a = math.Sqrt(4) 是非法语句，因为 a 的值是要在运行中计算出来的。
```

##### 批量声明常量

 像变量一样，也可以在一个语句中，同时声明多个常量。 

```go
package main

import "fmt"

const constant = "constant value"

const (
	constant1 = "constant value1"
	constant2 = "constant value2"
)

const (
	a = 1 // a==1
	b = 2 // b==2
	c     // c==2
	d     // d==2
)

func main() {
	fmt.Println(a, b, c, d)
}
```

 在带括号的 const 声明中，可以隐式重复表达式 - 这表示 `重复前面的表达式`。 

所以，`c` 与 `d` 的值等于 `b` 的值

![](images/批量声明常量.png)

##### iota

 Go 的关键字 `iota` 可以用来声明枚举常量。这个关键字可以生成一系列自增值，从 0 开始，每次声明增加 1。 

```go
const (
	a = iota // a == 0
	b = iota // b == 1
	c = iota // c == 2
	d        // d == 3 ( 相当于 d = iota )
)
```

 可以用  `_ ( 空白标识符 )` 跳过值。 `_` 会接收 `iota + 1` 的值，然后丢弃。 

```go
const (
    a1 = iota + 1 // iota =0; a == 0 + 1
    _             // iota =1;(implicitly _ = iota + 1 )
    b1            // iota =2;b == 3 (implicitly b = iota + 1 )
    c1            // iota =3;c == 4 (implicitly c = iota + 1 )
)
```



### 字符串

 Go语言中的字符串以原生数据类型出现，使用字符串就像使用其他原生数据类型（int、bool、float32、float64 等）一样。 Go 语言里的字符串的内部实现使用`UTF-8`编码。 字符串的值为`双引号(")`中的内容，可以在Go语言的源码中直接添加非ASCII码字符，例如： 

```go
var t1 string = "中文"
var t2 string = "en"
```

#### 字符串转义

Go 语言的字符串常见转义符包含回车、换行、单双引号、制表符等，如下表所示。

| 转义符 |                含义                |
| :----: | :--------------------------------: |
|  `\r`  |         回车符（返回行首）         |
|  `\n`  | 换行符（直接跳到下一行的同列位置） |
|  `\t`  |               制表符               |
|  `\'`  |               单引号               |
|  `\"`  |               双引号               |
|  `\\`  |               反斜杠               |

举个例子，我们要打印一个Windows平台下的一个文件路径：

```go
var s1 string = "c:\\User\\go"
fmt.Println(s1)
```

#### 多行字符串

 Go语言中要定义一个多行字符串时，就必须使用`反引号`字符：

```go
var s2 string = `
	c:\User\go
	hello
	`
fmt.Println(s2)
```

 反引号间换行将被作为字符串中的换行，但是所有的转义字符均无效，文本将会原样输出。 

#### 字符串的常用操作

|                方法                 |      介绍      |
| :---------------------------------: | :------------: |
|              len(str)               |     求长度     |
|           +或fmt.Sprintf            |   拼接字符串   |
|            strings.Split            |      分割      |
|          strings.contains           |  判断是否包含  |
| strings.HasPrefix,strings.HasSuffix | 前缀/后缀判断  |
| strings.Index(),strings.LastIndex() | 子串出现的位置 |
| strings.Join(a[]string, sep string) |    join操作    |

### byte和rune类型

组成每个字符串的元素叫做“字符”，可以通过遍历或者单个获取字符串元素获得字符。 字符用单引号（’）包裹起来，如： 

```go
package main

import "fmt"

func main() {
	c1 := '中'
	c2 := 'z'
	fmt.Printf("%T \t %v \t %x \n", c1, c1, c1)
	fmt.Printf("%T \t %v \t %x \n", c2, c2, c2)
}

// output:
int32    20013   4e2d 
int32    122     7a
```

Go 语言的字符有以下两种：

1. `uint8`类型，或者叫 byte 型，代表了`ASCII码`的一个字符。
2. `rune`类型，代表一个 `UTF-8字符`。

当需要处理中文、日文或者其他复合字符时，则需要用到`rune`类型。`rune`类型实际是一个`int32`。 

Go 使用了特殊的 `rune` 类型来处理 Unicode，让基于 Unicode 的文本处理更为方便，也可以使用 byte 型进行默认字符串处理，性能和扩展性都有照顾。 

```go
func traversalString() {
	str := "你好，我是golang"

	// byte --> uint8
	fmt.Printf("%T \t %v \n", str[0], str[0])
	for i := 0; i < len(str); i++ {
		fmt.Printf("%v(%c) ", str[i], str[i])
	}
	fmt.Println()
	s := []rune(str)[0]
	// rune --> int32
	fmt.Printf("%T \t %v \n", s, s)
	for _, char := range str {
		fmt.Printf("%v(%c) ", char, char)
	}
}
// output
uint8    228
228(ä) 189(½) 160( ) 229(å) 165(¥) 189(½) 239(ï) 188(¼) 140() 230(æ) 136() 145() 230(æ) 152() 103(g) 111(o) 108(l) 97(a) 110
(n) 103(g)
int32    20320
20320(你) 22909(好) 65292(，) 25105(我) 26159(是) 103(g) 111(o) 108(l) 97(a) 110(n) 103(g)
```

因为UTF8编码下一个中文汉字由3~4个字节组成，所以我们不能简单的按照字节去遍历一个包含中文的字符串，否则就会出现上面输出中第一行的结果。

字符串底层是一个byte数组，所以可以和`[]byte`类型相互转换。字符串是不能修改的 字符串是由byte字节组成，所以字符串的长度是byte字节的长度。 rune类型用来表示utf8字符，一个rune字符由一个或多个byte组成。