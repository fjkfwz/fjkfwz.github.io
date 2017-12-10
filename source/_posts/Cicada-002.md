title: Project Cicada# 002：K-9 Mail 源码分析
date: { { date } }
categories: Cicada
tags: [K-9 Mail,Android]
description:
---
**摘要**：K-9 Mail 是 Android 平台上使用 Java 语言开发的专业的开源邮件客户端，系统设计、代码实现、注释良好，支持 MS Exchange Server，邮件会话和推送，有着健壮的开发者社区，本文分析其技术架构以及主要的代码实现以便打包成库供后期开发使用。
<!--more-->
# 实现层级
K-9 Mail 将的主要的用于邮件会话的实体类打包成类库，其主要的实体类包括`com.fsck.k9.mail`提供用于通讯过程需要的类和用于对邮件提供面向编程思想进行邮件内容封装的实体类`com.fsck.k9.mail.internet`

上层包括用于实现用户数据持久化和用于充当用户 UI 信息控制的 Activity 的相关操作，其包名为`com.fsck.k9`类其中两个重要的类包括`Account`、`K9`，二者通过`SharedPreference`类来持久化数据

K9类继承自'android.app.Application',其主要的作用用于设置、获取客户端应用全局的配置性数据，包括获取邮件的频率、全局的主题设置、消息的推送、预览邮件内容的设置，其方法提供给其他任何的类使用

Account 类对应的是 MVC 模型中的`Model`类，它接收来自包括`Activity，Service，Broadcast 和 Receiver `等等机制传递过来的用户请求，其除了封装相应的用户信息，还被设计用于保存账户的各种设置，包括用于账户身份认证的 Identity，字体设计 FontSizes，通知设置 NotificationSetting 和邮件收发地址、草稿箱、在各种网络状态下是否启用数据的压缩，是否进行邮件加密等

# Library 类
## com.fsck.k9.mail
`com.fsck.k9.mail`中的核心类包括`Address`、`Folder`、`Message`、`Store`

![K9mail 中主要的类](http://7nar5o.com1.z0.glb.clouddn.com/class.png)

### Address
Address 类实现了 E-Mail 的地址的封装，其将获取收件人发件人，邮件目标地址的信息并将其拼接成固定数据格式的 String 流

### Store 类
`Store`类为抽象类，相当于远程或本地的 Store 代理其包括4个子类`imap`、`pop3`、`webdav`分别用于访问 IMAP,POP3，WebDav 服务器和本地 SQLite 数据可，Store 类根据用户 Account 设置的属性值`mStoreUri`来创建对应的 Store 子类方法

```
 Store store = sStores.get(uri);
        if (store == null) {
            if (uri.startsWith("imap")) {
                store = new ImapStore(storeConfig,
                        new DefaultTrustedSocketFactory(context),
                        (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE));
            } else if (uri.startsWith("pop3")) {
                store = new Pop3Store(storeConfig,
                        new DefaultTrustedSocketFactory(context));
            } else if (uri.startsWith("webdav")) {
                store = new WebDavStore(storeConfig);
            }

            if (store != null) {
                sStores.put(uri, store);
            }
        }

        if (store == null) {
            throw new MessagingException("Unable to locate an applicable Store for " + uri);
        }

        return store;
    }
```
对外通过提供接口 MessagingController 类来调用库中 Store 类来实现对底层的同一封装，在创建 EMail 账户，与 WebDav 通信也会调用其 Store 类，然后调用 Store 类并通过 RemoteStore 对邮件进行路由从而对不同的通信协议进行处理

### Message
Message 实现了对邮件内容的封装，有一个子类 MimeMessage 继承自 Message，其安照`metadata in RFC 822 and RFC 2045 style headers.`规定的数据结构来拼装数据，即以 OO 的方式通过 Adress 类封装邮件地址，Body 类来完成对邮件主体数据的封装从而封装整个邮件数据，K9 Mail 在其4个 Store 类中以内部类的形式提供了 MImeMessage 的子类，包括`ImapMessage，LocalMessage，Pop3Message 和 WebDavMessage）来操作每一种类型 Store 中的邮件

#### MIME 邮件的构成
>MIME 的全称是"Multipurpose Internet Mail Extensions"，中译为"多用途互联网邮件扩展"，指的是一系列的电子邮件技术规范，主要包括 RFC 2045、RFC 2046、RFC 2047、RFC 4288、RFC 4289和 RFC 2077。

顾名思义，MIME 是对传统电子邮件的一个扩展，现在已经成为电子邮件实际上的标准。

传统的电子邮件是1982年定下技术规范的，文件是 RFC 822。

它的一个重要特点，就是规定电子邮件只能使用 ASCII 字符。这导致了三个结果：1）非英语字符都不能在电子邮件中使用；2）电子邮件中不能插入二进制文件（如图片）；3）电子邮件不能有附件。

这实际上无法接受的，因此到了1992年，工程师们决定扩展电子邮件的技术规范，提出一系列补充规范，这就是 MIME 的由来。

下面是一封传统的电子邮件。

	From: "Tommy Lee" <lee@example.com>
	To: "Jack Zhang" <zhang@example.com>
	Subject: Test
	Date: Wed, 17 May 2000 19:08:29 -0400
	Message-ID: <NDBBIAKOPKHFGPLCODIGIEKBCHAA.lee@example.com>
	Hello World.

从上面可以看出，这封信的发信人地址是`lee@example`.com，收信人地址是`hang@example.com`，邮件主题是 Test，发送时间是2000年5月17日，邮件内容是`"Hello World."`。

在结构上，这封信分为三个部分：首先是信件头，然后是一个空行，最后是信件内容。收信人的客户端软件只会显示最后一部分，要查看全信，必须使用"查看原始邮件"功能。

MIME 对传统电子邮件的扩展，表现在它在信件头部分添加了几条语句，主要有三条。
第一条是：

	MIME-Version: 1.0

这条语句是必须的，而且1.0这个版本值是不变的，即使 MIME 本身已经升级了好几次。
有了这条语句，收信端就知道这封信使用了 MIME 规范。
第二条语句是：

	Content-Type: text/plain; charset="ISO-8859-1"

这一行是极端重要的，它表明传递的信息类型和采用的编码。
Content-Type 表明信息类型，缺省值为" text/plain"。它包含了主要类型（primary type）和次要类型（subtype）两个部分，两者之间用"/"分割。主要类型有9种，分别是`application`、`audio`、`example`、`image`、`message`、`model`、`multipart`、`text`、`video`。
每一种主要类型下面又有许多种次要类型，常见的有：

	text/plain：纯文本，文件扩展名.txt
	text/html：HTML 文本，文件扩展名.htm 和.html
	image/jpeg：jpeg 格式的图片，文件扩展名.jpg
	image/gif：GIF 格式的图片，文件扩展名.gif
	audio/x-wave：WAVE 格式的音频，文件扩展名.wav
	audio/mpeg：MP3格式的音频，文件扩展名.mp3
	video/mpeg：MPEG 格式的视频，文件扩展名.mpg
	application/zip：PK-ZIP 格式的压缩文件，文件扩展名.zip

详细的 Content-Type 列表，可以查看这里和这里。
如果信息的主要类型是"text"，那么还必须指明编码类型"charset"，缺省值是 ASCII，其他可能值有"ISO-8859-1"、"UTF-8"、"GB2312"等等。
整个 Content-Type 这一行，不仅使用在电子邮件，后来也被移植到了 HTTP 协议中，所以现在只要是在网上传播的 HTTP 信息，都带有 Content-Type 头，以表明信息类型。
前面已经说过，电子邮件的传统格式不支持非 ASCII 编码和二进制数据。因此 MIME 规定了第三条语句：

	Content-transfer-encoding: base64

这条语句指明了编码转换的方式。Content-transfer-encoding 的值有5种,"7bit"、"8bit"、"binary"、"quoted-printable"和"base64",其中"7bit"是缺省值，即不用转化的 ASCII 字符。

##### MIME 消息
总体来说，MIME 消息由消息头和消息体两大部分组成。现在我们关注的是 MIME 邮件，因此在以下的讨论中姑且称“消息”为“邮件”。邮件头中不允许出现空行。有一些邮件不能被邮件客户端软件识别，显示的是原始码，就是因为首行是空行。

###### 邮件头
邮件头包含了发件人、收件人、主题、时间、MIME 版本、邮件内容的类型等重要信息。每条信息称为一个域，由域名后加“: ”和信息内容构成，可以是一行，较长的也可以占用多行。域的首行必须“顶头”写，即左边不能有空白字符（空格和制表符）；续行则必须以空白字符打头，且第一个空白字符不是信息本身固有的，解码时要过滤掉。

邮件体包含邮件的内容，它的类型由邮件头的“Content-Type”域指出。常见的简单类型有 text/plain(纯文本)和 text/html(超文本)。邮件体被分为多个段，每个段又包含段头和段体两部分，这两部分之间也以空行分隔。常见的 multipart 类型有三种：multipart/mixed, multipart/related 和 multipart/alternative。从它们的名称，不难推知这些类型各自的含义和用处。它们之间的层次关系可归纳为下图所示：


可以看出，如果在邮件中要添加附件，必须定义 multipart/mixed 段；如果存在内嵌资源，至少要定义 multipart/related 段；如果纯文本与超文本共存，至少要定义 multipart/alternative 段。什么是“至少”？举个例子说，如果只有纯文本与超文本正文，那么在邮件头中将类型扩大化，定义为 multipart/related，甚至 multipart/mixed，都是允许的。

multipart 诸类型的共同特征是，在段头指定“boundary”参数字符串，段体内的每个子段以此串定界。所有的子段都以“--”+boundary 行开始，父段则以“--”+boundary+“--”行结束。段与段之间也以空行分隔。在邮件体是 multipart 类型的情况下，邮件体的开始部分(第一个“--”+boundary 行之前)可以有一些附加的文本行，相当于注释，解码时应忽略。段间也可以有一些附加的文本行，不会显示出来，如果有兴趣，不妨验证一下。
结合 boundary 定界和 multipart 层次关系图，。

在邮件头中，有很多从 RFC 822沿用的域名，MIME 也增加了一些。常见的标准域名和含义如下

| 域名	| 含义	| 添加者 |
| ------------- |:-------------|:-------------:|
| Received	| 传输路径	| 各级邮件服务器 |
| Return-Path	| 回复地址	| 目标邮件服务器 |
| Delivered-To	| 发送地址	| 目标邮件服务器 |
| Reply-To	| 回复地址	| 邮件的创建者 |
| From	| 发件人地址	| 邮件的创建者 |
| To	| 收件人地址	| 邮件的创建者 |
| Cc	| 抄送地址	| 邮件的创建者 |
| Bcc	| 暗送地址	| 邮件的创建者 |
| Date	| 日期和时间	| 邮件的创建者 |
| Subject	| 主题	| 邮件的创建者 |
| Message-ID	| 消息 ID	| 邮件的创建者 |
| MIME-Version	| MIME 版本	| 邮件的创建者 |
| Content-Type	| 内容的类型	| 邮件的创建者 |
| Content-Transfer-Encoding	| 内容的传输编码方式 | 	邮件的创建者 |

非标准的、自定义域名都以 X-开头，例如 X-Mailer, X-MSMail-Priority 等，通常在接收和发送邮件的是同一程序时才能理解它们的意义。
在段头中，大致有如下一些域

| 域名	| 含义 |
| ------------- |:-------------:|
| Content-Type	| 段体的类型 |
| Content-Transfer-Encoding	| 段体的传输编码方式 |
| Content-Disposition	| 段体的安排方式 |
| Content-ID	| 段体的 ID |
| Content-Location	| 段体的位置(路径) |
| Content-Base	| 段体的基位置 |

有的域除了值之外，还带有参数。值与参数、参数与参数之间以“;”分隔。参数名与参数值之间以“=”分隔。如例3的28-29行，Content-Type 域的值为“multipart/alternative”，此外有一个参数 boundary，值为"----=_NextPart_002_007C_01C3115F.80DFC5E0"。又如例3的第176行，Content-Disposition 域的值为“attachment”，此外有一个参数 filename，值为“readme.doc”。


Content-Type 都是“主类型/子类型”的形式。主类型有 text, image, audio, video, application, multipart, message 等，分别表示文本、图片、音频、视频、应用、分段、消息等。每个主类型都可能有多个子类型，如 text 类型就包含 plain, html, xml, css 等子类型。以 X-开头的主类型和子类型，同样表示自定义的类型，未向 IANA 正式注册，但大多已经约定成俗了。如 application/x-zip-compressed 是 ZIP 文件类型。在 Windows 中，注册表的“HKEY_CLASSES_ROOT/MIME/Database/Content Type”内列举了除 multipart 之外大部分已知的 Content-Type。

![邮件 body 的三种组成及它们之间的关系](http://7nar5o.com1.z0.glb.clouddn.com/邮件 body.png)

关于参数的形式，RFC 里有很多补充规定，有的允许带几个参数，较为常见的有

| 主类型 | 参数名 | 含义 |
| ------------- |:-------------|:-------------:|
| text	| charset	| 字符集 |
| image	| name | 名称 |
| application | name | 名称	|
| multipart	| boundary | 边界 |

其中字符集也能在 Windows 注册表的“HKEY_CLASSES_ROOT/MIME/Database/Charset”内见到。

Content-Transfer-Encoding 共有 Base64, Quoted-printable, 7bit, 8bit, Binary 等几种。其中7bit 是缺省的编码方式。电子邮件源码最初设计为全部是可打印的 ASCII 码的形式。非 ASCII 码的文本或数据要编码成要求的格式，如上面的三个例子。Base64, Quoted-Printable 是在非英语国家使用最广使的编码方式。Binary 方式只具有象征意义，而没有任何实用价值。
Base64将输入的字符串或一段数据编码成只含有`{'A'-'Z', 'a'-'z', '0'-'9', '+', '/'}`这64个字符的串，'='用于填充。其编码的方法是，将输入数据流每次取6 bit，用此6 bit 的值(0-63)作为索引去查表，输出相应字符。这样，每3个字节将编码为4个字符(3×8 → 4×6)；不满4个字符的以'='填充。

有的场合，以“=?charset?B?xxxxxxxx?=”表示 xxxxxxxx 是 Base64编码，且原文的字符集是 charset。如
Quoted-printable 根据输入的字符串或字节范围进行编码，若是不需编码的字符，直接输出；若需要编码，则先输出'='，后面跟着以2个字符表示的十六进制字节值。有的场合，以“=?charset?Q?xxxxxxxx?=”表示 xxxxxxxx 是 Quoted-printable 编码，且原文的字符集是 charset。在段体内则直接编码，适当时机换行，换行前额外输出一个'='。

近年来，国内多数邮件服务器已经支持8bit 方式，因此只在国内传输的邮件，特别是在邮件头中，可直接使用8bit 编码，对汉字不做处理。如果邮件要出国，还是老老实实地按 Base64或 Quoted-printable 编码才行。

内嵌资源也是 MIME 的一个发光点，它能使邮件内容变得生动活泼、丰富多彩。可在邮件的 multipart/related 框架内定义一些与正文关联的图片、动画、声音甚至 CSS 样式和脚本的段。通常在 HTML 正文内，使用超级链接与内嵌资源相联系。

	<BODY background=cid:007901c3111c$72b978a0$0100007f@bluesky bgColor=# ffffff>
	它指出用一个 Content-ID 为007901c3111c$72b978a0$0100007f@bluesky 的图片作为背景(cid:xxxxxxxx 也是一种超级链接)。而64-169行恰好就是这样一个内嵌资源。
	除了用 Content-ID 进行联系外，还有另外一种常用形式：用普通超级连接和 Content-Location。例如：
	在 HTML 正文中，
	... ...  ... ...
	<IMG SRC="http://www.dangdang.com/images/all/anti_joyo_dm_book.gif">
	... ...  ... ...
	<IMG SRC="http://www.dangdang.com/dd2001/getimage_small.asp?id=486341">
	... ...  ... ...
	对应的内嵌资源为
	Content-Type: image/gif; name="anti_joyo_dm_book.gif"
	Content-Transfer-Encoding: base64
	Content-Location: http://www.dangdang.com/images/all/anti_joyo_dm_book.gif
	... ... ... ...
	Content-Type: application/octet-stream; name="getimage_small.asp?id=486341"
	Content-Transfer-Encoding: base64
	Content-Location: http://www.dangdang.com/dd2001/getimage_small.asp?id=486341
	... ... ... ...
	另外，
	Content-Location: http://www.dangdang.com/images/all/anti_joyo_dm_book.gif
	与
	Content-Location: anti_joyo_dm_book.gif
	Content-Base: http://www.dangdang.com/images/all/
	是等效的。

### Folder
Folder 用于处理邮件文件夹的封装，Folder 也是一个抽象类，k9 mail 也同时在 Store 中对应提供4种类型的即`ImapFolder,Pop3Folder,LocalFolder 和 WebDavFolder，用于将数据封装成文件夹的格式

### Filter
包`com.fask.k9.mail.filter`，Android 数据流的加解密库包括 Base64的加解密，二进制流的加解密，十进制的加解密其用于做数据流的解析，数据的打包封装，将编码过的邮件的网络数据流解密成具有可读性的 String 流

### Power
包`com.fsck.k9.mail.power`中的 TaccingPowerManager 用于调控系统的资源管理，跟踪 Wakelock,对线程唤醒进行调度，从而安排邮件的获取，更新 Notification

### SSL
包`com.fsck.k9.mail.ssl`，Android SSL 加密库用于支持加密的邮件服务器，对邮件客户端到服务器端的通信进行

### Transport
包`com.fsck.k9.mail.transport`用于提供邮件底层的网络连接，包括认证设置（验证用户名，密码和客户端证书），并提供未加密的 SMTP 连接和 TLS 隧道数据连接的 SMTP 通讯，SSL 加密的 SMTP 通讯，其连接遵循 SMTP 邮件通讯协议

## K9包
K9和 Account 均使用`SharedPreferences`，`SharedPreferences`是平台下除 SQLite 外的另一种方便的数据持久化方式，是 Android 平台下最简单的外部数据读写方法，适用于保存不同用户的个性化设置信息

一个账户通过一个 UUID 定义，可以通过`mUuid`的属性来区分两个账户的设置信息。Account 类实现了接口`BaseAcount`，这个接口定义了基础的用户信息，能够获取、设置 EMail 账户及获取其信息

# 业务实现流程
## 接收邮件
k9mail 可以通过 IMAP 和 POP3两种方式来从服务器端读取邮件，它设计和实现上的亮点包括

	1.实现了用一个同一个流程无缝融合 IMAP,POP3和 WebDav 三种不同的账户类型，其从服务器端接收邮件的流程和代码是相同的
	2.通过抽象的方式实现获取邮件过程与账户类型无关，但没有增加不同获取方式之间代码的耦合度，通过代码将不同类型的账户类型路由到不同的 Store 子类，包括 IMAP，POP3和 WebDav，从而实现代码的松耦合。
	3.接收邮件时，通过``类来实现邮件结构的判断，从数据流的头信息，从而区分邮件数据流的块大小，先接收小邮件，后接收大邮件，接收的小邮件直接返回给 UI 线程进行更新，而将耗时任务转移到后台进行数据的接收工作，同时可以根据用户的设置判断自动接收邮件。判断邮件大小的区分可以通过 Account 用户自定义属性`MaximumAutoToDownload`通过`SharedPreference`储存到 XML 的 Key-Value 数据结构中。
	4.同时在接收邮件的过程中将进行服务器端与客户端直接的操作，包括客户端与服务器端邮件信息的同步

![](http://7nar5o.com1.z0.glb.clouddn.com/LoadMessage.png)

接收邮件的实现流程如下：
### 启动后台进程
用户点击`MessageListFragment`底栏`FooterView`选择`ManualSearch`时

```
 public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
        if (view == mFooterView) {
            if (mCurrentFolder != null && !mSearch.isManualSearch()) {

                mController.loadMoreMessages(mAccount, mFolderName, null);

```

Activity 会调用 MessagingController 的 loadMoreMessages 加载当前账户文件夹的下一批邮件

	mController.loadMoreMessages(mAccount, mFolderName, null);

在`loadMoreMessages`方法中，调用 `synchronizeMailbox()`来加载更多的邮件

	synchronizeMailbox(account, folder, listener, null);

synchronizeMailbox 方法在后台开启一个线程，并运行`synchronizeMailboxSynchronous()`方法来完成加载邮件，通知界面数据更新的操作
```
 public void synchronizeMailbox(final Account account, final String folder, final MessagingListener listener, final Folder providedRemoteFolder) {
        putBackground("synchronizeMailbox", listener, new Runnable() {
            @Override
            public void run() {
                synchronizeMailboxSynchronous(account, folder, listener, providedRemoteFolder);
            }
        });
    }
```
synchronizeMailboxSynchronous()方法通知界面监听器 Listener 来更新状态
```
for (MessagingListener l : getListeners(listener)) {
            l.synchronizeMailboxStarted(account, folder);
        }
        /*
         * We don't ever sync the Outbox or errors folder
         */
        if (folder.equals(account.getOutboxFolderName()) || folder.equals(account.getErrorFolderName())) {
            for (MessagingListener l : getListeners(listener)) {
                l.synchronizeMailboxFinished(account, folder, 0, 0);
            }

            return;
        }
```
### Messaging 同步文件夹
通知所用 MessagingListern 开始同步文件夹

	l.synchronizeMailboxStarted(account, folder);

如果是发件箱是`发件箱 Outbox 或者错误的文件夹 errors folder`则调用界面同步完成并返回

	l.synchronizeMailboxFinished(account, folder, 0, 0);

### 取得远程 Store
从 SQLite 中获取本地邮件获得本地邮件信息`Folder.OPEN_MODE_RW`更新最新的 UID`updateLastUid()`保存到一个 HashMap 中，根据 Account 对象`account.getRemoteStore()`取得远程 Store（Pop3Store,ImapStore,WebDavStore）及其文件夹
```
 final LocalStore localStore = account.getLocalStore();
            tLocalFolder = localStore.getFolder(folder);
            final LocalFolder localFolder = tLocalFolder;
            localFolder.open(Folder.OPEN_MODE_RW);
            localFolder.updateLastUid();
            List<? extends Message> localMessages = localFolder.getMessages(null);
            Map<String, Message> localUidMap = new HashMap<String, Message>();
            for (Message message : localMessages) {
                localUidMap.put(message.getUid(), message);
            }

            if (providedRemoteFolder != null) {
                remoteFolder = providedRemoteFolder;
            } else {
                Store remoteStore = account.getRemoteStore();
                remoteFolder = remoteStore.getFolder(folder);
                if (! verifyOrCreateRemoteSpecialFolder(account, folder, remoteFolder, listener)) {
                    return;
                }
```

从文件夹中获取远程邮件，判断远程邮件夹是否为为 HashMap 中确定是否下载这个远程邮件，把下一批下载到本地的邮件收集到一个 ArrayList<Message>中.

```
   remoteFolder.open(Folder.OPEN_MODE_RW);
                if (Expunge.EXPUNGE_ON_POLL == account.getExpungePolicy()) {
                    remoteFolder.expunge();
                }

            }

            /*
             * Get the remote message count.
             */
            int remoteMessageCount = remoteFolder.getMessageCount();

            int visibleLimit = localFolder.getVisibleLimit();

            if (visibleLimit < 0) {
                visibleLimit = K9.DEFAULT_VISIBLE_LIMIT;
            }

            final List<Message> remoteMessages = new ArrayList<Message>();
            Map<String, Message> remoteUidMap = new HashMap<String, Message>();

            final Date earliestDate = account.getEarliestPollDate();


            if (remoteMessageCount > 0) {
                /* Message numbers start at 1.  */
                int remoteStart;
                if (visibleLimit > 0) {
                    remoteStart = Math.max(0, remoteMessageCount - visibleLimit) + 1;
                } else {
                    remoteStart = 1;
                }
                int remoteEnd = remoteMessageCount;

                if (K9.DEBUG)
                    Log.v(K9.LOG_TAG, "SYNC: About to get messages " + remoteStart + " through " + remoteEnd + " for folder " + folder);

                final AtomicInteger headerProgress = new AtomicInteger(0);
                for (MessagingListener l : getListeners(listener)) {
                    l.synchronizeMailboxHeadersStarted(account, folder);
                }


                List<? extends Message> remoteMessageArray = remoteFolder.getMessages(remoteStart, remoteEnd, earliestDate, null);

                int messageCount = remoteMessageArray.size();

                for (Message thisMess : remoteMessageArray) {
                    headerProgress.incrementAndGet();
                    for (MessagingListener l : getListeners(listener)) {
                        l.synchronizeMailboxHeadersProgress(account, folder, headerProgress.get(), messageCount);
                    }
                    Message localMessage = localUidMap.get(thisMess.getUid());
                    if (localMessage == null || !localMessage.olderThan(earliestDate)) {
                        remoteMessages.add(thisMess);
                        remoteUidMap.put(thisMess.getUid(), thisMess);
                    }
                }

                for (MessagingListener l : getListeners(listener)) {
                    l.synchronizeMailboxHeadersFinished(account, folder, headerProgress.get(), remoteUidMap.size());
                }

            } else if (remoteMessageCount < 0) {
                throw new Exception("Message count " + remoteMessageCount + " for folder " + folder);
            }
```


### 同步数据
判断用户是否设置为本地 Folder 与远程服务器同步`account.syncRemoteDeletions()`，若值为1，则删除服务器上不存在但本地存在的邮件

```
/*
             * Remove any messages that are in the local store but no longer on the remote store or are too old
             */
            if (account.syncRemoteDeletions()) {
                List<Message> destroyMessages = new ArrayList<Message>();
                for (Message localMessage : localMessages) {
                    if (remoteUidMap.get(localMessage.getUid()) == null) {
                        destroyMessages.add(localMessage);
                    }
                }


                localFolder.destroyMessages(destroyMessages);

                for (Message destroyMessage : destroyMessages) {
                    for (MessagingListener l : getListeners(listener)) {
                        l.synchronizeMailboxRemovedMessage(account, folder, destroyMessage);
                    }
                }
            }
```

### 执行下载
执行`MessagingController.downloadMessages()`下载第3步确认下载的邮件到本地，下载结果是成功通知所有注册的`MessagingListener`

```
/*
             * Now we download the actual content of messages.
             */
            int newMessages = downloadMessages(account, remoteFolder, localFolder, remoteMessages, false);

            int unreadMessageCount = localFolder.getUnreadMessageCount();
            for (MessagingListener l : getListeners()) {
                l.folderStatusChanged(account, folder, unreadMessageCount);
            }
```

### 优先下载
在 downloadMessages()中计算未同步的邮件、小邮件、大邮件到本地，最后保存到 SQLite 之中

```
         * Grab the content of the small messages first. This is going to
         * be very fast and at very worst will be a single up of a few bytes and a single
         * download of 625k.
         */
        FetchProfile fp = new FetchProfile();
        fp.add(FetchProfile.Item.BODY);
        //        fp.add(FetchProfile.Item.FLAGS);
        //        fp.add(FetchProfile.Item.ENVELOPE);

        downloadSmallMessages(account, remoteFolder, localFolder, smallMessages, progress, unreadBeforeStart, newMessages, todo, fp);
        smallMessages.clear();
```
### 判断邮件 Body
通过`message.getBody() == null`来判断该邮件是否有内容，如果邮件 Body 为空则直接调用对应的 Folder 的 fetch()`remoteFolder.fetch()`方法来取得邮件；
、`localFolder.appendMessages()、localFolder.getMessage()`更新本地 Folder 信息；否则则通过`MessageExtractor.collectTextParts();`取得邮件的各个部分，再调用对应的`remoteFolder.fetchPart();`把邮件的每一个部分收集到本地,并构造到邮件中，然后`localFolder.appendMessages();`保存到 SQLite 中


```
 if (message.getBody() == null) {
    /*
   	* The provider was unable to get the structure of the message, so
    * we'll download a reasonable portion of the messge and mark it as
    * incomplete so the entire thing can be downloaded later if the user
    * wishes to download it.
    */
    fp.clear();
    fp.add(FetchProfile.Item.BODY_SANE);
    /*
     *  TODO a good optimization here would be to make sure that all Stores set
     *  the proper size after this fetch and compare the before and after size. If
     *  they equal we can mark this SYNCHRONIZED instead of PARTIALLY_SYNCHRONIZED
     */

    remoteFolder.fetch(Collections.singletonList(message), fp, null);

    // Store the updated message locally
    localFolder.appendMessages(Collections.singletonList(message));

    Message localMessage = localFolder.getMessage(message.getUid());


    // Certain (POP3) servers give you the whole message even when you ask for only the first x Kb
    if (!message.isSet(Flag.X_DOWNLOADED_FULL)) {
    /*
     * Mark the message as fully downloaded if the message size is smaller than
     * the account's autodownload size limit, otherwise mark as only a partial
     * download.  This will prevent the system from downloading the same message
     * twice.
     *
     * If there is no limit on autodownload size, that's the same as the message
     * being smaller than the max size
     */
    if (account.getMaximumAutoDownloadMessageSize() == 0 || message.getSize() < account.getMaximumAutoDownloadMessageSize()) {
        localMessage.setFlag(Flag.X_DOWNLOADED_FULL, true);
        } else {
        // Set a flag indicating that the message has been partially downloaded and
        // is ready for view.
        localMessage.setFlag(Flag.X_DOWNLOADED_PARTIAL, true;
        	}
        }
    } else {
        /*
         * We have a structure to deal with, from which
         * we can pull down the parts we want to actually store.
         * Build a list of parts we are interested in. Text parts will be downloaded
         * right now, attachments will be left for later.
         */

        Set<Part> viewables = MessageExtractor.collectTextParts(message);

        /*
         * Now download the parts we're interested in storing.
         */
        for (Part part : viewables) {
            remoteFolder.fetchPart(message, part, null);
        }
        // Store the updated message locally
        localFolder.appendMessages(Collections.singletonList(message));

        Message localMessage = localFolder.getMessage(message.getUid());
    }

```
### 设置重复下载
最后设置邮件下载标识防止重复下载`localMessage.setFlag(Flag.X_DOWNLOADED_PARTIAL, true)`

```
// Set a flag indicating this message has been fully downloaded and can be viewed.
localMessage.setFlag(Flag.X_DOWNLOADED_PARTIAL, true);

```
通过`MessagingListener`通知文件夹添加或更新邮件，更新进度

```
for (MessagingListener l : getListeners()) {
    l.synchronizeMailboxAddOrUpdateMessage(account, folder, localMessage);
    l.synchronizeMailboxProgress(account, folder, progress.get(), todo);
    if (!localMessage.isSet(Flag.SEEN)) {
                    l.synchronizeMailboxNewMessage(account, folder, localMessage);
                        }
                    }
```
## 发送邮件
发送邮件的过程中，客户端一边执行异步 AnsycTask，一边更新 UI 线程，发送完成后返回发送成功的页面。

发送邮件的过程如下：

![](http://7nar5o.com1.z0.glb.clouddn.com/sendemail.png)

### 启动后台

用户编写邮件的 Activity 是`MessageCompose`,在`MessageCpmpose`中触发`MenuItem`的`send`按钮启动发送邮件的过程

```
case R.id.send:
	mPgpData.setEncryptionKeys(null);
	onSend();
	break;
```
然后进行用户输入的检查，如果用户的发送地址，主题，邮件主题为空，则返回错误信息，使用 Toast 对用户进行提醒，并检查队列信息，如果之前没有邮件则调用`performSend()`，对用户的设置进行判断，进行不同方式的邮件加密（SignEncryptCallback and with encryptedData set in pgpData），调用`sendMessage()`方法执行一个新的`SendMessageTask()`,在这个类的`doInBackground()`在`MessagingController`中获取一个 Instance 执行`sendMessage()`动作
```
MessagingController.getInstance(getApplication()).sendMessage(mAccount, message, null);
```
调用 MessagingController 的`sendMessage()`完成操作

### 本地保存
在 MessageController 的`send()`中,首先根据当前的 account 账户找到`LocalStore`对象，再根据`localStore`句柄获取`LocalFolder`对象，然后调用`localFolder.appendMessages()`把待发送的邮件保存在本地。

```
LocalStore localStore = account.getLocalStore();
LocalFolder localFolder = localStore.getFolder(account.getOutboxFolderName());
localFolder.open(Folder.OPEN_MODE_RW);
localFolder.appendMessages(Collections.singletonList(message));
Message localMessage = localFolder.getMessage(message.getUid());
localMessage.setFlag(Flag.X_DOWNLOADED_FULL, true);
localFolder.close();
sendPendingMessages(account, listener);
```

### 发送邮件
接下来`MessagingController`调用`sendPendingMessages()`从发件箱中发送邮件，`sendPendingMessagesSynchronous()`内完成具体邮件的发送工作并通知 Listener 更新 UI 界面,并且支持输出发送消息进度的信息

	transport.sendMessage(message);

在`sendPendingMessagesSynchronous()`方法中，首先根据当前的 account 账户找到`LocalStore`对象，再根据`localStore`句柄获取`OutboxFolder`发件箱的`LocalFolder`对象后,调用向 MessagingController 注册的 Listerner`sendPendingMessagesStarted()`
```
Store localStore = account.getLocalStore();
localFolder = localStore.getFolder(
                  account.getOutboxFolderName());
if (!localFolder.exists()) {
    return;
}
for (MessagingListener l : getListeners()) {
    l.sendPendingMessagesStarted(account);
}
localFolder.open(Folder.OPEN_MODE_RW);
```
如果发件箱中的邮件标记为“删除”，则先销毁这部分邮件，然后循环发送每一封邮件，并为每一封邮件打上状态标记（Flag.X_SEND_IN_PROGRESS）

```
if (message.isSet(Flag.DELETED)) {
    message.destroy();
    continue;
	}
```
在发送的过程中，
调用向 MessagingController 注册的 Listerner`synchronizeMailboxProgress()`通知监听器发送已开始，
在邮件的发送过程中，循环调用`synchronizeMailboxProgress()`方法传递 progress 参数，在发送每一封邮件`transport.sendMessage()`的过程中，进度计数器会`progress++`,在邮件发送结束后调用`l.sendPendingMessagesCompleted();`
```
for (MessagingListener l : getListeners()) {
    l.synchronizeMailboxProgress(account, account.getSentFolderName(), progress, todo);
    }
```

`sendPendingMessagesSynchronous()`调用`transport.sendMessage(message)`发送每一封邮件，如果不存在已发送文件夹则将发出的邮件 destory，如果存在已发送文件夹则将已发送邮件从未发送文件夹转移到已发送文件夹，发送过程调用`EOLConvertingOutPutStrean`以数据流的方式将待发送的邮件写入服务器端。

```
localFolder.moveMessages(Collections.singletonList(message), localSentFolder);
```
最后调用`processPendingCommands()`同步服务器端已发送文件夹。

## 删除邮件

### 删除一组邮件

![K9Mail 删除邮件的流程](http://7nar5o.com1.z0.glb.clouddn.com/deletemesssage.png)

1.k9mail 在 Activity MessageList 方法`onCustomKeyDown`中设置删除按钮的响应事件,根据删除按钮所在的界面(`MassageListFragment`和`MessageViewFragment`的删除按钮)调用`onDelect`方法删除邮件

```
if (mDisplayMode == DisplayMode.MESSAGE_LIST) {
    mMessageListFragment.onDelete();
} else if (mMessageViewFragment != null) {
    mMessageViewFragment.onDelete();
}
```

`MassageListFragment`再分别调用`MessagingController`的方法`deleteThreads()`和`deleteMessages()`去删除一个会话邮件或是一组邮件,`MessageViewFragment`调用`MessagingController`的`deleteMessages()`方法删除一个邮件

```
private void onDeleteConfirmed(List<LocalMessage> messages) {
        if (mThreadedList) {
            mController.deleteThreads(messages);
        } else {
            mController.deleteMessages(messages, null);
        }
    }
```
在执行删除操作之前,kmail 会`disable`删除按钮,防止连续误删

`MessagingController`执行删除操作调用`actOnMessages()`方法启动后台线程`deleteThreadsSynchronous()`并把任务压栈道名为 mCommands 的 BlockingQueue<Command>类型的队列中

```
private void putBackground(String description, MessagingListener listener, Runnable runnable) {
        putCommand(mCommands, description, listener, runnable, false);
```
使在删除过程中，界面仍能够保持正常的响应，在操作过程中通知 MessagingListener 通知界面

2.删除邮件之前先通知 MessagingListener 删除界面邮件，删除邮件时，判断是否存在垃圾箱，当前文件夹为垃圾箱，则将邮件打上`Flag.DELETED`的标签，
```
if (folder.equals(account.getTrashFolderName()) || !account.hasTrashFolder()) {
    localFolder.setFlags(messages, Collections.singleton(Flag.DELETED), true);
} else {
    localTrashFolder = localStore.getFolder(account.getTrashFolderName());
    if (!localTrashFolder.exists()) {
        localTrashFolder.create(Folder.FolderType.HOLDS_MESSAGES);
    }
    if (localTrashFolder.exists()) {
        uidMap = localFolder.moveMessages(messages, localTrashFolder);
    }
}
```
如果待删除邮件在发件箱内，将待删除的邮件移入垃圾箱，则把要删除的邮件移入垃圾箱，同时向服务器执行 PendingExpunge()在服务器上移动响应的文件夹；然后通过 Account 获取 LocalStore 后获取 LocalFolder，通过 Account 的方法`getDeletePolicy()`获取用户删除操作执行不同的动作，如果设置删除动作为“已读”则给该邮件打上“已读”的标签，如果用户策略为删除则打上“删除”标签，执行真正的删除操作。
```
if (folder.equals(account.getOutboxFolderName())) {
    for (Message message : messages) {
    // If the message was in the Outbox, then it has been copied to local Trash, and has
    // to be copied to remote trash
        PendingCommand command = new PendingCommand();
        command.command = PENDING_COMMAND_APPEND;
        command.arguments =
            new String[] {
            account.getTrashFolderName(),
            message.getUid()
        };
        queuePendingCommand(account, command);
    }
    processPendingCommands(account);
} else if (account.getDeletePolicy() == DeletePolicy.ON_DELETE) {
    if (folder.equals(account.getTrashFolderName())) {
        queueSetFlag(account, folder, Boolean.toString(true), Flag.DELETED.toString(), uids);
    } else {
        queueMoveOrCopy(account, folder, account.getTrashFolderName(), false, uids, uidMap);
                }
        processPendingCommands(account);
    } else if (account.getDeletePolicy() == DeletePolicy.MARK_AS_READ) {
        queueSetFlag(account, folder, Boolean.toString(true), Flag.SEEN.toString(), uids);
        processPendingCommands(account);
    }

```
调用`processPendingCommands()`方法将删除操作保存到 SQLite 的`Pending_commands`的表中
```
List<PendingCommand> commands = localStore.getPendingCommands();
```
执行完成后则从表中删除

3.执行删除操作，获取到远程文件夹
```
Store remoteStore = account.getRemoteStore();
Folder remoteFolder = remoteStore.getFolder(folder);
```
取得到 ImapFolder，在`ImapFolder`中执行真正的删除操作
```
remoteFolder.open(Folder.OPEN_MODE_RW);
if (remoteFolder.getMode() != Folder.OPEN_MODE_RW) {
    return;
}
remoteFolder.expunge();
} finally {
            closeFolder(remoteFolder);
```
`ImapFolder`调用另一个内部类`ImapConnection`向服务器发送`expunge()`命令，`ImapFolder`处理`expunge()`的响应。

## IMAP 邮件的解析、封装、保存

![服务器响应解析](http://7nar5o.com1.z0.glb.clouddn.com/服务器响应解析.png)

在获取邮件的过程中，通过`message.getBody() == null`来判断该邮件是否有内容，如果邮件 Body 为空则直接通过`MessageController`调用`ImapStore`的内部类`ImapFolder`的`fetch()`方法来获取邮件；否则则通过`MessageExtractor.collectTextParts();`取得邮件的各个部分，再调用对应的`remoteFolder.fetchPart();`把邮件的每一个部分收集到本地,并构造到邮件中，然后`localFolder.appendMessages();`保存到 SQLite 中

`fetch()`方法的操作流程如下：首先检查到服务器的 TCP 连接（若无连接则抛出异常）-> 构造 IMAP 命令 -> 通过 ImapConnection 将命令发送到邮件服务器 -> 读取、解析服务器响应

```
mConnection.sendCommand(String.format("UID FETCH %s (%s)",
	combine(uidWindow.toArray(new String[uidWindow.size()]), ','),
    combine(fetchFields.toArray(new String[fetchFields.size()]), ' ')), false);
    ImapResponse response;
    int messageNumber = 0;

    ImapResponseCallback callback = null;
    if (fp.contains(FetchProfile.Item.BODY) || fp.contains(FetchProfile.Item.BODY_SANE)) {
        callback = new FetchBodyCallback(messageMap);
    }
    response = mConnection.readResponse(callback);

        ImapMessage imapMessage = (ImapMessage) message;

        Object literal = handleFetchResponse(imapMessage, fetchList);
```

首先通过`mConnection.sendCommand()`方法向服务器传输命令，根据邮件 UID 来获取邮件，K9Mail 通过使用了`java.net.Socket`类通过 Socket 连接邮件服务器，k9mail 与邮件服务器建立连接和管理连接的实现封装在`ImapConnection`之中，连接过程通过域名服务器(DNS)解析@host 域名，返回的 IP 地址逐个尝试连接服务器，直到连接到邮件服务器后将不再尝试后面的 address，在与服务器的连接过程中，`ImapConnection`调用`ImapResponseParser`对服务器返回的响应先做一遍解析，提高对不规范邮件的错容性，尽可能的显示所有能够正常显示的邮件或者对于实在无法解析的邮件显示邮件的部分内容


调用`ImapMessage`类解析服务器响应，将返回的数据封装成为`ImapMessage`类型，

```
        String bodyString = (String)literal;
        InputStream bodyStream = new ByteArrayInputStream(bodyString.getBytes());
        imapMessage.parse(bodyStream);

```
`Folder`,`Message`类其作用近似于接口但相对于接口它提供了一些通用方法的实现，例如判断发送时间先后的`olderThan()`方法，在实际处理业务逻辑的过程中使用的的是它们的子类，Message 类只有一个子类`MimeMessage`，邮件头`MimeHearder`封装、Email 地址`Address`封装,邮件体`MimeBody`封装等都定义在`MimeMessage`之中，`MimeMessage`有4个子类是响应 Store 的内部类用于实现对应`Store`类的具体功能。

K9Mail 实际解析邮件并且实现封装的类为`MimeMessage`主要的实现方法是调用的是 Apache 的开源邮件解析库 mimej4来实现，K9mail 主要使用了 mime4j 的编解码、解析邮件并封装为对象、处理特殊字符这三个方面的功能，`MimeHearder`类用于封装 Mime 邮件的头部，提供了获取邮件头信息（收件人、抄送人地址）的处理，拼装的方法

![](IMAP 邮件的分装)(http://7nar5o.com1.z0.glb.clouddn.com/message.png)

在`MimeMessage`中实际完成解析工作的是类方法`parse(InputStream in)`和内部类`MimeMessageBuilder`，二者的主要功能是提供邮件的解析机制完成邮件的解析并封装为`MimeMessage`对象
```
private void parse(InputStream in, boolean recurse) throws IOException, MessagingException {
...

MimeStreamParser parser = new MimeStreamParser(parserConfig);
parser.setContentHandler(new MimeMessageBuilder());
if (recurse) {
    parser.setRecurse();
}
try {
    parser.parse(new EOLConvertingInputStream(in));
}
...
```
在解析过程中，遇到邮件内容的关键字`head`,`MultiPart`,`body`,`BodyPart`,`epilogue`、`preamble`
`field`等会触发响应类的解析工作，例如当开始解析头信息时，调用`Part.class`对 header 进行解析
```
public void startHeader() {
    expect(Part.class);
}
```

一个 MimeMessage 对象可以有一个`MultiPart`(同时又是一个 Body 类型)的 mBody，这个 Body 可以有一组数量不限的 BodyPart，每个 BodyPart 又是一个 Body 类型，同时每个`MultiPart`还可以有一个 Part 类型的父对象，这样就构成了一组递归关系，从父对象往 BodyPart 解析直到没有内容为止。

MimeMessageBuilder 是负责将解析好邮件的各个部分组装成为 k9mail 对邮件内容的封装类，最终解析好的邮件会被封装为 MimeMessage 对象
```
private class MimeMessageBuilder implements ContentHandler
```
`ContentHandler`类是`MimeMessageBuilder`类的接口，`ContentHandler`是 mime4j 的接口

mime4j 的 MimeStreamParser 类负责具体的解析工作，在解析过程中会以事件触发机制调用之前注册的`ContentHandler`(`MimeMessageBuilder`),通知其使用构建结果构建的对象
```
case T_BODY
BodyDescriptor desc = mimeTokenStream.getBodyDescriptor()
InputStream bodyContent
if (contentDecoding) 
    bodyContent = mimeTokenStream.getDecodedInputStream()
} else 
    bodyContent = mimeTokenStream.getInputStream()
    handler.body(desc, bodyContent)
break;
```
解析完成后调用`localFolder.appendMessages(Collections.singletonList(message))`将 MimeMessage 对象保存到 SQlite 的`message`表中，k9mail 先提取各个字段的值保存到`ContentValue`（相当于 Map，其中数据可以在 SQLiteDatabase 操作数据库使用）中，然后调用`android.database.sqlite`的`insert`方法将 ContentValue 的数据保存到 message 表中并创建在`threads`中的入口

```
ContentValues cv = new ContentValues();
    cv.put("message_part_id", rootMessagePartId);
    cv.put("uid", uid);
    cv.put("subject", message.getSubject());
    cv.put("sender_list", Address.pack(message.getFrom()));
    cv.put("date", message.getSentDate() == null
            ? System.currentTimeMillis() : message.getSentDate().getTime());
    cv.put("flags", this.localStore.serializeFlags(message.getFlags()));
    cv.put("deleted", message.isSet(Flag.DELETED) ? 1 : 0);
    cv.put("read", message.isSet(Flag.SEEN) ? 1 : 0);
    cv.put("flagged", message.isSet(Flag.FLAGGED) ? 1 : 0);
    cv.put("answered", message.isSet(Flag.ANSWERED) ? 1 : 0);
    cv.put("forwarded", message.isSet(Flag.FORWARDED) ? 1 : 0);
    cv.put("folder_id", mFolderId);
    cv.put("to_list", Address.pack(message.getRecipients(RecipientType.TO)));
    cv.put("cc_list", Address.pack(message.getRecipients(RecipientType.CC)));
    cv.put("bcc_list", Address.pack(message.getRecipients(RecipientType.BCC)));
    cv.put("preview", preview);
    cv.put("reply_to_list", Address.pack(message.getReplyTo()));
    cv.put("attachment_count", attachmentCount);
    cv.put("internal_date", message.getInternalDate() == null
            ? System.currentTimeMillis() : message.getInternalDate().getTime());
    cv.put("mime_type", message.getMimeType());
    cv.put("empty", 0);

    String messageId = message.getMessageId();
    if (messageId != null) {
    cv.put("message_id", messageId);
    }

	if (oldMessageId == -1) {
    long msgId = db.insert("messages", "uid", cv);

    // Create entry in 'threads' table
    cv.clear();
    cv.put("message_id", msgId);

    if (rootId != -1) {
        cv.put("root", rootId);
    }
    if (parentId != -1) {
        cv.put("parent", parentId);
    }

    db.insert("threads", null, cv);
    } else {
    db.update("messages", cv, "id = ?", new String[] { Long.toString(oldMessageId) });
    }
```
### mime4j
开源项目 mime4j 是一个优秀的邮件内容解析、构建和处理库，它主要有两个特点：
	1.使用回调机制报告邮件解析事件的发生
	在解析过程中，当遇到开始和结束邮件头、邮件体等构成邮件元素的时候，mime4j 会通过回调的方式对外发起调用，这种行为类似于 SAX XML 解析器，有效简化了邮件的解析和解析结果封装的实现。
	2.提高了对不规范邮件的兼容性
	mime4j 对不规范邮件的兼容性达到了有 Perl 编写的邮件解析工具 MIME:Tools 的水平，二者在邮件解析结果的差别在0.5%，主要是在不规范垃圾邮件的解析上。

## 添加用户的流程

![添加用户的流程](http://7nar5o.com1.z0.glb.clouddn.com/bindingaccount.png)

当用户不存在或用户在`Activity AccountList`中点击 bottombar 的添加新用户时，会启用 Intent 打开`Activity AccountSetupBasic`进行用户信息的相关配置，先进行一遍邮件地址的解析工作，如果是常用的邮件域名（为名为`R.xml.providers`文件中的域名），则根据响应的配置调用 finishAutoSetup()自动完成配置的工作，否则则需要手动配置邮件服务器的信息
```
 XmlResourceParser xml = getResources().getXml(R.xml.providers);
```
`R.xml.providers`文件中邮件配置的定义如下：
```
<provider id="gmail" label="Gmail" domain="gmail.com">
    <incoming uri="imap+ssl+://imap.gmail.com" username="$email" />
    <outgoing uri="smtp+ssl+://smtp.gmail.com" username="$email" />
</provider>
```
finishAutoSetup()方法如下：
```
private void finishAutoSetup() {
        String email = mEmailView.getText().toString();
        String password = mPasswordView.getText().toString();
        String[] emailParts = splitEmail(email);
        String user = emailParts[0];
        String domain = emailParts[1];
        try {
            String userEnc = UrlEncodingHelper.encodeUtf8(user);
            String passwordEnc = UrlEncodingHelper.encodeUtf8(password);

            String incomingUsername = mProvider.incomingUsernameTemplate;
            incomingUsername = incomingUsername.replaceAll("\\$email", email);
            incomingUsername = incomingUsername.replaceAll("\\$user", userEnc);
            incomingUsername = incomingUsername.replaceAll("\\$domain", domain);

            URI incomingUriTemplate = mProvider.incomingUriTemplate;
            URI incomingUri = new URI(incomingUriTemplate.getScheme(), incomingUsername + ":" + passwordEnc,
                    incomingUriTemplate.getHost(), incomingUriTemplate.getPort(), null, null, null);

            String outgoingUsername = mProvider.outgoingUsernameTemplate;

            URI outgoingUriTemplate = mProvider.outgoingUriTemplate;


            URI outgoingUri;
 			if (mAccount == null) {
                mAccount = Preferences.getPreferences(this).newAccount();
            }
            mAccount.setName(getOwnerName());
            mAccount.setEmail(email);
            mAccount.setStoreUri(incomingUri.toString());
            mAccount.setTransportUri(outgoingUri.toString());

            setupFolderNames(incomingUriTemplate.getHost().toLowerCase(Locale.US));

            ServerSettings incomingSettings = RemoteStore.decodeStoreUri(incomingUri.toString());
            mAccount.setDeletePolicy(AccountCreator.getDefaultDeletePolicy(incomingSettings.type));

            // Check incoming here.  Then check outgoing in onActivityResult()
            AccountSetupCheckSettings.actionCheckSettings(this, mAccount, CheckDirection.INCOMING);
        }
```
finishAutoSetup()的流程如下：

1.先搜寻默认的 Provider `findProviderForDomain(domain)`
2.然后根据模板信息用模板信息拼装`incomingUri`和`outgoingUri`
3.并调用`Preferences.getPreferences(this).newAccount()`方法建立一个新的 Account，并配置一些用户信息，并保存用于从远程获取邮件的 StoreUri（`incomingUri`），和向远程发送邮件的 TransportUri(outgoingUri）
4.配置填写完成后调用`Activity AccountSetupCheckSetting`进行配置信息的检查

之后传递 Intent 的 EXTRA_ACCOUNT 和 EXTRA_CHECK_DIRECTION 参数，打开`Activity AccountSetupCheckSetting`进行配置信息的检查
```
public static void actionCheckSettings(Activity context, Account account,
            CheckDirection direction) {
    Intent i = new Intent(context, AccountSetupCheckSettings.class);
    i.putExtra(EXTRA_ACCOUNT, account.getUuid());
    i.putExtra(EXTRA_CHECK_DIRECTION, direction);
    context.startActivityForResult(i, ACTIVITY_REQUEST_CODE);
}
```
执行`execute()`开启一个新的 AsyncTask
```
new CheckAccountTask(mAccount).execute(mDirection);
```
清理完错误的认证信息后，执行 checkServerSettings()方法
```
clearCertificateErrorNotifications(direction);
checkServerSettings(direction);
setResult(RESULT_OK);
finish();
```
checkServerSettings()方法进行服务器的获取发送测试
```
switch (direction) {
    case INCOMING: {
        checkIncoming();
        break;
    }
    case OUTGOING: {
        checkOutgoing();
        break;
    }
}
```
`Activity AccountSetupBasic`通过 Intent 传递过来的`CheckDirection`参数是`INCOMING`,则进行获取邮件的测试`checkIncoming()`,`checkIncoming()`方法进行如下操作：
```
Store store = account.getRemoteStore()
publishProgress(R.string.account_setup_check_settings_check_incoming_msg);
store.checkSettings();
MessagingController.getInstance(getApplication()).listFoldersSynchronous(account, true, null);
MessagingController.getInstance(getApplication()).synchronizeMailbox(account, account.getInboxFolderName(), null, null);
```
1.先根据`account`获取远程 Store，如果 RemoteStore 不存在则异步创建一个 Pop3Store，该类包含着该账户所需的所有远程储存信息，同时 Activity 加载等待动画
2.调用 Pop3Store 的 checkSettings()方法配置 Pop3Store
```
Pop3Folder folder = new Pop3Folder(mStoreConfig.getInboxFolderName());
folder.open(Folder.OPEN_MODE_RW);
```
创建一个新的 Pop3Folder，用这个 Folder 的`open()`方法与邮件服务器通信以及发送其他命令

3.打开一个线程调用`MessagingController`的`listFoldersSynchronous()`方法,该方法用于从邮件服务器中下载所有 Folder，listFoldersSynchronous()`方法的主要流程如下：
```
Store localStore = account.getLocalStore();
localFolders = localStore.getPersonalNamespaces(false);
if (refreshRemote || localFolders.isEmpty()) {
    doRefreshRemote(account, listener);
    return;
}
```
首先根据`Account`获取`LocalStore`,根据`LocalStore`获取`PersonalNamespaces`如果远程目录有新邮件或者本地文件夹为空，则执行`doRefreshRemote()`方法，`doRefreshRemote()`方法的主要流程如下：
```
 for (Folder remoteFolder : remoteFolders) {
    if (localFolderNames.contains(remoteFolder.getName()) == false) {
        LocalFolder localFolder = localStore.getFolder(remoteFolder.getName());
        foldersToCreate.add(localFolder);
    }
    remoteFolderNames.add(remoteFolder.getName());
}
```
4.打开一个线程调用`MessagingController`的`synchronizeMailbox()方法，该方法用于从邮件服务器中同步收件箱，`synchronizeMailbox()方法在后台执行`synchronizeMailboxSynchronous()`方法，`synchronizeMailboxSynchronous()`方法通知监听器后调用`processPendingCommandsSynchronous()`方法先邮件服务器发送命令`getInboxFolderName()`获取账户文件夹的名字

## LocalStore 数据库
K9 Mail 在本地以 email 账号为单位，以 UUID 作为 SQLite 数据库名称，分别保存一个账号的文件夹（folders），邮件（messages），附件（attachment），邮件头信息（headers），提交到邮件服务器命令的栈（pending_commands），会话（邮件之间的关系 threads），不同的账户有着各自的数据库

### 打开数据库
LockableDataBase 类中定义了打开数据库的操作和避免读写出错的数据库锁，系统根据用户的 UUID 获取数据库文件的位置 databaseFile，从
而找到数据库打开数据库文件

```
final File databaseFile = storageManager.getDatabase(uUid, providerId);
private void doOpenOrCreateDb(final File databaseFile) {
        if (StorageManager.InternalStorageProvider.ID.equals(mStorageProviderId)) {
            // internal storage
            mDb = context.openOrCreateDatabase(databaseFile.getName(), Context.MODE_PRIVATE,
                    null);
        } else {
            // external storage
            mDb = SQLiteDatabase.openOrCreateDatabase(databaseFile, null);
        }
    }
```

### 数据库设计 Schema
在新建数据库过程中 LockableDataBase 类根据 StoreSchemaDefinition 的数据库结构建立数据库表，同时升级软件版本是也根据 Schema 重新建立数据库表，db.getVersion()=29的 Schema 结构如下：

```
db.execSQL("CREATE TABLE folders (id INTEGER PRIMARY KEY, name TEXT, "
       + "last_updated INTEGER, unread_count INTEGER, visible_limit INTEGER, status TEXT, "
       + "push_state TEXT, last_pushed INTEGER, flagged_count INTEGER default 0, "
       + "integrate INTEGER, top_group INTEGER, poll_class TEXT, push_class TEXT, display_class TEXT, notify_class TEXT"
       + ")");

db.execSQL("CREATE TABLE messages (" +
        "id INTEGER PRIMARY KEY, " +
        "deleted INTEGER default 0, " +
        "folder_id INTEGER, " +
        "uid TEXT, " +
        "subject TEXT, " +
        "date INTEGER, " +
        "flags TEXT, " +
        "sender_list TEXT, " +
        "to_list TEXT, " +
        "cc_list TEXT, " +
        "bcc_list TEXT, " +
        "reply_to_list TEXT, " +
        "attachment_count INTEGER, " +
        "internal_date INTEGER, " +
        "message_id TEXT, " +
        "preview TEXT, " +
        "mime_type TEXT, "+
        "normalized_subject_hash INTEGER, " +
        "empty INTEGER, " +
        "read INTEGER default 0, " +
        "flagged INTEGER default 0, " +
        "answered INTEGER default 0, " +
        "forwarded INTEGER default 0, " +
        "message_part_id INTEGER" +
        ")");
```

![数据库 scheme](http://7nar5o.com1.z0.glb.clouddn.com/scheme.png)
#### Folders 表
该表存放了当前账户所有的文件夹，该表以 ID 为主键，在 name 字段建立了索引，包括“文件夹设置”中的所有选项、显示（diaplay_class）、同步（poll_class）和推送（push_class）级别，k9mail 支持`NONE NO_CLASS INHERITED FIRST_CLASS`、同步的时间(last_pushed)，是否将该文件夹下的邮件整合到全局收件箱（integrate），以及标星数

字段 top_group 按照数字等级说明一个文件夹是否是“收件箱、垃圾箱、草稿箱、归档文件夹、发件箱、反垃圾邮件箱、已发送文件夹、错误文件夹”其中之一并且设置为置顶文件夹

#### Messages 表
该表保存了一份邮件除了附件（保存在 attachments 之中）及头信息（保存在 headers 之中）的所有信息，每封邮件都在表中对应了一条记录，字段 id 为主键，在表上建立了如下索引：
	1.在字段 uid 和 folder_id 上建立了索引 msg_uid
	2.在字段 folder_id、delect 和 internal_data 建立了索引 msg_folder_id_delect_data
	3.在字段 empty 上建立了索引 msg_empty
	4.在字段 read 上建立了索引 msg_read
	5.在字段 flagger 上建立了索引 msg_flagged

字段 flag 是邮件的标识与服务器端保持一致，标识的值包括：`DELETED SEEN FLAGGED ANSWERED FORWARDED`等，字段`deleted flagged answered forwarded`与字段 flag 的标识相对应，字段`read`则对应着标识`SEEN`，用于标识 flag 是否存在对应的标识，类型为 INTEGER，`0：存在；1：不存在`，默认值为0

字段 empty 表明邮件是否为空，`0:邮件非空；1：空`；attachment_account 表明一个邮件附件的总数目，对应`LocalMessage`类的属性`mAttachmentAccount`
#### 表 message_parts
表 message_parts 包括除了 message 表外的所有信息，在 schema version 29中 k9mail 将表 attachments 和表 header 整合到表 message_parts 之中

##### 表 attachments
该表以 id 字段为主键，储存着邮件的附件信息，通过字段`message_id`对应着响应的 Message，该表的每条字段关联着一个附件文件的链接，通过`content_url`从本地保存中获取到附件文件

字段 store_data 保存着字符串类型的 Android 邮件头信息；字段`content_disposition`保存着邮件头信息中`Content_Disposition`的值，通过该字段可以获取附件的相关数据

##### 表 header
该表以 id 为主键，保存着邮件的头信息，与 Attachment 相似，一封邮件可以在该表中存在多条记录，字段`name``value`以键值对 key-value 的方式保存着头信息中的所有字段，在字段`message_id`上建立了索引`header_folder`

#### 表 threads
该表以 id 为主键，为客户端提供了邮件会话支持的功能，按顺序保存着一个会话中邮件的关系，该表的索引有：
1.在字段`message_id`上建立了索引`threads_message_id`
2.在字段`root`上建立了索引`threads_root`
3.在字段`parent`上建立了索引`threads_parent`

k9 mail 使用了 LocalStore 内部类 ThreadInfo 作为 threads 的实体类，ThreadInfo 中的属性 threadId 对应着该表之中的 Id 字段

#### 表 pending_command
该表以 id 为主键，保存着提交到服务器的命令队列，其作用是保证这些命令不会丢失，防止客户端发生意外或意外退出时而导致命令丢失的情况发生，保证客户端与服务器端数据的一致性，字段 Argements 中保存着命令参数

### 触发器
SQLite 触发器可以在执行完一段特定的数据库更改后，触发完成后续的数据库操作，避免了开发专门的数据库代码
1.触发器 set_thread_root
该触发器 thread 表，在该表中新增一条计入后,对新增的的记录设置 root 字段的值为 id 的值
```
db.execSQL("CREATE TRIGGER set_thread_root " +
    "AFTER INSERT ON threads " +
    "BEGIN " +
    "UPDATE threads SET root=id WHERE root IS NULL AND ROWID = NEW.ROWID; " +
    "END");
```

2.触发器 delete_folder
该触发器作用于 message 表，在从 folder 表中删除记录前起作用，在从 folder 中删除记录前先将 message 表中属于待删除的文件夹的所有邮件自动删除
```
db.execSQL("CREATE TRIGGER delete_folder" +
	"BEFORE DELETE ON folders" +
	"BEGIN DELETE FROM messages WHERE old.id = folder_id;" +
	"END;");
```

3.触发器 delete_message
该触发器作用于 message 表和 hearder 表（message_parts 表），在从从 messages 表中删除记录前起作用，用于自动删除邮件的附件头信息
```
 db.execSQL("CREATE TRIGGER delete_message " +
    "BEFORE DELETE ON messages " +
    "BEGIN " +
    "DELETE FROM message_parts WHERE root = OLD.message_part_id;" +
    "END");
```
### 操作数据库
LocalStore 是唯一对外公开操作数据库的类，其内部包括9个内部类、1个接口，以邮件、文件夹为核心，提供了以面向对象方式围绕邮件和文件夹的一系列操作，包括向数据表插入新的数据，删除数据，更新数据库，获取数据等方法，k9 mail 为不同的内部类设计了基类，即把不同的实体封装成为不同的类，又减少了代码的重复。



