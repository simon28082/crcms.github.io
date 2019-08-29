---
title: Golang 锁的简单使用
date: 2019-08-28 06:01:34
tags:
    - golang
    - sync.mutex
    - lock
---
## 简述
Golang中的锁机制主要包含互斥锁和读写锁

## 互斥锁
互斥锁是传统并发程序对共享资源进行控制访问的主要手段。在`Go`中主要使用
`sync.Mutex`的结构体表示。

一个简单的示例：
```golang
func mutex()  {
	var mu sync.Mutex
	mu.Lock()
	fmt.Println("locked")
	mu.Unlock()
}
```

或者也可以使用`defer`来实现，这在整个函数流程中全部要加锁时特别有用，还有一个好处就是可以防止忘记`Unlock`
```golang
func mutex()  {
	var mu sync.Mutex
	mu.Lock()
	defer mu.Unlock()
	fmt.Println("locked")
}
```

互斥锁是开箱即用的，只需要申明`sync.Mutex`即可直接使用
```golang
var mu sync.Mutex
```

互斥锁应该是`成对出现`，在同步语句不可以再对`锁加锁`，看下面的示例：
```golang
func mutex()  {
	var mu sync.Mutex
	mu.Lock()
	fmt.Println("parent locked")
	mu.Lock()
	fmt.Println("sub locked")
	mu.Unlock()
	mu.Unlock()
}
```
此时则会出现`fatal error: all goroutines are asleep - deadlock!`错误

同样，如果多次对一个锁解锁，则会出现`fatal error: sync: unlock of unlocked mutex`错误
```golang
func mutex()  {
	var mu sync.Mutex
	mu.Lock()
	fmt.Println("locked")
	mu.Unlock()
	mu.Unlock()
}
```

那么在`goroutine`中是否对外部锁加锁呢？
```golang
func mutex()  {
	var mu sync.Mutex
	fmt.Println("parent lock start")
	mu.Lock()
	fmt.Println("parent locked")
	for i := 0; i <= 2; i++ {
        go func(i int) {
            fmt.Printf("sub(%d) lock start\n", i)
            mu.Lock()
            fmt.Printf("sub(%d) locked\n", i)
            time.Sleep(time.Microsecond * 30)
            mu.Unlock()
            fmt.Printf("sub(%d) unlock\n", i)
        }(i)
    }
	time.Sleep(time.Second * 2)
	mu.Unlock()
	fmt.Println("parent unlock")
	time.Sleep(time.Second * 2)
}
```
先看上面的函数执行结果
```
parent lock start
parent locked
sub(0) lock start
sub(2) lock start
sub(1) lock start
parent unlock // 必须等到父级先解锁，后面则会阻塞
sub(0) locked // 解锁后子goroutine才能执行锁定
sub(0) unlock
sub(2) locked
sub(2) unlock
sub(1) locked
sub(1) unlock
```
为了方便调试，使用了`time.Sleep()`来延迟保证`goroutine`的执行
从结果中可以看出，当所有的`goroutine`遇到`Lock`时都会阻塞，而当`main`函数中的`Unlock`执行后，会有一个优先（无序）的`goroutine`来占得锁，其它的则再次进入阻塞状态。

**总结：**
- 互斥锁必须成对出现
- 同级别互斥锁不能嵌套使用
- 父级中如果存在锁，当在`goroutine`中执行重复锁定操作时`goroutine`将被阻塞，直到原互斥锁解锁，多个`goroutine`将会争抢当前锁资源，其它继续阻塞。

## 读写锁
读写锁和互斥锁不同之处在于，可以分别针对读操作和写操作进行分别锁定，这样对于性能有一定的提升。
读写锁，对于多个写操作，以及写操作和读操作之前都是互斥的这一点基本等同于互斥锁。
但是对于同时多个读操作之前却非互斥关系，这也是相读写锁性能高于互斥锁的主要原因。

读写锁也是开箱即用型的
```
var rwm = sync.RWMutex
```

读写锁分为写锁和读锁：
- 写锁定和写解锁
```
rwm.Lock()
rwm.Unlock()
```
- 读锁定和读解锁
```
rwm.RLock()
rwm.RUnlock()
```

读写锁的读锁和写锁不能交叉相互解锁，否则会发生`panic`，如：
```golang
func rwMutex()  {
	var rwm sync.RWMutex

	rwm.Lock()
	fmt.Println("locked")
	rwm.RUnlock()
}
```
`fatal error: sync: RUnlock of unlocked RWMutex`

对于读写锁，同一资源可以同时有多个读锁定，如：
```golang
func rwMutex()  {
	var rwm sync.RWMutex

	rwm.RLock()
	rwm.RLock()
	rwm.RLock()
	fmt.Println("locked")
	rwm.RUnlock()
	rwm.RUnlock()
	rwm.RUnlock()
}
```
但对于写锁定只能有一个（和互斥锁相同），同时使用多个会产生`deadlock`的`panic`，如：
```golang
func rwMutex()  {
	var rwm sync.RWMutex

	rwm.Lock()
	rwm.Lock()
	rwm.Lock()
	fmt.Println("locked")
	rwm.Unlock()
	rwm.Unlock()
	rwm.Unlock()
}
```

**在`goroutine`中，写解锁会试图唤醒所有想要进行读锁定而被阻塞的`goroutine`。**

**而读解锁会在已无任何读锁定的情况下，试图唤醒一个想进行写锁定而被阻塞的`goroutine`。**

下面看一个完整示例：
```golang
func rwMutex() {
	var rwm sync.RWMutex

	for i := 0; i <= 2; i++ {
        go func(i int) {
            fmt.Printf("go(%d) start lock\n", i)
            rwm.RLock()
            fmt.Printf("go(%d) locked\n", i)
            time.Sleep(time.Second * 2)
            rwm.RUnlock()
            fmt.Printf("go(%d) unlock\n", i)
        }(i)
    }
    // 先sleep一小会，保证for的goroutine都会执行
    time.Sleep(time.Microsecond * 100)
    fmt.Println("main start lock")
    // 当子进程都执行时，且子进程所有的资源都已经Unlock了
    // 父进程才会执行
    rwm.Lock()
    fmt.Println("main locked")
    time.Sleep(time.Second)
    rwm.Unlock()
}
```
```
go(0) start lock
go(0) locked
go(1) start lock
go(1) locked
go(2) start lock
go(2) locked
main start lock
go(2) unlock
go(0) unlock
go(1) unlock
main locked
```

反复执行上述示例中，可以看到，写锁定会阻塞`goroutine`
最开始先在`main`中`sleep 100ms` ，保证子的`goroutine`会全部执行，而每个子`goroutine`会`sleep 2s`。
此时会阻塞整个`main`进程，当所有子`goroutine`执行结束，读解锁后，main的写锁定才会执行。

再看一个读锁定示例：
```golang
func rwMutex5() {
	var rwm sync.RWMutex

	for i := 0; i <= 2; i++ {
		go func(i int) {
			fmt.Printf("go(%d) start lock\n", i)
			rwm.RLock()
			fmt.Printf("go(%d) locked\n", i)
			time.Sleep(time.Second * 2)
			rwm.RUnlock()
			fmt.Printf("go(%d) unlock\n", i)
		}(i)
	}

	fmt.Println("main start lock")
	rwm.RLock()
	fmt.Println("main locked")
	time.Sleep(time.Second * 10)
}
```
```
main start lock
main locked
go(1) start lock
go(1) locked
go(2) start lock
go(2) locked
go(0) start lock
go(0) locked
go(0) unlock
go(1) unlock
go(2) unlock
```

可以看到读锁定却并不会阻塞`goroutine`。

**总结：**
- 读锁定和写锁定对于写操作都是互斥的
- 读锁定支持多级嵌套，但写锁定无法嵌套执行
- 如果有写锁定，当多个读解锁全部执行完成后，则会唤起执行写锁定
- 写锁定会阻塞`goroutine`（在Lock()时和互斥锁一样，RLock()时先也是等到RUnlock()先执行，才有锁定机会）