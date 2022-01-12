### 1、安装jdk

下载安装包： https://www.oracle.com/java/technologies/downloads/#java8-windows

![image-20220112164203793](D:\jeffchan\oddworld-learn-note\java基础\java基础课件.assets\image-20220112164203793.png)

下载x64的，然后一直安装即可。



### 2、java语言概述

#### 2.1、人机交互方式

- 命令行（CLI）:  比较灵活，需要相关控制台，需要记住一些指令，较麻烦。

  基本的doc命令：

  ```yaml
  window:
    dir:  列出对应目录的所有文件
    cd: 进入某个目录
    cls: 清理控制台窗口
    
  linux：
    ls: 列出对应目录的所有文件
    cd: 进入某个目录
    clear: 清理控制台窗口
  ```

  

- 图形界面（GUI）：直观，简单容易操作



#### 2.2、2021年12月份语言的排行

![image-20220111162405502](D:\jeffchan\oddworld-learn-note\java基础\java基础课件.assets\image-20220111162405502.png)



#### 2.3、java的两大核心机制

- java虚拟机（JVM）

  - java程序的运行要借助jvm
  - 不同的平台都有对应的jvm，jvm屏蔽不同平台的实现细节，让java程序可以一次编译，到处运行。

- 垃圾回收（GC）: 

  将不再使用的内存回收，就是垃圾回收。

  java会主动回收这些不再使用的内存，不需要程序员负责回收，程序员也无法精确干预。



#### 2.4、java语言的特点

- 面向对象
  - 类和对象
  - 继承，多态，封装
- 平台无关性
  - 不同平台都有对应的jvm，所以java程序可以一次编译，到处运行



#### 2.5、jdk和jre的区别

- jdk是java的开发工具，它包括jre，还有一些编译器（javac），jmap等分析内存和线程的工具，面向的是java的开发人员。
- jre是java的运行环境，主要是对已经编译好的程序进行相关的运行。



#### 2.6、java的API

​	运用程序接口就是java提供的系统类库。比如tools.jar、dt.jar等包里面的类等。

- 这里有系统类库，本地类库和第三方类库
  - 系统类库： java自身提供的类库
  - 本地类库：项目中，自己写的
  - 第三方类库： 引入别人的jar包才导入的类，比如你要使用spring的功能，你引入的spring的jar包就是第三方提供的类库。

#### 2.7、注释

- 单行注释 //
- 多行注释/**   多行内容  **/



#### 2.8、helloworld的编写

- 创建一个Hello.txt的文件
- 然后加入以下内容

```java
public class Hello{

   public static void main(String[] args){
     System.out.println("helloworld!");
  }
}
```

- 修改.txt为java
- javac Hello.java
- java Hello



### 3、安装Idea开发工具



### 4、基本语法