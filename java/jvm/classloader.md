# 类加载器

## 什么是类加载器？
类加载器（class loader）用来加载 Java 类到 Java 虚拟机中。一般来说，Java 虚拟机使用 Java 类的方式如下：Java 源程序（.java 文件）在经过 Java 编译器编译之后就被转换成 Java 字节代码（.class 文件）。

类加载器负责读取 Java 字节代码，并转换成 java.lang.Class类的一个实例。每个这样的实例用来表示一个 Java 类。通过此实例的 newInstance()方法就可以创建出该类的一个对象。实际的情况可能更加复杂，比如 Java 字节代码可能是通过工具动态生成的，也可能是通过网络下载的。


## 请描述下类加载机制，然后说明说明是反射，以及反射的常见调用方式？
1. 类加载机制：java源文件经过编译后产生一个字节码文件。java虚拟机把描述类的数据加载到内存，对数据进行处理后变成一个对象实例，而这个对象为Class类实例；
2. 反射机制：运行时加载，使用编译器位置的类获取其中完整够着并生成对象的实体或对其调用；
3. 调用方式：Class.forName()静态方法，可以利用类名在classpath中查找类并且装载到内存返回这个class。加载类的过程采用懒惰方式（即检查发现如果已加载了就不再加载，直接返回已经加载的类，相当于手工检查内存中是否已加载了某个类。）newinstance方法会利用默认构造器创建类实例。

## Java 虚拟机是如何判定两个 Java 类是相同的？
Java 虚拟机不仅要看类的全名是否相同，还要看加载此类的类加载器是否一样。只有两者都相同的情况，才认为两个类是相同的。即便是同样的字节代码，被不同的类加载器加载之后所得到的类，也是不同的。比如一个 Java 类 com.example.Sample，编译之后生成了字节代码文件 Sample.class。两个不同的类加载器 ClassLoaderA和 ClassLoaderB分别读取了这个 Sample.class文件，并定义出两个 java.lang.Class类的实例来表示这个类。这两个实例是不相同的。对于 Java 虚拟机来说，它们是不同的类。试图对这两个类的对象进行相互赋值，会抛出运行时异常 ClassCastException。

## 类加载机制题，请问以下代码的打印结果？
```java
class Grandpa
{
    static
    {
        System.out.println("爷爷在静态代码块");
    }
}    
class Father extends Grandpa
{
    static
    {
        System.out.println("爸爸在静态代码块");
    }
    public static int factor = 25;
    public Father()
    {
        System.out.println("我是爸爸~");
    }
}
class Son extends Father
{
    static 
    {
        System.out.println("儿子在静态代码块");
    }
    public Son()
    {
        System.out.println("我是儿子~");
    }
}
public class InitializationDemo
{
    public static void main(String[] args)
    {
        System.out.println("爸爸的岁数:" + Son.factor);    //入口
    }
}

```
结果答案：  
>爷爷在静态代码块  
>爸爸在静态代码块  
>爸爸的岁数:25

### 类加载器七个阶段
1. 加载：把代码数据加载到内存
	- 通过一个类的全限定名来获取其定义的二进制字节流。
	- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
	- 在Java堆中生成一个代表这个类的 java.lang.Class 对象，作为对方法区中这些数据的访问入口。
2. 验证：当jvm加载完class字节码文件并在方法去创建对应的class对象之后，jvm便会开始启动字节码校验
	1. jvm规范校验：jvm会对字节流进行文件格式校验；元数据验证;符号引用验证;
	2. 代码逻辑校验(字节码验证)：jvm会对代码组成的数据流和控制流进行校验，确保jvm运行该字节码文件不会出现致命错误，比如方法的返回参数不一致等；
3. 准备（重要）：当完成字节码文件校验之后，jvm便会开始为类变量分配内存并初始化。
	1. 内存分配对象：java中的变量有[类变量]和[类成员变量]两种类型，[类变量]指的是static修饰的变量，而其他所有类型的变量属于[类成员变量]，在准备阶段JVM只会为[类变量]分配内存，而不会为[类成员变量]分配内存.[类成员变量]的内存分配需要等到初始化阶段才开始。
	2. 初始化类型：在准备阶段，JVM会为类变量分配内存，并为其初始化。但是这里的初始化指的是为变量赋予java语言中该数据类型的零值，而不是用户代码里的初始化的值。
	
		例如以下代码在准备阶段诸侯，age的值将是0，而不是12.
	```java
		public static int age = 12;
	```
	
		但是如果一个变量是常量（static final修饰）的话，那么在准备阶段，属性便会被赋期望的值。
4. 解析：当通过准备阶段之后，JVM针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类进行解析。将在常量池的符号引用替换为直接引用。
5. 初始化（重要）：到了初始化阶段，用户定义的java代码才真正开始执行，在这个阶段，JVM会根据语句执行顺序对类对象进行初始化，一般来说当JVM遇到下面5种情况才会触发初始化：
	1. 遇到new、getstatic、putstatic、invokestatic这四条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。场景：使用new关键字初始化、读取或者设置一个类的静态字段（被final修饰已在编译器把结果放到常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
	2. 使用java.lang.reflect包的方法对其类进行反射调用的时候，如果没有类初始化，则需要先触发初始化。
	3. 当初始化一个类的时候，如果发现其父类还没有执行初始化，则需要先触发其父类的初始化。
	4. 当使用jdk1.7动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果Ref_getstatic、ref_putstatic,ref_invokestatic的句柄的，并且这个方法句柄所对应的的类还没有进行初始化，则先触发初始化。
	
	> JVM初始化步骤:  
	>	1.假如这个类还没有被加载和连接，则程序先加载并连接该类  
	>	2.假如该类的直接父类还没有被初始化，则先初始化其直接父类  
	>	3.假如类中有初始化语句，则系统依次执行这些初始化语句
6. 使用：当JVM完成初始化之后，JVM便开始从入口方法开始执行程序代码。
7. 卸载：当用户程序代码执行完毕之后，JVM便开始销毁创建class对象，最后负责运行的jvm也退出内存。

### 分析
首先程序到 main 方法这里，使用标准化输出 Son 类中的 factor 类成员变量，但是 Son 类中并没有定义这个类成员变量。于是往父类去找，我们在 Father 类中找到了对应的类成员变量，于是触发了 Father 的初始化。

但根据我们上面说到的初始化的 5 种情况中的第 3 种，我们需要先初始化 Father 类的父类，也就是先初始化 Grandpa 类再初始化 Father 类。于是我们先初始化 Grandpa 类输出：「爷爷在静态代码块」，再初始化 Father 类输出：「爸爸在静态代码块」。

最后，所有父类都初始化完成之后，Son 类才能调用父类的静态变量，从而输出：「爸爸的岁数：25」。

### 例子1
```java
class Grandpa
{
    static
    {
        System.out.println("爷爷在静态代码块");
    }
}    
class Father extends Grandpa
{
    static
    {
        System.out.println("爸爸在静态代码块");
    }
    public static int factor = 25;
    public Father()
    {
        System.out.println("我是爸爸~");
    }
}
class Son extends Father
{
    static 
    {
        System.out.println("儿子在静态代码块");
    }
    public Son()
    {
        System.out.println("我是儿子~");
    }
}
public class InitializationDemo
{
    public static void main(String[] args)
    {
        System.out.println("爸爸的岁数:" + Son.factor);    //入口
    }
}
```

### 分析1
- 首先程序到 main 方法这里，使用标准化输出 Son 类中的 factor 类成员变量，但是 Son 类中并没有定义这个类成员变量。于是往父类去找，我们在 Father 类中找到了对应的类成员变量，于是触发了 Father 的初始化。
- 但根据我们上面说到的初始化的 5 种情况中的第 3 种，我们需要先初始化 Father 类的父类，也就是先初始化 Grandpa 类再初始化 Father 类。于是我们先初始化 Grandpa 类输出：「爷爷在静态代码块」，再初始化 Father 类输出：「爸爸在静态代码块」。
- 最后，所有父类都初始化完成之后，Son 类才能调用父类的静态变量，从而输出：「爸爸的岁数：25」。


### 例子2
```java
class Grandpa
{
    static
    {
        System.out.println("爷爷在静态代码块");
    }
    public Grandpa() {
        System.out.println("我是爷爷~");
    }
}
class Father extends Grandpa
{
    static
    {
        System.out.println("爸爸在静态代码块");
    }
    public Father()
    {
        System.out.println("我是爸爸~");
    }
}
class Son extends Father
{
    static 
    {
        System.out.println("儿子在静态代码块");
    }
    public Son()
    {
        System.out.println("我是儿子~");
    }
}
public class InitializationDemo
{
    public static void main(String[] args)
    {
        new Son();     //入口
    }
}
```

### 分析2
- 首先在入口这里我们实例化一个 Son 对象，因此会触发 Son 类的初始化，而 Son 类的初始化又会带动 Father 、Grandpa 类的初始化，从而执行对应类中的静态代码块。因此会输出：「爷爷在静态代码块」、「爸爸在静态代码块」、「儿子在静态代码块」。
-  当 Son 类完成初始化之后，便会调用 Son 类的构造方法，而 Son 类构造方法的调用同样会带动 Father、Grandpa 类构造方法的调用，最后会输出：「我是爷爷~」、「我是爸爸~」、「我是儿子~」。

### 例子3
```java
public class Book {
    public static void main(String[] args)
    {
        staticFunction();
    }
    static Book book = new Book();
    static
    {
        System.out.println("书的静态代码块");
    }
    {
        System.out.println("书的普通代码块");
    }
    Book()
    {
        System.out.println("书的构造方法");
        System.out.println("price=" + price +",amount=" + amount);
    }
    public static void staticFunction(){
        System.out.println("书的静态方法");
    }
    int price = 110;
    static int amount = 112;
}
```

### 分析3
- 在上面两个例子中，因为 main 方法所在类并没有多余的代码，我们都直接忽略了 main 方法所在类的初始化。但在这个例子中，main 方法所在类有许多代码，我们就并不能直接忽略了。
- 当 JVM 在准备阶段的时候，便会为类变量分配内存和进行初始化。此时，我们的 book 实例变量被初始化为 null，amount 变量被初始化为 0。
- 当进入初始化阶段后，因为 Book 方法是程序的入口，根据我们上面说到的类初始化的五种情况的第四种：当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。JVM 会对 Book 类进行初始化。
- JVM 对 Book 类进行初始化首先是执行类构造器（按顺序收集类中所有静态代码块和类变量赋值语句就组成了类构造器），后执行对象的构造器（先收集成员变量赋值，后收集普通代码块，最后收集对象构造器，最终组成对象构造器）。


### 总结方法论
- 确定类变量的初始值。在类加载的准备阶段，JVM 会为类变量初始化零值，这时候类变量会有一个初始的零值。如果是被 final 修饰的类变量，则直接会被初始成用户想要的值。
初始化入口方法。当进入类加载的初始化阶段后，JVM 会寻找整个 main 方法入口，从而初始化 main 方法所在的整个类。当需要对一个类进行初始化时，会首先初始化类构造器，之后初始化对象构造器。
- 初始化类构造器。初始化类构造器是初始化类的第一步，其会按顺序收集类变量的赋值语句、静态代码块，最终组成类构造器由 JVM 执行。
- 初始化对象构造器。初始化对象构造器是在类构造器执行完成之后的第二部操作，其会按照执行类成员变成赋值、普通代码块、对象构造方法的顺序收集代码，最终组成对象构造器，最终由 JVM 执行。

## 双亲委派模型（Parent Delegation Model）？
类的加载过程采用双亲委派机制，这种机制能更好的保证 Java 平台的安全性.
类加载器 ClassLoader 是具有层次结构的，也就是父子关系，其中，Bootstrap 是所有类加载器的父亲，如下图所示：
<img src="https://raw.githubusercontent.com/ppj19891020/pictures/master/baise/9.png"/>
该模型要求除了顶层的 Bootstrap class loader 启动类加载器外，其余的类加载器都应当有自己的父类加载器。子类加载器和父类加载器不是以继承（Inheritance）的关系来实现，而是通过组合（Composition）关系来复用父加载器的代码。每个类加载器都有自己的命名空间（由该加载器及所有父类加载器所加载的类组成，在同一个命名空间中，不会出现类的完整名字（包括类的包名）相同的两个类；在不同的命名空间中，有可能会出现类的完整名字（包括类的包名）相同的两个类）



### 双亲委派模型的工作过程？
- 当前 ClassLoader 首先从自己已经加载的类中查询是否此类已经加载，如果已经加载则直接返回原来已经加载的类。

	> 每个类加载器都有自己的加载缓存，当一个类被加载了以后就会放入缓存， 等下次加载的时候就可以直接返回了。
- 当前 ClassLoader 的缓存中没有找到被加载的类的时候，委托父类加载器去加载，父类加载器采用同样的策略，首先查看自己的缓存，然后委托父类的父类去加载，一直到 bootstrap ClassLoader.
	> 当所有的父类加载器都没有加载的时候，再由当前的类加载器加载，并将其放入它自己的缓存中，以便下次有加载请求的时候直接返回。

### 为什么这样设计呢？
主要是为了安全性，避免用户自己编写的类动态替换 Java 的一些核心类，比如 String，同时也避免了重复加载，因为 JVM 中区分不同类，不仅仅是根据类名，相同的 class 文件被不同的 ClassLoader 加载就是不同的两个类，如果相互转型的话会抛java.lang.ClassCaseException.