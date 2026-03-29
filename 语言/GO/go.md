# 路由管理

## goframe

### 如何将函数与路径绑定在一起

```go
    s.BindHandler("/", func(r *ghttp.Request) {
        r.Response.Write("Hello World!")
    })
```

**BindHandler("路径",函数)**

通过`Server`对象的`BindHandler`方法*绑定路由*以及*路由函数*。在本示例中，我们绑定了`/`路由，并指定路由函数返回`Hello World`。

当然 我们也可以写一个函数 来代替func



### 批量绑定控制器

> s.BindObject("路径", obj)

```go
s.BindObject("/user", user.Controller{})
```

这样就可以一次性 将user包下面的方法给绑定到对应的路径下面

- `BindObject(path, obj)` 中，基础路径 `path` 需要**显式传入**（比如 `/user`），每次绑定都要手动指定。

GoFrame 的 `Bind` 方法为了兼容 “值类型” 和 “指针类型” 的控制器实例，内部做了**自动转换处理（统一转为指针类型）**，以确保指针接收者的方法能被正确识别和绑定。这就是为什么传入 `Hello{}` 和 `NewHello()`（返回 `*Hello`）会得到相同结果的底层原因。



### 分组

`RouterGroup.Bind(obj)` 中，基础路径来自**当前路由组（RouterGroup）预先定义的路径**（比如 `group := s.Group("/api")` 中，`/api` 就是基础路径），后续绑定无需重复写，直接继承组的路径。

```go
			s.Group("/api", func(group *ghttp.RouterGroup) {
				group.Middleware(ghttp.MiddlewareHandlerResponse)
				group.Group("/hello", func(helloGroup *ghttp.RouterGroup) {
					helloGroup.Bind(hello.NewHello())
				})

				group.Group("/user", func(userGroup *ghttp.RouterGroup) {
					userGroup.Bind(user.Controller{})
				})
           //分成了两个组 hello和user
           //但是 他们都在 api的下面 所以是api/user  api/hello
			})
```



### 规范路由

GoFrame中提供了规范化的路由注册方式，注册方法如下 

```go
func Handler(ctx context.Context, req *Request) (res *Response, err error)
```

其中`Request`与`Response`为自定义的结构体。

通过如下方式指定请求方法与路径

```go
type HelloReq struct {
    g.Meta `path:"/hello" method:"get"`
}
```





## 获取请求参数Req

```go
func (c *Hello) Param(ctx context.Context, req *hello.ParamReq) (res *hello.ParamRes, err error) {
	r := g.RequestFromCtx(ctx)
	name := r.GetQuery("name")
    data := r.GetQueryMap(g.Map{"name": name, "age": 2}) 
	r.Response.Writeln(name.String() + "2778")
	return
}
```

> g.RequestFromCtx(ctx) -->  从上下文中提取当前 HTTP 请求对象 `*ghttp.Request`。

在 GoFrame 框架中，每个 HTTP 请求处理时，框架会自动将当前请求的 `*ghttp.Request` 对象存入上下文 `ctx` 中，方便在函数调用链中传递和获取（无需显式在函数参数中传递 `*ghttp.Request`）。



>ctx context.Context -->是传递请求生命周期和元数据的载体。

- 里面包含了当前请求的元信息（如请求 ID、超时控制等），`g.RequestFromCtx` 通过这个上下文 “找到” 对应的请求实例。
- 示例中的意义：在 `Param` 方法中，通过 `g.RequestFromCtx(ctx)` 从上下文获取当前请求对象 `r`，后续可以通过 `r` 操作请求（如获取参数、写入响应等）。

> r.GetQuery("name")  -->获取 URL 查询参数中 key 为 "name" 的值。

> data := r.GetQueryMap(g.Map{"name": name, "age": 2})  这个name 和2 就是默认值 让这个其实获取的就是{"age":"1231","name":"啊啊啊","rwqe":"123123"} 这种格式的 当然 只要有 就写上去 即使是你不想要的



### Get方法

- **`GetQuery\*` 系列方法**（如 `GetQuery`、`GetQueryMap`）：仅用于获取 **URL 查询参数**（对应 HTTP 的 GET 方法参数）。
- **`GetForm\*` 系列方法**（如 `GetForm`、`GetFormMap`）：用于获取 **POST 表单参数**（如 `application/x-www-form-urlencoded` 或 `multipart/form-data` 类型的 POST 数据）。
- **`GetJson\*` 系列方法**：用于获取 **POST JSON 数据**（`application/json` 类型的 POST 数据）。

- `getRouterMap` 用于获取 **动态路由** {}里面的数据 



### 从req中直接取数值 goland推荐的

```go

func (c *Hello) Param(ctx context.Context, req *hello.ParamReq) (res *hello.ParamRes, err error) {
	r := g.RequestFromCtx(ctx)
	r.Response.Write(req)
	return
}

```

- 无需关心参数的原始提交方式（GET/POST/JSON 自动兼容）；
- 自动过滤无关参数（只保留 `ParamReq` 中定义的字段）；
- 支持通过结构体标签（如 `v:"required"`）做参数校验（框架会自动返回校验错误）。



### 自己写一个结构体 取接住

```go
func (c *Hello) Param(ctx context.Context, req *hello.ParamReq) (res *hello.ParamRes, err error) {
	r := g.RequestFromCtx(ctx)
	type User struct {
		Name string
		Age  int
	}
	var u User
	err = r.ParseForm(&u)
	if err != nil {
		r.Response.Write(u)
	}
	return
}

```



### p d

```go
type ParamReq struct{
	Name string `p:"name" d:"无名"`
    Age  int `json:age`
}
如果 前端不是name 但是你又不想新建一个字段  那么就用 p 起一个别名 也鞥适用于name
d 默认值
josn 可以让返回值返回为json的样式
```









## 相应Res

> r.Response.Writeln() 最常用的 直接返回文字 结构体 甚至是html标签

r.Response.

- Writeln 换行
- Write 不换行
- Writef 可以有占位符
- WriteJson()





# 数据库

## 前期准备

### 配置文件

> *manifest/config/config.yaml*

```yml
server:
    address: ":8000"         # 服务监听端口
    openapiPath: "/api.json" # OpenAPI接口文档地址
    swaggerPath: "/swagger"  # 内置SwaggerUI展示地址
 
database:
  default:
    link:   "mysql:root:12345678@tcp(127.0.0.1:3306)/star?loc=Local"
    debug:  true
```

## CRUD

### 查询

> `g.Model("表名")` 返回是模型对象

`g.Model("products")` 会创建一个与 `products` 表绑定的模型对象（后续所有操作都针对该表）。



获得到 g.MOdel()以后 比如 query：= g.MOdel(“一个表名”)就可以通过

**products, err := query.One()** 查询一个 返回值是 查询的返回值 和错误数量



- qu.**One**()查询一条
- qu.*All*() 查全
- qu.**Fields**("id", "vend_id").All()  查询那两个字段
- qu.**FieldsEX**("id", "vend_id").All() 查询 除了那两个字段
- qu.**Value**("id") 查id这一个字段的一个数据
- qu.**Array**("id")查id这一个字段的所有
- qu.**Count**（） 用于查询并返回记录数。



> 如果对于同一个g.Model() 的对象 query 使用的话 就会在原有的基础上 查询 （上一条的查询语句 会影响到吓一跳）

所以我们的解决思路就是 --->  每次都查询的是g.Model().查询的内容





> ### Where

- qu.**Where**("id", 1).All() --> 查询id=1

- qu.**Where**("id=2").All() ---> 查询id=2

- qu.**WhereGT**("id",2).All() ---> 查询id>   GTE >=

- LT --><   LTE--> < =
- Where("level=? OR money >=**?**", 1, 1000000)  使用?作为占位符
- Where("level", 1)**.WhereOr**("money >=", 1000000)  或者直接使用WhereOr来表示or

> ### Order

pro, error := qu.WhereGTE("id", 2).OrderAsc("prod_price").All()

> ### Group

pro, error := qu.WhereGTE("id", 2).Group("prod_price").All()

> ### 分页

pro, error := qu.Limit(1,3).All()

pro, error := qu.Page(3,5).All() 第三页 每页五个



#### 结构体映射

```go
	type Products struct {
		Id        int
		VendId    int
		ProdName  string
		ProdPrice float64
	}
	var products *Products
	_ = g.Model("products").Scan(&products) 其实这个的返回值是error
	req.Response.WriteJson(products)

```

> Scan 的是& 因为你是直接把这个对象给改了

然后有一点

var products *Products 和var products Products的返回结果是一样的

但是在查询查不到的地方 用指针的是Null  而用对象的是返回语句 “sqlxxx查不到”



> 把var products *Products 改成var products 【】\*Products 就是查询所有的了

### 插入

### insert

```go
result, err := qu.Data(data).Insert()
等价的两句话
result, err := qu.Insert(data)
```



返回值

```go
{
    "Result": {
        "Locker": {}
    },
    "Affected": 1 影响的数量
}
```



#### Replace

```go
result, err := qu.Replace(data)
```

根insert的区别就在于 replace是替换 就是即使是主键冲突 也可以改 当然 也可以添加 

而insert就是会报错

3.save 如果写入的数据中存在主键或者唯一索引时，更新原有数据，否则写入一条新数据。

#### 批量插入

```go
	type Products struct {
		Id        int
		VendId    int
		ProdName  string
		ProdPrice float64
	}
	data := g.List{//一系列键值对集合
		{"vend_id": 10, "prod_id": 11, "prod_name": "商品2", "prod_price": 100.0},
		{"vend_id": 20, "prod_id": 11, "prod_name": "商品2", "prod_price": 100.0},
		{"vend_id": 40, "prod_id": 11, "prod_name": "商品2", "prod_price": 100.0},
		{"vend_id": 40, "prod_id": 11, "prod_name": "商品2", "prod_price": 100.0},
	}
	result, err := qu.Insert(data)
	if err == nil {
		req.Response.WriteJson(result)
	}
```

或者是g.Array 但是里面就要存g.map或者结构体

```go
    data := g.Array{//存g.map或者结构体
        g.Map{
            "vend_id":    10,       // 正确字段名（避免拼写错误）
            "prod_name":  "商品A",   // 必填字段
            "prod_price": 99.9,     // 价格字段
        },
        g.Map{
            "vend_id":    10,
            "prod_name":  "商品B",
            "prod_price": 199.9,
        },
    }
```



### Update

#### `Update`

```go
// UPDATE `user` SET `status`=1 WHERE `status`=0 ORDER BY `login_time` asc LIMIT 10
g.Model("user").Data("status", 1).Order("login_time asc").Where("status", 0).Limit(10).Update()

//或者直接使用update
g.Model("user").Update("status=1", 1)
```

 用于数据的更新，往往需要结合 `Data` 及 `Where` 方法共同使用。 `Data` 方法用于指定需要更新的数据， `Where` 方法用于指定更新的条件范围。同时， `Update` 方法也支持直接给定数据和条件参数。

#### `Decremet&Increment`

```go
result, err := qu.WhereLTE("id", 5).Increment("prod_price",10) 
//让那些 id《=5的 加10的price
```



### 删除

#### Delete()

```
g.Model("user").Where("uid", 10).Delete()
```





## 事务

### 普通的写法

```go
	db := g.DB()//是 g 提供的数据库操作方法，用于获取一个数据库连接对象
//如果有上下文 也就那个 ctx context.Context 可以直接写成 db.Begin(ctx)  这个req.Context()==ctx
	if tx, err := db.Begin(req.Context()); err == nil {
		r, err := tx.Save("user", g.Map{
			"id":   1,
			"name": "john",
		})
		if err == nil {
			tx.Commit()
		} else {
			tx.Rollback()
		}
		fmt.Println(r)
	}
```

```go
data := new(Products)
	tx, err := g.DB().Begin(req.Context())//开启事务
	if err != nil {
		req.Response.WritelnExit("发生错误: " + err.Error())//事务开启时失败
	}

	md := tx.Model("book")
	result, err := md.Insert(data)

	if err == nil {//插入失败
		tx.Commit()
		req.Response.WriteJson(result)
	} else {
		tx.Rollback()
		req.Response.Writeln("发生错误: " + err.Error())
	}
```



### 闭包操作



```go
	g.DB().Transaction(context.TODO(), func(ctx context.Context, tx gdb.TX) error {
		data := new(Products)
		tx, err := g.DB().Begin(req.Context())
		if err != nil {
			req.Response.WritelnExit("发生错误: " + err.Error())
		}
		md := tx.Model("book")
		result, err := md.Insert(data)

		if err == nil {
			req.Response.WriteJson(result)
		} else {
			req.Response.Writeln("发生错误: " + err.Error())
		}
		return err
	})
```

这个方法 就是通过判断error方法 来实现对于自动提交和rollback





## 原生sql 

> 你想听我原生家庭的故事吗

```go
sql := "INSERT INTO `book` (`name`, `author`, `price`) VALUES (?, ?, ?)"
db := g.DB()
result, err := db.Exec(req.Context(), sql, g.Array{3, 7})
```



## DAO

自动生成的 先去hack/config.yml中进行

然后在执行 gf gen dao

| `/internal/dao`          | 数据操作对象 | 通过对象方式访问底层数据源，底层基于 `ORM` 组件实现。往往需要结合 `entity` 和 `do` 共同使用。该目录下的文件开发者可扩展修改。 |
| ------------------------ | ------------ | ------------------------------------------------------------ |
| `/internal/model/do`     | 数据转换模型 | 数据转换模型用于业务模型到数据模型的转换，由工具维护，用户不能修改。工具每次生成代码文件将会覆盖该目录。关于 `do` 文件的介绍请参考： - [数据模型与业务模型](https://goframe.org/docs/design/project-models) - [DAO-工程痛点及改进](https://goframe.org/docs/design/project-dao-improvement) - [利用指针属性和do对象实现灵活的修改接口](https://goframe.org/docs/core/gdb-practice-using-pointer-and-do-for-update-api) |
| `/internal/model/entity` | 数据模型     | 数据模型由工具维护，用户不能修改。工具每次生成代码文件将会覆盖该目录。 |

do 在插入的时候不会将其他的给顶替掉

而 entity 的却会 



### 连表查询

![image-20251114192410449](F:\记录\GO\image-20251114192410449.png)

在entity表里面 写入其他表的指针 然后 json表示的是 在关联查出的名字 然后orm表示的是链表的 whit"xx"="xx"



```
db := g.DB()
db.With(entity.Dept{}).Scan(&emp)
```



> db.With(entity.Dept{} 也就是entity表里面的构造器 也是要链表 ).Scan(&emp  是这个表的对象 也是)





## Service层

作用：

- 定义接口
- 定义接口变更
- 定义一个获取接口实例的函数
- 定义一个接口实现的注册方式



dcdevicegroup.go

dc_device_group.go

dc_device_group.go





# WebSocket



<img src="F:\记录\GO\image-20251207094527958.png" alt="image-20251207094527958" style="zoom:33%;" />

WebSocket 通信始于 **HTTP 握手**，客户端通过 `Upgrade` 请求头将 HTTP 连接升级为 WebSocket 连接。握手成功后，双方通过**消息帧**（文本、字节或控制帧如 ping/pong）进行通信。控制帧用于维护连接，例如 ping/pong 用于检测连接是否存活。



#### `type Conn`：网络连接的通用抽象

`Conn`是Go语言定义的网络连接接口，涵盖了所有网络类型（TCP、UDP、Unix域套接字等）的公共操作：

## >websocket升级器

```go
// wsUpgrader WebSocket 升级器配置
var wsUpgrader = websocket.Upgrader{
	// CheckOrigin 允许任何来源（开发环境）
	// 生产环境中应该实现适当的来源检查以确保安全
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
	// Error 处理升级失败的错误
	Error: func(w http.ResponseWriter, r *http.Request, status int, reason error) {
		// 在这里实现错误处理逻辑
		g.Log().Errorf(r.Context(), "WebSocket upgrade error: %v", reason)
	},
}
```







> ### 原生

```go
package websocket


import (
	"net/http"

	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
	"github.com/gorilla/websocket"
)

// WebSocketController WebSocket 控制器
type WebSocketController struct{}

// New 创建 WebSocket 控制器实例
func New() *WebSocketController {
	return &WebSocketController{}
}

// wsUpgrader WebSocket 升级器配置
var wsUpgrader = websocket.Upgrader{
	// CheckOrigin 允许任何来源（开发环境）
	// 生产环境中应该实现适当的来源检查以确保安全
	CheckOrigin: func(r *http.Request) bool {
		return true
	},
	// Error 处理升级失败的错误
	Error: func(w http.ResponseWriter, r *http.Request, status int, reason error) {
		// 在这里实现错误处理逻辑
		g.Log().Errorf(r.Context(), "WebSocket upgrade error: %v", reason)
	},
}

// Echo WebSocket Echo 服务器处理器
// 路径: /ws
func (c *WebSocketController) Echo(r *ghttp.Request) {
	// 将 HTTP 连接升级为 WebSocket
	ws, err := wsUpgrader.Upgrade(r.Response.Writer, r.Request, nil)
	if err != nil {
		r.Response.Write(err.Error())
		return
	}
	defer ws.Close()

	// 获取请求上下文用于日志记录
	var ctx = r.Context()
	logger := g.Log()

	// 消息处理循环
	for {
		// 读取传入的 WebSocket 消息
		msgType, msg, err := ws.ReadMessage()
		if err != nil {
			break // 连接关闭或发生错误
		}
		// 记录接收到的消息
		logger.Infof(ctx, "received message: %s", msg)
		// 将消息回显给客户端
		if err = ws.WriteMessage(msgType, msg); err != nil {
			break // 写入消息时出错
		}
	}
	// 记录连接关闭
	logger.Info(ctx, "websocket connection closed")
}

```





> ### 使用`gorilla/websocket`库



```go
package main

import (
    "log"
    "net/http"
    "github.com/gorilla/websocket"
)

// 定义 WebSocket 升级器
var upgrader = websocket.Upgrader{
    CheckOrigin: func(r *http.Request) bool { return true },
}

// 存储所有客户端连接
var clients = make(map[*websocket.Conn]bool)
var broadcast = make(chan string) // 广播消息通道

// 处理 WebSocket 连接
func handleConnections(w http.ResponseWriter, r *http.Request) {
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Printf("upgrade error: %v", err)
        return
    }
    defer ws.Close()

    // 注册新客户端
    clients[ws] = true

    for {
        var msg string
        err := ws.ReadJSON(&msg)
        if err != nil {
            log.Printf("read error: %v", err)
            delete(clients, ws)
            break
        }
        // 将消息发送到广播通道
        broadcast <- msg
    }
}

// 广播消息给所有客户端
func handleBroadcast() {
    for {
        msg := <-broadcast
        for client := range clients {
            err := client.WriteJSON(msg)
            if err != nil {
                log.Printf("write error: %v", err)
                client.Close()
                delete(clients, ws)
            }
        }
    }
}

func main() {
    http.HandleFunc("/ws", handleConnections)
    go handleBroadcast() // 启动广播 goroutine
    log.Fatal(http.ListenAndServe(":8080", nil))
}


```

- `clients` 存储所有活跃连接，`broadcast` 通道用于消息分发。
- 每个客户端连接运行在独立 goroutine 中，处理消息读取。
- `handleBroadcast` 循环监听广播通道，将消息推送给所有客户端。





>### 并发处理

```go
package main

import (
    "log"
    "net/http"
    "sync"
    "github.com/gorilla/websocket"
)

var upgrader = websocket.Upgrader{CheckOrigin: func(r *http.Request) bool { return true }}
var clients = make(map[*websocket.Conn]bool)
var mutex = sync.RWMutex{} // 保护 clients 并发访问

func handleConnections(w http.ResponseWriter, r *http.Request) {
    ws, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        log.Printf("upgrade error: %v", err)
        return
    }

    // 注册客户端
    mutex.Lock()
    clients[ws] = true
    mutex.Unlock()

    defer func() {
        mutex.Lock()
        delete(clients, ws)
        mutex.Unlock()
        ws.Close()
    }()

    for {
        var msg string
        err := ws.ReadJSON(&msg)
        if err != nil {
            log.Printf("read error: %v", err)
            break
        }
        // 广播消息（简化示例）
        mutex.RLock()
        for client := range clients {
            client.WriteJSON(msg)
        }
        mutex.RUnlock()
    }
}
```



> 性能优化 通过临时的bufferPool 

```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 1024)
    },
}

func readMessage(ws *websocket.Conn) ([]byte, error) {
    buf := bufferPool.Get().([]byte)
    defer bufferPool.Put(buf)
    _, data, err := ws.ReadMessage()
    return data, err
}

```



> 心跳

```go
func handlePingPong(ws *websocket.Conn) {
    ticker := time.NewTicker(30 * time.Second)
    defer ticker.Stop()
	
    for {
        select {
        case <-ticker.C:
            if err := ws.WriteMessage(websocket.PingMessage, []byte{}); err != nil {
                log.Println("ping error:", err)
                return
            }
        }
    }
}

```



> 多房间聊天

```go
type Room struct {
    clients map[*websocket.Conn]bool
    broadcast chan string
}

func (r *Room) handleBroadcast() {
    for msg := range r.broadcast {
        for client := range r.clients {
            client.WriteJSON(msg)
        }
    }
}

```







#  GC

垃圾回收的过程 分为两个半独立的组件

> 赋值器 & 回收器

赋值器：这一名称本质上是在指代用户态的代码。因为对垃圾回收器而言，用户态的代码仅仅只是在修改对象之间的引用关系，也就是在对象图（对象之间引用关系的一个有向图）上进行操作。

回收器：执行垃圾回收的代码



> 根对象 --> 垃圾回收器在标记的过程中最先检查的对象

- 全局变量 ：程序在**编译期**就能确定的那些存在于程序整个生命周期的**变量**。
- 执行栈 ：每个goroutine 都包含自己的执行栈 这些执行栈包含**栈上的变量**以及指向**分配的堆内存区块的指针**
- 寄存器 寄存器的值可能表示一个**指针**，参与计算的这些指针可能指向某些赋值器分配的堆内存区块



> GC方法 追踪式GC

从根对象出发 根据对象之间的引用信息 一步步推导知道扫描完整个堆并且确定要保存的对象 从而确定回收的对象



> 三色标记法

三色抽象规定了三种不同类型的对象，并用不同的颜色相称	

- 白色对象**（可能死亡）**：未被回收器访问到的对象。在回收开始阶段，所有对象均为白色，当回收结束后，白色对象均不可达。
- 灰色对象**（波面）**：已被回收器访问到的对象，但回收器需要对其中的一个或多个指针进行扫描，因为他们可能还指向白色对象。
- 黑色对象**（确定存活）**：已被回收器访问到的对象，其中所有字段都已被扫描，黑色对象中任何一个指针都不可能直接指向白色对象。



<img src="F:\记录\GO\image-20251220221017875.png" alt="image-20251220221017875" style="zoom:33%;" />





