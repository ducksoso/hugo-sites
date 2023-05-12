---
title: "Go Note Magic"
date: 2023-05-12T14:36:32+08:00
draft: false
---



# Go注释的魔力

-----

## 注释语法

----

继承于 C 语言的语法，Go 支持常见的单行注释和多行注释，使用 `// `和` /* ... */`

```go
// 行尾的所有内容都是注释
var foo int;  // 注释可以从任一行中的任何地方开始

/*
多行注释
*/
```

尽管Go编译器会忽略注释的内容，但这并不意味着Go工具集会完全忽略注释的内容。本文将会介绍一些特殊格式的注释的使用。



## 1. Godoc 文档文本

----

Go 中最常见的「神奇」注释形式可能是对 Go 内置文档工具 godoc 的注释。godoc 的工作原理是扫描所有 .go 的数据。在一个包中的文件 (忽略任何 _test.go 文件) 中查找声明之前的注释 (没有任何中间代码或空行)。godoc 将使用注释的文本来形成包的文档。

例如，要记录一个函数，只需在其声明之前的行上放置一行或多行注释：

```go
/ Foo 将是 foo 给定的字符串，如果字符串不能被 foo 赋值
// 返回一个 error
func Foo(s string) error {
    ...
}
```



在 **导出** 和 **包级 **类型、函数、方法、变量或常量声明之前，可以使用相同的注释样式：

```go
package objects

// Object是一个通用的东西
type Object struct {}

// Bar将是bar的Object，如果不是则返回error
func (o Object) Bar() error {
    return nil
}

// List包含所有当前注册的Object
var List []Object

// MaxCount确定允许的最大对象数
const MaxCount = 50
```

因为只导出包级别的注释，所以开发人员可以自由地在方法 / 函数体中使用注释，而不必担心注释会被不小心添加到公共文档中。

`godoc`也提供一种通过解析包申明前的注释来生成包级别的文档的方法：

```go
// 包对象完成基本对象的叙述
package objects
```

需要注意的是 godoc 在生成包的索引（请看示例 https://golang.org/pkg/ ）的时候使用第一句包文档注释，因此一定要写相应的包描述。

如你所见，注释产生一种简单的方法来为开发者和使用者提供文档，而不需要复杂的语法和额外的文档文件



## 2. 构建约束

----

Go 语言中注释的第二个特殊用途是构建约束。

Go 作为一种编程语言的关键特性是它支持各种操作系统和体系结构。通常情况下，相同的代码可以用于多个平台。但是在某些情况下，特定于操作系统或体系结构的代码应该只用于特定的目标。标准的 go build 工具可以通过理解以操作系统名称和 / 或体系结构结尾的程序应该只用于匹配这些标记的目标来处理某些情况。例如，一个名为 foo_linux.go 的文件，将只会被 Linux 系统编译。 foo_amd64.go 用于 AMD64 架构，foo_windows_amd64.go 用于运行在 AMD64 架构上的 64 位 Windows 系统。

然而，这些命名约束在更复杂的情况下会失败，例如当同一代码可以用于多个 (但不是所有) 操作系统时。在这些情况下，Go 有构建约束的概念 —— 在编译 Go 程序时，go build 将读取经过特殊设计的注释，以确定要引用哪些文件。

构建约束的注释遵循以下规则:

- 以前缀`+build`开始，后跟一个或多个空格
- 位于文件顶部在包声明之前
- 在它和包声明之间至少有一个空行，以防止它被视为包文档

而不是将文件命名为 `foo_linux.go`。我们可以把下面的注释放在文件`foo.go`的开头：

```go
// +build linux
```

然而，当引用多个体系结构和 / 或操作系统时，构建约束的威力就显现出来了。Go 为组合构建约束制定了以下规则：

- 以 ! 开头的构建标记是无效的
- 用空格分隔的构建标记在逻辑上是或
- 以逗号分隔的构建标记在逻辑上是和
- 在多行上构建约束是逻辑和

根据上面的规则，下面的约束将把文件限制为 Linux 或 Darwin(MacOS):

```go
// +build linux darwin
```

而这个约束同时需要 Windows 和 i386

```
// +build windows,386
```

上面的约束也可以写在下面的两行中：

```go
// +build windows
// +build 386
```



**使用 ! 进行编译区分**

两个文件中一个是hash tag，一个是 int tag，在编译的时候通过`go build -tags xxx`指定哪些不被编译，否则只有不带 ! 的tag都会被编译进包。

display_hash.go
```go
// +build hash !display_alternatives

// 上面
package main

import "fmt"

type DisplayName string

func Print(name DisplayName) {
  fmt.Printf("%s\n", name)
}

func MakeDisplayName(name string) DispalyName {
  return DispalyName(name)
}

```

display_int.go
```go
// +build int

package main

import (
    "fmt"
    "encoding/hex"
    "encoding/binary"
)

type DisplayName uint64

func Print(name DisplayName) {
    fmt.Printf("%d\n", name)
}

func MakeDisplayName(name string) DisplayName {
    h, err := hex.DecodeString(name)
    if err != nil {
        panic(fmt.Sprintf("decode hex string failed. cause: %v\n", err))
    }
    fmt.Printf("data: %v\n", h)

    value := binary.BigEndian.Uint16(h)
    return DisplayName(value)
}
```

编译`display_int.go`

编译执行过程`go build -tags "display_alternatives int"`

编译`display_hash.go` 

编译执行过程`go build -tags hash`



**这里面有个问题，不传递-tags，为什么也会编译，因为在 go/build/build.go中的match方法中有这么一句：**

```go
if strings.HasPrefix(name, "!") { // negation
    return len(name) > 1 && !ctxt.match(name[1:])
}
```

也就是说，只要有!（不能只是!），tag不在BuildTags中时，总是会编译。
回到刚才的 tag，hash 和 int，在执行`go build -tags "display_alternatives int"`，会导致 ! 无效，所以就不会执行 hash的代码。

<font color=red>**构建约束以一行`+build`开始的注释** ，在`+build`之后列出了一些条件，在这些条件成立时，该文件应包含在编译的包中</font>

除了指定操作系统和体系结构之外，构建约束可以通过 ignore 标记的常见用法来完全忽略文件（任何不匹配有效架构或操作系统的文本是可以工作的）：
```go
// +build ignore
```

应该注意的是，这些构建约束（以及前面提到的命名约定）也适用于测试文件，因此可以以类似的方式执行特定于体系结构 / 操作系统的测试。

### 主要解决问题

1. go的同一包下面，如果方法同名是会报错的，于是使用构建约束解决了代码解耦问题。
2. 指定哪些文件能够被编译，以及在编译的时机，主要是通过`go build -tags xxx`



## 3. 生成代码

----

Go 语言注释的另一个有趣的用法是通过 `go generate` 命令生成代码。 `go generate` 是 Go 语言标准工具包的一部分，它通过运行用户指定的外部命令以编程方式生成源 (或其他) 文件。`go generate` 的工作方式是扫描 `.go` 程序，寻找其中包含要运行的命令的特殊注释，然后执行它们。

具体来说，`go generate` 查找以 `go:generate` 开头的注释（注释标记和文本开始之间没有空格），如下：
```go
//go:generate <command> <arguments>
```

与构建约束不同，`go:generate`注释可以位于`.go`源文件中的任何位置（尽管典型的Go语言习惯用法是将它们放在文件的开头）。

`go:generate`的常见用法是通过这里提供的`stringer`工具提供人类可读的常量值。`stringer`文档提供了下面的示例来解释它的操作。给定一个自定义类型，`Pill`，枚举常量：

```go
type Pill int

const (
  Placebo Pill = iota
  Aspirin
  Ibuprofen
  Paracetamol
  Acetaminophen = Paracetamol
)
```

运行命令`stringer -type Pill`将会创建一个新的源文件`pill_string.go`，它提供了以下方法：

```go
func (p Pill) String() string
```

例如，它允许打印常量的名称：

```go
fmt.Printf("pill type: %s", pill)
```

但是需要记住包中每个适用类型的正确命令和参数可能很复杂，因此我们可以将以下注释添加到包的`.go`文件中。

```go
//go:generate stringer -type=Pill
```

然后运行`go:generate`将会触发`stringer`的调用，适用正确的参数来创建我们的`Pill`字符串方法。在这种情况下，我们可以看到`stringer`和`go generate`对程序员带来的巨大好处，尤其是在包中有多个自定义类型的情况下。



## 4. cgo

---

Go 语言中注释的一个特殊用法是 C 语言集成工具 Cgo。 Cgo 允许 Go 程序直接调用 C 语言代码，允许在 Go 中重用已建立的 C 语言库。要在 Go 程序中使用 Cgo，首先要导入伪包「C」。一旦导入，Go 程序就可以引用像 `C.size_t` 这样的原生 C 类型和 `C.putchar() `这样的函数。

然而，C 语言编程的某些方面是很难转换的。为了处理这些问题，Cgo 特别使用了` import 「C」` 语句之前的注释（在 Cgo 术语中称为 序言 ）来提供各种 C 语言特定的配置项。

其中一项是 `#include` 指令。几乎每个 C 程序都需要`#include`指令来指示头文件的位置。Go 语言没有任何本地对应的命令 (`import` 在包上工作，而不是头文件)，所以 Cgo 解析序言中的`#include`语句。例如：

```go
// #include <stdio.h>
// #include <errno.h>
import "C"
```

序言部分的注释不仅仅限于`#include`语句，实际上，import 语句之前的任何注释都将视为标准的C代码，然后可以通过 `C` 包进行引用。比如，序言：

```go
// #include <stdio.h>
//
// static void myprint(char *s) {
//    printf("%s\n", s)
// }
import "C"
```

然后我们可以在Go中引用这个新定义的C函数，如下所示：

```go
C.myprint(C.string("foo"))
```

最后，为了处理编译器和类似的选项，Cgo引入来`#Cgo`指令，该指令可用于设置环境变量、编译器标记和运行 `pkg-config` 命令，如下所示：

```go
// #cgo CFLAGS: -DPNG_DEBUG=1
// #cgo amd64 386 CFLAGS: -DX86=1
// #cgo LDFLAGS: -lpng
// #cgo pkg-config: png cairo
// #include <png.h>
import "C"
```

所有这些序言的定义帮助适用 `Cgo` 的程序与 go 构建工具无缝集成，而不需要额外复杂的创建文件或其他脚本。



## 5. 编译指示

---

> 形如 `//go:` 就是Go语言的编译指示的实现方式。相信看过 Go SDK 的同学并不陌生，经常能在代码函数声明的上一行看到这样的写法。
>
> 编译器源码 里可以看到全部的指示，但是要注意，`//go:`是连续的，`//` 和 `go` 之间并没有空格。
>
> **参考：**
>
> https://golang.org/cmd/compile/#hdr-Compiler_Directives



### 常用指示详解

1.  `//go:noinline`

   **noinline 顾名思义，不要内联**

   **inline内联：inline，是在编译期间发生的，将函数调用处替换为被调用函数主体的一种编译器优化手段。**

   **使用 `inline` 有一些优势，同时也有一些问题。**

   **优势：**

   - 减少函数调用的开销，提供执行速度。
   - 复制后的更大函数体为其他编译优化带来可能性，如：过程间优化
   - 消除分支，并改善空间局部性和指令顺序性，同样可以提高性能

   **劣势：**

   - 代码复制带来的空间增长
   - 如果有大量重复代码，反而会降低缓存命中率，尤其对CPU缓存是致命的

   实际使用中，对于是否使用内联，要谨慎考虑，并做好平衡，以使它发挥最大的作用。简单来说，对于短小而且工作较少的函数，使用内联是有效益的。

   例子：

   ```go
   func appendStr(word string) string {
       return "new " + word
   }
   ```

   执行 `GOOS=linux GOARCH=386 go tool compile -S main.go > main.S` 我截取有区别的部分展出它编译后的样子：

   ```
       0x0015 00021 (main.go:4)    LEAL    ""..autotmp_3+28(SP), AX
       0x0019 00025 (main.go:4)    PCDATA    $2, $0
       0x0019 00025 (main.go:4)    MOVL    AX, (SP)
       0x001c 00028 (main.go:4)    PCDATA    $2, $1
       0x001c 00028 (main.go:4)    LEAL    go.string."new "(SB), AX
       0x0022 00034 (main.go:4)    PCDATA    $2, $0
       0x0022 00034 (main.go:4)    MOVL    AX, 4(SP)
       0x0026 00038 (main.go:4)    MOVL    $4, 8(SP)
       0x002e 00046 (main.go:4)    PCDATA    $2, $1
       0x002e 00046 (main.go:4)    LEAL    go.string."hello"(SB), AX
       0x0034 00052 (main.go:4)    PCDATA    $2, $0
       0x0034 00052 (main.go:4)    MOVL    AX, 12(SP)
       0x0038 00056 (main.go:4)    MOVL    $5, 16(SP)
       0x0040 00064 (main.go:4)    CALL    runtime.concatstring2(SB)
   ```

   可以看到，它并没有调用 `appendStr` 函数，而是直接把这个函数体的功能内联了。
   那么话说回来，如果你不想被内联，怎么办呢？此时就该使用 `go//:noinline` 了，像下面这样写：

   ```go
   //go:noinline
   func appendStr(word string) string {
       return "new " + word
   }
   ```

   编译后是：

   ```
       0x0015 00021 (main.go:4)    LEAL    go.string."hello"(SB), AX
       0x001b 00027 (main.go:4)    PCDATA    $2, $0
       0x001b 00027 (main.go:4)    MOVL    AX, (SP)
       0x001e 00030 (main.go:4)    MOVL    $5, 4(SP)
       0x0026 00038 (main.go:4)    CALL    "".appendStr(SB)
   ```

   此时编译器就不会做内联，而是直接调用 `appendStr` 函数。

   

2.  `//go:nosplit`

   `nosplit` 的作用是：跳过栈溢出检测

   **栈溢出是什么？**
   正是因为一个 `Goroutine` 的起始栈大小是有限制的，且比较小的，才可以做到支持并发很多 `Goroutine`，并高效调度。 stack.go 源码中可以看到，`_StackMin` 是 2048 字节，也就是 2k，它不是一成不变的，当不够用时，它会动态地增长。 那么，必然有一个检测的机制，来保证可以及时地知道栈不够用了，然后再去增长。 回到话题，`nosplit` 就是将这个跳过这个机制。

   **优劣**
   显然地，不执行栈溢出检查，可以提高性能，但同时也有可能发生 `stack overflow` 而导致编译失败。

   

3. `//go:noescape`

    `noescape` 的作用是：禁止逃逸，而且它必须指示一个只有声明没有主体的函数。

   **逃逸是什么？**

   Go 相比 C、C++ 是内存更为安全的语言，主要一个点就体现在它可以自动地将超出自身生命周期的变量，从函数栈转移到堆中，逃逸就是指这种行为。

   **优劣**

   最显而易见的好处是，GC 压力变小了。 因为它已经告诉编译器，下面的函数无论如何都不会逃逸，那么当函数返回时，其中的资源也会一并都被销毁。 不过，这么做代表会绕过编译器的逃逸检查，一旦进入运行时，就有可能导致严重的错误及后果。

4. `//go:norace`

   norace 的作用是：跳过竞态检测 我们知道，在多线程程序中，难免会出现数据竞争，正常情况下，当编译器检测到有数据竞争，就会给出提示。如：

   ```go
   var sum int
   
   func main() {
       go add()
       go add()
   }
   func add() {
       sum++
   }
   ```

   执行 `go run -race main.go` 利用` -race` 来使编译器报告数据竞争问题。你会看到：

   说明两个 `goroutine` 执行的 `add()` 在竞争。



绝大多数情况下，无需在编程时使用 `//go: Go` 语言的编译器指示，除非你确认你的程序的性能瓶颈在编译器上，否则你都应该先去关心其他更可能出现瓶颈的事情。



## 小结

---

这里一共提及5种注释的含义
