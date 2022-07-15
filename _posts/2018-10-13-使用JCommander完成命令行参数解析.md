---
layout: post
title: 使用JCommander完成命令行参数解析
categories: [JCommander, 命令行]
description: 使用JCommander完成命令行参数解析
keywords: JCommander, 命令行
tags: JCommander, 命令行
---
## 概述
JCommander是一个小巧的解析命令行参数的Java框架。
### 参考

```
http://hao.jobbole.com/jcommander/
http://jcommander.org/#_overview
https://github.com/cbeust/jcommander
```
![jcommander](/images/post/201810/jcommander1.png)
### 代码样例
```
https://github.com/viakiba/springboot/tree/master/springboot_jcommander
```
其实简单的java程序就可以，但是我想用SpringBoot来搞，都一样在这里。
### 引入
```
<dependency>
			<groupId>com.beust</groupId>
			<artifactId>jcommander</artifactId>
			<version>LATEST</version>
</dependency>
```

## 注解解释
这是一个以注解为基准的解析参数的Java框架。
```
@Parameters(separators = "= ")
// Parameters 用在类名上，separators的值用于分割key和value可以自定义。
public class CommandBean {
    //放在属性上 names 指定key的名字可以多个 required 标识此参数必须传入 description 此命令的表述  hiden如果为true 则不会显示在usage提示中。
    @Parameter(names = {"-p", "-port", "--port"}, required = true,
            description = "run mode. The agent/monitor modes are available.",hiden = true)
    public int port;

    @Parameter(names = {"-c", "-contexts", "--contexts"}, required = true,
            description = "run mode. The agent/monitor modes are available.")
    public String contexts;

    //略
}
```

```
// java -jar xxx -p=8005 -c=contextvalue
public static void main(String[] args) {
		//处理命令
		CommandBean commandBean = new CommandBean();
		JCommander jCommander = new JCommander(commandBean);
		try{
			jCommander.setProgramName("JcommanderApplication");
			jCommander.setAcceptUnknownOptions(true);
			jCommander.parse(args);
			System.out.println(commandBean.port);
		}catch (Exception e){
      //命令不对 或者不全 required为true的没有输入 usage 方法会输出命令提示，name / description 的内容会格式化输出。
			jCommander.usage();
			e.printStackTrace();
		}
    //此时 commandBean 这个变量的属性都会被传入的参数赋值。
    System.out.println(commandBean.toString())
}
```

## 总结
这个组件在需要命令行启动时处理处理传入参数能够很好的提高开发效率。
