一、环境准备

参考：https://blog.csdn.net/u012373815/article/details/53266301

二、配置选择

三、注意事项

四、问题解决

```
运行错误一：
    运行wordcount简单程序提示：
    Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/hadoop/fs/FSData...
原因：
    初步怀疑是maven配置中的hadoop-common包未加载完成，清理红色错误依赖后运行正常
```

```
运行错误二：

    scala.Predef$.refArrayOps([Ljava/lang/Object;)Lscala/collection/mutable/ArrayOps
原因：
    系统默认scala版本与本机自带版本冲突，spark默认依赖2.11
解决方法：
    将scala-sdk从2.12换为2.11

    File -> Project Structure -> Global libraries -> Remove SDK -> Rebuild.
```



