---
layout: post
title: 游戏后端开发中常用的不停机修复方案
categories: [java]
description: 游戏后端开发中常用的不停机修复方案
keywords: java, 热修复, 游戏
---

## 缘由

```
游戏服务器开发中，上线之后难免会遇到bug，如果每次都停服更新，毕竟会造成一定的损失。此时，热修复就显得颇为需要。Java从Java5开始就具有了原生修改class的能力，在Java6中进行了增强。
推荐阅读：
https://www.ibm.com/developerworks/cn/java/j-lo-jse61/index.html
同时 Java也可以在Groovy的支持下进行Groovy脚本执行的能力，从而可以执行groovy的代码从而也能扩展热修复的能力。
```

## 实践
### 环境准备

```
引入了 groovy 与 test 支持
https://github.com/viakiba/viakiba/tree/master/javaHotFix
查看lib下的jar文件 引入到环境变量中
```

### 执行Groovy脚本

Groovy执行的脚本有两种方式，一种是文件一种是方法体字符串。

```txt

    @Test
    public void testGroovyFile() throws IOException {
        GroovyShell groovyShell = new GroovyShell();
        Object evaluate = groovyShell.evaluate(new File("C:\\Users\\89264\\Desktop\\hotload\\src\\com\\company\\GroovyDemo.java"));
    }
    @Test
    public void testGroovyScript() throws IOException {
        GroovyShell groovyShell = new GroovyShell();
        /*
        public String main(String[] args){
            System.out.println("=================");
            return "<><><><>";
        }
         */
        Script script = groovyShell.parse("String test(String[] args){\n" +
                "        System.out.println(\"=================\");\n" +
                "        return \"<><><><>\";\n" +
                "    }" );
        String[] strings = new String[1];
        strings[0] = "dsa";
        Object main = script.invokeMethod("test","");
        System.out.println(main);
    }

```

GroovyDemo文件见 https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/bean/GroovyDemo.java

这么操作可以进行线上的数据修复 比如配置等

### 反射

游戏中有很多单例的实现，如果这些实现有问题的问题话，可以借助groovy脚本使用反射替换这个实例。

```txt
https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/ReflectionTest.java
```

### Instrumentation 
借助 Instrumentation 实现class重新载入 即热更新

#### 局限性
```txt
	***********热更规则：copy自jdk-1.6-api*******************************************************
	1.如果重定义的方法有活动的堆栈帧，那么这些活动的帧将继续运行
		原方法的字节码。将在新的调用上使用此重定义的方法。 
	2.此方法不会引起任何初始化操作，JVM 惯例语义下发生的初始化除
		外。换句话说，重定义一个类不会引起其初始化方法的运行。静
		态变量的值将与调用之前的值一样。 
	3.重定义类的实例不受影响。 
	4.重定义可能会更改方法体、常量池和属性。重定义不得添加、移除、
		重命名字段或方法；不得更改方法签名、继承关系。
	5.如果此方法抛出异常，则不会重定义任何类。 
	******************************************************************************************
```

#### 实现

```txt

1. 实现 InstrumentationHolder
    https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/hotreload/InstrumentationHolder.java
    解释：
        这个 holder 会在 MANIFEST.MF 描述文件中声明，会在执行jar文件声明的 main 之前执行，执行这个方法的premain方法。此后此文件中的 Instrumentation 会在重新更换 class 的时候 在 HotReloadService 中使用。

2. 实现 编译操作
    https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/hotreload/HotReloadTask.java
    解释：
        从代码字符串编译到class文件所进行的实现。 见 https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/HotLoadTest.java 文件 第34行代码进行的实现。此时的字符串更新内容是写死在文件里的，其实也可以通过接口传进来 比如socket / rest 接口 等
        注意 此文件中的的 GS_DEPLOY_CONTEXT_CLASSPATH_DIR 声明的路径 这个就是 jar的执行路径 注意修改到自己上面。

3. 触发更新
    https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/hotreload/HotReloadService.java
    解释：
        实现 1中的重新定义class操作。

4. 执行类
    https://github.com/viakiba/viakiba/blob/master/javaHotFix/src/com/company/hotreload/HotReloadTask.java
    解释：
        把上面三个操作耦合在一起实现测试例子。

```

#### 打包
重点就是描述文件不一样！！！
我使用的IDEA打包，不重复写了，可以参考这个：
```html
https://www.cnblogs.com/blog5277/p/5920560.html

注意 MANIFEST.MF 文件这个描述文件在打包的时候。需要打两次，两次的区别就是这个 MANIFEST.MF 描述文件不一样。

javaAgent jar 描述文件内容如下:

    Manifest-Version: 1.0
    Premain-Class: com.company.hotreload.InstrumentationHolder
    Can-Redefine-Classes: true
构建出的jar假设命名为 hotloadAgent.jar

可执行jar 描述文件内容如下:

    Manifest-Version: 1.0
    Can-Redefine-Classes: true
    Main-Class: com.company.HotLoadTest

构建出的jar假设命名为 hotload.jar

文件可以在如下地址找到 可进行对比：
    https://github.com/viakiba/viakiba/tree/master/javaHotFix/file

这里面的俩 jar 文件可以直接用下面的指令验证。
```

#### 效果

```shell
java -javaagent:hotloadAgent.jar -jar hotload.jar com.company.HotLoadTest
```
输出内容：
![hotfix](/images/post/202003/1.png)

### 结束

这估计是 最简单mingliao的热更新实现demo了。
```txt
还有另外一种实践 不过我个人认为没有上面的实践具有工程意义。
https://github.com/youxijishu/game-hot-update
```
