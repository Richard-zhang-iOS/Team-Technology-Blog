### 什么是gradle
简单的说gradle是一套构建工具，所谓构建工具就是根据一堆输入信息，生成一系列产物，复杂的说gradle是一套框架，所有的参数配置其实都严格对应相应的API，我们没有必要可以记住这些API，我们需要掌握的是大体流程，然后借助官方SDK+API来完成自己的需求。
### 为什么会有gradle
在Gradle爆红之前，常用的构建工具是ANT，然后又进化到Maven。ANT和Maven这两个工具其实也还算方便，现在还有很多地方在使用。但是二者都有一些缺点，所以让更懒得人觉得不是那么方便。比如，Maven编译规则是用XML来编写的。XML虽然通俗易懂，但是很难在xml中描述if{某条件成立，编译某文件}/else{编译其他文件}这样有不同条件的任务。于是gradle就应运而生，gradle采用的语言是groovy，groovy能让你写java语言像写脚本一样简单，同时gradle是一种DSL语言，即Domain Specific Language，领域相关语言。什么是DSL，说白了它是某个行业中的行话。在gradle的体现有很多，比如sourceSets代表源文件的集合等。
### 如何使用gradle
1.首先我们需要学习一下groovy，值得一提的是groovy语言在编译的时候已经编译为java类字节码，然后运行在JVM虚拟机中，可以理解为它扩展了Java语言，写法十分的奔放，比如定义变量使用`def`定义，但是这个关键字也不是必需的，语句结尾可以不使用`;`，单引号严格对应Java中的双引号，双引号则和脚本语言中的双引号类似，当含有`$`符号时，则它会`$`表达式先求值，三引号中的字符串支持换行。暂时先了解这些，详细的可以直接看`GDK`。
2.接下来我们就介绍一下gradle的行话，在gradle中，每一个待编译的工程都是一个`Project`对象，每个`Project`在编译的时候包含多个`Task`，比如一个`Android`项目的编译可能就包含**Java源码编译Task**、**资源编译Task**、**JNI编译Task**、**lint检查的Task**、**打包生成APK的Task**、**签名Task**等等。一个`Project`包含的`Task`的数量是由编译脚本执行的插件决定的，插件就是用来定义`Task`，并且具体执行这些`Task`的东西。而gradle作为一个框架，主要负责定义流程和规则，具体的编译工作则是通过编译插件来完成，比如编译Java有Java插件，编译Groovy有Groovy插件，编译Android APP有Android APP插件，编译Android Library有Android LIbrary插件。该![图片](https://raw.githubusercontent.com/sakurajiang/Picture/master/%E8%87%AA%E5%AE%9A%E4%B9%89Gradle%E6%8F%92%E4%BB%B6/%E6%A8%A1%E5%9D%97%E6%95%B0%E7%9B%AE.png)中有两个`Project`，根据gradle设计，每一个`Project`都对应一个`build.gradle`对象，而`build.gradle`就是该`Project`的编译脚本，如果只有这些，那么你编译的就需要进入到每一个`Project`中，然后执行编译脚本，这样效率很低，于是gradle提供了**Multi-Projects Build**，即多个项目一起编译，实现很简单，就是再项目根目录增加一个`build.gradle`文件，然后再增加一个`settings.gradle`，前者可以什么都不写，后者就是通过`include`标签将所有的`Project`包含进来，如：`include ':app', ':buildsrc'`，同时我们还可以在这里添加一些初始化的函数，比如：
```
	def initSomeValue(){
    Properties properties = new Properties();
    File fileProperties = new File(rootDir.getAbsolutePath()+"/local.properties");
    properties.load(fileProperties.newDataInputStream());
    gradle.ext.myApi = properties.getProperty("sdk.dir");
    println("init"+gradle.myApi);
}
```
该函数就是将`SDK`的目录赋值给gradle的额外属性`myApi`中，下次直接通过`gradle.myApi`获取该属性。目前我们了解的内容如下：
**每一个Project都必须设置一个build.gradle文件。
对于multi-projects build，需要在根目录下也放一个build.gradle，和一个settings.gradle。
一个Project是由若干tasks来组成的，当gradle xxx的时候，实际上是要求gradle执行xxx任务。这个任务就能完成具体的工作。**
下面说一下gradle的工作流程，简单地说就是先执行`settings.gradle`，完成初始化过程，然后解析每一个`build.gradle`，其中的`Task`会被添加到有向图中，用于解决依赖关系，最后就是执行阶段，执行gradle XXX，gradle就会将这个xxx任务链上的所有任务全部按依赖顺序执行一遍。接下来介绍几种对象：
1.Gradle对象，当执行gradle XXX的时候，gradle会从默认的配置脚本中构造一个Gradle对象，在整个构建过程中只有一个Gradle对象，Gradle对象的数据类型就是Gradle。我们一般很少去定制这个默认的配置脚本。
2.Project对象，每一个`build.gradle`对象都会生成一个`Project`对象，在gradle中，`Project`对象对应的是Build Script，在`build.script`中，我们一般需要做如下几件事，分别是：应用插件、设置属性等。
3.`Settings`对象，每个`settings.gradle`都会生成一个`Settings`对象，注意，对于其他gradle文件，除非定义了class，否则会转换成一个实现了Script接口的对象。这一点和3.5节中Groovy的脚本类相似。
讲了这么多，那么一般怎么用呢？**对于Java和Groovy而言，我们一般会把公共的方法放在一个类中，然后在别的类中import该类，但是在Gradle中，一般都是通过在`build.gradle`中定义插件，然后在插件中定义`Task`来完成**，定义`Task`可以通过task函数来定义，一个`Task`包含多个Action，所以`Task`有两个函数，分别是doFirst和doLast，即最先执行和最后执行。普及了基础知识，接下来直接看这个项目吧,代码里差不多都有注释。这里需要再补充的就是自定义gradle插件步骤：
1.新建一个module（我一般是选择Android Library）

2.除了main文件夹和build.gradle全部删除，注意main文件夹下的内容也要删除

3.在main下新建groovy文件夹

4.在groovy新建package，名字自己随意取，跟包名类似

5.在package下新建name.groovy文件，name自己取，如下：

```
package com.sakurajiang.test

import org.gradle.api.Plugin
import org.gradle.api.Project

public class MyPlugin implements Plugin<Project> {
    void apply(Project project) {
        def log = project.logger
        log.error "========================";
        log.error "精简的MyPlugin，开始修改Class!";
        log.error "========================";
    }
}```

6.在main目录下新建resources文件夹

7.在resources文件夹下新建META-INF文件夹

8.在META-INF文件夹下新建gradle-plugins文件夹

9.在gradle-plugins文件夹下新建name.properties文件，这里的文件名就是引用该插件时用到的名字。其中的内容是implementation-class=com.sakurajiang.test.MyPlugin，后面的值就是步骤5中的文件的包名加类名。

10.在该module的build.gradle文件中配置如下属性

```
apply plugin: 'groovy'
apply plugin: 'maven'
dependencies {
    compile gradleApi() //gradle sdk
    compile localGroovy() //groovy sdk
}
repositories {
    jcenter()
}

group='sakurajiang'
version='1.0.0'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: uri('../repo'))
        }
    }
}```

11.选择AS中右边的gradle，选择自定义的插件，选择upload中的uploadArchives

12.在需要使用该插件的module中的build中添加如下代码：
```
buildscript {
    repositories {
        maven {
            url uri('../repo')
        }
    }
    dependencies {
        classpath 'sakurajiang:buildsrc:1.0.0'
    }
}
apply plugin: 'Z'
```
其中'Z'就是步骤9中的name，其中的classpath就是group：modulename：version。

13.选择Make Project，至此就可以在Build中看到自定义插件中的信息了。自定义插件到此完成。