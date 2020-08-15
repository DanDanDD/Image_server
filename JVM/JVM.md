## 从栈帧看字节码如何在JVM中流转

怎么查看字节码文件？

对象初始化后，具体字节码怎么执行？

#### 分析字节码的小工具：

①  javap    将 .class 字节码文件解析成可读的文件格式

javap -v 

javap -p 打印私有字段和方法

```java
javap -p -v HelloWorld
```

在 **Stack Overflow** 上有一个非常有意思的问题：我在某个类中增加一行注释之后，为什么两次生成的 **.class** 文件，它们的 **MD5** 是不一样的？

这是因为在 **javac** 中可以指定一些额外的内容输出到字节码。经常用的有

* javac -g:lines 强制生成 LineNumberTable。
* javac -g:vars  强制生成 LocalVariableTable。
* javac -g 生成所有的 debug 信息

②  **jclasslib** 图形化工具，直观的查询字节码中的内容

​         Idea 中有插件，可以从 plugins 中搜索到



##### 类的初始化发生在类的加载阶段

##### 对象的创建方式有什么？

* new

* Class 的 newInstance 方法

* Constructor 类的 newInstance 方法

* 反序列化                                      （未调用构造方法）

* Object 的 clone 方法                  （未调用构造方法）

  

![image-20200815005040962](C:\Users\氮蛋\AppData\Roaming\Typora\typora-user-images\image-20200815005040962.png)

A 和 B 会被加载到元空间的方法区，进入 main 方法后，将会交给执行引擎执行。

这个执行过程是在栈上完成的，其中有几个重要的区域，包括虚拟机栈、程序计数器等



#### 查看字节码

##### 命令行查看

↓ 编译源代码 A.java

```java
javac -g:lines -g:vars A.java
```

这将强制生成 LineNumberTable 和 LocalVariableTable

后使用 javap 命令查看 A 和 B 的字节码

```java
javap -p -v A.class
javap -p -v B.class
```

这个命令不仅会输出行号、本地变量表信息、反编译汇编代码，还会输出当前类用到的常量池等信息

```java
javap 中显示的字样
1: invokespecial #1                  // Method java/lang/Object."<init>":()V
```

对象的初始化，首先调用了 Object 类的初始化方法，注意 <init> 

##### clinit  和 init

<clinit>方法在类加载过程中执行的，而<init>在对象实例化时执行,所以<clinit>一定比<init>先执行

<init>:会将变量初始化,初始化语句块，构造器方法等操作顺序封装到该方法中

- 父类的静态变量初始化
- 父类的静态代码块
- 子类的静态变量初始化
- 子类的静态代码块
- 父类的变量初始化
- 父类的初始化代码块
- 父类的构造方法
- 子类的变量初始化
- 子类的初始化代码块
- 子类的构造方法