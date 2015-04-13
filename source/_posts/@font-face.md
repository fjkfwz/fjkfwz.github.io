title: 在网页中插入特殊字体,浅谈@font-face
date: 2014-7-26
categories: 前端设计
tags: [字体,前端]
description: 本篇文章包括（1）介绍@font-face（2）如何在网页中使用@font-face插入特殊字体（3）中文在线@font-face——有字库（4）修改hexo主题字体
---
**摘要**：*本篇文章包括（1）介绍@font-face（2）如何在网页中使用@font-face插入特殊字体（3）中文在线@font-face——有字库（4）修改hexo主题字体*
<!--more-->
自从在hexo上安装了pacmen主题之后，就一直对pacman主题`header`上的`textlogo`和`footer`上`intro_line`的字体很感兴趣，因为我的电脑和手机上并没有安装该字体，但是无论是在任何终端上，浏览的体验都是一致的，然后查找了一些资料，发现这是`CSS3`中的一个功能模块`@font-face`，主要用于实现网页字体多样性。

#@font-face功能

制作网站难免有些字体不是默认的，通过@font-face可以加载自己特定的字体，来实现特定的文字效果。
@font-face语句是css中的一个功能模块，用于实现网页字体多样性(设计者可随意指定字体，不需要考虑浏览者电脑上是否安装)。主要是把自己定义的Web字体嵌入到你的网页中，随着@font-face模块的出现，我们在Web的开发中使用字体不怕只能使用Web安全字体，
@font-face 不能说他是什么新东西了，在 CSS2.0 规范中就有了这玩意儿，IE4.0 开始就已经出现，只是当时用的不是特别广泛，后来在 CSS2.1 草案中又被删掉。随着 web 的急速发展，@font-face 价值越来越凸显，然后再次被纳入 CSS3 草案中。@font-face 是个什么东西，本文不做过多说明，不太清楚的童鞋可以看这里
>http://www.w3schools.com/css/css3_fonts.asp

#@font-face文件
而由于网页中使用的字体类型，也是各浏览器对字体类型有不同的支持规格。 字体格式类型主要有几个大分类：TrueType、Embedded Open Type 、OpenType、WOFF 、SVG。

TrueType

>Windows和Mac系统最常用的字体格式，其最大的特点就是它是由一种数学模式来进行定义的基于轮廓技术的字体，这使得它们比基于矢量的字体更容易处理，保证了屏幕与打印输出的一致性。同时，这类字体和矢量字体一样可以随意缩放、旋转而不必担心会出现锯齿。

EOT– Embedded Open Type (.eot)

>EOT是嵌入式字体，是微软开发的技术。允许OpenType字体用@font-face嵌入到网页并下载至浏览器渲染，存储在临时安装文件夹下。

OpenType(.otf)

>OpenType是微软和Adobe共同开发的字体，微软的IE浏览器全部采用这种字体。致力于替代TrueType字体。

WOFF–WebOpen Font Format (.woff)

>WOFF（Web开发字体格式）是一种专门为了Web而设计的字体格式标准，实际上是对于TrueType/OpenType等字体格式的封装，每个字体文件中含有字体以及针对字体的元数据（Metadata），字体文件被压缩，以便于网络传输。

SVG(Scalable Vector Graphics) Fonts (.svg)

>SVG是由W3C制定的开放标准的图形格式。SVG字体就是使用SVG技术来呈现字体，还有一种gzip压缩格式的SVG字体。

<table><tr><th>format 格式</th><th>Font 格式</th><th>后缀名</th></tr><tr><td>truetype</td><td>TrueType</td><td>.ttf</td></tr><tr><td>opentype</td><td>OpenType</td><td>.ttf, .oft</td></tr><tr><td>truetype-aat</td><td>TrueType with Apple Advanced Typography extensions</td><td>.ttf</td></tr><tr><td>embedded-opentype</td><td>Embedded OpenType</td><td>.eot</td></tr><tr><td>svg</td><td>SVG Font</td><td>.svg, .svgz</td></tr>
</table>

#@font-face声明字体
由于每种浏览器对@font-face的兼容性不同，不同的浏览器对字体的支持格式不同，这就意味着在@font-face中我们至少需要.woff,.eot两种格式字体，甚至还需要.svg等字体达到更多种浏览版本的支持。
<table><tr><th>浏览器</th><th>支持类型</th></tr><tr><td>IE6,7,8</td><td>仅支持 Embedded OpenType(.eot) 格式。</td></tr><tr><td>Firefox 3.5</td><td>支持 TrueType、OpenType(.ttf, .otf) 格式。</td></tr><tr><td>Firefox 3.6</td><td>支持 TrueType、OpenType(.ttf, .otf) 及 WOFF 格式。</td></tr><tr><td>Chrome,Safari,Opera</td><td>支持 TrueType、OpenType(.ttf, .otf) 及 SVG Font(.svg) 格式。</td></tr></table>

为了使@font-face达到更多的浏览器支持，Paul Irish写了一个独特的@font-face语法叫Bulletproof @font-face:

    @font-face {
	font-family: 'YourWebFontName';
	src: url('YourWebFontName.eot?') format('eot');/*IE*/
	src:url('YourWebFontName.woff') format('woff'), url('YourWebFontName.ttf') format('truetype');/*non-IE*/
    }

但为了让各多的浏览器支持，你也可以写成：

    @font-face {
	font-family: 'YourWebFontName';
	src: url('YourWebFontName.eot'); /* IE9 Compat Modes */
	src: url('YourWebFontName.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
             url('YourWebFontName.woff') format('woff'), /* Modern Browsers */
             url('YourWebFontName.ttf')  format('truetype'), /* Safari, Android, iOS */
             url('YourWebFontName.svg#YourWebFontName') format('svg'); /* Legacy iOS */
    }


#使用@font-face实现网页中插入特殊字体的过程
##获取特殊字体
这里我们要用到的是Single Malta字体
![Single Malta](http://picture-lotors.qiniudn.com/lotors%E6%8D%95%E8%8E%B7.JPG)
要得到Single Malta字体，不外乎两种途径，其一找到付费网站购买字体，其二就是到免费网站DownLoad字体。当然要给钱的这种傻事我想大家都不会做的，那我们就得到免费的地方下载，在哪有呢？我平时都是到Google Web Fonts和Dafont.com寻找自己需要的字体，当然网上也还有别的下载字体的地方，这个Demo使用的是Dafont.com的Single Malta字体，这样就可以到这里下载Single Malta：
>http://www.dafont.com/single-malta.font

##获取@font-face所需字体格式
由于不同浏览器支持的字体文件不同，我们需要想办法获得@font-face所需的.eot,.woff,.ttf,.svg字体格式，这里我们用到了一个网站fontsquirrel，可以方便的帮助我们实现字体格式的转换
>http://www.fontsquirrel.com/tools/webfont-generator

![fontsquirrel](http://picture-lotors.qiniudn.com/lotorstransttf.JPG)
进入网站后点击`Add Fonts`按钮，上传我们刚刚下载好的字体文件（要先把SingleMalta.ttf从.zip压缩包中解压出来），通常选择中间默认的`OPTIMAL`选项即可，然后勾选`Agreement`,点击`DOWNLOAD YOU KIT`，稍等片刻，即可下载我们所需要的字体文件。
解压压缩包包可以看到已经转换好的字体文件，如下图：
![](http://picture-lotors.qiniudn.com/lotorstransferfft.JPG)
大家可以看到，解压缩出来的文件格式，里面除了@font-face所需要的字体格式外，还带有一个DEMO文件，如果你不清楚的也可以参考下载下来的DEMO文件，我在这里不对DEMO说明问题，我主要是给大家介绍如何把下载下来的文件有价值的运用到我们的项目中。
##应用@font-face到我们的项目中
这里我们新建一个DOME项目，目录包括一个fonts文件夹，里面存放的转换好的字体文件；一个css文件夹，用于存放ccs文件；和一个index.html入口文件。目录和html如下
![](http://i1368.photobucket.com/albums/ag185/Mckee_Andy/index_zps500bc614.jpg)

style.css代码
```
   @font-face {
      font-family: 'SingleMaltaRegular';
      src: url('../fonts/singlemalta-webfont.eot');
      src: url('../fonts/singlemalta-webfont.eot?#iefix') format('embedded-opentype'),
           url('../fonts/singlemalta-webfont.woff') format('woff'),
       url('../fonts/singlemalta-webfont.ttf') format('truetype'),
       url('../fonts/singlemalta-webfont.svg#SingleMaltaRegular') format('svg');
      font-weight: normal;
      font-style: normal;
   }
      h1.SingleMalta{
     font-family: 'SingleMaltaRegular';
   }
   body{
    background-color: rgb(123,234,234);
   }
```
上传到服务器，效果如下
![](http://i1368.photobucket.com/albums/ag185/Mckee_Andy/fontresult_zpsb9fc4c31.jpg)
##更改hexo的textlog字体
pacman的字体资源是储存在pacman主题文件夹下`source/font`之中的，关于@font-face声明是储存在`source/css/_base/font.styl`配置文件之中，查看pacman的配置文件_config.yml找到字体选项Font通过注释可以了解到，页面的字体配置是保存在`source/css/_base/variable.styl`之中，如下图所示

![](http://picture-lotors.qiniudn.com/lotorsfont2.JPG)

首先将解压出来的.eot,.woff,.ttf,.svg字体文件放入`font`文件夹中，找到`source/css/_base/font.styl`配置文件，可以看到pacman已经帮我们建立了一个通用的声明，所以我们无需修改声明，只需要在`variable.styl`中修改页面字体配置即可，将`font-custom-family``font-custom-filename`改成我们的字体名和文件名即可
```
font-custom-family = "SingleMaltaRegular"（随意）
font-custom-filename = singlemalta-webfont
```
刷新页面即可看到效果，如下图：
![](http://picture-lotors.qiniudn.com/lotorsresult.JPG)
#中文@font-face
上面的这些@font-face转换网站都不支持中文字体的转换，中文字体文件相对于英文显得过于庞大，很长一段时间都被认为是不适合嵌入网页的。

直到几年前，这个问题终于被一个日本网站解决了，他用的技术就是截取法，在前端置入一个js脚本，脚本自动根据网页内容适时生成一个小字库（只包含当前网页内容的小字库）然后自动转换成.ttf、.eot、.woff、.svg等格式嵌入网页中，从页实现@font-face效果。体验和英文@font-face差不多，效果非常漂亮。但日文@font-face网站对于中文网页还无法支持。

如果是你想在你的网页上使用中文简体@font-face服务，也不是不可能，推荐一个中文@font-face网站——“有字库”。

使用时，只需要引用一段js脚本代码或者一段css代码，网站就会自动帮你截取网页需要的小字库并生成.ttf、.eot、.woff、.svg等格式文件，你可以将各文件下载下来，也可以托管在这个网站上，非常方便。去试试吧
##有字库测试
```
<html>
	<head>
	<title>font test</title>
    <meta http-equiv="content-type" charset="utf-8"  content="text/html"
	</head>
	<body>
	<h1 class="SiYuanRegular">中国文字之美</h1>
    <p class="SiYuanRegular">思源黑体测试</p>
	<h1 >中国文字之美</h1>
    <p >思源黑体测试</p>
	</body>	//有字库生成的JS脚本插入</body>和</html>之间
    <script type="text/javascript" src="http://www.youziku.com/UserDownFile/jquery.min.js"></script>
<script type="text/javascript" src="http://www.youziku.com/UserDownFile/jquery.md5.js"></script>
<script type="text/javascript"> 
    function youziku46827() {
        var resultStr = $(".SiYuanRegular").text();
        var md5 = "";
        resultStr = Trim(resultStr);
        resultStr = SelectWord(resultStr);
        md5 = $.md5("08d5ae7dc2e04646b67b8f08909cc995"+"SiYuanRegular" + resultStr);
        $.getJSON("http://www.youziku.com/webfont/CSSPOST?jsoncallback=?", { "id": md5, "guid": "08d5ae7dc2e04646b67b8f08909cc995", "type": "5" }, function (json) {
            if (json.result == 0) {/*alert("需要生成");*/
                $.post("http://www.youziku.com/webfont/PostCorsCreateFont", { "name": "SiYuanRegular", "gid": "08d5ae7dc2e04646b67b8f08909cc995", "type": "5", "text": resultStr }, function (json) {
                if (json == "0") { /*alert("参数不对");*/
                } else if (json == "2") {/*alert("超过每日生成字体数的上限");*/
                } else if (json == "3") { /*alert("当前正在生成请稍后");*/
                } else {/*alert("正在生成");*/
                }
            });
            }
            else {/*alert("下载css文件");*/
                loadExtentFile("http://www.youziku.com/webfont/css?id=" + md5 + "&guid=" + "08d5ae7dc2e04646b67b8f08909cc995" + "&type=5");
            }
        });
    }
    (function youziku() {
    if (window.location.href.toString().substring(0, 7) == "file://") {
            alert("你当前是通过双击打开html文件，进行本地测试的，这样看不到字体效果，一定要通过本地建立的虚拟网站或发布到外网进行测试。详见有字库的使用说明。");
        }else{
        youziku46827();
        }
    })()
</script>
</html>	
```
测试效果
![有字库测试](http://i1368.photobucket.com/albums/ag185/Mckee_Andy/fontcop_zpsea3b8452.jpg)
