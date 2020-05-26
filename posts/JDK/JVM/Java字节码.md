# Java 字节码

## 说说字节码

class 文件 

jar 文件

## 从 Hello Word 开始

![](https://i.loli.net/2020/05/22/z7hsZIH1QRMTibm.png)

从 hello world 开始

`Hello.java`

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, World");
    }
}
```

class 文件本质上是一个二进制文件，可以使用 xxd 命令查看。

```shell
$ javac Hello.java
$ xxd Hello.class
00000000: cafe babe 0000 0034 0022 0a00 0600 1409  .......4."......
00000010: 0015 0016 0800 170a 0018 0019 0700 1a07  ................
00000020: 001b 0100 063c 696e 6974 3e01 0003 2829  .....<init>...()
00000030: 5601 0004 436f 6465 0100 0f4c 696e 654e  V...Code...LineN
00000040: 756d 6265 7254 6162 6c65 0100 124c 6f63  umberTable...Loc
00000050: 616c 5661 7269 6162 6c65 5461 626c 6501  alVariableTable.
00000060: 0004 7468 6973 0100 074c 4865 6c6c 6f3b  ..this...LHello;
```



## javap

```
$ javap -c Hello.class
Compiled from "Hello.java"
public class Hello {
  public Hello();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: getstatic     #2  // Field java/lang/System.out:Ljava/io/PrintStream;
       3: ldc           #3  // String Hello, World
       5: invokevirtual #4  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
       8: return
}
```

## Class 文件结构

Class 文件是类似于结构体的一种数据结构：

```
ClassFile {
    u4             magic;
    u2             minor_version;
    u2             major_version;
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
    u2             access_flags;
    u2             this_class;
    u2             super_class;
    u2             interfaces_count;
    u2             interfaces[interfaces_count];
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

`u4` 和 `u2` 是 Java 虚拟机定义的无符号类型，其他类型会在下文中详细介绍。

包含了以下几个部分：

- 魔数（Magic Number）
- 版本号（Minor&Major Version）
- 常量池（Constant Pool）
- 类访问标记(Access Flags)
- 类索引（This Class）
- 超类索引（Super Class）
- 接口表索引（Interfaces）
- 字段表（Fields）
- 方法表（Methods）
- 属性表（Attributes）

![](https://i.loli.net/2020/05/22/bLKNEJ2yRpYP8tC.png)



### 魔数

四个字节，作为 .class 文件的表示，虚拟机在加载类文件之前会检查这四个字节，如果不符合会拒绝加载。

![](https://i.loli.net/2020/05/22/24cbOA9CZkDijyB.png)

### 版本

在魔数之后的四个字节分别表示**副版本号**（Minor Version）和**主版本号**（Major Version），如下图所示：

![](https://i.loli.net/2020/05/22/Yi8qPkZAKUdrDXz.png)

主版本号说明了类被编译时使用的版本，如果类文件的版本号高于 JVM 自身的版本号，加载该类会被直接抛出`java.lang.UnsupportedClassVersionError`异常。

这里的主版本是 52（0x34），对应的是 Java 8 的版本。

 ### 常量池

常量池是作为 class 文件的大头，稍微会有点复杂。

在 JVM 字节码中，只有较大的整数和字符串常量才会放入常量池中。

**比较小的操作数和常用的数字 0 之类的会被内嵌在字节码中。**

常量池在 class 文件的定义中包含了两部分：

```
{
    u2             constant_pool_count;
    cp_info        constant_pool[constant_pool_count-1];
}
```

1. 常量池大小（cp_info_count），两个字节，表示变长结构的长度。假设值为 n，那么常量的有效索引为 1 ~ n-1，0 作为保留索引表示不指向任何常量池项。
2. 常量池项（cp_info）集合，最多包含 n-1 个项。因为 `Long` 类型和 `Double` 类型的常量会占用两个索引的位置，这会导致实际的常量池个数 会比 n-1 小。

常量池项都由两部分组成：表示类型的 tag 和表示内容的字节数字：

```
cp_info {
    u1 tag;
    u1 info[];
}
```



javap -v 查看类文件的常量池

```
$ javap -v HelloWorld

Constant pool:
   #1 = Methodref          #6.#15         // java/lang/Object."<init>":()V
   #2 = Fieldref           #16.#17        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = String             #18            // Hello, World
  ...
  #27 = Utf8               println
  #28 = Utf8               (Ljava/lang/String;)V
```



### 类访问标记

![](https://i.loli.net/2020/05/23/W37CnoqF4kgA5Oi.png)

由两个字节表示总共可以有 16 个标记位可供使用，目前只使用了其中的 8 个。



| 名称            | 值   | 说明                                   |
| --------------- | ---- | -------------------------------------- |
| ACC\_PUBLIC     | 1    | 标识是否是 public                      |
| ACC\_FINAL      | 10   | 标识是否是 final                       |
| ACC\_SUPER      | 20   | 已经不用了                             |
| ACC\_INTERFACE  | 200  | 标识是类还是接口                       |
| ACC\_ABSTRACT   | 400  | 标识是否是 abstract                    |
| ACC\_SYNTHETIC  | 1000 | 编译器自动生成，不是用户源代码编译生成 |
| ACC\_ANNOTATION | 2000 | 标识是否是注解类                       |
| ACC\_ENUM       | 4000 | 标识是否是枚举类                       |



### 类索引表



### 字段表



### 方法表



### 属性表



## 虚拟机

### 基于栈的虚拟机

虚拟机常见的实现方式有两种：Stack based 的和 Register based。比如基于 Stack 的虚拟机有Hotspot JVM、.net CLR，这种基于 Stack 实现虚拟机是一种广泛的实现方法。而基于 Register 的虚拟机有 Lua 语言虚拟机 LuaVM 和 Google 开发的安卓虚拟机 DalvikVM。



### 栈帧

JVM 经典实现，Hotspot JVM 就是一个基于栈的虚拟机，每个线程都有一个虚拟机栈，存储了「栈帧」。每次方法调用都会涉及到栈帧的创建销毁。

栈帧（Stack Frame）是用于支持虚拟机进行方法调用和方法执行的数据结构，栈帧随着方法调用而创建，随着方法结束而销毁，栈帧的存储空间分配在 Java 虚拟机栈中。

每个栈帧拥有自己的：

- **局部变量表（Local Variables）**
- **操作数栈（Operand Stack）** 
- **指向运行时常量池的引用**

#### 局部变量表

局部变量表的大小在编译期间就已经确定

Java 虚拟机使用局部变量表来完成方法调用时的参数传递，当一个方法被调用时，它的参数会被传递到从 0 开始的连续局部变量列表位置上。当一个实例方法（非静态方法）被调用时，第 0 个局部变量是调用这个实例方法的对象的引用（也就是我们所说的 this ）

![](https://i.loli.net/2020/05/23/mW4B75deJhEuCQK.png)

#### 操作数栈

在方法调用时，操作数栈一般用来准备调用方法的参数和接收方法返回的结果。

![](https://i.loli.net/2020/05/23/DpPGoRw2LAN9UBi.png)

整个 JVM 指令执行的过程就是局部变量表与操作数栈之间不断 load、store 的过程：



## 字节码指令



- 加载和存储指令，比如 iload 将一个整形值从局部变量表加载到操作数栈
- 控制转移指令，比如条件分支 ifeq
- 对象操作，比如创建类实例的指令 new
- 方法调用，比如 invokevirtual 指令用于调用对象的实例方法
- 运算指令和类型转换，比如加法指令 iadd
- 线程同步，monitorenter 和 monitorexit 两条指令来支持 synchronized 关键字的语义
- 异常处理，比如 athrow 显式抛出异常

### 加载存储



### 控制转移



### 对象操作



### 方法调用

- invokestatic：用于调用静态方法
- invokespecial：用于调用私有实例方法，比如构造器，以及使用 super 关键字调用父类的实例方法或构造器，和所实现接口的默认方法
- invokevirtual：用于调用非私有实例方法
- invokeinterface：用于调用接口方法
- invokedynamic：用于调用动态方法



方法绑定：

* 静态绑定：编译时能确定目标方法。
* 动态绑定：需要在运行时根据调用者类型动态识别。





## 参考

1. [栈式虚拟机和寄存器式虚拟机？ - RednaxelaFX的回答 - 知乎 ](https://www.zhihu.com/question/35777031/answer/64575683)
2. [虚拟机随谈（一）：解释器，树遍历解释器，基于栈与基于寄存器，大杂烩](https://www.iteye.com/blog/rednaxelafx-492667)

