# D1

## 任务

- 安装golang
- 基本语法
  - 变量
  - 类型（基础类型 结构体 接口 切片 map....）
  - 函数
  - 执政
  - 流程控制





## 数据类型

数字类型

| 序号 | 类型和描述                                                   |
| :--- | :----------------------------------------------------------- |
| 1    | **uint8** 无符号 8 位整型 (0 到 255)                         |
| 2    | **uint16** 无符号 16 位整型 (0 到 65535)                     |
| 3    | **uint32** 无符号 32 位整型 (0 到 4294967295)                |
| 4    | **uint64** 无符号 64 位整型 (0 到 18446744073709551615)      |
| 5    | **int8** 有符号 8 位整型 (-128 到 127)                       |
| 6    | **int16** 有符号 16 位整型 (-32768 到 32767)                 |
| 7    | **int32** 有符号 32 位整型 (-2147483648 到 2147483647)       |
| 8    | **int64** 有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |







## 切片

> 对于数组的抽象  可变 变长动态数组

```go
make([]T, length, capacity)
	_ = make([]int, 5) //创建一个长度&容量为5的切片
	// 创建一个整型切片
	// 其长度为 3 个元素，容量为 5 个元素
	_ = make([]int, 3, 5)
}
```



len（） 获取长度 cap（）获取容量

```go
	fmt.Println("numbers[1:4] ==", numbers[1:4]) //1->4
	/* 默认下限为 0*/
	fmt.Println("numbers[:3] ==", numbers[:3]) //小于三的
	/* 默认上限为 len(s)*/
	fmt.Println("numbers[4:] ==", numbers[4:])//大于四的
```



 copy() append

```go
	num = append(num, 0)
	就是在切片后面加了0  也可以多个
		num = append(num, 4,5,3,1,6)
```



### range

```go
	for _, nums := range num {
		fmt.Println(nums)
	}
```



### Map

```go
maparr := make(map[int]int)
maparr[1] = 3
使用字面量创建 Map
m := map[string]int{
    "apple":  1,
    "banana": 2,
    "orange": 3,
} //初始化
delete(maparr, 1)
fmt.Print(maparr[1])
```



### 接口

```go
package main

import "fmt"

// 定义接口
type area interface {
	add(id int) int
	delete(flag bool)
}

// 定义Book结构体，并实现area接口的所有方法
type Book struct {
	id   int
	name string
}

// 实现area接口的add方法（带Book接收者）
func (i int) add(num int) int {
	return i + num
}

//报错中，只有用户自定义的类型（如通过type MyInt int创建的新类型）才能绑定方法。内置的int、string等类型属于 “非本地类型”，不允许直接为其添加方法。

type MyInt int

func (i MyInt) add(num int) int {
	return int(i) + num
}

// 实现area接口的delete方法（带Book接收者）
func (b Book) delete(flag bool) {
	if flag {
		fmt.Println("删除成功")
	} else {
		fmt.Println("删除失败")
	}
}

func main() {
	// 创建实现了area接口的结构体实例
	var b area = Book{id: 1, name: "Go编程"}
	// 通过实例调用接口方法
	num := b.add(4)
	fmt.Println(num) // 输出：5
	b.delete(true)   // 输出：删除成功
}


```



## Errors

接口类型s

```go
type error interface {
    Error() string
}

```



### New() --> 返回一个错误消息

```go
package main

import (
	"errors"
	"fmt"
)

func main() {
	err := errors.New("this is an error")
	fmt.Println(err) // 输出：this is an error
}

```



### 自定义Error

```go
package main

import (
	"errors"
	"fmt"
)
//建立结构体
type MyError struct {
	Code int
	Msg  string
}
//接口
func (e *MyError) Error() string {
	return fmt.Sprintf("Code: %d, Msg: %s", e.Code, e.Msg)
}

func getError() error {
	return &MyError{Code: 404, Msg: "Not Found"}
}

func main() {
	err := getError()
	var myErr *MyError
	if errors.As(err, &myErr) {
		fmt.Printf("Custom error - Code: %d, Msg: %s\n", myErr.Code, myErr.Msg)
	}
}

```



## git

#### 仓库初始化

git init

### clone

git clone +地址

### 创建文件

```bash
echo "这是第一个测试文件" > test.txt  # 创建并写入内容
```

### 加到暂存区

```
git add test.txt
```

提交到版本库 

```
git commit -m"解释说明"
```

创建新分支

```
git checkout -b newbranch
```

提交

```
echo "在feature分支添加新内容" >> test.txt
git add test.txt
git commit -m "feature分支：修改test.txt，添加新内容"
```

分支与maser合并

- 先回到主分支

  - ```
    git checkout main
    ```

    - 将feature-branch合并到主分支：

      ```bash
      git merge feature-branch
      ```

    - 若需要，将合并后的代码推送到远程仓库：

      ```bash
      git push -u origin master
      ```
    
    git remote add origin <远程仓库地址>

## Go并发编程

### Goroutine

程序员只需要定义很多个任务，让系统去帮助我们把这些任务分配到CPU上实现并发执行呢？

Go语言中的goroutine就是这样一种机制，goroutine的概念类似于**线程**，但 goroutine是由**Go的运行时（runtime）调度和管理的**。Go程序会智能地将 goroutine 中的任务合理地分配给每个CPU。Go就是因为它在语言层面已经内置了调度和上下文切换的机制。



> 在func前面加一个go 实现线程



### 使用sync.WaitGroup线程同步

```go
package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func hello(i int) {
	defer wg.Done()
	fmt.Println("hello GGGG", i)
}
func main() {
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go hello(i)
	}
	wg.Wait()
}
```

goroutine是并发执行的，而goroutine的调度是随机的。

wg.Add(1)

每次激活想要被等待完成的`goroutine`之前，先调用Add()，用来设置或添加要等待完成的`goroutine`数量

wg.Done()

每次需要等待的`goroutine`在真正完成之前，应该调用该方法来人为表示`goroutine`完成了，该方法会对等待计数器减1

wg.Wait()

在等待计数器减为0之前，Wait()会一直阻塞当前的`goroutine`



<img src="F:\记录\GO\image-20251117143100833.png" alt="image-20251117143100833" style="zoom: 33%;" />

没用的后果 就是

<img src="F:\记录\GO\image-20251117143123849.png" alt="image-20251117143123849" style="zoom:33%;" />





### defer

defer是go中一种延迟调用机制，**defer后面的函数只有在当前函数执行完毕后才能执行**，将延迟的语句按defer的逆序进行执行，也就是说先被defer的语句最后被执行，最后被defer的语句，最先被执行，通常用于释放资源。

`defer` 语句通常用于确保一个函数调用在程序执行结束时发生，常见的用例包括文件关闭、锁释放、资源回收等



### runtime

#### runtime.Gosched()

**让当前正在运行的协程（goroutine）主动让出 CPU 时间片**，将执行权交还给 Go 调度器，以便调度器可以安排其他等待运行的协程执行。

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	go func(s string) {
		for i := 0; i < 2; i++ {
			fmt.Println(s)
		}
	}("world")
	// 主协程
	for i := 0; i < 2; i++ {
		// 切一下，再次分配任务
		runtime.Gosched()
		fmt.Println("hello")
	}
}

```

这样的话就是每次要输出hello的时候先暂停一下 看看有没有其他的进程在执行 然后把自己的时间片让出来 给其他人

#### runtime.Goexit()

> 结束进程

#### runtime.GOMAXPROCS

> 多少个OS线程来同时执行Go代码 默认一般都是机器上的核心数



### Channel

Go语言的并发模型是**CSP**，提倡通过**通信共享内存**而不是通过共享内存而实现通信。

channel是可以让一个goroutine发送特定值到另一个goroutine的通信机制。

类似队列 先进先出

`ctnm`这个channel是一个无缓冲通道 只能通一个接受一个放开 要不然不能 会产生死锁

`可以没有发送 但是必须有接受`

```go
package main

import (
	"fmt"
	"time"
)

func recv(c chan int) {
	ret := <-c
	fmt.Println("接收成功", ret)
}
func main() {
	ch := make(chan int)
	go recv(ch)

	time.Sleep(10 * time.Second) // 等待子协程就绪
	ch <- 10
	fmt.Println("发送成功")

	time.Sleep(30 * time.Second) // 等待子协程打印完成
}

```

#### 当然也有 有缓冲的

```go
	ch :=make(chan int ,34)
```

#### 优雅的关闭通道

```go
close(ch)
```

####  如何优雅的从通道循环取值

```go
package main

import "fmt"

//
//func main() {
//	ch := make(chan int, 1) // 创建一个容量为1的有缓冲区通道
//	ch <- 10
//	fmt.Println("发送成功")
//
//}

func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go func() {
		for i := 0; i < 100; i++ {
			ch1 <- i
		}
		close(ch1)
	}()

	go func() {
		for {
			i, ok := <-ch1 // 通道关闭后再取值ok=false
			if !ok {
				break
			}
			ch2 <- i * i
		}
		close(ch2)
	}()
	for i := range ch2 { // 通道关闭后会退出for range循环
		fmt.Println(i)
	}

}

```



#### 单向通道

```go
var ch2 chan<- float64 只用于写
var ch3 <-chan int 只用于读
```





### select

虽然确实我们可以	

```go
for{
    // 尝试从ch1接收值
    data, ok := <-ch1
    // 尝试从ch2接收值
    data, ok := <-ch2
    …
}
```

通过这个方式来实现从多个通道来对接受值接受需求 但是这么做一点都不优雅

所以Go内置了一个select关键字来对于这个场景 来让这个更加优雅 可以同时操作多个通道

```go
	chan1:=make(chan int ,4)
	chan2:=make(chan int ,4)
	
	select {
	case <-chan1:
		// 如果chan1成功读到数据，则进行该case处理语句
	case chan2 <- 1:
		// 如果成功向chan2写入数据，则进行该case处理语句
	default:
		// 如果上面都没有成功，则进入default处理流程
	}
```



> select可以同时监听一个或多个channel，直到其中一个channel ready

```go
package main

import (
	"fmt"
	"time"
)

func test1(ch chan string) {
	time.Sleep(time.Second * 5)
	ch <- "test1"
}
func test2(ch chan string) {
	time.Sleep(time.Second * 10)
	ch <- "test2"
}

func main() {
	// 2个管道
	output1 := make(chan string)
	output2 := make(chan string)
	// 跑2个子协程，写数据
	go test1(output1)
	go test2(output2)
	// 用select监控
	select {
	case _ = <-output1:
		fmt.Println("s1先写完")
	case s2 := <-output2:
		fmt.Println("s2=", s2)
	}
}

```





### GC

了解





## 常用库

### fmt

- Print

  - ```go
    func Print(a ...interface{}) (n int, err error) 
    func Printf(format string, a ...interface{}) (n int, err error)
    func Println(a ...interface{}) (n int, err error)
    ```

- ####  Fprint ****

常用这个函数往文件中写入内容

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
    fmt.Println("打开文件出错，err:", err)
    return
}
name := "枯藤"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

- `os.O_CREATE`：如果目标文件**不存在**，则自动创建该文件；如果文件已存在，则不影响（不会覆盖）。
- `os.O_WRONLY`：以**只写模式**打开文件（只能向文件写入数据，不能读取）。
- `os.O_APPEND`：以**追加模式**写入数据（新写入的内容会添加到文件末尾，而不是覆盖原有内容）。

`0644`：文件权限（仅在文件被创建时生效）

权限值由 3 组数字组成，分别对应：**文件所有者（owner）**、**所属组（group）**、**其他用户（others）** 的权限。
八进制 0644 中，前缀 0 表示这是八进制数，后面的 6、4、4 分别对应三组权限：
第一位 6（所有者权限）：6 = 4（读权限） + 2（写权限） → 所有者可以读和写该文件。
第二位 4（所属组权限）：4 = 4（读权限） → 同组用户只能读该文件，不能写。
第三位 4（其他用户权限）：4 = 4（读权限） → 其他用户只能读该文件，不能写。

#### Sprint

Sprint系列函数会把传入的数据生成并返回一个字符串。

没什么好说的 跟printf差不多

####  Errorf

是Go标准库中的函数，可以创建一个新的错误。这个函数接受一个格式化字符串和一些参数，返回一个新的错误：
的优点在于其支持格式化字符串，**这使得我们可以方便地在错误信息中包含一些动态的数据**。

#### Scan

```go
 fmt.Scan(&name, &age, &married)
```

#### Scanf

```go
	fmt.Scanf("1:%s 2:%d 3:%t", &name, &age, &married)
	强制要求按照格式输入不然不给过
```



#### Scanln

在遇到换行时才停止扫描



### Io

> 文件开

```go
file, err := os.Open("./main.go") 返回值 一个是
记得关
file.Close()
```

> 写文件

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	file, err := os.Create("./xxx.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer file.Close()
	for i := 0; i < 5; i++ {
		file.WriteString("ab\n")
		file.Write([]byte("cd\n"))
	}
}
```



Http

get

**func Get(url string) (resp \*Response, err error)**

```go
// 发送 GET 请求（语法糖）
resp, err := http.Get("https://httpbin.org/get")
if err != nil {
    panic(err)
}

```

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {
    resp, err := http.Get("https://www.5lmh.com/")
    if err != nil {
        fmt.Println("get failed, err:", err)
        return
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("read from resp.Body failed,err:", err)
        return
    }
    fmt.Print(string(body))
}

```



```go
func main() {
    apiUrl := "http://127.0.0.1:9090/get"
    // URL param
    data := url.Values{}
    data.Set("name", "枯藤")
    data.Set("age", "18")
    u, err := url.ParseRequestURI(apiUrl)
    if err != nil {
        fmt.Printf("parse url requestUrl failed,err:%v\n", err)
    }
    u.RawQuery = data.Encode() // URL encode
    fmt.Println(u.String())
    resp, err := http.Get(u.String())
    if err != nil {
        fmt.Println("post failed, err:%v\n", err)
        return
    }
    defer resp.Body.Close()
    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Println("get resp failed,err:%v\n", err)
        return
    }
    fmt.Println(string(b))
}
```



post

**func Post(url, contentType string, body io.Reader) (resp \*Response, err error)**

```go
func main() { // 发送 POST 请求（语法糖）
	resp, err := http.Post("https://httpbin.org/post", "application/json", nil)
	if err != nil {
		panic(err)
	}
	fmt.Println(resp)
}
```



#### 优雅的写url

```go
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		return
	}
	param := make(url.Values)
	param.Set("key", "1")//覆盖
	param.Add("key2", "2")//追加

	_, err = http.DefaultClient.Do(req)
	if err != nil {
		panic(err)
	}
```



#### 优雅的写Request

```go
	// 1. 定义请求体内容（JSON 字符串）
	jsonBody := `{"name": "张三", "age": 20}`
	// 2. 将 JSON 字符串转为 io.Reader 类型（用 bytes.Buffer 包装）
	bodyReader := bytes.NewBufferString(jsonBody)

	// 3. 创建请求（注意：POST 方法更适合带请求体）
	req, err := http.NewRequest("POST", "https://httpbin.org/post", bodyReader)	
// 创建 GET 请求
	req, err := http.NewRequest(http.MethodGet, "https://httpbin.org/get", nil)
	if err != nil {
		panic(err)
	}

	// 设置请求头
	req.Header.Add("Accept", "*/*")
	req.Header.Add("Accept-Language", "en-US,en;q=0.9")
	req.Header.Add("Authorization", "Token 12345")
	req.Header.Add("User-Agent", "Go-net/http")

	// 发送请求
	_, err = http.DefaultClient.Do(req) //发送 Do
	if err != nil {
		panic(err)
	}

```



#### 百度

```go
package main

import (
    "fmt"
    "io/ioutil"
    "net/http"
)

func main() {

    url := fmt.Sprintf("https://www.baidu.com/s?wd=济南")
    resp, err := http.Get(url)//这个Get会自动的帮你发送 也就是Do
    if err != nil {
       panic(fmt.Sprintf("ulr有错误 %s", err))
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
       panic(fmt.Sprintf("body有错误 %s", err))
    }
    fmt.Println(string(body))
    
}
```





### Json

```go
json.Marshal(v any)
这样就可以让一个结构体变成json格式 返回的是一个切片
```

```go
	b, err = json.MarshalIndent(p, "", "     ")//p prefix indent  -->   prefix 定义前缀字符， indent 设置缩进
	if err != nil {
		fmt.Println("json err ", err)
	}
	fmt.Println(string(b))
这个是带前缀的

```







# D2

## Http

### 工作原理

1. 客户端通过tcp和服务器及逆行联系以后（443），并且在一般的tcp链接握手中请求证书
2. 服务器返回公钥
3. 客户端产生随机密钥
4. 客户端使用公钥对 对称密钥进行加密
5. 向服务端发送对称密钥
6. 双方通过对称加密进行通信



### 请求方法

| 1    | GET  | 从服务器获取资源。用于请求数据而不对数据进行更改。例如，从服务器获取网页、图片等。 |
| ---- | ---- | ------------------------------------------------------------ |
| 2    | POST | 向服务器发送数据以创建新资源。常用于提交表单数据或上传文件。发送的数据包含在请求体中。 |
| 3    | PUT  | 向服务器发送数据以更新现有资源。如果资源不存在，则创建新的资源。与 POST 不同，PUT 通常是幂等的，即多次执行相同的 PUT 请求不会产生不同的结果 |
|      |      |                                                              |

### 权限码

- 1XX 消息响应

- 2XX 成功

- 3XX 重定向

- 4XX 客户端错误
  - 400 客户端请求的语法错误，服务器无法理解
  - 401请求要求用户的身份认证
  - 保留
  - 403权限
  - 404网页无响应
  - 405 方法禁止
- 5XX 服务器错误
  - 503 服务器不可用



### 报文

1. 请求行
   1. 请求方法：GET、POST
   2. 请求目标：通常是一个 URL ，表明了要操作的资源。
   3. 版本号：HTTP 协议版本
   4. 例子 -->GET / HTTP/1.1  
2. 请求头 -->一群kv值
3. 请求体
   1. 介绍响应报文 --》状态码就在这里
   2. 首部字段
   3. 介绍功能

### 练习

```go
	// 设置方法，url，添加header
	req, err := http.NewRequest("GET", "https://example.com", nil)
	//req, err := http.Get("https://example.com")
	if err != nil {
		fmt.Println("Error creating HTTP request:", err)
		return
	}
	req.Header.Add("Authorization", "Bearer <token>")
	req.Header.Add("Content-Type", "application/json")
	
```

```go

	resp, err := http.Get("https://httpbin.org/get")
	if err != nil {
		fmt.Println("请求失败:", err)
		return
	}
	defer resp.Body.Close()

	// 1. 获取单个响应头的值（最常用）
	contentType := resp.Header.Get("Content-Type")
	contentLength := resp.Header.Get("Content-Length")
	date := resp.Header.Get("Date")

	fmt.Println("Content-Type:", contentType)     // 响应体格式（如 application/json）
	fmt.Println("Content-Length:", contentLength) // 响应体长度（字节数）
	fmt.Println("Date:", date)                    // 服务器响应时间

	// 2. 如果头字段有多个值（例如 Set-Cookie 可能有多个）
	cookies := resp.Header["Set-Cookie"] // 返回 []string
	fmt.Println("所有 Cookie:")
	for _, cookie := range cookies {
		fmt.Println("-", cookie)
	}

	// 读取响应体（不影响头信息获取）
	body, _ := io.ReadAll(resp.Body)
	fmt.Println("\n响应体内容:", string(body))


/*
	resp, err := http.Get("https://httpbin.org/get") 发送请求 返回一个resp
		req, err := http.NewRequest("GET", "https://example.com", nil) 返回的是req 没有发送
*/
```



## 星辰英语本

### 项目初始化

```cmd'
 go install github.com/gogf/gf/cmd/gf/v2@latest
```

```cmd
gf init 项目名字
```

```go
go get -u github.com/gogf/gf/contrib/drivers/mysql/v2  //下载mysql驱动
```

在main。go中引入

```go
    _ "github.com/gogf/gf/contrib/drivers/mysql/v2"
```



修改配置

hack/conig

```
gfcli:
  gen:
    dao:
      - link: "mysql:root:123456@tcp(127.0.0.1:3306)/exe?charset=utf8mb4&parseTime=true&loc=Local"
```

使用命令gf gen dao 就可以生成





### 注册接口

1. 在api/users/v1/users.go中

   ```go
   package v1
   
   import "github.com/gogf/gf/v2/frame/g"
   
   type RegisterReq struct {
   	g.Meta   `path:"users/register" method:"post"`
   	Username string `json:"username"`
   	Password string `json:"password"`
   	Email    string `json:"email"`
   }
   type RegisterRes struct {
   }
   
   ```

   

然后执行命令

> gf gen ctrl





`Logic` 是业务逻辑层 类似java中的impl

```go
	package users

import (
	"context"
	"star/internal/dao"
	"star/internal/model/do"
)

func (u *Users) Register(cxt context.Context, username, password, email string) error {
	_, err := dao.Users.Ctx(cxt).Data(do.Users{
		Username: username,
		Password: password,
		Email:    email,
	}).Insert()
	if err != nil {
		return err
	}
	return err
}

```



| entity | 业务实体抽象，承载业务数据 | 可能不直接对应表      | 业务层数据传递（入参、返回值） |
| ------ | -------------------------- | --------------------- | ------------------------------ |
| do     | 数据库表的直接映射         | 与表结构完全对应      | ORM 交互（数据读写的载体）     |
| dao    | 封装数据库操作逻辑         | 负责执行 SQL/ORM 操作 | 业务层调用，实现数据访问       |





```go
dao.Users.DB()//*gdb.DB（GoFrame 的数据库连接对象）。 用于执行底层数据库操作（如原生 SQL、事务控制等）
dao.Users.Ctx() //（GoFrame 的查询模型对象）
//返回一个与当前 DAO 绑定的查询模型，用于通过链式操作构建数据库查询（如条件筛选、排序、分页等）。
dao.Users.Ctx().Data()
//是 “数据容器”，负责承载插入 / 更新时的具体数据。
```



用户重复检验

```go
func (u *User) checkUser(cxt context.Context, username string) error {
	count, err := dao.User.Ctx(cxt).Where("username", username).Count()
	if err != nil {
		return err
	}
	if count > 0 {
		return gerror.New("用户已存在")
	}
	return nil
}
```



## JWT



## 中间件

该方式也是主流的 `WebServer` 提供的请求流程控制方式， 基于中间件设计可以为 `WebServer` 提供更灵活强大的插件机制。

```go
package middleware

import (
    "net/http"

    "star/internal/consts"

    "github.com/gogf/gf/v2/net/ghttp"
    "github.com/golang-jwt/jwt/v5"
)

func Auth(r *ghttp.Request) {
    var tokenString = r.Header.Get("Authorization")
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        return []byte(consts.JwtKey), nil
    })
    if err != nil || !token.Valid {
        r.Response.WriteStatus(http.StatusForbidden)
        r.Exit()
    }
    r.Middleware.Next()
}
```



`r.Middleware.Next()` 这段代码是中间件的控制核心，放在函数的最后面表示它是一个**前置中间件**，代表请求前调用。放在最前面则是**后置中间件**，在请求后生效。

`r.Header.Get("Authorization")`从`HTTP Header`中获取`Authorization`字段，即获取前端传过来的`Token`。`jwt.Parse` 解析`Token`后再通过`token.Valid`验证是否有效，如果失效则返回 `HTTP StatusForbidden 403` 状态码，即权限不足，反之调用`r.Middleware.Next()`进入下一步。



在`Logic`中不能直接获取`HTTP`对象，需要使用`g.RequestFromCtx(ctx).Request`





# D3

## redis



```yml
# Redis 配置示例
redis:
  # 单实例配置示例1
  default:
    address: 127.0.0.1:6379
    db:      1

  # 单实例配置示例2
  cache:
    address:     127.0.0.1:6379
    db:          1
    pass:        123456
    idleTimeout: 600
```



## 使用

```
g.Redis() 获取 Redis 客户端对象:
redis := g.Redis("cache")
```

```go
	var ctx = gctx.New()//获取ctx
	_, err := g.Redis().Set(ctx, "key", "value")//set 操作
	if err != nil {
		g.Log().Fatal(ctx, err)
	}
	value, err := g.Redis().Get(ctx, "key")//取数据
	if err != nil {
		g.Log().Fatal(ctx, err)
	}
	fmt.Println(value.String())
```

```go
// 执行 SET 命令
result, err := redis.Do(ctx, "SET", "key", "value")

// 执行 GET 命令
value, err := redis.Do(ctx, "GET", "key")

// 执行 HSET 命令
result, err := redis.Do(ctx, "HSET", "hash", "field", "value")

// 执行 HGETALL 命令
hash, err := redis.Do(ctx, "HGETALL", "hash")
```

```go
// 执行 SET 命令 
result, err := redis.Set(ctx, "key", "value")

// 执行 GET 命令
value, err := redis.Get(ctx, "key")

// 执行 HSET 命令
result, err := redis.HSet(ctx, "hash", "field", "value")

// 执行 HGETALL 命令  
hash, err := redis.HGetAll(ctx, "hash")

```





## Lunix

ls: 列出目录

cd：切换目录

pwd：显示目前的目录

mkdir：创建一个新的目录

rmdir：删除一个空的目录

cp: 复制文件或目录

cp /root/install.sh /home -->制 root目录下的install.sh 到 home目录下

rm: 移除文件或目录

经典 rm -rf

mv: 移动文件与目录，或修改文件与目录的名称



### 编辑文件

vi /vim 

```
vi world.txt  # 输入i  a 进入编辑模式
# 进入编辑模式，输入内容
hello linux world !
# 保存并退出
:wq 
```

**i** 切换到输入模式，以输入字符。

**x** 删除当前光标所在处的字符。

**:** 切换到底线命令模式，以在最底一行输入命令



### 查看日志

1. tail -->tail 命令可以用于查看日志文件的最后几行或实时追踪日志文件。
   - -n: 指定显示行数。 -f: 以跟随模式运行。 -c: 指定要显示的字符数。 -r: 从文件末尾倒序显示。
   - tail -n 100 server.log 看后100行
2. head --> 看前几行 跟tail一样
3. cat 全文搜索
   - cat server.log 看全部文件

4.grep命令—查找文件内容

grep -5 'parttern' inputfile 查找 符合的前后5行



### 查看系统状态

top 看实时监控 CPU、内存、进程等

<img src="F:\记录\GO\image-20251121090649806.png" alt="image-20251121090649806" style="zoom:50%;" />

df -h 查看所有磁盘分区的总容量、已用、可用

du -sh 目录 查看目录的总磁盘占用

lsof -i 端口号's：查看所有网络接口的 IP 地址、MAC 地址、网络状态等

### 端口监听

grep 端口监听

netstat -an 网络连接状态

### 压缩解压

- 压缩：zip test.zip test/
- 解压：unzip test.zip



### 用户

添加账号 useradd 选项 用户名

-c comment 指定一段注释性描述。

-d 目录 指定用户主目录，如果此目录不存在，则同时使用-m选项，可以创建主目录。

-g 用户组 指定用户所属的用户组。

-G 用户组，用户组 指定用户所属的附加组。

-m　使用者目录如不存在则自动建立。

-s Shell文件 指定用户的登录Shell。

-u 用户号 指定用户的用户号，如果同时有-o选项，则可以重复使用其他用户的标识号。



删除用户 userdel 选项 用户名

修改帐号 usermod 选项 用户名



### 文件权限

chmod 权限 文件

![权限](https://i-blog.csdnimg.cn/blog_migrate/c0f98e47843a0296761c58d66b2950fa.png)







## 租赁关系

在多租户架构中，不同租户贡共享一个应用程序实例及其基础设置。这就意味着所有租户使用的硬件资源都被统一管理（包含硬件的型号也能得到统一），不再需要像独立部署模式中的一样分散管理

**租户关系**是互联网技术领域中，特别是在云计算和 SaaS (软件即服务) 架构下的一种核心概念，指多个客户 (租户) 共享同**一套软件系统和基础设施**，同时各自的**数据和配置相互隔离的关系模**式。

特点 ：共享软件实例 资源

## 物模型

是物联网领域中对物理设备或虚拟实体进行**标准化、结构化定义的数字抽象模型**，是物理实体在数字世界的 "数字孪生体"。

设备提供一套统一的 "数字说明书"

本质 `JSON`



### 三大核心组件

1. #### 属性

| 字段名称 | 字段说明         | 约束条件                                                     |
| -------- | ---------------- | ------------------------------------------------------------ |
| 名称     | 参数中文名       | “仅支持中文、英文大小写、数字、部分常用符号（下划线，减号，括弧，空格），必须以中文、英文或数字开头，长度不超过40个字符。” |
| 标识符   | 参数唯一英文标识 | 支持大小写字母、数字和下划线、不超过50个字符。               |
| 数据类型 |                  | 必选，可选整数型、浮点型、枚举型、字符串。                   |
| 枚举项   | 枚举值和解释     | 仅枚举值参数。 分为参数值和参数描述，参数值支持整形，不超过2个字符，参数描述支持中文、英文、数字、下划线，不超过20个字符，枚举项数量可自定义。 |
| 取值范围 | 数据范围         | 仅整形、浮点数。 可自定义，输入的数值范围不超过各类型数据所能表示的范围。 |
| 步长     | 取值间隔         | 仅整形、浮点数。 步长是指设备上报或下发数值时，递增或递减的间隔。步长只能是一个正数；整数型最小步长为1；浮点数最小步长为10^(-7)；最大步长不能超出取值范围的差值。 |
| 数据长度 | 字符串长度       | 仅字符串参数。 整数，表示字符串最大长度，取值1-2048          |
| 单位     | 数据单位         |                                                              |
| 读写权限 | 读写权限         | 可选“读”“写”“读写” 表示参数的读写权限                        |

1. #### 方法

设备可执行的**操作或功能**，需要外部触发，通常有输入和输出参数

| 字段名称 | 字段说明                                                     | 约束条件                                                     |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 名称     | 参数中文名                                                   | 仅支持中文、英文大小写、数字、部分常用符号（下划线，减号，括弧，空格），必须以中文、英文或数字开头，长度不超过40个字符。 |
| 标识符   | 参数唯一英文标识                                             | 支持大小写字母、数字和下划线、不超过50个字符。               |
| 调用方式 | 异步调用是指云端执行调用后直接返回,不会关心设备的回复消息,如果服务为同步调用,云端会等待设备回复,否则会调用超时。 | 异步调用或同步调用任选其一。                                 |
| 输入参数 |                                                              | 输入参数只可选择当前设备的属性，可多选，可为空。             |
| 输出参数 |                                                              | 输出参数只可选择当前设备的属性，可多选，可为空。             |
| 描述     | 参数描述                                                     | 100字以内                                                    |

1. #### 事件 

   备在特定条件下**主动上报的信息**，用于通知外部系统状态变化或异常

| 字段名称 | 字段说明         | 约束条件                                                     |
| -------- | ---------------- | ------------------------------------------------------------ |
| 名称     | 参数中文名       | 仅支持中文、英文大小写、数字、部分常用符号（下划线，减号，括弧，空格），必须以中文、英文或数字开头，长度不超过40个字符。 |
| 标识符   | 参数唯一英文标识 | 支持大小写字母、数字和下划线、不超过50个字符。               |
| 输出参数 |                  | 输出参数只可选择当前设备的属性，可多选，可为空。             |
| 描述     | 参数描述         | 100字以内                                                    |



### 2. 典型应用场景

#### 智能家居：环境监测系统

**物模型核心定义**：

- **属性**：温度、湿度、光照强度、空气质量指数 (PM2.5)
- **服务**："设置告警阈值"、"校准传感器"
- **事件**："温湿度异常告警"、"空气质量恶化通知"

**实际应用**：当传感器检测到室内温度超过 30℃时，自动触发事件上报，系统可联动空调开启并发送通知给用户。



## Modbus通信协议详解：工业自动化的经典协议

DEF:**串行通信协议**，是全球第一个真正用于工业现场的**总线协议**  设备之间的数据交换



<img src="F:\记录\GO\image-20251119164140610.png" alt="image-20251119164140610" style="zoom:50%;" />

<img src="F:\记录\GO\image-20251119164235434.png" alt="image-20251119164235434" style="zoom:50%;" />

### Modbus帧结构

Modbus协议的数据帧结构分为以下几个部分：

1. **设备地址段**：标识目标设备的地址。
2. **功能代码段**：标识操作类型（如读、写）。
3. **数据段**：包含具体的操作数据。
4. **错误校验段**：用于检测数据传输是否正确。



Modbus协议支持多种传输方式

- **串口传输**：如RS-232、RS-485。
- **网络传输**：如Modbus TCP/IP。

###  CRC16校验



CRC16校验是一种常用的错误检测算法，用于确保数据传输的可靠性。发送方和接收方使用相同的算法计算校验值，并进行对比。





## MQTT协议

### 发布者 代理 订阅者

`MQTT`使用的发布/订阅消息模式，它提供了一对多的消息分发机制，从而实现与应用程序的解耦。

这是一种消息传递模式，**消息不是直接从发送器发送到接收器**（即点对点），而是由`MQTT server`分发的。

<img src="F:\记录\GO\image-20251119165257227.png" alt="image-20251119165257227" style="zoom:50%;" />

服务器分发消息，因此必须是发布者，但绝**不是订阅者**！

客户端可以发布消息（发送方）、订阅消息（接收方）或两者兼而有之。

客户端（也称为节点）是一种智能设备，如微控制器或具有 TCP/IP 堆栈和实现 MQTT 协议的软件的计算机。

### QoS（Quality of Service levels）

服务质量

MQTT 在这里帮助避免信息丢失及其服务质量水平

**QoS 0**

这一级别会发生**消息丢失或重复**，消息发布依赖于底层TCP/IP网络。即：<=1

**QoS 1**

QoS 1 承诺消息将至少传送一次给订阅者。

**QoS 2**

使用 QoS 2，我们保证消息仅传送到目的地一次。

带有唯一消息 ID 的消息会存储两次，首先来自发送者，然后是接收者。QoS 级别 2 在网络中具有最高的开销，因为在发送方和接收方之间需要两个流。

<img src="F:\记录\GO\image-20251119165649783.png" alt="image-20251119165649783" style="zoom:33%;" />

### MQTT 数据包结构

- `固定头（Fixed header）`表示数据包类型及数据包的分组类标识；
- `可变头（Variable header）`数据包类型决定了可变头是否存在及其具体内容；
- `消息体（Payload）`，表示客户端收到的具体内容；



<img src="F:\记录\GO\image-20251119165752497.png" alt="image-20251119165752497" style="zoom:33%;" />

#### 固定头

- 消息类型
- 标识位 / DUP
  - 在不使用标识位的消息类型中，标识位被作为保留位。如果收到无效的标志时，接收端必须关闭网络连接：
- QOS 服务保证
- RET 
  - 发布保留标识，表示服务器要保留这次推送的信息，如果有新的订阅者出现，就把这消息推送给它，如果设有那么推送至当前订阅者后释放。
- 剩余长度

#### 可变头

可变头的内容因数据包类型而不同，较常的应用是做为包的标识：



#### 消息体

- `CONNECT`，消息体内容主要是：客户端的ClientID、订阅的Topic、Message以及用户名和密码
- `SUBSCRIBE`，消息体内容是一系列的要订阅的主题以及`QoS`。
- `SUBACK`，消息体内容是服务器对于`SUBSCRIBE`所申请的主题及`QoS`进行确认和回复。
- `UNSUBSCRIBE`，消息体内容是要订阅的主题。







# D4

## 掌握g.cfg()

```go
//func exe() {
//
//	var ctx = gctx.New()
//	fmt.Println(g.Cfg().Get(ctx, "viewpath"))
//	fmt.Println(g.Cfg().Get(ctx, "database.default.0.role"))
//
//	var ctx = gctx.New()
//	fmt.Println(gcfg.Instance().Get(ctx, "viewpath"))
//	fmt.Println(gcfg.Instance().Get(ctx, "database.default.0.role"))
//}

```

## g.log()

```go
	ctx := context.Background()
	g.Log().Debug(ctx, "这是一条DEBUG级别的日志")
	g.Log().Info(ctx, "这是一条INFO级别的日志")
	g.Log().Error(ctx, "这是一条ERROR级别的日志")
```



## golang操作数据库

```go
package main

import (
	"database/sql"
	"fmt"
	"log"

	_ "github.com/go-sql-driver/mysql"
)

// 定义一个结构体来存储查询结果（假设你的表有id, userName, passWord字段）
type User struct {
	id       int
	userName string
	passWord string // 使用sql.NullInt64处理可能为NULL的值
}

func main() {
	// 连接数据库
	db, err := sql.Open("mysql",
		"root:123456@tcp(127.0.0.1:3306)/exe?charset=utf8mb4&parseTime=true&loc=Local")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// 验证连接
	err = db.Ping()
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println("数据库连接成功！")

	// 1. 基本查询（查询所有用户）
	fmt.Println("\n=== 查询所有用户 ===")
	rows, err := db.Query("SELECT id, userName, passWord FROM user")
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	// 遍历查询结果
	for rows.Next() {
		var user User
		// 将查询结果扫描到结构体变量中
		err := rows.Scan(&user.id, &user.userName, &user.passWord)
		if err != nil {
			log.Fatal(err)
		}
		fmt.Printf("id: %d, userName: %s, passWord: %s\n", user.id, user.userName, user.passWord)
	}
}

```

 

## json序列化和反序列化

```go
// package main
//
// import (
//
//  "encoding/json"
//  "fmt"
//
// )
//
//  type Person struct {
//     Name  string
//     Hobby string
//  }
//
//  func main() {
//     p := Person{"5lmh.com", "女"}
//     // 编码json
//     b, err := json.Marshal(p)
//     if err != nil {
//        fmt.Println("json err ", err)
//     }
//     fmt.Println(string(b))
//
//     // 格式化输出
//     b, err = json.MarshalIndent(p, "", "     ")
//     if err != nil {
//        fmt.Println("json err ", err)
//     }
//     fmt.Println(string(b))
//  }
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

// 定义一个示例结构体（需序列化的对象）
type Person struct {
    Name string `json:"name"` // JSON 字段名映射
    Age  int    `json:"age"`
}

// 序列化结构体为 JSON 并写入文件
func writeToFile(data Person, filePath string) error {
    // 将结构体转换为 JSON 字节
    jsonData, err := json.MarshalIndent(data, "", "  ")
    if err != nil {
       return fmt.Errorf("序列化 JSON 失败: %v", err)
    }

    // 将 JSON 字节写入文件
    err = os.WriteFile(filePath, jsonData, 0644)
    if err != nil {
       return fmt.Errorf("写入文件失败: %v", err)
    }
    return nil
}

// 从文件读取 JSON 并反序列化为结构体
func readFromFile(filePath string) (Person, error) {
    // 读取文件内容
    data, err := os.ReadFile(filePath)
    if err != nil {
       return Person{}, fmt.Errorf("读取文件失败: %v", err)
    }

    // 将 JSON 字节反序列化为结构体
    var person Person
    err = json.Unmarshal(data, &person)
    if err != nil {
       return Person{}, fmt.Errorf("反序列化 JSON 失败: %v", err)
    }
    return person, nil
}

func main() {
    filePath := "person.json" // 存储 JSON 的文件路径

    // 步骤1：创建结构体实例并序列化到文件
    person := Person{
       Name: "张三",
       Age:  30,
    }
    err := writeToFile(person, filePath)
    if err != nil {
       fmt.Println("错误:", err)
       return
    }
    fmt.Println("已将结构体序列化并保存到", filePath)

    // 步骤2：从文件读取 JSON 并反序列化为结构体
    loadedPerson, err := readFromFile(filePath)
    if err != nil {
       fmt.Println("错误:", err)
       return
    }
    fmt.Println("从文件读取并反序列化的结果:")
    fmt.Printf("姓名: %s, 年龄: %d\n", loadedPerson.Name, loadedPerson.Age)
}
```

### 1. 配置文件示例（`config/config.toml`）

toml











```toml
[server]
  port = 8080
  host = "127.0.0.1"

[database]
  link = "mysql:user:password@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4"

[app]
  name = "my-gf-app"
  mode = "dev"  # 多环境标识：dev（开发）、test（测试）、prod（生产）
```

### 2. 读取配置的代码示例

go



运行









```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
)

func main() {
	ctx := context.Background()
	// 读取简单配置项
	appName := g.Cfg().MustGet(ctx, "app.name").String()
	serverPort := g.Cfg().MustGet(ctx, "server.port").Int()
	dbLink := g.Cfg().MustGet(ctx, "database.link").String()

	g.Log().Info(ctx, "应用名称:", appName)
	g.Log().Info(ctx, "服务端口:", serverPort)
	g.Log().Info(ctx, "数据库链接:", dbLink)
}
```

## 三、日志处理（`g.Log` 配置级别与输出）

### 日志配置与输出示例

go



运行









```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/log"
)

func main() {
	// 配置日志：同时输出到控制台和文件，级别为 DEBUG
	g.Log().SetConfigWithMap(g.Map{
		"Level": log.LevelDebug, // 日志级别：Debug/Info/Warn/Error等
		"Console": g.Map{
			"Enabled": true, // 启用控制台输出
		},
		"File": g.Map{
			"Enabled":  true,  // 启用文件输出
			"Path":     "logs",// 日志存储目录
			"Rotate":   true,  // 按天分割日志
			"RotateBy": "day",
		},
	})

	ctx := context.Background()
	g.Log().Debug(ctx, "这是一条 DEBUG 级别的日志")
	g.Log().Info(ctx, "这是一条 INFO 级别的日志")
	g.Log().Error(ctx, "这是一条 ERROR 级别的日志")
}
```

## 四、路由与控制器

### 1. 路由注册（`BindHandler` + 控制器注解）

go



运行









```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

func main() {
	s := g.Server()

	// 方式1：直接绑定 Handler
	s.BindHandler("/hello", func(r *ghttp.Request) {
		r.Response.Write("Hello, GoFrame!")
	})

	// 方式2：控制器注解（定义控制器并绑定方法）
	type UserController struct{}
	s.BindHandler("/user/list", (*UserController).List)

	s.Run() // 启动服务，监听 8080 端口
}

// UserController 控制器示例
type UserController struct{}

// List 方法：处理 /user/list 路由，返回 JSON 响应
func (c *UserController) List(r *ghttp.Request) {
	r.Response.WriteJson(g.Map{
		"code": 0,
		"msg":  "success",
		"data": g.Map{"total": 10, "list": []string{"user1", "user2"}},
	})
}
```

### 2. 路由参数获取（路径、查询、JSON 数据）

go



运行









```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

func main() {
	s := g.Server()

	// 1. 路径参数（如 /user/123）
	s.BindHandler("/user/{id}", func(r *ghttp.Request) {
		id := r.GetInt("id") // 获取路径参数 id
		r.Response.Writef("User ID: %d", id)
	})

	// 2. 查询参数（如 /user?name=john&age=20）
	s.BindHandler("/user", func(r *ghttp.Request) {
		name := r.GetString("name") // 查询参数 name
		age := r.GetInt("age")      // 查询参数 age
		r.Response.WriteJson(g.Map{"name": name, "age": age})
	})

	// 3. JSON 数据（POST 请求体）
	s.BindHandler("POST:/user", func(r *ghttp.Request) {
		var user struct{ Name string; Age int }
		if err := r.ParseJson(&user); err != nil {
			r.Response.WriteJson(g.Map{"code": 1, "msg": err.Error()})
			return
		}
		r.Response.WriteJson(g.Map{"code": 0, "msg": "success", "data": user})
	})

	s.Run()
}
```

## 五、数据库操作（ORM + 条件查询 + 事务）

### 1. CRUD 操作（Insert/FindOne/Update/Delete）

```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
)

// User 数据模型（对应数据库表 user）
type User struct {
	ID   int    `json:"id" gorm:"primaryKey"`
	Name string `json:"name"`
	Age  int    `json:"age"`
}

func main() {
	ctx := context.Background()

	// 1. 插入数据
	user := User{Name: "张三", Age: 30}
	result, err := g.DB().Model("user").Insert(ctx, user)
	if err != nil {
		g.Log().Error(ctx, "插入失败:", err)
		return
	}
	id, _ := result.LastInsertId()
	g.Log().Info(ctx, "插入成功，ID:", id)

	// 2. 查询单条数据
	var getOne User
	err = g.DB().Model("user").Where("id", id).FindOne(ctx, &getOne)
	if err != nil {
		g.Log().Error(ctx, "查询失败:", err)
		return
	}
	g.Log().Info(ctx, "查询单条:", getOne)

	// 3. 更新数据
	rows, err := g.DB().Model("user").Where("id", id).Data(g.Map{"age": 31}).Update(ctx)
	if err != nil {
		g.Log().Error(ctx, "更新失败:", err)
		return
	}
	g.Log().Info(ctx, "更新行数:", rows)

	// 4. 删除数据
	rows, err = g.DB().Model("user").Where("id", id).Delete(ctx)
	if err != nil {
		g.Log().Error(ctx, "删除失败:", err)
		return
	}
	g.Log().Info(ctx, "删除行数:", rows)
}
```

### 2. 条件查询（Where/And/Or + 分页 / 排序）

```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
)

func main() {
	ctx := context.Background()

	// 条件查询 + 分页 + 排序
	var users []g.Map
	err := g.DB().Model("user").
		Where("age > ?", 20).   // 基础条件
		And("name LIKE ?", "张%"). // 且条件
		Order("id DESC").       // 按 ID 倒序
		Limit(10).              // 每页 10 条
		Offset(0).              // 第 1 页（偏移量 0）
		Select(ctx, &users)
	if err != nil {
		g.Log().Error(ctx, "查询失败:", err)
		return
	}
	g.Log().Info(ctx, "查询结果:", users)

	// 统计符合条件的总数
	count, err := g.DB().Model("user").Where("age > ?", 20).Count(ctx)
	if err != nil {
		g.Log().Error(ctx, "统计失败:", err)
		return
	}
	g.Log().Info(ctx, "符合条件的总数:", count)
}
```

### 3. 事务处理（Begin/Commit/Rollback）



```go
package main

import (
	"context"
	"github.com/gogf/gf/v2/frame/g"
)

func main() {
	ctx := context.Background()

	// 开启事务
	tx, err := g.DB().Begin(ctx)
	if err != nil {
		g.Log().Error(ctx, "开启事务失败:", err)
		return
	}
	defer tx.Rollback(ctx) // 若未提交，自动回滚

	// 执行 SQL 操作（插入 + 更新）
	_, err = tx.Model("user").Insert(ctx, g.Map{"name": "事务测试", "age": 25})
	if err != nil {
		g.Log().Error(ctx, "插入失败:", err)
		return
	}

	_, err = tx.Model("user").Where("name", "事务测试").Update(ctx, g.Map{"age": 26})
	if err != nil {
		g.Log().Error(ctx, "更新失败:", err)
		return
	}

	// 提交事务
	if err = tx.Commit(ctx); err != nil {
		g.Log().Error(ctx, "提交事务失败:", err)
		return
	}
	g.Log().Info(ctx, "事务执行成功！")
}
```









# D5

- [x] redis的命令行操作
- [ ] mysql的命令行操作
- [x] linux的命令操作
- [ ] 学习xuiAdmin









## Redis

启动

```
redis-cli.exe -h 127.0.0.1 -p 6379
```

设置键值对:

```
set myKey abc
```

取出键值对:

```
get myKey
```

```cmd
getrange key st ed #范围取值

127.0.0.1:6379> getrange key 0 1
"aa"


127.0.0.1:6379> mget k1 k2 k3 #取多个值
1) "1"
2) "2"
3) "3"


#哈希的取值
127.0.0.1:6379> hmset hk1 name 1 age 3
OK
127.0.0.1:6379> hmget hk1 name age
1) "1"
2) "3"


#list的操作
127.0.0.1:6379>  LPUSH lkey 2
(integer) 1
127.0.0.1:6379>  LPUSH lkey 4
(integer) 2
127.0.0.1:6379>  LPUSH lkey 2
(integer) 3
127.0.0.1:6379>
127.0.0.1:6379>  LPUSH lkey 25
(integer) 4
127.0.0.1:6379>  lrange lkey
(error) ERR wrong number of arguments for 'lrange' command
127.0.0.1:6379>  lrange lkey 0 10
1) "25"
2) "2"
3) "4"
4) "2"



#set的操作
127.0.0.1:6379> SADD skey 213
(integer) 1
127.0.0.1:6379> SADD skey 2
(integer) 1
127.0.0.1:6379> SADD skey 234
(integer) 1
127.0.0.1:6379> SADD skey 23414
(integer) 1
127.0.0.1:6379> Smembers skey 23414
(error) ERR wrong number of arguments for 'smembers' command
127.0.0.1:6379> Smembers skey
1) "2"
2) "213"
3) "234"
4) "23414"


#Zset的操作
127.0.0.1:6379>  ZADD runoobkey 1 redis
(integer) 1
127.0.0.1:6379>  ZADD runoobkey 2 redis
(integer) 0
127.0.0.1:6379>  ZADD runoobkey 4 redis
(integer) 0
127.0.0.1:6379>  ZADD runoobkey 5 redi
(integer) 1
127.0.0.1:6379>  Zrange runoobkey 0 10
1) "redis"
2) "redi"
127.0.0.1:6379>  ZADD runoobkey 5 rediqwer
(integer) 1
127.0.0.1:6379>  ZADD runoobkey 6 rediqweqweqwr
(integer) 1
127.0.0.1:6379>  Zrange runoobkey 0 10
1) "redis"
2) "redi"
3) "rediqwer"
4) "rediqweqweqwr"
127.0.0.1:6379>exit #退出
```

## linux操作

> vi text.txt 进入文件操作

<img src="F:\记录\GO\image-20251121090349950.png" alt="image-20251121090349950" style="zoom:33%;" />

> top查看cpu的信息

<img src="F:\记录\GO\image-20251121110214172.png" alt="image-20251121110214172" style="zoom:33%;" />

> journalctl 查看日志文件

<img src="F:\记录\GO\image-20251121110639891.png" alt="image-20251121110639891" style="zoom:33%;" />



```bash
# 直接查看内容
cat /var/log/myapp.log
# 分页查看
less /var/log/myapp.log
# 实时跟踪日志
tail -f /var/log/myapp.log
```

<img src="F:\记录\GO\image-20251121110905389.png" alt="image-20251121110905389" style="zoom:33%;" />

> free -h 查看内存

>![image-20251121110950455](F:\记录\GO\image-20251121110950455.png)



> df -h 查看磁盘



![image-20251121111044708](F:\记录\GO\image-20251121111044708.png)





> ifconfig 查看网络

![image-20251121111134627](F:\记录\GO\image-20251121111134627.png)



>查看TCP/UDP监听端口
>
>ss -tuln
>
>查看80端口的进程
>
>lsof -i :80





<img src="F:\记录\GO\image-20251121111237192.png" alt="image-20251121111237192" style="zoom:50%;" />

![image-20251121111250975](F:\记录\GO\image-20251121111250975.png)



> 压缩
>
> zip 包名 路径
>
> 解压
>
> unzip  包名 路径

![image-20251121111509765](F:\记录\GO\image-20251121111509765.png)



> sodu useradd zds 加用户
>
> sodu passwd zds 改密码

![image-20251121111916002](F:\记录\GO\image-20251121111916002.png)



> ls -l 查看文件权限

![image-20251121112214628](F:\记录\GO\image-20251121112214628.png)



![image-20251121112420256](F:\记录\GO\image-20251121112420256.png)



## mysql

```cmd
mysql -u root -p
create database db_1;
```



![image-20251121113731394](F:\记录\GO\image-20251121113731394.png)

> mysql> create table db_2.t_1(
>     ->  name varchar(50)
>     -> );  创建表

![image-20251121114730228](F:\记录\GO\image-20251121114730228.png)

> 增加

```mysql
insert into t_3 values(1,'440111200011111101','Jim','Green');

select * from t_3;

select name,sfz from t_3;

update t_3 set sfz = '440100200010100001';

delete from t_3 where id = 2;

## JWT
```

##

- GenerateToken 生成
- ParseToken 解析
- DeleteToken 删除
- GetAccessToken 获取访问

通过读取config包里面的东西

```yaml
# JWT配置
jwt:
    signingKey: 39c54195e73304e74a8429b178965865
    expiresTime: 7d
    bufferTime: 1d
    issuer: xiujie # 发行人
    
signingKey：用于签名验证的密钥
expiresTime：token有效期为7天
bufferTime：缓冲时间为1天，用于提前刷新token
issuer：指定token发行方为"xiujie"
```





## 登录逻辑

- 先获取ip

- 通过LoginParams 封装logininfor
- 校验验证码 service.SysCaptcha().VerifyCaptcha(ctx, param.CaptchaID, param.CaptchaValue)
- 获取用户信息 service.SysUser().GetUserByUsernameAndPassword(ctx, param.TenantId, param.Username, param.Password)
- 构建临时的上下文
- 生成token
- 保存登录日志
- 保存在线列表

失败为1



## 获取token

从请求头"Authorization"字段或查询参数"access_token"中获取token信息
如果两个地方都为空，则返回错误
优先使用Authorization头中的Bearer token，如果为空则使用查询参数中的token





## 权限

#### 权限码

```go
"cpr:"             // 角色权限码前缀+角色权限
"cpm:"             // 菜单权限码前缀+菜单权限
"cpd:"             // 部门权限码前缀+部门编码
"cpp:"             // 岗位权限码前缀+岗位编码
"cpu:"             // 用户权限码前缀+用户名
"cpc:current:user" // 当前登录用户默认权限
```









## 获取服务器信息



### IMonitorServer获取所有的信息

**mi2ntior.go**

```go
	GetGoInfo(ctx context.Context) (res *model.GoRunInfo)
	// GetHostInfo 获取主机信息
	GetHostInfo(ctx context.Context) (data []byte)
	// GetSysLoad 获取系统负载信息
	GetSysLoad(ctx context.Context) (data []byte)
	// GetCpuInfo 获取CPU信息
	GetCpuInfo(ctx context.Context) (data []byte)
	// GetMemInfo 获取内存信息
	GetMemInfo(ctx context.Context) (data []byte)
	// GetDiskInfo 获取磁盘信息
	GetDiskInfo(ctx context.Context) (data []byte)
	// GetNetStatusInfo 获取网络信息
	GetNetStatusInfo(ctx context.Context) (data []byte)
```



### 获取电脑信息

>var gm runtime.MemStats 定义一个能够获取电脑信息的结构
>
>runtime.ReadMemStats(&gm)  写入

- 获取主机信息

```go
	hostInfo, _ := host.Info()
	timestamp, _ := host.BootTime()
```





# D6

## MODUSBUS

- Modbus-RTU
- Modbus-ASCII
- Modbus-TCP



> MODBUS 采用的是主从协议 --> 总线上每次只有一个数据进行传输，即主机发送，从机应答



## Modbus-RTU协议

**设备必须要有RTU协议！这是Modbus协议上规定的，且默认模式必须是RTU，ASCII作为选项**。

> **帧结构 = 地址 + 功能码+ 数据 + 校验**

- **地址**: 占用一个字节，范围0-255，其中有效范围是1-247，其他有特殊用途，比如255是广播地址(广播地址就是应答所有地址，正常的需要两个设备的地址一样才能进行查询和回复)。
- **功能码**：占用一个字节，功能码的意义就是，知道这个指令是干啥的，比如你可以查询从机的数据，也可以修改数据，所以不同功能码对应不同功能。
- **数据**：根据功能码不同，有不同结构，在下面的实例中有说明。
- **校验**：为了保证数据不错误，增加这个，然后再把前面的数据进行计算看数据是否一致，如果一致，就说明这帧数据是正确的，我再回复；如果不一样，说明你这个数据在传输的时候出了问题，数据不对的，所以就抛弃了。



![image-20251124102232494](F:\记录\GO\image-20251124102232494.png)

![image-20251124102252420](F:\记录\GO\image-20251124102252420.png)

- **发送**：从机的地址+我要干嘛的功能码+我要查的寄存器的地址+我要查的寄存器地址的个数+校验码
- **回复**：从机的地址+主机发我的功能码+要发送给主机数据的字节数+数据+校验码



# JWT

- GenerateToken 生成
- ParseToken 解析
- DeleteToken 删除
- GetAccessToken 获取访问

通过读取config包里面的东西

```yaml
# JWT配置
jwt:
    signingKey: 39c54195e73304e74a8429b178965865
    expiresTime: 7d
    bufferTime: 1d
    issuer: xiujie # 发行人
    
signingKey：用于签名验证的密钥
expiresTime：token有效期为7天
bufferTime：缓冲时间为1天，用于提前刷新token
issuer：指定token发行方为"xiujie"
```





## 登录逻辑

- 先获取ip

- 通过LoginParams 封装logininfor
- 校验验证码 service.SysCaptcha().VerifyCaptcha(ctx, param.CaptchaID, param.CaptchaValue)
- 获取用户信息 service.SysUser().GetUserByUsernameAndPassword(ctx, param.TenantId, param.Username, param.Password)
- 构建临时的上下文
- 生成token
- 保存登录日志
- 保存在线列表

失败为1



## 获取token

从请求头"Authorization"字段或查询参数"access_token"中获取token信息
如果两个地方都为空，则返回错误
优先使用Authorization头中的Bearer token，如果为空则使用查询参数中的token





# 权限

#### 权限码

```go
"cpr:"             // 角色权限码前缀+角色权限
"cpm:"             // 菜单权限码前缀+菜单权限
"cpd:"             // 部门权限码前缀+部门编码
"cpp:"             // 岗位权限码前缀+岗位编码
"cpu:"             // 用户权限码前缀+用户名
"cpc:current:user" // 当前登录用户默认权限
```









# mintior.go

## IMonitorServer获取所有的信息



```go
	GetGoInfo(ctx context.Context) (res *model.GoRunInfo)
	// GetHostInfo 获取主机信息
	GetHostInfo(ctx context.Context) (data []byte)
	// GetSysLoad 获取系统负载信息
	GetSysLoad(ctx context.Context) (data []byte)
	// GetCpuInfo 获取CPU信息
	GetCpuInfo(ctx context.Context) (data []byte)
	// GetMemInfo 获取内存信息
	GetMemInfo(ctx context.Context) (data []byte)
	// GetDiskInfo 获取磁盘信息
	GetDiskInfo(ctx context.Context) (data []byte)
	// GetNetStatusInfo 获取网络信息
	GetNetStatusInfo(ctx context.Context) (data []byte)
```



## 获取电脑信息

>var gm runtime.MemStats 定义一个能够获取电脑信息的结构
>
>runtime.ReadMemStats(&gm)  写入

- 获取主机信息

```go
	hostInfo, _ := host.Info()
	timestamp, _ := host.BootTime()
```







1.设备的参数属性从哪里调取？













- [x] 新加字段 登陆时间的字段 gf dao 一下
- [x] 所有的写在一个文件像下面
- [x] uuid32
- [x] view 返回向 要把新嘉的字段 返回  最后登录时间
- [x] defer 不用加







- [ ] 建立机构
- [ ] 建立工厂
- [ ] 工厂&机构绑定
- [ ] 用账号登录 设备设置
- [ ] 生产产品
- [ ] 建立订单
- [ ] 设备上来自动连接打印机
- [ ] 





11.27



```
            {
                "deviceId": 166,
                "deviceName": "1127测试设备",
                "deviceSecret": "049c161d",
                    "deviceSn": "00000054",
                "onlineState": 2,
                "lastOnlineTime": "2025-11-27 15:22:18",
                "lastLoginTime": "2025-11-27 15:12:18",
                "svrCode": "hztcp01",
                "remark": "",
                "deviceParams": "",
                "deviceProperties": "{\"properties\":[{\"key\":\"uartId\",\"value\":\"COM3\",\"time\":1764228138210},{\"key\":\"uartState\",\"value\":\"1\",\"time\":1764228138210},{\"key\":\"uartSendLastTime\",\"value\":\"0\",\"time\":1764228138210},{\"key\":\"uartRecvLastTime\",\"value\":\"0\",\"time\":1764228138210},{\"key\":\"startup\",\"value\":\"600\",\"time\":1764228138210}],\"time\":1764228138210}",
                "tenantId": "000000",
                "createdAt": "2025-11-27 15:02:32"
            },
```







- gf gen service：按 logic 文件的 package 声明分组生成

- gf gen ctrl：按 API 文件的目录结构（api/{模块名}/v1/）生成到对应 controller 目录

因此，要让 service 生成到 hzdevelopercenter.go，需要确保 logic 文件的包名是 hzdevelopercenter；要让 controller 生成到 hzdevelopercenter 目录，需要将 API 文件放在 api/hzdevelopercenter/v1/ 下。









```sql
2025-11-28T09:54:17.442+08:00 [DEBU] {c8620c3fd3087c1861902111fe71a759} [ 26 ms] [default] [hz_iot] [rows:1  ] SELECT COUNT(1) FROM `pc_product_test_device` LEFT JOIN `dc_debug_device` ddi ON (ddi.device_sn COLLATE utf8mb4_general_ci = pc_product_test_device.device_sn COLLATE utf8mb4_general_ci) LEFT JOIN `pc_device_info` pdi ON (pdi.device_sn COLLATE utf8mb4_general_ci = ddi.device_sn COLLATE utf8mb4_general_ci) LEFT JOIN `pc_product_info` ppi ON (ppi.product_code COLLATE utf8mb4_general_ci = ddi.product_code COLLATE utf8mb4_general_ci) WHERE ((`pc_product_test_device`.`tenant_id`='000000') AND (`pc_product_test_device`.`device_sn`='00000053')) AND `pdi`.`deleted_at` IS NULL AND `ppi`.`deleted_at` IS NULL
2025-11-28T09:54:17 [DEBU] {c8620c3fd3087c1861902111fe71a759} sMiddleware.ResponseHandler method: GET path: /api/v1/hzdevelopercenter/pcProductTestDevice/list, ================ end ================
```





```
package hzdevelopercenter

import (
	"context"
	"xiuadmin/internal/model"
	"xiuadmin/internal/service"

	v1 "xiuadmin/api/hzdevelopercenter/v1"
)

func (c *ControllerV1) PcProductTestDeviceList(ctx context.Context, req *v1.PcProductTestDeviceListReq) (res *v1.PcProductTestDeviceListRes, err error) {
	list, totalCount, err := service.GenPcProductTestDevice().List(ctx, &req.PcProductTestDeviceListParam)
	if err != nil {
		return
	}
	if list == nil {
		list = []*model.PcProductTestDeviceListModel{
			{ProductCode: ""},
		}
	}
	res = new(v1.PcProductTestDeviceListRes)
	//模拟一些数据
	res.Items = list
	res.PageResult.Page = req.Page
	res.PageResult.PageSize = req.PageSize
	res.PageResult.Total = totalCount
	return
}
func (c *ControllerV1) PcProductTestDeviceExport(ctx context.Context, req *v1.PcProductTestDeviceExportReq) (res *v1.PcProductTestDeviceExportRes, err error) {
	err = service.GenPcProductTestDevice().Export(ctx, &req.PcProductTestDeviceListParam)
	return
}
func (c *ControllerV1) PcProductTestDeviceView(ctx context.Context, req *v1.PcProductTestDeviceViewReq) (res *v1.PcProductTestDeviceViewRes, err error) {
	data, err := service.GenPcProductTestDevice().View(ctx, &req.PcProductTestDeviceViewParam)
	if err != nil {
		return
	}

	res = new(v1.PcProductTestDeviceViewRes)
	res.PcProductTestDeviceViewModel = data
	return
}
func (c *ControllerV1) PcProductTestDeviceEdit(ctx context.Context, req *v1.PcProductTestDeviceEditReq) (res *v1.PcProductTestDeviceEditRes, err error) {
	res = new(v1.PcProductTestDeviceEditRes)
	err = service.PcProductTestDevice().Edit(ctx, &req.PcProductTestDeviceEditParam)
	return
}
func (c *ControllerV1) PcProductTestDeviceDelete(ctx context.Context, req *v1.PcProductTestDeviceDeleteReq) (res *v1.PcProductTestDeviceDeleteRes, err error) {
	res = new(v1.PcProductTestDeviceDeleteRes)
	err = service.PcProductTestDevice().Delete(ctx, &req.PcProductTestDeviceDeleteParam)
	return
}
```



