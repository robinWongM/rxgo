## 设计说明
原有的代码已经提供了实现一个 `operator` 的大部分基础，本软件包主要在原有代码的基础上依葫芦画瓢，新增了一个常用的 `operator`：`DistinctUntilChanged`。关于该操作符本身的用法，请参考 [RxGo 相关文档](https://github.com/ReactiveX/RxGo/blob/master/doc/distinctuntilchanged.md)

`DistinctUntilChanged` 这个操作符相比其他已经实现的转换类操作符（如 `map`, `filter`等）最大的区别在于其具有状态。操作符需要记住上一次流经该操作符的 value，并与当前源发出的 value 进行比较，来决定是否传递这个值。于是我们使用了最常用的方法——闭包，来解决这个状态记忆的问题。

## 单元测试结果

```bash
codespace ➜ ~/workspace/rxgo (master ✗) $ go test -v
=== RUN   TestRange
--- PASS: TestRange (0.00s)
=== RUN   TestRangeWithCancel
--- PASS: TestRangeWithCancel (0.00s)
=== RUN   TestStart
--- PASS: TestStart (0.00s)
=== RUN   TestAnySouce
--- PASS: TestAnySouce (0.00s)
=== RUN   TestJust
--- PASS: TestJust (0.00s)
=== RUN   TestFromSlice
--- PASS: TestFromSlice (0.00s)
=== RUN   TestFromChan
--- PASS: TestFromChan (0.00s)
=== RUN   TestFromObservable
--- PASS: TestFromObservable (0.00s)
=== RUN   TestThrow
Completed
--- PASS: TestThrow (0.00s)
=== RUN   TestEmpty
--- PASS: TestEmpty (0.00s)
=== RUN   TestNeverWithCancel
--- PASS: TestNeverWithCancel (0.00s)
=== RUN   TestMain
map debug  Receive value  120
map debug  Receive value  40
map debug  Receive value  80
map debug  Down 
Just 240
Just 80
Just 160
Filter 0
Filter 7
Filter 2
--- PASS: TestMain (0.00s)
=== RUN   TestObserver
test observer observed value  1
test observer observed value  2
test observer observed value  3
test observer Down 
--- PASS: TestObserver (0.00s)
=== RUN   TestTreading
test flatMap observed value  11
test flatMap observed value  21
test flatMap observed value  31
test flatMap Down 
test flatMap again observed value  11
test flatMap again observed value  21
test flatMap again observed value  31
test flatMap again Down 
--- PASS: TestTreading (0.00s)
=== RUN   TestAnyTranform
--- PASS: TestAnyTranform (0.00s)
=== RUN   TestMap
--- PASS: TestMap (0.00s)
=== RUN   TestFlatMap
--- PASS: TestFlatMap (0.00s)
=== RUN   TestFilter
--- PASS: TestFilter (0.00s)
=== RUN   TestDistinctUntilChanged
--- PASS: TestDistinctUntilChanged (0.00s)
PASS
ok      github.com/robinwongm/rxgo      0.006s
```

## 功能测试结果

我们只需要编写一个简单的使用该库的程序来测试即可。本次测试使用的程序源代码如下：

```
package main

import (
	"fmt"

	"github.com/robinwongm/rxgo"
)

func main() {
	// declare an observable
	rxgo.Just(0, 1, 2, 3, 3, 4, 4, 4, 5, 5, 3, 2, 1).Map(func(x int) int {
		return 3 * x
	}).Filter(func(x int) bool {
		return x > 10
	}).DistinctUntilChanged(func(x int) int {
		return x
	}).Subscribe(func(x int) {
		fmt.Println(x)
	})
}
```

实际输出：
```
$ go run main.go
12
15
```