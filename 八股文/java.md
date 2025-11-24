## [java新特性]()

### java8

- **用元空间替代了永久代**	

- **引用了Lambda表达式**

- **函数式接口**

  - Predicate ->条件判断

  - ```java
    // Predicate<T> 用于条件判断
    Predicate<Integer> isEven = n -> n % 2 == 0;
    Predicate<String> isEmpty = String::isEmpty;
    Predicate<String> isNotEmpty = isEmpty.negate();  // 取反
    
    List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> evenNumbers = numbers.stream()
        .filter(isEven)
        .collect(Collectors.toList());
    
    ```

  - Function 数据转换

  - ```java
    // Function<T, R> 用于转换
    Function<String, Integer> stringLength = String::length;
    Function<Integer, String> intToString = Object::toString;
    
    // 函数组合
    Function<String, String> addPrefix = s -> "前缀-" + s;
    Function<String, String> addSuffix = s -> s + "-后缀";
    Function<String, String> combined = addPrefix.andThen(addSuffix);
    String result = combined.apply("鱼皮"); // "前缀-鱼皮-后缀"
    
    ```

    

- **引入了日期类**

  - 从原来的Date&Calendar 引入新的LoacalDate
    - 解决了一些旧日历的线程安全..可变性...之类的

- **接口默认方法、静态方法**

  - **接口升级**：比如 `Java 8` *中对 `Collection` 接口新增了 `stream()` 等默认方法，所有集合实现类（如 `ArrayList`、`HashMap`）无需修改即可直接使用这些方法。
  - **提供通用实现**：接口中可以定义通用逻辑的默认方法，减少实现类的重复代码。

- **新增Stream流式接口**

  - 新增一些中间的stream API 能够让对集合的处理更加的`优雅`
    - filter()
    - map()
    - sorted()
    - distinct()
    - limit()
    - skip()
  - 也有一些终端的操作 让
    - collection()
    - foreach()
    - count()
    - findFirst()
    - anyMatch()//是否有匹配的
    - reduce()//归约操作

- **引入Optional类**

- **新增了CompletableFuture,StampedLock...并发实现类**



### java11

- HTTP客户端的API

- String类的新特性

  - isBlank()-->判空
  - strip() -->出去空白字符
  - lines()-->将字符串分割成Stream
  - repeat()-->重复字符串

- File类

- 改进了原先难用的FileReader FileWriter

  - ```java
    // 写入文件 Files.writeString
    String content = "这是一个测试文件\n包含多行内容\n中文支持测试";
    Path tempFile = Files.writeString(
        Paths.get("temp.txt"), 
        content,
        StandardCharsets.UTF_8
    );
    
    // 读取文件 Files.readString
    String readContent = Files.readString(tempFile, StandardCharsets.UTF_8);
    System.out.println("读取的内容:\n" + readContent);
    
    ```

  

### java17 `代码设计安全性` `JDK稳定性`

- **虚拟线程**

  - 传统的线程 每一个线程都对应着一个真实的线程 那么 虚拟线程 就可以 栈内存按需分配（初始可小至 KB 级），创建 / 销毁成本极低。

  - 可轻松创建百万甚至千万级

  - 由 JVM 的**虚拟线程调度器**管理，最终映射到平台线程执行。

  - 使用简单

    - ```java
      // 直接创建虚拟线程
      public void handleSingleUser(Long userId) {
          Thread.ofVirtual().start(() -> {
              // 要异步执行的任务
              User user = userService.findById(userId);
              processUser(user);
          });
      }
      
      ```

  - **Switch匹配方式重大跟新**

    - 支持条件判断
    - 可以匹配对象类型 然后直接转换 优雅~~

  - **Record模式（不想看 极简化不是我的代码风格）**

    - 记住一次性可以提取所有的信息即可



### java25

- **Scoped** **Value**
  - 这个方法允许在`线程`内以及`子线程`之间安全高效的`共享不可变数据`
  - 并且开销小 why
    - 因为是这样的比如一个线程里面有一个资源 你开1000个 总不能复制1000个吧 不然太慢了  所以ScopedValue 就是为了不让复制1000个而产生的





## 双亲委派模型

**Def**:当类加载器 去加载某个类的时候 会委派父类加载器 ---直到->根类加载器 BootStrap ClassLoader ，当父类的无法加载时 才给本类加载  

**为了什么？：**

1. 安全性 比如java.lang.object 只能由根类加载器使用 因此如果有人替换了Object也没事 、
2. 一致性 保证一个类在JVM中只会被加载一次 让我们每次用的都是用一个类对象
3. <img src="F:\记录\八股文\image-20251024152138996-1761290506783-1.png" alt="image-20251024152138996" style="zoom:33%;" />

 三种类加载器 

1. 启动类加载器 --》虚拟机的一部分 由C++实现的 负责加载<JAVA_HOME>\lib 目录中或被 -Xbootclasspath 指定的路径中的并且文件名是被虚拟机识别的文件。
2. 扩展类加载器--》java实现的，独立于虚拟机 负责加载 <JAVA_HOME>\lib\ext 目录中或被 java.ext.dirs 系统变量所指定的路径的类库。
3. 应用程序类加载器--》java实现的，独立于虚拟机 主要负责加载用户类路径 ( classPath ) 上的类库，如果我们没有实现自定义的类加载器那这玩意就是我们程序中的默认加载器。

 
