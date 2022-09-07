# KeyWords-Go 

## go 

### 启用协程

- 
    ```go
    go func(msg string) { 
        fmt.Println(msg)   
    }("going")
    ```

### [协程池](https://github.com/bytedance/gopkg/tree/develop/util/gopool) 

- 
  ```go
    gopool.Go(
        func(){       
        /// do your job  
        }
    )
  ```

## error ，defer 与 recover

### error

- error 是go定义的接口，并未对其进行规范。

- errors.New()

- fmt.Errorf("")

- [Go2提案](https://go.googlesource.com/proposal/+/master/design/go2draft-error-values-overview.md)

    - 其中提到了几种 error 的规范方式。
    - if err == nil {} 判断

### [defer](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-defer/)

- defer 的实现是由编译器和运行时共同完成的

- 经常被用于关闭文件描述符、关闭数据库连接以及释放资源

- 
  ```go
  tx := db.Begin()
  defer tx.Rollback() 
  ```

#### 关键

- 作用域
    - 只会在当前函数和方法返回之前被调用
    - 先进后出 先声明的defer语句在后声明的之后执行。
- 预计算
    - defer关键字会立刻拷贝函数中引用的外部参数.
    - 
        ```go
            func main() {
                startedAt := time.Now()
                defer fmt.Println(time.Since(startedAt))
                
                time.Sleep(time.Second)
            }
            // $ go run main.go
            // 0s
        ```

    - 
        ```go
            func main() {
                startedAt := time.Now()
                defer func() { fmt.Println(time.Since(startedAt)) }()
                
                time.Sleep(time.Second)
            }
            // go run main.go
            // 1s
        ```

    - 虽然调用 defer 关键字时也使用值传递，但是因为拷贝的是函数指针，所以 time.Since(startedAt) 会在 main 函数返回前调用并打印出符合预期的结果。

- 数据结构

- 执行机制
    - 中间代码生成阶段的 cmd/compile/internal/gc.state.stmt 会负责处理程序中的 defer，该函数会根据条件的不同，使用三种不同的机制处理该关键字

- 分配方式 （三种不同类型 defer 的设计与实现原理）
    - 堆分配 (最初方式，性能最差)

    - 栈分配 (1.13引入，性能替身百分之30)

    - 开放编码 (1.14引入，性能做好)
        - 有条件开启
        - 编译期间判断 defer 关键字、return 语句的个数确定是否开启开放编码优化
        - 如果函数中 defer 关键字的数量多于 8 个或者 defer 关键字处于 for 循环中，那么我们在这里都会禁用开放编码优化，使用上两节提到的方法处理 defer。

### panic 与 recover

- 与 recover 通常一起出现

- 调用 panic 后会立刻停止执行当前函数的剩余代码，并在**当前Goroutine** 中递归执行调用方的 defer；

- recover 可以中止 panic 造成的程序崩溃。它是一个只能在 defer 中发挥作用的函数，在其他作用域中调用不会发挥作用；

- 
    ```go
        func panic(v interface{})
        pannic("XXX 错误")
    ```
- 
    ```go
    defer func() {
        if err1 := recover(); err1 != nil {
            log.GameSystemLog().Error("err........", zap.Any("error", err1))
        }
    }()
    ```

## [channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/) 无锁管道


- 概述 ： channel是一个缓冲器，遵循先入先出的设计。
    - 先从 Channel 读取数据的 Goroutine 会先接收到数据；
    - 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；
- 异步与同步
    - channel的容量会使得其长度为0（同步管道），长度大于0（异步管道<未满的前提>）
    - 特殊：chan struct{} 类型的异步 Channel — struct{} 类型不占用内存空间，不需要实现缓冲区和直接发送（Handoff）的语义；
- ch <- i 发送数据
    - 直接发送
        - 当存在等待的接收者时，通过 runtime.send 直接将数据发送给阻塞的接收者；
        - 当缓冲区存在空余空间时，将发送的数据写入 Channel 的缓冲区；
        - 当不存在缓冲区或者缓冲区已满时，等待其他 Goroutine 从 Channel 接收数据；  
    - 阻塞发送
- 接收数据，i <- ch
  i, ok <- ch
    - 直接接收

    - 阻塞接收

- 关闭 channel

    - close(ch)

## select

- Go 语言的 select 与操作系统中的 select 比较相似

- 特点

    - 与switch结构类似，但是 case 参数不同，select的必是 channel的收发 .

    - select 能在 Channel 上进行非阻塞的收发操作；

    - select 在遇到多个 Channel 同时响应时，会随机执行一种情况；
- 阻塞
    - 直接阻塞

        - 空 select 会直接阻塞当前 Goroutine，导致 Goroutine 进入无法被唤醒的永久休眠状态。
    - 单一管道
        
        - 如果当前的 select 条件只包含一个 case 。
        - 当 case 中的 Channel 是空指针时，会直接挂起当前 Goroutine 并陷入永久休眠。
    - 非阻塞

        - 发送
            - runtime.selectnbsend，它为我们提供了向 Channel 非阻塞地发送数据的能力
            - 在不存在接收方或者缓冲区空间不足时，当前 Goroutine 都不会阻塞而是会直接返回

        - 接收
            - 从 Channel 中接收数据可能会返回一个或者两个值
            - runtime.selectnbrecv 会直接忽略返回的布尔值，而 runtime.selectnbrecv2 会将布尔值回传给调用方

        - 在默认情况下，编译器会使用如下的流程处理 select 语句
            - https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-select/#%e5%b8%b8%e8%a7%81%e6%b5%81%e7%a8%8b
            - 随机生成一个遍历的轮询顺序 pollOrder 并根据 Channel 地址生成锁定顺序 lockOrder；
            - 根据 pollOrder 遍历所有的 case 查看是否有可以立刻处理的 Channel；
            - 如果存在，直接获取 case 对应的索引并返回；
            - 如果不存在，创建 runtime.sudog 结构体，将当前 Goroutine 加入到所有相关 Channel 的收发队列，并调用 runtime.gopark 挂起当前 Goroutine 等待调度器的唤醒；
            - 当调度器唤醒当前 Goroutine 时，会再次按照 lockOrder 遍历所有的 case，从中查找需要被处理的 runtime.sudog 对应的索引；

## for 与 range

### fori（经典循环）

-   ```go
        for i := 0; i < 10; i++ {
		    println(i)
	    }
    ```
### range (范围循环)
-   ```go
        arr := []int{1, 2, 3}
        for index, value := range arr {
            println("index 是 " + strconv.Itoa(index))
            println("value 是 " + strconv.Itoa(value))
        }
    ```

-   ```go
    // 值得注意的是 不要 引用临时的 v 变量的地址 他是循环使用的。
    // &arr[i] 进行替代
    arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
    for _, v := range newArr {
		fmt.Println(*v)
	}
    // 输出： 3 3 3
    ```

- 简单的经典循环相比，范围循环在 Go 语言中更常见。

- 编译器会在编译期间将所有 for-range 循环变成经典循环。

### array / slice

-   ```go
        arr := [3]int{1, 2, 3} //数组前面会指定长度
        for index, value := range arr {
            println("index 是 " + strconv.Itoa(index))
            println("value 是 " + strconv.Itoa(value))
        }
        // 切片初始化 几种方式
        // slice1 := make([]int, len, cap)
        // slice2 := []int{10, 20, 30, 40}
        // 切片创建新的切片 
        // slice3 := slice2[:]
    ```
    

### string

-   ```go
    str := "12345"
	for _,v := range str {
		fmt.Println(v)
	}
    ```

### map

-   ```go
        hash := map[string]int{
            "1": 1,
            "2": 2,
            "3": 3,
        }
        for k, v := range hash {
            println(k, v)
        }
    ```

### channel

-  ```go
    c := make(chan int, 1)
    c <- 1
    for e := range c {
        fmt.Println(e)
        close(c)
        // 不关闭 会一直等待循环 channel
    } 
    ```

 
## 并发

    Go 语言在 sync 包中提供了用于同步的一些基本原语，包括常见的 sync.Mutex、sync.RWMutex、sync.WaitGroup、sync.Once 和 sync.Cond

### Mutex (互斥锁)

-   ```go
    state := 0
	lock := &sync.Mutex{}
	for i := 1000; i > 0; i-- {
		go func() {
			lock.Lock()
			defer lock.Unlock()
			state = state + 1
		}()
	}
	time.Sleep(time.Second * 5)
	fmt.Println(state)
    ```

### RWMutex (读写锁)

-   ```go
    // 只读的地方使用读锁  lock.RLock()
    // 有写的地方用写锁    lock.Lock()
    func TestXXXXX(t *testing.T) {
	state := 0
	lock := &sync.RWMutex{}
	for i := 1000; i > 0; i-- {
		go func() {
			lock.Lock()
			defer lock.Unlock()
			state = state + 1
		}()
		go func() {
			lock.RLock()
			defer lock.RUnlock()
			fmt.Println(state)
		}()
	}
	time.Sleep(time.Second * 5)
	fmt.Println("xxxxxxxxxxxxxxxxx")
	fmt.Println(state)
}
    ```

### WaitGroup

-   ```go
    wg := sync.WaitGroup{}
	wg.Add(3) // 调用 done 三次，wait 放行
	for i := 1; i < 4; i++ {
		go func() {
			time.Sleep(time.Duration(i) * time.Second)
			wg.Done()
		}()
	}
	wg.Wait()
	fmt.Println("over")
    ```


### once

-   ```go
    // 保证在 Go 程序运行期间的某段代码只会执行一次
    o := &sync.Once{}
	for i := 0; i < 10; i++ {
		o.Do(func() {
			fmt.Println("only once")
		})
	}
    ```

### Cond 

-   ```go
    var status int64
    func Test_Cond(t *testing.T) {
        c := sync.NewCond(&sync.Mutex{})
        for i := 0; i < 10; i++ {
            go listen(c)
        }
        time.Sleep(3 * time.Second)
        go broadcast(c)

        ch := make(chan os.Signal, 1)
        signal.Notify(ch, os.Interrupt)
        <-ch
    }

    func broadcast(c *sync.Cond) {
        c.L.Lock()
        atomic.StoreInt64(&status, 1)
        c.Broadcast()
        c.L.Unlock()
    }

    func listen(c *sync.Cond) {
        c.L.Lock()
        for atomic.LoadInt64(&status) != 1 {
            c.Wait()
        }
        fmt.Println("listen")
        c.L.Unlock()
    }
    ```

### context

-   ```go
    /** 拿 context 做并发控制，核心是利用
    ctx, cancel := context.WithCancel(context.Background())
	ctx, timeout := context.WithTimeout(context.Background(), 1*time.Second)
	ctx, deadline := context.WithDeadline(context.Background(), time.Now().Add(1*time.Second))


    cancel , timeout ,deadline 发起的 通知，然后所以监听 ctx.Done() 管道的协程就会终止。
    */
    package main
    import (
        "context"
        "fmt"
        "sync"
        "time"
    )
    func main() {
        var wg sync.WaitGroup
        ctx, stop := context.WithCancel(context.Background())
        wg.Add(1)
        go func() {
            defer wg.Done()
            worker(ctx)
        }()
        time.Sleep(3*time.Second) //工作3秒
        stop() //3秒后发出停止指令
        wg.Wait()
    }

    func worker(ctx context.Context){
        for {
            select {
            case <- ctx.Done():
                fmt.Println("下班咯~~~")
                return
            default:
                fmt.Println("认真摸鱼中，请勿打扰...")
            }
            time.Sleep(1*time.Second)
        }
    }
    ```
## GC内存

### 内存分配

### 栈内存管理

### 垃圾收集


## 其他

- 其实 go 的效率高很大程度上来源于 io 是非阻塞的，这一块可以看一下 go 的调度与网络轮训器实现，他们是go的根本。