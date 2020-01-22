# 1 Introduction
Golang 是一门新的语言，尽管它借鉴了一些当前已存在的一些语言的思想，但其还是有一些独特的特性用于golang高效编程。为了更好的使用go编写代码，重要的是理解go的特性与惯用法(properties and idioms). 另外比较重要的是掌握go编码中约定俗成的惯用法，如命名，代码格式，程序结构，等以便其他的go程序员容易的理解你写的代码。

这篇文档对于编写结构清晰，地道的go代码提供一些建议。你首先需要阅读[language specification](https://golang.org/ref/spec) , [Tour of Go](https://tour.golang.org/), 以及[How to Write Go Code](https://golang.org/doc/code.html)

## Examples
[go语言的源码包](https://golang.org/src/)不仅仅是作为核心库的功能，还是如何使用go语言的示例。并且很多包都包含可直接运行的代码示例，你可以直接在golang.org的网站中运行这些代码示例，如string的[Map](https://golang.org/pkg/strings/#example_Map)函数。如果你有疑问，或关注于其具体实现，库中的文档，代码，示例可以提供一些解答，思路和背景。

# 2 Formatting
代码格式的问题是最容易引起争论(contentinous)却没什么重大意义(consequential)的问题。

在go语言中，我们采用了一种特殊的方法，让机器管理代码格式的问题。`gofmt`工具将读取程序文件并源代码以标准的格式进行输出。

下面作为示例，说明了不需要花费大量试讲将结构体的字段的注释对齐， `Gofmt` 将完成格式化工作。

考虑如下声明(程序员输入格式)：
```go
type T struct {
    name string // name of th object
    value int // its value
}
```

`Gofmt`将上面的代码按列对齐：
```go
type T struct {
    name    string  // name of the object
    value   int     // its value
}
```

标准包中的go代码都是经过`gofmt`格式化过的。
Some formatting details remain , Very briefly:

Indentation缩进：
    使用tab进行缩进。`gofmt`默认使用tab，除非必须使用空格的时候才使用空格。

Line length行长度：
    go没有单行长度的限制，如果感觉行太长可以换行并用一个tab缩进
Parentheses括号：
    Go需要的括号比C和Java少：控制结构(if, for , switch)在其语法中不需要括号。

# 3 Commentary
go提供了C风格的块注释/* */和C++风格的行注释//。一般使用行注释；块注释大多数用于包的注释，但可以用于注释大块的代码。

`godoc`从go源码文件提取文档内容。出现在top-level declarations之前的注释,且中间没有新的行，会与声明一起提取，作为解释性的文字。

每个包都应该包含一个`package comment`包注释，是一个块注释，用于说明包的用法。对于多文件的源码包，包注释只需要在一个源码文件中出现即可，任何一个源码文件均可。包注释需要介绍包并提供一些关于包的整体信息。它会出现在`godoc`页面的开始位置，具体设置情况如下：

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp
```

如果包比较简单， 包注释也会比较简洁：

```go
// Package path implements utility routines for
// manipulating slash-separated filebane paths.
```

注释不需要额外的格式化，如banners of stars. 生成的输出格式甚至不会以固定宽度的字体出现。所以不需要使用空格来对齐。像`gofmt`一样，`godoc`会处理格式。

取决于具体环境，`godoc`可以不会重新格式话注释，所以要确保：正确拼写，标点符号，句子结构，折叠长行等。

在一个包内，任何top-level declaration之前的注释都被视为此声明的`doc comment` . Every exported (capitalized) name in a program should hava doc comment.程序中每个可导出的名称都应该有文档注释。

文档注释采用完整的句子才能达到最佳效果，允许更宽泛的自动展示。第一个句子应该是一句概括性的句子以声明的名称开头。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每个文档注释均以其描述的名称开头，就可以使用go工具的`doc`子命令配合`grep`获得输出想要的东西。假定你没有记住"Compile"这个名字，但是想查找regular expression的parsing function，则可采用如下命令：
```bash
$ go doc -all regexp | grep -i parse
```

如果包中的文档注释以"This function..."开头，`grep`命令将不会有帮助你记住名称的能力。但因为包采用名称作为文档注释的开头，就能联想到你寻找的内容：
```bash
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
$
```

go的声明语法允许分组声明。 单个的文档注释可以介绍一组相关的常量或变量。但这种声明是总体的呈现的，所以针对这种声明的注释通常比较泛泛。

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

分组也可以显示成员之间的关系，比如由mutex保护的一组变量
```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

# 4 Names
如果其他语言一样，go中的命名也非常重要，甚至会产生语义影响，如：对于一个名称在包外是否可见取决于它的第一个字母是否大写。因此需要花费一定的时间去讨论go语言中的naming conventions.

## 4.1 Package names
在包导入后，报名称为包内容的访问入口。在
```go
import "bytes"
```
后，代码中就可以访问`bytes.Buffer`。每个使用这个包的人都可以通过这个名称来访问包中的内容，这隐含的要求是要求包名符合：short简短, concise简明, evocative见名知意.习惯上，包采用小写单个单词的方式命名。不应该使用下划线或首字母大写的方式。包名是导入时需要的唯一默认名称，不需要在所全局源代码中唯一。

另一个惯例是包名是源代码目录的名称，在路径为`src/encoding/base64`的包导入为`encoding/base64`,而不是base64或encoding_base64或encodingBase64.

## 4.2 getters
go不提供自动的对getters和setters的支持。自己实现getters和setters并没有什么问题，也值得这么做，but it is niether idiomatic or necessary to put `Get` into the getter's name. 如果有一个字段称为`owner`(小写，非导出)，则getter的方法应该称为`Owner`(大写，可导出)，而非`GetOwner` . 大写字母即为可导出的这种规定为区分方法和字段提供了遍历。若要提供setter方法，`SetOwner`是个不错的选择。实践中两种命名都很合理：
```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

## 4.3 Interface names
按约定，只包含一个方法的接口应该以方法名加上er后缀来命名，或相似的修改来构造一个名词：`Reader, Writer, Formatter, CloseNotifier`等。

诸如此类的命名有很多，遵循它们及其代表的函数名会让事情变得简单。`Read, Write, Close, Flash, String`等都具有典型的前面和意义。为了避免冲突，请不要使用这些名称给你的方法命名，除非你明确知道它们的签名和意义相同。反之，如果你的类型实现了一个与众所周知的类型相同含义的方法，那就使用相同的命名和签名；将字符串转换方法命名为`String`而不是`ToString` .

## 4.4 MixedCaps驼峰记法
最后，go中约定使用驼峰记法MixedCaps或mixedCaps.

# 5 Semicolons分号
与C相同的是，Go的正式语法使用分号来结束语句；与C不同的是，这些分号并不在源码中出现。取而代之，词法分析器在扫描代码的时候会使用一条简单的规则来自动插入分号，所以输入的代码文本中就不需要输入分号了。

规则是这样的，如果一个新行之前的最后一个标记是标识符(包括int, float64)，仅含字面意义的如数字，字符常量，或下面标记之一：
```go
break continue fallthrough return ++ -- ) }
```
则词法分析将始终在该标记后面插入分号。这条规则可以概括为：如果新行前的标记为语句的末尾，则插入分号。

分号也可在闭括号之前直接省略，因此像
```go
go func() { for { dst <- <-src } }()
```
这样的语句无需分号。通常Go程序只在诸如for循环子句这样的地方使用分号，以此来将初始化器，条件及增量元素分开。如果你在一行中写多个语句，也需要用分号隔开。

>警告：无论如何，你都不应该将一个控制结构(if, for, switch, 或select)的左大括号放到下一行。如果这样做，就会在大括号前插入一个分号，这可能引起不需要的效果。你应该这样写。

```go
if i < f() {
    g()
}
```
而不是这样写
```go
if i < f() // wrong!
{ // wrong!
    g()
}
```
# 6 Control structures
Go中的结构控制与C有许多相似之处，但一些关键的地方是不同的。Go不再使用do或while循环，只有一个更通用的for;

switch有更灵活一点；

if和switch可如同for的方式接受可选的初始化语句；

break和continue语句接受一个可选的标签来区别什么可以break或continue；

此外，Go还有一个包含类型选择和多路通信复用器的新控制结构：select。

Go的语法也有些不同：没有圆括号，而其主题必须始终使用大括号括起来。

## 6.1 if
在go语言中，if结构如下：
```go
if x > 0 {
    return y
}
```
强制的大括号促使你将简单的if语句分成多行。特别是在主题中包含return或break等控制语句的时候，这种编码风格的好处一试便知。

由于if和switch可接受初始化语句，因此用他们来设置局部变量十分常见。
```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```
在go的库中，你会发现当if语句没有执行到下一条语句时，亦其执行体以break,continue, goto或return结束时，不必要的else语句可以省略。
```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```
下面的例子是一种的常见的情况，即代码必须防范一系列的错误条件。若控制流成功继续，则说明程序已排除错误。由于出错时将以return结束，之后的代码也就无需else了。
```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.stat()
if err != nil {
    f.Close()
    return err
}
```
## 6.2 Redeclaration and reassignment重新声明与再次赋值
顺便提一下：前面一节中的最后的示例中演示了短声明`:=`的使用方法，声明调用`os.Open`读取
```go
f, err := os.Open(name)
```
该语句声明了两个变量`f,err` . 接下来的一行调用`f.Stat()`
```go
d, err := f.Stat()
```
看起来似乎是声明了`d`和`err`。需要注意的是，机关上面两条语句中都出现了`err`，但这种重复是合法的。`err`在第一条语句中声明，但在第二条语句中只是被赋值。这意味着调用`f.Stat`使用已经存在的变量`err`, `err`仅是被重新赋值了而已。

在满足下面的条件下，已经被声明的变量v，仍可出现在`:=`声明中：
- 本次声明与已声明的变量v处于同一作用域(如果v已经在外部的作用域声明过，则本次声明将会创建新的变量)
- 初始化时对应类型的值赋给v
- 本次声明中至少有另一个变量是新声明的

这个特性简直就是纯粹的使用主义的体现，它使得我们可以很方便地只使用一个err值，例如， 在一个相当长if-else语句链中，你会发现它用的很频繁。
值得一提的是，即便Go中的函数形参和返回值在词法上处于大括号之外，但他们的作用域和该函数仍然相同

## 6.3 For
Go中的for循环与C类似，但又有些不同。Go中的for统一了C中的for和while的用法，并且去掉了`do-while`。 Go中的for有三种形式，只有只用使用了分号。

```go
// 如同C的for
for init; condition; post { }

// 如同C的while
for condition { }

// 如同C的for(;;)
for { }
```
短声明使得在循环中声明索引变量变得非常容易
```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```
如果你想变量数组，切片，字符串或映射或从channel中读取信息，range子句能处理这种循环。
```go
for key, value := range oldMap {
    newMap[key] = value
}
```
如果你只想要range返回的第一个元素(key or index),丢掉第二个：
```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```
如果你仅需要range返回的第二个元素(the value)，使用空白标识符，下划线`_` ， 来丢弃第一个元素：
```go
sum := 0
for _, value := range array {
    sum += value
}
```
空白标识符有多种用法，详见[后面部分](https://golang.org/doc/effective_go.html#blank)

对于字符串，range提供了更多的便利。它通过解析UTF-8，将每个独立的Unicode的Code point分离出来。错误的编码占用一个字节，并使用rune U+FFFD代替(rune 是Go的内置类型，是Go对单个Unicode code point的称谓，详情参考[the language specification](https://golang.org/ref/spec#Rune_literals))

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```
输出
```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```
最后Go没有逗号操作符，++和--是语句而不是表达式。因此如果你想在一个for循环中使用多个变量，你应该使用平行赋值
```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```
## 6.4 Switch
Go中的switch比C的更通用。其表达式无需为常量或整数。case表达式会自上而下逐一求解直到匹配位置。若switch没有表达式，它将匹配true。因此我们可以将if-else-if-else链携程一个switch，这也更符合Go的风格。
```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```
There is no automatic fall through, 但case语句可以使用逗号分割列举相同的处理条件。
```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```
尽管它们在Go中的用法和其他类C语言差不多，但break语句可以使switch提前终止。但有时候需要跳出循环，而不是跳出switch结构。这种情况下，在Go语言中，通过在循环前面设置一个标签，然后使用break跳到那个标签。示例如下：
```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
    }
```
同样，continue语句也可以接受一个可选的标签，但它仅用于循环。

作为本节的结束，下面的示例使用两个switch语句对byte slice进行了比较：
```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```
## 6.5 Type switch
switch也可用于发现接口变量的动态类型。这样的*type switch*使用将关键字`type`放入圆括号内的type  assertion语法来实现。如果switch在表达式中声明了变量，则变量在每个子句中将会有对应的类型。在这类switch结构的case语句中复用这些名称，也是符合Go风格的惯用方法，在每个case语句中声明使用相同的名字声明一个新的变量，但是具有不同的类型。
```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```

# 7 Functions
## 7.1 Multiple return values
Go语言的特性之一就是函数和方法可以返回多个值。这种形式可以改善C中一些笨拙的习惯：将错误值返回(例如用-1表示EOF)和修改通过地址传入的参数。

在C中，写入操作发生的错误会用一个负数标记，而错误代码会隐藏在某个不确定的位置。而在Go中，Write会返回写入的字节数以及一个错误："是的，你写入了一些字节，但并未全部写入，因为设备已满".os包中中用于写文件的Write方法的签名如下：
```go
func (file *File) Write(b []byte) (n int, err error)
```
如同文档描述的那样，它返回写入的字节数和一个non-nil的error，在n != len(b)的时候。这是一种常见的编码风格，详情参考错误处理的部分。

一个类似的方法避免为了模拟引用参数而传入指针来返回值的需求。这里一个简单的函数从一个字节切片的某个位置中获取数字，返回这个数字及下一个位置：
```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```
你可以使用这个函数扫描输入切片b中的数字：
```go
for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```
## 7.2 Named result parameters
Go函数的返回或结果参数可以给定一个名称，且可以想常规的参数一样使用，就像输入参数一样。当给定了名称后，在函数开始之前，需要对参数使用其对应类型的0值进行初始化。这种情况下，如果函数执行了return语句而return没有参数，则当前结果参数的值将作为函数的返回值

这种名称并不是强制的，但是它可以是代码变得更加简短，清晰：它们可以起到文档的作用。如果我们命名了`nextInt`函数的结果参数，则会变成如下形式，很明显其返回值是`int`类型：
```go
func nextInt(b []byte, pos int) (value, nextPos int) {}
```
因为命名的结果参数已初始化，并绑定到无参数的return语句，则函数变得更加简单和清晰。下面是一个使用命名结果参数的`io.ReadFull`的版本：
```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```
## 7.3 Defer
Go的defer语句用于预设一个函数调用(即推迟执行函数)，该函数会在函数执行defer的返回之前立即执行。这种方式非比寻常，但确实处理一些场景的有效方式，如资源必须被释放而不管函数采用何种路径返回。典型的狮子就是解锁互斥和关闭文件。
```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```
延迟调用一个函数如`Close`有两个优点。首先，它保证了你永远不会忘记关闭文件，如果你后来又编辑了这个函数添加了一个新的返回路径，这是经常容易犯的错误。其次，有意义的是close靠近open，这使得比把close放到行数末尾更加清晰。

被推迟函数的实参(如果该函数方法还包括接受者)在推迟执行时就会求值，而不是在调用执行时才求值。这样避免担心变量在函数执行是发生改变，这也意味着单个推迟的调用会推迟多个函数的执行。下面是个简单的例子
```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```
被推迟的函数会按LIFO的顺序执行，所以上面的代码在函数返回使打印4 3 2 1 0 。一个更具有实际意义的例子是用简单的方法来追踪函数在程序中的执行。

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```
我们可以充分利用这个特点，及被推迟函数的实参在`defer`执行时才会被求值。追踪路线可以为不追踪路线设置实参。
```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```
输出
```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```
对于习惯其他语言中块级资源管理的程序员，defer似乎有点怪异，但它最有趣而强大的应用恰恰来自于其基于函数而非块的特点。在panic和revoer两节中，将看到关于它可能性的其它例子。

# 8 Data
## 8.1 Allocation with *new*
Go提供了两种分配流派，内置函数`new`和`make`。它们做的事情不同，应用与不同的类型，这有点令人困惑，但是规则却很简单。首先来看`new`。它是用于分配内存的内置函数，但是与其它语言的同名函数不同，它不会初始化内存，仅仅将内存置零。也就是说, `new(T)`为类型为T的新元素分配已指令的内存空间，并返回它的地址，一个类型为*T的值。用Go的术语说即是，它返回了一个指针，该指针指向新分配的，类型为T的零值。

既然new返回的内存已经置零，那么当你设计数据结构时，每种类型的零值就不必进一步初始化了，这意味着该数据结构的使用者只需用`new`创建一个新的对象就能正常工作。例如，`bytes.Buffer`的文档中提到"零值的Buffer就是已经准备就绪的缓冲区". 同样，`sync.Mutex`并没有显式的构造函数或Init方法，而是零值的`sync.Mutex`就已经被定义为已解锁的互斥锁了。

"零值属性"可以带来各种好处。考虑以下类型声明：
```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```
SyncedBuffer类型的值在内存分配或声明后就就绪可用了。下面的代码片段中，p和v无需进一步处理即可正确工作。
```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```
## 8.2 Constructors and composite literals
有时候零值不够好，需要一个初始化构造函数，下面的代码示例来自于os包

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```
上面的代码显得过于冗长，我们可以通过*composite literal*来简化它。*composite literal*是一个表达式，该表达式在每次求值是都会创建新的实例。

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```
需要指出的是，与C不同，返回局部变量的地址是完全没有问题的。该变量关联的内存中的数据在函数返回后依然存在。实际上，每次获取一个*composite literal*的地址时，都会将一个新的实例分配内存，影刺我们可以将上面的最后两行代码合并：
```go
    return &File{fd, name, nil, 0}
```
*compposite leteral*的字段必须按顺序全部列出。但如果以`field:value`的形式明确地标出元素，初始化字段时就可以按任何顺序出现，未给出的字段值将赋0值，因此有如下形式：
```go
return &File{fd: fd, name: name}
```
少数情况下，如果*composite literal*不包含任何字段，它将创建该类型的零值。表达式`mew(File)`和`&File{}`是等价的。
*composite literal*同样可用于创建数组、切片及映射，字段标签是索引还是映射的键则根据最使用的情况确定。在下例初始化过程中，无论Enone, Eio和Einval的值是什么，只要他们的标签不同就行。

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

## 8.3 allocation with make
内置函数`make(T, args)`的使用目的与`new(T)`不同。make仅用于slices,maps,channels 。 并返回一个类型为T(不是*T)的初始化值(不是零值)。造成这种使用差异的原因是这三种类型的底层引用数据结构要求在使用前必须初始化。以slice为例，它需要三个元素来描述：一个指向数据(内部是一个数组)的指针,长度，和容量。且在这写元素初始化前，这个slice是*nil*。对于slices,maps, channels，`make`初始化了其内部的底层数据结构，并准备了用于使用的值。例如：
```go
make([]int, 10, 100)
```
分配了一个包含100个整数空间的数组，接着创建了一个长度为10，容量为100，指针指向底层数组前10个元素的slice结构。(当使用make创建slice的时候，容量参数可以省略，详情参考slice部分)。相反，`new([]int)`返回一个指向新分配的，已置零的切片结构，即一个值为nil的slice的指针。下面的实例演示了make和new的区别：

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```
必须记住的是，make仅适用于slice，maps， channels，且不返回指针。为了获取显示的指针，使用new，或者显式的使用变量的地址。

## 8.4 Arrays
在规划内存的详细布局时，数组是很有用的，有时它能帮助避免内存再次分配。但更重要的是数组是构建slice的基础。



