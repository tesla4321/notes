# go defer 分析


今天看到公众号 Go 101 的一个推送，名字叫《一道小面试题》，里面分析了一个go程序
```go
package main

import "fmt"

func f(n int) (r int) {
  defer func() {
    r += n
    recover()
  }()
  
  var f func()
  defer f()
  f = func() {
    r += 2
  }

  return n + 1
}

func main() {
  fmt.Println(f(3))
}
```
然后说输出结果是7，并且分析的时候说
>1. 在一个函数的return之后，它的返回值可以在它其中的defer调用代码中被修改（这是为什么r += n对返回值造成了影响的原因）。
>2. 未被执行的一个defer调用语句中的被调用函数值是在此defer语句执行的时候被估值的。当此估值被确定之后，其将不再会被修改（这是为什么r += 2未被执行的原因）。即使被调用的函数的估值结果为nil，此defer调用语句也不会造成panic。只有当此函数在此defer调用语句的父函数的返回退出阶段中此defer调用语句被执行的时候才会造成panic。比如在此测试中，延迟调用defer f()不会阻止后续的return返回语句的执行。

这里的第二条看的有点懵，然后我把`defer f()` 放到f函数定义的下面
```go
package main

import "fmt"

func f(n int) (r int) {
  defer func() {
    r += n
    recover()
  }()
  
  var f func()
  
  f = func() {
    r += 2
  }
  defer f()

  return n + 1
}

func main() {
  fmt.Println(f(3))
}
```
#### 发现输出结果变成了9。未执行r += 2的原因是上面那个代码的f没有定义。
然后我尝试把`r += 2`改成`n += 2`，发现结果还是9，在函数结束之后改变`n`为什么会改变返回值呢？
我又将`n += 2`这个defer和上面那个defer顺序换了下
```go
package main

import "fmt"

func f(n int) (r int) {

  var f func()
  
  f = func() {
    n += 2
  }
  defer f()
  defer func() {
    r += n
    recover()
  }()

  return n + 1
}

func main() {
  fmt.Println(f(3))
}
```
输出结果变成了7，那看来是和两个defer的执行顺序有关，defer的执行顺序是倒过来的，所以上面的代码先执行的 `r += n`然后再执行`n += 2`。对比先执行`n += 2`再执行`r += n`的结果然后结合之前第一个结论
> 在一个函数的return之后，它的返回值可以在它其中的defer调用代码中被修改（这是为什么r += n对返回值造成了影响的原因）。
可以得出一个结论：如果要多次修改返回值r的值，需要把修改r的defer函数放在前面（后执行），在后面的defer函数（先执行）里面修改参数类似这里的`n`会对包含类似`r += n`这种包含`n`的后执行defer函数产生影响。
