# Scala 柯里化 用法 原理 详解 应用场景 及干什么用的 {#articleContentId}

  

# Scala 柯里化

### 概念

**柯里化(Currying)：** 指的是将原来接受**两个参数**的函数变成新的接受**一个参数**的函数的过程。新的函数返回一个**以原有第二个参数为参数**的函数。

### 用法

**非柯里化：**

```java
//非柯里化函数定义
def add(x:Int,y:Int)=x+y
```

那么我们应用的时候，应该是这样用：**add(1,2)**

**柯里化**

```java
//柯里化使用多个参数列表
def add(x:Int)(y:Int) = x + y
```

那么我们应用的时候，应该是这样用：**add(1)(2)**,最后结果都一样是3，这种方式（过程）就叫柯里化。

**应用实例：**

```java
object cury_func {
  def plainOldSum(x:Int,y:Int)= x + y  //非柯里化函数定义
  def curriedSum(x:Int)(y:Int)= x + y  //柯里化使用多个参数列表

  def main(args:Array[String]): Unit ={
    println(plainOldSum(1,5))
    println(curriedSum(1)(5))
   
    //通过柯里函数curriedSum定义变量
    val onePlus=curriedSum(1)_ //_ 第二个参数列表的占位符，
    println(onePlus(2)) //传入的是第二个参数
  }
}
```

### 原理

add(1)(2) 实际上是**依次调用两个普通函数**（非柯里化函数），第一次调用使用一个参数 x，返回一个**函数类型**的值，第二次使用参数y调用**这个函数类型**的值。

**演变过程：**

```java
def add(x:Int,y:Int)=x+y
def add(x:Int)=(y:Int)=>x+y
def add(x:Int)(y:Int) =x+y
```

**实质上：**
最先**演变**成这样一个方法：

```java
def add(x:Int)=(y:Int)=>x+y
```

那么这个函数是什么意思呢？ **接收一个x为参数，返回一个匿名函数**，该匿名函数的定义是：接收一个Int型参数y，函数体为x+y。现在我们来对这个方法进行调用。

```java
val result = add(1)
```

![在这里插入图片描述](https://raw.githubusercontent.com/ArkNg/NoteBook/master/2021/02/uPic/2019111418143086-20210213152400712.png)
返回一个result，那result的**值**应该是一个**匿名函数**：**(y:Int)=>1+y**
所以为了得到结果，我们继续调用result。

```java
val sum = result(2)
```

最后打印出来的结果就是3。

### 应用场景（及干什么用的）

使用柯里化特性可以**将复杂逻辑简单化**
能将很多常漏掉的主函数业务逻辑之外的处理暴露在函数的定义阶段
提高代码的健壮性，使函数功能更加细腻化和流程化

在定义主函数功能的时候常常出现这种情况，函数体中的代码越写越长，本来可以拆分的功能，由于写代码的时候逻辑思绪根本停不下来，不愿意中途打断去定义那些繁琐的辅助函数，往往一个函数就一路走到头，最后使主函数代码段很长，不便理解，功能复杂，需要后期慢慢拆分功能。小象在学习使用scala函数式柯里化风格后，在思绪停不下来又需要拆分功能的时候，就将定义好需要拆分的辅助功能函数的声明按照柯里化风格定义在函数声明中，继续完成主函数功能，等主函数功能完成后，在慢慢按照顺序去实现这些辅助功能。为了减少或隐藏辅助功能的传入简化接口中不必要的辅助方法参数传入，可以创建主函数的重载。使用scala柯里化风格可以简化主函数的复杂度，提高主函数的自闭性，提高功能上的可扩张性(事实证明：流水化生产式最高效和安全的，代码编写也一样，一个函数实现维护一个功能的完成处理(逻辑处理，相关异常处理等)，也是极简设计，优秀代码体检的追求)。

例如 设计一个获取本地文本文件的所有行数据的功能，主函数功能主要是创建文件流读取文件的所有行，在读取过程中，需要做很多的辅助操作如判断本地文件是否存在和可读和关闭文件流。使用scala的函数式柯里化代码看上去将变得非常优雅

```scala

def getLines(filename:String):List[String]={
    getLines(filename)(isReadable)(closeStream)
}
def getLines(filename: String)(isFileReadable: (File) => Boolean)(closableStream: (Closeable) => Unit):List[String] = {
    val file = new File(filename)
    if (isFileReadable(file)) {
      val readerStream = new FileReader(file)
      val buffer = new BufferedReader(readerStream)
      try {
        var list: List[String] = List()
        var str = ""
        var isReadOver = false
        while (!isReadOver) {
          str = buffer.readLine()
          if (str == null) isReadOver = true
          else list = str :: list
        }
        list.reverse
      } finally {
        closableStream(buffer)
        closableStream(readerStream)
      }
    } else {
      List()
    }
  }
 
  def isReadable(file: File) = {
    if (null != file && file.exists() && file.canRead()) true
    else false
  }
  def closeStream(stream: Closeable) {
    if (null != stream) {
      try {
        stream.close
      } catch {
        case ex => Log.error(“[”+this.getClass.getName+”.closeStream]”,ex.getMessage)
      }
    }
  }
```

转载自：https://blog.csdn.net/yangguo_2011/article/details/30730185?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.control