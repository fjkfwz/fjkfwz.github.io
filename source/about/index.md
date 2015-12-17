title: Resume
date:
type: "about"
---
#基本信息

>- **姓名：** 林国强    *GUOQIANG LIN*
- **学历：** 福建农林大学  12级生物科学
- **PhoneNum:** 186.504.868.94
- **EmailAdd:** fjkfwz@gmail.com
- **Telegram:** https://telegram.me/masukio

***
#项目
##Openwrt Remote Manage Toolkits
**描述：** JSON-RPC远程系统调用工具集。
**包含：**
>- rpcshell ：系统级的RPC服务端守护进程，你可以向服务端发送带Shell命令的JSON调用系统资源。由WebShell修改而来，将请求/回调封装为JSON格式。
- rpcconfig：由rpcshell和本地配置解析工具uci封装的RPC接口，能够方便的对配置文件进行修改。
- rpcloader：由rpcshell和dynamic Loader封装的RPC接口，实现应用程序动态加载远程Module模块。

##Openwrt H3C inode Client
**描述：** 第三方H3C inode客户端。
**文档：** http://masukio.tk/2015/07/07/H3C-inode-Linux/
**服务器端：**
基于@liuqun/njit8021xclient项目第一版用Python重写

修改：
>- 绕过服务器端版本验证：大量发送数据包直到服务器正确响应
- 添加新的加密方式：猜测数据加密方式及构造认证信息
- 添加心跳认证：定期发送心跳包确保连接
- Module：动态加载不同的加密方式/认证信息以适应不同校区的认证
- 断线重连：定时发送ping数据包，若掉线则restart
- 管理页面：用Lua编写的Luci界面

第二版用C重写整合了Remote Manage Toolkits编译进Openwrt系统镜像

**客户端：**
Android管理器：基于JSON-RPC开发的Android客户端，能够查看Openwrt的基本信息，操作/管理应用
微信后台管理器：
>- 绑定：但设备连上网络会向微信后台发送设备状态信息，并返回设备码
- 后台：数据储存在MySQL，服务器用JSON与设备通讯并封装成微信消息返回给微信服务器

***

#实验项目

##Zhihu Bubble

**描述：**基于非监督学习算法的知乎智能推荐系统
**进度：**
>- 完成简易版知乎单机爬虫抓取Followee List,获取少量数据集
- 测试聚类/NMF模型，测试协同过滤，缓存Redis
- 分析每个Filter Bubble(https://en.wikipedia.org/wiki/Filter_bubble)特征
- 计算特征的打分矩阵，选取Top10特征，爬虫抓取每个特征对应的文章列表
- 计算文章的打分矩阵，输出Hotspot文章列表

**Next：**

>- 编写MapReduce版本爬虫，Monogodb持久化连接
- Redis/Bloom Filter过滤重复连接
- 制作Docker镜像，方便部署
- 编写MapReduce版本算法训练模型/测试CUDA
- 用Flask搭建一个前端/设计API接口

***

#Skill Stack

##算法
掌握基本的数据结构及算法(CLRS)及其部分MapReduce算法
掌握常见的数据结构及算法(LeetCode)，B/B+Tree，LruCache，Bloom Filter等
了解基本的Machine Learning，协同，CUDA算法

##语言
**JAVA:**
>- 熟悉JDK 8/Kotlin基本语法，熟悉JDK基础库及第三方常用库的使用
- 熟悉JAVA的异常处理机制
- 熟悉Reflect机制，class的动态加载，class Loader机制
- 了解JAVA的并发编程模型，公共内存/锁机制

**Python:**
>- 了解Python2.7.基本语法，Lambda函数式，基础库
- 熟悉第三方常用库及统计学库(Numpy)的使用，图形库matplotlib
- 了解Python多进程/线程模型，GIL
- 熟悉RE模块，正则表达式匹配
- 熟悉Python的异常处理机制

**JVM:**
>- 了解JAVA编译字节码文件
- 了解Sun HotSpot/Dalvik JVM 编译执行流程
- 了解GC回收机制及原理

**Pattern:** 熟悉Code Refactoring的基本准则，了解OOP Design Pattern
**OOP:** 熟悉面向对象的编程思想，抽象，封装，闭包，回调
**VCS：**
>- 熟悉Git/SVN的常用操作及冲突处理，diff/patch/quilt的使用
- 熟悉Git-flow：[http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)

**RFC通信协议：** 熟悉XMPP，POP3/IMAP/STMP(数据格式MIME)通信协议
**REST:** 熟悉RESTful API/RPC架构的设计原则及开发
**数据格式:** 熟悉XML/JSON的数据格式，JAVA/Python库的文件生成及解析，GSON的使用
**设计模式:** 了解MVC/MVP，MVVM
**加密算法：**
>- 了解常用的对称/非对称加密算法，消息摘要算法，数字签名算法
- 配置Nginx实现HTTPS传输

##后端开发
**PHP：**
>- 了解PHP语法，能够快速搭建产品原型，主要用于为Android开发搭建后台
- 框架：ThinkPHP，Smarty模板

**linux：**
>- 能够熟练使用\*nix基础命令(busybox)，Vim，网络配置
- 常用网络服务的搭建/配置，iptable/ipset配置，Qos技术原理及配置
- 编译工具链GNU toolchain，Makefile文件的编写
- 熟悉Nginx反代，均衡负载
常用的linux系统如下：CentOS，Ubuntu 14.x LTS，Debian，(Openwrt)

**数据库：**
>- 熟悉sqlite/MySQL基础CURD SQL操作及常用语句及JDBC接口程序，MySql主从备份
- 熟悉Mongodb常用语句及JDBC/mongopy接口程序的使用
- 了解Mongodb Shard，Replication基本配置，事务/锁机制，增量备份，Raft选举
- 了解Hive常用HQL操作，metadata，数据仓库的概念

**动态缓存：**熟悉Redis，Memcached set/get操作
**Hapdoop：**能够编写经典算法的分布式版本，了解hadoop生态
**容器技术：**
>- 了解服务器端虚拟化技术：KVM，Docker，Hyper
- Docker镜像制作

**云服务：** GCP(APP/Container Engine)，Firebase，AWS(EC2，EMR)，阿里云ECS，DaoCloud
**安全：** 了解常用的加解密算法(CSSIP)，VLAN网段划分，容灾热备

##前端开发
**基础：** 熟悉基本HTML/CSS/JavaScript语句，了解基础DOM操作，事件传递机制，ECMAScript规格
**框架：** 熟悉基础框架：Sass，jQuery，BootStrap的基础语法，DOM选择器的使用
**Tools:** 了解前端自动构建工具集Yeoman，Bower，Grunt的使用
**Node.js：** 了解Node.js的基础语法及包的使用

##Android
**基础：**
>- 熟悉Android的基本组件，Layout，常用控件
- 熟悉数据持久化的开发,网络编程，事件传递机制
- 熟悉自定义View，了解Android IPC机制，Remote View机制
- 了解JNI编程，动态加载技术
- 熟悉Activity，Fragment，Application生命周期，Activity Stack
- 了解Sensor开发

**并发：**熟悉Android的线程和线程池，AsyncTask
**网络框架：**熟悉OkHTTP，Volley，Retrofit的使用
**优化：**熟悉Android View组件的常用优化，缓存策略，LruCache/DiskLruCache
**开发：**IDE:Android Studio，熟悉基本的调试工具，ANR日志分析，MAT内存泄露分析
**源码：**阅读过Android核心代码
>- 了解系统/进程启动流程
- Bundle native/JNI消息传递机制
- 核心组件的类调用，编译流程

**Android5+:**熟悉Android5新增/@Deprecated API，运行时权限管理，Dove，Data Binding
**Design：**熟悉Material Design/HIG设计规范及design library组件的使用

##嵌入式开发
**驱动：**能够根据时序图/硬件文档编写简单的驱动
**Openwrt系统：**阅读过openwrtBB源码，熟悉构建/编译流程，能够进行系统移植
**Openwrt开发：**能够在Openwrt进行应用程序及Luci应用界面的开发
**通信协议: **熟悉RFID，NFC，Zigbee，Bluetooth协议
**开发板：** 单片机80C51，Ardunio UNO，Raspberry Pi V1，Easylink M150
**项目：** 寻路径独轮车，3D Matrix，数码管时钟，四轴飞行器

##生物信息
**NGS：** 熟悉第二代测序技术illumina Hiseq(X10,2500),lon Torrent,PacBio测序原理/read/depth/coverage/accuracy
**基因芯片：** 了解Affymetrix，Agilent生物芯片原理
**算法：** 熟悉BLAST序列比对算法，HMM模型
**Tools：** 熟悉常用的生物信息分析工具GATK，bowtie，数据格式转换，Pipleline搭建

##其他
**Bitcoin:** Bitcoin爱好者，熟悉BlockChain技术
**Tools:** Markdown，LaTeX/MathJax，XMind，Axure，Sublime Text

***
##Groups:

Microsoft Student Partners(MSP)
Google Developer Group (GDGxXiamen)
视觉与学习青年学者研讨会(VALSE)
全球创客马拉松(Makerthon)



