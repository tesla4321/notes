## 两个小问题记一下

1. go的运算符(operator)优先级和其他语言不太一样。go中的&等位运算跟* /是同等的。比==这种比较运算符高。和C语言，java都不一样。
看源码的时候才注意到的。
```go
func isPowerOfTwo(x int) bool {
	return x&(x-1) == 0
}
```

1 *  /  %  <<  >>  &  &^
---------
2 +  -  |  ^
-------------
3 ==  !=  <  <=  >  >=
-----------
4 &&
------------
5 ||
-------------


2. slice 做append的时候会导致内存地址变化，指向的底层看情况有时候会变有时候不变。
```go
func main() {
	gg := []string{"a","b","c","d"}
	bb := deletes(gg, 1)
	fmt.Println(bb)
	fmt.Println(gg)
	
}

func deletes(s []string, i int) []string {
	s = append(s[:i], s[i+1:]...)
	return s
}
```
输出
```
[a c d]
[a c d d]
```
