### 线程

* `OS`线程会被操作系统内核调度。
* 线程都有一个固定内存快(一般2MB)做栈。
* 线程上下文切换成本高。
* 线程 有线程`ID`

### goroutine

* `goroutine` 是由`go`内部调度器调度， `Go`调度器的工作和内核的调度是相似
的 但是这个调度器只关注单独的`Go`程序中的`goroutine`。

* 单个 `goroutine` 只需要`2kb`的起始内存， 栈的大小会根据需要动态地伸缩。而
`goroutine`的栈的最大值有`1GB`。

* `goroutine` 的切换只是调度器内部切换不涉及内核切换成本低。
* `goroutine` 和线程是 `m:n`的关系 通过`n`个线程 执行`m`个`goroutine`。
* `go`调度器通过`GOMAXPROCS`(默认是运行机器上的CPU核数)来决定会有多少个操作系统的线程同时执
行Go的代码， `GOMAXPROCS`是前面说的`m:n`调度中的`n`
* `GOMAXPROCS`的环境变量来显式地控制这个参数，或者也可以在运行时用`runtime.GOMAXPROCS`函数来修改它。
* `goroutine`没有`ID`