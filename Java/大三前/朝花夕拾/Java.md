# Java

# 面向对象初级部分

### 为什么 非static可以调用static而非static不能调用static

因为在程序创立之初，static就跟上面的class一起存在了，而非static需要实例化以后才能存在。

## 构造器

```java
public class Person {
    int age;
    String name;
        public Person() {
    }
    public Person(int age) {
        this.age = age;
    }
    public Person(String name) {
        this.name = name;
    }
}
```

**1.与类名相同**

**2.没有返回值**

**3.new的本质就是在调用构造方法**

![image-20240903144853276](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20240903144853276.png)

注意：如果建立了有参构造器，像

```java
public Person(String name) {
    this.name = name;
}
```

那么在new一个对象的时候就要使用有参的形式，比如

```java
Person xm=new Person("xiaoming");
```

要不然就要在类里面，加一个无参构造器

```java
public Person() {}
```

无参构造器的作用：

1.初始化对象的值

## 重写：需要有继承关系，子类重写父类的方法：

1.方法名要相同 

2.参数列表必须相同

3.修饰符：范围可以扩大但不能缩小

4.抛出的异常：范围可以缩小不能扩大



## 多态

1.多态是方法的多态，属性没有多态

2.父类和子类要有联系，类型转换异常

3.存在条件：继承关系，方法需要重写，父类引用指向子类对象

注意：static属于类，不属于实例//final变量//private方法不能多态

### 动态绑定机制***



## 

## 关键字

### instanceof

作用就是检测是否具有父子关系

类名 instanceof 类名 结果就是True 和false

### This 与Super

这两个关键词的创建就是为了解决子类全局变量与子类内部变量，以及子类与父类的变量如果相同，该如何区分的问题

this 就是自己，super就是父类

基本用法就是 this.XXX 或者Super.XXX

如果父类的父类。。。。也有相同的变量名字，则依照就近原则

Super 也可以直接访问爷类

### ==与equal()的区别

其实都是判断是否相等，一个是数值 equal的相等，另一个是地址==的相等

注意object中的equal是重写了==所以有时候也是判断地址

### finlalize(释放资源——》亡语)

就是现在要开始考虑资源的占用率了，不用的资源就要关掉或者删除

这个方法是在object里面的，如果不重写的话，这B养的万一灭有一点用

woc 换了 被删除了 哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈哈 不学了 哈哈哈哈哈哈哈哈哈哈哈哈哈

## 可变参数

将参数相同代码实现功能基本相同的函数，用可变参数进行整合

```java
/*这里的参数，可以是0-多.把sum看作是数组
可变参数可以是 数组
可变参数可以与其他参数在一起，但是可变参数一定要是在最后
*/
package EXE.Demon04;

public class VarParament {
    public static void main(String[] args) {
        T t1=new T();
        t1.varparament("小明",11,15,15,15,510,5,15,1501,0,99);
    }
}
class T {
    void varparament(String name,int ... num)
    {
        int sum=0;
        for(int i=0;i<num.length;i++)
        {
            sum+=num[i];
        }
        System.out.println("名字"+name+sum);
    }
}


```



——————————————————————————————————

## 内部类

两种分类方式

1定义在局部位置上

局部内部类----》有类名

{

定义在外部类的局部位置

1.可以访问外部类的所有成员，包括私有的

2.作用域：仅仅在它的方法或者代码块中

3.访问内部类 直接访问

4.

}

匿名内部类-----》没有类名

2.定义在成员位置上

成员内部类

静态内部内（static）

## 匿名内部类

简化开发，如果一个类只想用一次 不想再用了， 于是使用这个，就可以只生成一个对象

类名 对象名=new 类名(){        这里面直接重构      }

编译类型是就类 

但是运行类型变成了类$01 



## 成员内部类

## 代码块

在一个类里面例如

> ***类的五大成员====属性 方法 构造器 代码块 内部类***

一种构造器的应用

![image-20240909194920061](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20240909194920061.png)

```java
class Person{
    //普通代码块
    {
        一般用来赋初始值
            只有创建的时候才会调用
            
            每次建造就会执行一次
    }
    //静态代码块
    static{
        只执行一次，
        第一执行，只执行一次
    }
    //
    Person(){
        //构造器
    }
    
}
```

在静态代码块/属性与普通代码块/属性的时候

调用的顺序是 父类的静态》子类静态》父类普通》父构造》子类普通》子构 。相同等级按照先后顺序

## 静态导入包

```java
import static java.lang.Math.random;
这样以后的代码就可以不用写Math.random();
就可以直接写random();
有趣的用法
```



## 静态变量(类变量)

静态变量就是类变量 可以让所有的变量共同使用。

所有对象一起使用

如果只有public 没有static 对象不可以使用





## 抽象类（abstract）

```java
public abstract class  Person {
   abstract void  run();//正确写法
   abstract void  run(){}//错误写法，抽象类 不能有主体，它的实现是通过继承它的子类来实现的
    /*1. 子类继承了Person也需要
    2.  一个类只能继承一个抽象类，而一个类却可以实现多个接口
    抽象类的存在的意义是什么？？？？
    -》就是
    */
}
```



## 重载与重写的区别

![1725531981115](F:\Wechat\WeChat Files\wxid_13iogzk6c5q612\FileStorage\Temp\1725531981115.png)

## 向上转型与向下转型

```java
package EXE.Demon01;

public class Main {
    public static void main(String[] args) {
        Animal cat = new Cat();
    }
}

class Animal {
    int age;
    String name;

    void eat() {
        System.out.println("eat");
    }

    void sleep() {
        System.out.println("sleep");
    }
}

class Cat extends Animal {
    Cat() {
    }

    void run() {
        System.out.println("cats run");
    }

}

```

就向这样的就是向上转型。

编译类型是Animal 但是数据类型还是Cat

注意

1. 向上转型后，子类单独定义的方法会丢失（父类并不知道子类定义的新属性与方法）
2. 父类引用可以指向子类对象，但是子类引用不能指向父类对象
3. 如果子类中重写了父类的方法，那么调用这个方法的时候，将会调用子类中的方法
4. 访问属性看编译类型，访问方法看运行类型

## 接口

作用：

1.约束

2.定义一些方法，让不同的人实现

3.是个抽象类

4.里面的变量都是public abstract final 

5.implements可以实现多个接口

### 接口的多态数组（程序猿们太叼了）

比如一个A B C都是实现了English接口

定义一个English数组，那么这个数组就可以放A B C



## 简易计算器

```java
package Calculator;

import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyAdapter;

public class Main {
    public static void main(String[] args) {
        new Calculator();
    }
}

class Calculator extends Frame {
    Calculator() {
        TextField tf1 = new TextField(10);
        TextField tf2 = new TextField(10);
        TextField tf3 = new TextField(20);
        Label label = new Label("+");
        Button button = new Button("=");
        button.addActionListener(new AddActionListener(tf1, tf2, tf3));
        setLayout(new FlowLayout());
        add(tf1);
        add(label);
        add(tf2);
        add(button);
        add(tf3);
        setVisible(true);
        pack();
    }
}

class AddActionListener implements ActionListener {
    TextField tf1, tf2, tf3;

    AddActionListener(TextField tf1, TextField tf2, TextField tf3) {
        this.tf1 = tf1;
        this.tf2 = tf2;
        this.tf3 = tf3;
    }

    public void actionPerformed(ActionEvent e) {
        int num1 = Integer.parseInt(tf1.getText());
        int num2 = Integer.parseInt(tf2.getText());
        tf3.setText("" + (num1 + num2));
    }
}

```

# 中级部分

## 集合

动态保存任意多个对象

![image-20240919194743834](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20240919194743834.png)

```java
List list =new Arrylist();
//增加
list.add(obj);
//删除
list.remove(0);
删除第0个元素
//查找
list.contains("XXX");
//元素个数
list.size()
//是否为空
  list.isEmpty();
//清空
list.clear()
    
//
    





```

## List 

arrylist

vector



## Set

特点：

1. 存放是无序的，但是取出是固定的
2. 不能存放重复的元素
3. 可以加NULL但是只能一个
4. 对象不能通过索引获取，因为没有get方法。 只能用迭代器和for each
5. 



### Hashset

1. 是set接口的实现类
2. 实际上是HashMap
3. 红黑树：数组加链表 每个数组里面存放的是头节点节点后面是链表。链表多了（链表8个并且数组超过64，就会变成树 ）



## Map

1. 存放的是K-V类型  要存入两个 其中是以K来排序 有点像hash

Man的常用方法

1. put
2. remove 根据K删除映射关系
3. get 根据K获取值
4. size
5. isEmpty
6. clear
7. containsKey查找K是否存在

## 泛型

我不知道到底有什么 但是我知道一定有 并且有不止一个数据类型的东西

## 枚举和注解

## paint

```Java
        //画圆
        //g.drawOval(10, 10, 100, 100);
        System.out.println("正在画画~~");

        //画直线 drawLine(int x1, int y1, int x2, int y2)
        //g.drawLine(10, 10, 100, 100);

        //画矩形边框 drawRect(int x, int y, int width, int height)
        //g.drawRect(10, 10, 100, 100);

        //填充矩形 fillRect(int x, int y, int width, int height)
        //g.setColor(Color.BLUE);//设置画笔颜色
        //g.fillRect(10, 10, 100, 100);

        //填充椭圆 fillOval(int x, int y, int width, int height)
        //g.setColor(Color.RED);
        //g.fillOval(10, 10, 100, 100);

        //画图片 drawImage(Image img, int x, int y, ..)
        //1.获取图片资源，/bg.png 表示在该项目的根目录去获取 bg.jpg 图片资源
        //Image image = Toolkit.getDefaultToolkit().getImage(Panel.class.getResource("/bg.jpg"));
        //g.drawImage(image, 0, 0, 1920, 1200, this);

        //画字符串 drawString(String str, int x, int y)
        //给画笔设置颜色和字体
        //g.setColor(Color.red);
        //g.setFont(new Font("隶书", Font.BOLD, 50));
        //这里设置的100，100是“北京你好”的左下角
        //g.drawString("北京你好", 100, 100);
```

## 进程与线程

比如有一个进程 有两个线程  是main 与Cat

main 结束了以后并不会导致进程的结束 要等到 cat结束了以后才能结束进程

进程产生线程 线程依附于进程

如果直接使用run方法而不是start的话 那么其实就是用了一下run方法 只是将他作为一个函数使用 并不会产生线程

线程的入口是run函数 但是run函数其实是调用了start()函数 start函数最终其实是给到的是start0函数 这是最底层的了 而start0是用C/C++写的

![image-20240924214741433](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20240924214741433.png)

#### 关键字synchronized 可以让线程一个个来

```java
package EXE.Thread;

public class SellTick {
    public static void main(String[] args) {
        Sell sell = new Sell();
        new Thread(sell).start();
        new Thread(sell).start();
        new Thread(sell).start();

    }
}

class Sell implements Runnable {
    public static int tick = 50;

    synchronized void sell() {
        if (tick > 0) {
            System.out.println(Thread.currentThread().getName() + "当前窗口售票成功，剩余：" + tick);
            tick--;
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

    @Override
    public  void run() {
        while (true) {
            sell();
        }
    }
}

```

```java
package EXE.Thread;

public class SellTick_Thread {
    public static void main(String[] args) {
        Sell02 sell03 = new Sell02();
        Sell02 sell02 = new Sell02();
        Sell02 sell01 = new Sell02();
        sell03.start();
        sell02.start();
        sell01.start();
    }
}

class Sell02 extends Thread {
    public static int tick = 50;

    synchronized static void sell() {
        if (tick > 0) {
            System.out.println(Thread.currentThread().getName() + "当前窗口售票成功，剩余：" + tick);
            tick--;
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }

    @Override
    public  void run() {
        while (true) {
            sell();
        }
    }
}

```

## 互斥锁

不知道，等一下查一下吧

## 死锁

## 异常体系图

## 鼠标监听

## 窗口监听

## 键盘监听

## 设计模式

### 单例设计模式

#### 饿汉式

```java
/*
将构造器私有化
在类里面直接调用对象 用static
写一个public static的函数来返回对象
*/
package EXE.Static;

public class Main {
    public static void main(String[] args) {
        A c=A.getA();
    }
}

class A {
    int n;
    private static A a = new A(55);

    private A(int n) {
        this.n = n;
    }

    public static A getA() {
        return a;
    }

}
```

为什么要用这个，

1.为了让一个类只能产生一个对象。

比如如果有一个类，他是个重要的核心，占用资源很高，所以不能执行多次，只能执行一次

**是在类加载的时候 就创建了对象的实例**

**不存在线程安全问题**

**可能会浪费对象，因为已经创建了**

#### 懒汉式

```java
package EXE.Static;

public class Main {
    public static void main(String[] args) {
    }

    static class A {
        public static int n;
        private static A a;//默认是null

        private  A(int n) {
            this.n = n;
        }

        public static A getA() {
            if (a == null) {
                a= new A(5);
            }
            return a;
        }

    }
}
```

**是在类加载的时候就创建了对象的实例**

存在线程安全问题



## I/O流

![image-20241010214035229](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241010214035229.png)

![image-20241010220227289](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241010220227289.png)

### 序列化

你看啊 这个 数据可以存储在文件里 那么 对象可不可以呢？？

就是序列化 需要对象 继承seriable

### 流

字符流 字节流 输入流 输出流  节点流

### 读入 文件转换乱码问题

![image-20241012181645248](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241012181645248.png)

# 网络

## socket

就像一个口 所有的链接的数据传送 都是通过这个口来实现的

口来负责两个服务器的链接 

如何确定是哪两台服务器——》通过 ip以及端口

注意流要关闭 不然会造成资源浪费

```java
package Int.PJ1;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.security.Provider;

public class Ser {
    public static void main(String[] args) throws IOException {
        //建立端口的
        ServerSocket serverSocket = new ServerSocket(8888);
        //如果又socket的请求 便会创建一个socket 来接受 返回一个socket
        Socket socket = serverSocket.accept();
        System.out.println("服务器" + socket.getClass());
        InputStream inputStream = socket.getInputStream();
        byte[] buffer = new byte[1024];
        int len = 0;
        while((len=inputStream.read())!=-1){
            System.out.print((char)len);
        }
        serverSocket.close();
        inputStream.close();
        socket.close();
    }
}
fu'wu
```

```java
package Int.PJ1;

import java.io.OutputStream;
import java.net.InetAddress;
import java.net.Socket;

public class Cil {
    public static void main(String[] args) throws Exception {
        //通过ip 以及端口来获取是哪一个
        Socket socket = new Socket(InetAddress.getLocalHost(), 8888);
        System.out.println("客户端链接："+socket.getClass());
        //用OutputStream流 来连接上socket的流
        OutputStream out = socket.getOutputStream();
        out.write("999".getBytes());
        out.close();
        socket.close();
    }
}
用户端
```

```java
BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(outputStream));
解读 为了使用字符流输入 但是BufferedWriter只能接受Write类 于是就需要转换来把outputStream通过new OutputStreamWriter()来转换成
```

```java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
同理
```

小秘密：当客户端与服务端链接的时候 虽然看似是客户端找到了服务端的端口 客户端似乎是没有端口 其实客户端也是会产生临时端口    

# 反射

反射机示意图

![image-20241020185741522](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241020185741522.png)

吊的地方就是可用通过配置文件来在不修改源码的情况下 使用不同的东西

![image-20241021233817199](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241021233817199.png)

![image-20241021234048064](C:\Users\33882\AppData\Roaming\Typora\typora-user-images\image-20241021234048064.png)
