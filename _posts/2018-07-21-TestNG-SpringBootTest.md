---
layout: post
title: SpringBoot 中借助 TestNG/SpringBootTest 实现测试
categories: [TestNG, SpringBoot]
description: SpringBoot 中借助 TestNG/SpringBootTest 实现测试
keywords: SpringBoot, TestNG, Idea
---

** 代码地址 **
```
  https://github.com/viakiba/viakiba/tree/master/SpringTest-testNG
```

### 背景

> 最近在项目中需要对接一个第三方系统，对于这个系统的接口进行测试的时候后面的接口请求信息需要结合前面接口的相应结果 （大概四层）。虽然 junit 也可以完成这种需求，但是有时只想执行其中前两三次的测试时，junit 实现就显得比较牵强。所以就引出来接下来的 TestNG.项目本身是基于 SpringBoot 开发的，测试代码也写在了这个里面测试代码需要依赖一些配置信息以及服务类。

### TestNG

> TestNG 是一个测试框架，其灵感来自 JUnit 和 NUnit，但引入了一些新的功能，使其功能更强大，使用更方便。
TestNG 是一个开源自动化测试框架;TestNG表示下一代(Next Generation的首字母)。 TestNG类似于 JUnit (特别是 JUnit 4)，但它不是JUnit框架的扩展。它的灵感来源于 JUnit 。它的目的是优于 JUnit ，尤其是在用于测试集成多类时。 TestNG的创始人是 Cedric Beust (塞德里克·博伊斯特)。
TestNG消除了大部分的旧框架的限制，使开发人员能够编写更加灵活和强大的测试。 因为它在很大程度上借鉴了Java注解( JDK5.0引入的)来定义测试，它也可以显示如何使用这个新功能在真实的 Java 语言生产环境中。

#### 特点

```
注解
TestNG 使用 Java 和面向对象的功能
支持综合类测试(例如，默认情况下，不用创建一个新的测试每个测试方法的类的实例)
独立的编译时测试代码和运行时配置/数据信息
灵活的运行时配置
主要介绍“测试组”。当编译测试，只要要求 TestNG 运行所有的“前端”的测试，或“快”，“慢”，“数据库”等
支持依赖测试方法，并行测试，负载测试，局部故障
灵活的插件 API
支持多线程测试
```
> 来自 https://www.yiibai.com/testng/

#### 实践
TestNG 使用 XML 结合代码实现灵活的测试需求。（开发工具 IDEA ）</br>
##### 测试代码
相对于代码更加关注xml文件的书写与配置。
```
import org.testng.annotations.AfterGroups;
import org.testng.annotations.BeforeClass;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;

/**
 * @author viakiba
 *  testNG 样例
 * @description
 * @date Create in 17:56 2018/7/22
 */
public class SequenceTest {
    int i ;

    @BeforeClass(groups = "before")
    public void beforeTest0(){
        i = 10;
        System.out.println("beforeTest0");
    }

    @Test(priority = 1,groups = {"sequence"})
    public void test1(){
        System.out.println("test1 + sequence ");
    }

    @Test(priority = 3,groups = {"sequence","filter1"})
    public void test2(){
        System.out.println("test3 + sequence + filter1 ");
    }

    @Test(priority = 2,groups = {"sequence","filter2"})
    public void test3(){
        System.out.println("test2+ sequence + filter2");
    }

    @Test(priority = 4,groups = {"sequence","filter1","filter3"})
    public void test4(){
        System.out.println("test4 + sequence + filter1");
    }

    @Parameters({"testParam"})
    @Test(priority = 5,groups = {"sequence","param"})
    public void test5(int testParam){
        System.out.println(testParam);
        System.out.println("test5+sequence+param");
    }

    @Test(priority = 5,groups = {"before"})
    public void test5( ){
        System.out.println(i);
        System.out.println("test5+sequence+param");
    }

    @AfterGroups(groups = {"before"})
    public void afterTest0(){
        i = -1;
        System.out.println("afterTest0" + i);
    }
}
```

##### priority
###### 内容如下
这个测试 xml 配置，会按照 @Test 里的 priority 标识优先运行数字小的测试方法，然后是数字大的。（此时不关注 group 属性）
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="1">
    <test verbose="1" preserve-order="true" name="SequenceTest">
        <!--
            在不指定group时，会执行所有测试方法。
                    verbose  ： 控制台输出的详细内容等级,0-10级（0无，10最详细）
                    preserve-order ：是否按照排序执行 默认为true
                    name ： 必选项，<suite>的名字，将出现在reports里
                    更多详细参数 请参考：
                        https://blog.csdn.net/u011138533/article/details/52174446
         -->
        <classes>
            <!-- 可以多个 -->
            <class name="org.vk.test.testng.SequenceTest" />
        </classes>
    </test>
</suite>
```
###### 触发方式
![priority 触发方式](/images/post/201807/testNG0.png)

##### Group 归组筛选
这个测试 xml 配置，会按照 @Test 里的 priority 标识优先运行数字小的测试方法，然后是数字大的。同时测试的方法 @Test 里的 group 属性应该存在 filter1

###### 样例1
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="3">
    <test verbose="2" preserve-order="true" name="SequenceTest">
        <!--
            group 筛选执行样例
                这里的run标签可以只存在include/exclude
                    对于测试类而言，如果只存在exclude则除了exclude的方法其他类中存在的都会执行。
                    存在include则只执行include包含的。
        -->
        <groups>
            <define name="filter_group">
                <include name="filter1"/>
                <include name="filter2"/>
            </define>
            <define name="filter_group1">
                <include name="filter1"/>
            </define>
            <define name="filter_group2">
                <include name="filter2"/>
            </define>
            <run>
                <include name="filter_group"/>
                <exclude name="filter_group2"/>
                <!-- filter1,filter2 - filter2 = filter1 只运行 filter1 -->
            </run>
        </groups>
        <classes>
            <class name="org.vk.test.testng.SequenceTest" >
            </class>
        </classes>
    </test>
</suite>
```
###### 样例2
include 也可以包括 exclude,反之亦然。
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">

<suite name="Suite" parallel="classes" thread-count="3">
    <parameter name="testParam" value="11"></parameter>
    <test verbose="2" preserve-order="true" name="SequenceTest">
        <groups>
            <define name="filter_group">
                <include name="filter1"/>
                <include name="filter2"/>
            </define>
            <define name="filter_group1">
                <include name="filter1"/>
            </define>
            <define name="filter_group2">
                <include name="filter2"/>
            </define>
            <define name="filter_group3">
                <include name="filter3"/>
            </define>
            <run>
                <include name="filter_group">
                    <exclude name="filter3"/>
                    <!--<exclude name="filter_group3"/>-->
                    <!-- 上下等效 -->
                </include>
                <exclude name="filter_group2"/>
            </run>
        </groups>
        <classes>
            <class name="org.vk.test.testng.SequenceTest" >
            </class>
        </classes>
    </test>
</suite>
```
##### Parameters参数传递

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="3">
    <test verbose="2" preserve-order="true" name="SequenceTest">
        <parameter name="testParam" value="12" />
        <!--这里参数声明对于整个测试suite都有效-->
        <groups>
            <define name="param_group">
                <include name="param"/>
            </define>
            <run>
                <include name="param_group"></include>
            </run>
        </groups>
        <classes>
            <class name="org.vk.test.testng.SequenceTest" >
                <!--此处声明的参数的值优先级更高 仅对这个类有效-->
                <!--<parameter name="testParam" value="11"/>-->
            </class>
        </classes>
    </test>
</suite>
```

##### before/after 方法
有 before/after 相关开头的TestNG注解可以在 @Test 方法执行前后 进行执行前准备和执行后首位，这里以 BeforeClass/AfterGroups 为例。
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="3">
    <test verbose="2" preserve-order="true" name="SequenceTest">
        <!--这里参数声明对于整个测试suite都有效-->
        <groups>
            <define name="before">
                <include name="before"/>
            </define>
            <run>
                <include name="before"></include>
            </run>
        </groups>
        <classes>
            <class name="org.vk.test.testng.SequenceTest" >
            </class>
        </classes>
    </test>
</suite>
```

#### SpringBoot 引入 TestNG
基本可以使用 TestNG 所有配置。注意继承 AbstractTestNGSpringContextTests ，并使用 SpringBootTest 注解。
##### 代码准备
![SpringBoot 待测准备](/images/post/201807/SpringBootTest.png)
##### 样例1
指定引入需要注入的类
###### 代码如下
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.testng.AbstractTestNGSpringContextTests;
import org.testng.annotations.Test;
import org.vk.demo.EatCompentConfig;
import org.vk.demo.PeopleEatService;
import org.vk.demo.PeopleEatServiceImpl;

/**
 * @author viakiba
 *  classes需要注入的类
 *      如果依赖比较简单较少，可以指定classes的具体类
 * @description
 * @date Create in 20:38 2018/7/22
 */
@SpringBootTest(classes = { PeopleEatServiceImpl.class , EatCompentConfig.class })
//@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
public class EatServiceTest extends AbstractTestNGSpringContextTests {
    @Autowired
    private PeopleEatService peopleEatServiceImpl;
    @Test(groups = "eatOne")
    public void test1(){
        int i = peopleEatServiceImpl.eatService();
        System.out.println(i * 1);
    }
    @Test(groups = "eatTen")
    public void test2(){
        int i = peopleEatServiceImpl.eatService();
        System.out.println(i*10);
    }
}
```

###### XML配置
这个只会执行 test2() 方法
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="1">
    <test verbose="1" preserve-order="true" name="SequenceTest">
        <groups>
            <define name="eatOne" >
                <include name="eatOne"/>
            </define>
            <define name="eatTen">
                <include name="eatTen"/>
            </define>
            <define name="eatOneTen">
                <include name="eatOne"/>
                <include name="eatTen"/>
            </define>
            <run>
                <include name="eatTen"/>
                <!--<include name="eatOneTen"/>-->
            </run>
        </groups>
        <classes>
            <!-- 可以多个 -->
            <class name="org.vk.test.springtest_testng.EatServiceTest" />
        </classes>
    </test>
</suite>
```

如果依赖关系比较复杂，实现依赖注入指定的 class 过多，这是可以指定 Application.class .

###### class指定Application
这里时间了两个样例一个是只指定 Application 完成 bean 的初始化，其次是参数传递(TestNG实现)
```
package org.vk.test.springtest_testng;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.testng.AbstractTestNGSpringContextTests;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;
import org.vk.Application;
import org.vk.demo.EatCompentConfig;
import org.vk.demo.PeopleEatService;
import org.vk.demo.PeopleEatServiceImpl;

/**
 * @author viakiba
 *  classes需要注入的类
 * @description
 * @date Create in 20:38 2018/7/22
 */

@SpringBootTest(classes = { Application.class })
//@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
public class EatAppTest extends AbstractTestNGSpringContextTests {
    @Autowired
    private PeopleEatService peopleEatServiceImpl;
    @Test(groups = "eatOne")
    public void test1(){
        int i = peopleEatServiceImpl.eatService();
        System.out.println(i * 1);
    }
    @Test(groups = "eatTen")
    public void test2(){
        int i = peopleEatServiceImpl.eatService();
        System.out.println(i*10);
    }
    @Parameters({"a","b"})
    @Test(groups = "eatTenAB")
    public void test3(int a , int b){
        int i = peopleEatServiceImpl.eatService();
        System.out.println(i*10);
        System.out.println(a+b);
    }
}
```

###### XML配置

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="1">
    <test verbose="1" preserve-order="true" name="SequenceTest">
        <groups>
            <define name="eatOne" >
                <include name="eatOne"/>
            </define>
            <define name="eatTen">
                <include name="eatTen"/>
            </define>
            <define name="eatOneTen">
                <include name="eatOne"/>
                <include name="eatTen"/>
            </define>
            <run>
                <include name="eatTen"/>
                <!--<include name="eatOneTen"/>-->
            </run>
        </groups>
        <classes>
            <!-- 可以多个 -->
            <class name="org.vk.test.springtest_testng.EatAppTest" />
        </classes>
    </test>

</suite>
```

###### 传入参数样例

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "http://testng.org/testng-1.0.dtd">
<suite name="Suite" parallel="classes" thread-count="1">
    <test verbose="1" preserve-order="true" name="SequenceTest">
        <groups>
            <define name="eatAb" >
                <include name="eatTenAB"/>
            </define>
            <run>
                <include name="eatAb"/>
                <!--<include name="eatOneTen"/>-->
            </run>
        </groups>
        <classes>
            <!-- 可以多个 -->
            <class name="org.vk.test.springtest_testng.EatAppTest" >
                <parameter name="b" value="5"/>
                <parameter name="a" value="10"/>
            </class>
        </classes>
    </test>
</suite>
```

### 总结
以上 SpringBoot 项目中可以借助 TestNG 实现灵活的测试需求，并且借助 Spring DI 特性可以更加精炼的写出测试代码，完成依赖注入。

```
参考
  https://testng.org/doc/documentation-main.html
  https://blog.csdn.net/u011138533/article/details/52174446
  https://www.cnblogs.com/iceb/p/7119987.html?utm_source=itdadao&utm_medium=referral
  https://i.itest.ren/2016/05/06/Java%E6%B5%8B%E8%AF%95%E6%A1%86%E6%9E%B6-TestNG/
  https://blog.csdn.net/u011138533/article/details/52174446
  https://blog.csdn.net/u011138533/article/details/52174446
  https://blog.csdn.net/taiyangdao/article/details/77096823
  https://blog.csdn.net/douya43/article/details/73555843
  https://segmentfault.com/a/1190000010854538
```
