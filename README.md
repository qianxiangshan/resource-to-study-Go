resource-to-study-Go
====================

Go杂记

Go中未提供的try-catch-finnally结构。，但是可以采用多返回值的函数来处理错误。
Panic和recover用于恢复致命错误，不参与程序结构的处理。


Assert断言不存在是因为程序员会因此不再考虑错误处理。 从此处可以看出，我们编程是需要进行详细的错误处理，除非是致命错误
 Hoare's Communicating Sequential Processes, or CSP. ？？？？

命名：清晰，准确，例如bytes包 中buffer类型。包中对外可见的类型采用  paclagename.varname形式调用。故当报名可以说明标量的范围时不需要将范围夹在变量上。例如无需将buffer命名为byteBuffer。。


包大小：划分包的内容时要谨慎，能小则小


包路径清晰

Go get怎么使用咧   怎么在项目组中共享自己的包给别人
Go get works with packages, not URLs. 


如何确认某一类型实现了某个接口：
1、type T struct{} //确认自己的类型实现了接口var _ I = T{}   // Verify that T implements I.这里是一个类型转换，编译器会检查，。因为go不同于java等语言有显式的实现关键字impliment
2.可以显式的添加函数，来标记某一接口类型。如下
type Fooer interface {    Foo()    ImplementsFooer()//必须实现此方法，以提醒user其类型}
bype Bar struct{}func (b Bar) ImplementsFooer() {}func (b Bar) Foo() {}



接口的实现问题：接口不具有传递性？？：即实现了T接口的类型L，不能表示T接口，在其他类型实现和T接口相关的类型时。例子
type Opener interface {   Open() Reader}func (t T3) Open() *os.File // os.File是Reader类型，但是T3并没有实现Opener接口。
。
type Equaler interface {    Equal(Equaler) bool}
未实现EQUALER
type T intfunc (t T) Equal(u T) bool { return t == u } // does not satisfy Equaler
实现EQUALER
type T2 intfunc (t T2) Equal(u Equaler) bool { return t == u.(T2) }  // satisfies Equaler
从上述中可以看出，equal方法存在漏洞，即需要自己检查传入参数是否是T2类型

Interface{}变量包含两部分{value，type} .interface变量为nil意味着value和type均为nil

此处引出一个问题，当我们对一个接口变量进行判断nil时，并不意味着其值是nil即为nil。这里是一个坑。
例子：
func returnsError() error {	var p *MyError = nil	if bad() {		p = ErrBad	}	return p // Will always return a non-nil error.}此处理解为p为nil即err是空，但实际结果步非如此。实现此功能需要如下代码：
func returnsError() error {	if bad() {		return ErrBad	}	return nil //直接返回nil。这里有另外一个问题，返回值的接收，定义了errbad变量，不意味着其type等于非空吗，那么怎么能算全部nil呢？？此处接口的值未存储值，多以未nil，由于返回的为nil所以返回值是不会是接口类型的，此处需要使用：=来定义返回值的接收者，否则仍然会进入深坑}
建议判断接口为空时，采用接口内容判断，而不要使用是否为空判断。
切记：当接口的类型变量存储过具体的值（包括nil）后，那么接口的值将不会是nil。


关于值传递和引用传递：
Map and slice values behave like pointers，此处意味着map和slice始终是被共享的，除非主动复制一份。
但是接口变量复制是全部的复制，包含内容的复制，此后两者的内容改变无关。
Go中指针不能运算，并且go中指针和指针所指向的值类型不同，那么只在调用时，指针和指针所指对象才在表面上相同。
、
指针传递，两个作用，节省空间，方便修改原始值
均为值传递,但是可以传递值的指针，以此来减少开销，或者对原始内容进行更改、

func main() {
	done := make(chan bool)

	values := []string{"a", "b", "c"}
	for _, v := range values { //此处v被三个routine共享，所以当输出时，v的值//为c。此处是一个坑，可以采用匿名函数传值的方式解决。因为创建routine的时间略长
		go func() {
			fmt.Println(v) //结果为3个c。原因如上
			done <- true
		}()
	}

	// wait for all goroutines to complete before exiting
	for _ = range values {
		<-done
	}
}

Go通过多返回值来处理错误。


GO的包名和包路径， 包名可以不唯一，但是包路径必须唯一。
包路径从gopath的src下开始算。包名建议为包路径的最后一个文件夹名。
Go test 命令后加包路径。即当导入包的时候，其实写的是包路径。



Go自带的test框架。
测试包时，一般采用包名_test.go的名字来命名测试文件（与被测试包位于同一个包下，需要导入testin包）。
测试文件内采用func（t *Testing）的函数书写测试方法（此方法被系统自动调用。t.Error等方法来输出测试用例通过还是失败。）。


f, err := os.Open(name) //此处定义一个errif err != nil {    return err}d, err := f.Stat()//此处不再新定义err而是使用上述err。在go中被引用的变量均不会被销毁，所以在使用变量时要注意变量是否会被后续变量覆盖。当然不再同一个范围内的相同变量不会产生此类冲突if err != nil {    f.Close()    return err}codeUsing(f, d)



Switch c{
func shouldEscape(c byte) bool {    switch c {    case ' ', '?', '&', '=', '#', '+', '%': //此处神奇        return true    }    return false}

怎么监控goroutine是否完全执行完毕

Godoc怎么使用


Defer：
一个方法中可以定义多个defer，采用栈的方式存储defer

编译过的包才可以import ，否则不行滴
Init可以多次定义，但是普通的方法不可以。用于处理包中的数据，私有方法
在包中变量初始化过后调用init方法


_空变量，用于放弃变量的使用权，告诉编译器放弃滴该变量的检查是否使用
例子‘：
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.//让未使用的包存在，var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd  //fd虽然未使用，但是依然可以编译通过}

Import _ “net/http/pport” 此含义表示需要使用此包中的init函数初始化的数据，但是不需要明示调用此包中导出的方法
此init方法中绑定了四个url处理的函数，





