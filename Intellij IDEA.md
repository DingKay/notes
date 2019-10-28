## Intellij IDEA

## <a id="IDEA简介">IDEA简介</a>

* 一 、IDEA的介绍(#IDEA简介)

  Idea是捷克JetBrians公司的产品，是目前最好用的Java开发工具之一。

  公司旗下的产品还有：

  ​	WebStorm：用于开发JavaScript、Html5、CSS3等前端技术。

  ​	PyCharm：用于开发Python

  ​	PhpStorm：用于开发PHP

  ​	···

* 二 、IDEA的主要功能介绍

  ​	语言支持上：

  * 安装插件后支持PHP、Python、Ruby、Scala、Kotlin、Clojure

  * SQL类支持：PostgreSQL、MySQL、Oracle、SQL Server
  * 基本JVM：Java、Groovy
  * 支持的框架：Spring MVC、GWT、Vaadin、Play、Grails、Web Services、JSF、Struts、Hibernate、Flex
  * 额外支持的语言代码提示：HTML5、CSS3、SASS、LESS、JavaScript、CoffeScript、Node.js、ActionScript
  * 支持的容器：Tomcat、WomEE、WebLogin、JBoss、Jetty、WebSphere

* 三、IDEA的主要优势(相比较于Eclipse)

  1. 强大的整合能力
  2. 提示功能的快速、便捷
  3. 提示功能范围广
  4. 好用的快捷键和代码模板
  5. 精准搜索

* 四、[IDEA下载地址](https://www.jetbrains.com/)

---

* 五、IDEA的版本关系

  IDEA每四个月发布一个版本，一年发布三个版本

  	> 比如说 ：
  	>
  	> 		2017.1 一般为3月份发布的
  	>
  	> 		2017.2一般为7月份发布的
  	>
  	> 		2017.3一般为11月份发布的

  ## IDEA安装前的准备

  > - 2GB RAM Minimum ，4GB RAM Recommended
  > - 1.5GB Hard Disk Space + At Least 1GB For Caches
  > - 1024 × 768 Minimum Screen Resolution
  > - JDK

  *最好安装在固态硬盘上*

  `*如果需要完全卸载IDEA 则再次运行IDEA的安装程序 跟着提示完全卸载IDEA*`

  ### IDEA的安装总结

  ---

  从安装上来看，Intellij IDEA对硬件的要求似乎不是很高，可是实际在开发过程中并不是这样的，因为Intellij IDEA在执行时会有大量的缓存、索引文件，所以如果你正在使用Eclipse、MyEclipse 想通过Intellij IEDA来缓解计算机的卡、慢等问题，这基本上是不可能的，本质上应该对硬件进行升级。

  * IDEA的目录结构

    *bin*目录中的`idea.exe.vmoptions`或者`idea64.exe.vmoptions` 文件 可以修改IDEA的启动配置

    > -Xms 初始内存数
    >
    > -Xmx 最大内存数 可以减少GC的回收频率
    >
    > -XX：ReservedCodeCachesSize 所保留的缓存代码大小
    >
    > ···

  * 在`C：`盘符中会自动生成配置目录等

    > config 以及 system目录
    >
    > 这是IDEA的各种配置的保存目录，这个设置目录有一个特性，就是你删除掉整个目录之后，重新启动IDEA会自动再帮你生成一个全新的默认配置，所以很多时候如果你把Intellij IDEA配置修改坏了，没关系，删掉该目录，一切都会还原到默认。

    ---

    `IDEA中的 Project ≈ Eclipse 中的WorkSpace`

    * IDEA 的Module(模块) ≈ Eclipse 的Project![IDEA的Module](images\IDEA\IDEA的Module.png)

      Tips：*Module的删除需要再Module Setting中先移除--再删除*

    ---

    * 工程下的 `.idea 以及 .iml`文件都是IDEA工程特有的 类似于Eclipse工程下的`.settings .classpath .project`等。

  ### IDEA的常用配置

  ---

  * Appearance & Behavior(外观和行为)：

     * Appearance：UI Options -- Theme 设置主题

  * Editor：

     * Editor -- General -- Change Font Size(Zoom) with Ctrl + Mouse Wheel ==配合Ctrl键加上鼠标滚轮可以调整字体大小==

     * Editor -- General -- Show Quick Documentation On Mouse Move 显示快速的文本提示 ==当鼠标悬浮在类、方法上时可以显示一个快速的文档提示==

     * Editor -- General -- Insert Import On Paste `ALL` `Add Unambiguous Import On The Fly` `Optimize Import On The Fly(For Current Project)`  自动导包功能 ==Alt + Enter 手动导包==

     * Editor -- General -- Appearance -- Show Method Separators 设置方法的分隔符

     * Editor -- General -- Code Completion Case Sensivice Completion `None` ==不区分大小写进行提示==

     * Editor -- General -- Editor Tabs `Show Tabs In Single Row` ==设置Tab页多行显示==

     * Editor -- Color Scheme|Console Font -- Font -- Size 设置控制台字体 字体大小等

    * Editor -- Code Style -- Java -- Import Class Count To Use Import With “*” 设置导入类的个数为X个时自动更改为导入所有

    * Editor -- File and Code Templates -- File Header 设置文件头部信息

      > /**
      >
      >   *@author
      >
      >   *@create ${YEAR} - ${MONTH} - ${DAY} ${TIME}
      >
      > **/

    * Editor -- File Encoding ==设置当前工程的字符编码集==

      `Transparent native-to-ascii conversion` 涉及到本地的ASCII码的转换

      *更改字符集 `reload` 以某种字符集重新加载 但不改变原有文件的字符集

      `convert`即对该文件做某种字符集的转换 有可能出现乱码

  * Build,Execution,Deployment：

    * Compiler --Build Project Automatically | Compile Independent Module In Parallel ==设置工程自动编译 并对当前多个模块进行编译==

  * Power Save Mode 省电模式

    ···

  ---

  # IDEA 的快捷键


	## IDEA的Template(模板)

* Editor -- Postfix Completion：系统自带的无法修改

* Editor -- Live Templates：用户可以自定义修改

  >两者都提供了模板，Postfix Templates 较 Live Templates 速度更快。

---

#### 一、常用的模板：

主函数快速模板：public void static main(String [] args)   **`psvm`**

快速输出模板：System.out.println()	**`sout`**

* 配合使用

  输出方法参数:**`soutp`**

  输出方法名称:**`soutm`**

  输出方法变量:**`soutv`** ==就近原则==

  输出自定义变量:**`xxx.sout`**

  ---

循环模板：for(int i =0 ;i < xx ;i++) **`fori`**

增强型for循环模板：for(String s : arr) **`iter` `itar`**

集合遍历模板：**`list.for增强型循环` `list.fori`从头部开始顺序遍历 `list.forr`从尾部逆序遍历**

条件判断模板：**`ifn`  如果等于null `inn` 如果不等于null**

 * 增强型 **`xx.null` `xx.nn`**

定义属性快捷模板：

* private static final **`prsf`** ==当定义了一个private私有的变量在Module(模块)中，project(工程)可以使用Module(模块)下的私有属性，不过需要在iml文件中配置，快捷键为 `alt + enter`  ==Add dependency on module 'xxx'==![](images\IDEA\Module增加依赖.png)

* public static final **`psf`**
* public static final int **`psfi`**
* public static final String **`psfs`**

