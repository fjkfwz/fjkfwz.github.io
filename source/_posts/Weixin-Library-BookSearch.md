title: 微信公众平台图书馆查询开发记录
date: 2014-7-25
categories: 微信平台开发记录
tags: [微信公众平台,开发,笔记,PHP,编码,QRcode,验证码,模拟登入,图书馆]
description: 记录图书馆查询微信公众平台开发全过程，主要内容包括：（1）Fidder2 解析图书馆登入系统过程（2）模拟图书馆登入系统过程（3）机器识别验证码（4）Unicode 转 UTF-8编码过程（5）微信公众账号的绑定（6）Memcache 缓存技术(7)正则表达式
---
**摘要**：这篇文章记录图书馆查询微信公众平台开发全过程，主要内容包括：（1）Fidder2解析图书馆登入系统过程（2）模拟图书馆登入系统过程（3）机器识别验证码（4）Unicode 转 UTF-8 编码过程（5）微信公众账号的绑定（6）Memcache 缓存技术(7)正则表达式
<!--more-->
这是一篇记录我的第一个微信应用——福建农林大学图书馆服务的开发记录，应用架设在 Sina App Engine,开发目标是解决手机登入 web 查看图书不便的问题，并提供响应的服务功能，比如（1）图书5天内图书到期的提醒（2）图书的查询服务（3）图书借阅的查询服务（4）暂时图书馆没有藏书的到书提醒（5）免验证码登陆，算是对自己多天折腾的总结并提供有相关需求的朋友一些思路。

# Fidder2解析图书馆登入系统过程
## Fidder2介绍
>Fiddler 是一个 http 协议调试代理工具，它能够记录并检查所有你的电脑和互联网之间的 http 通讯，设置断点，查看所有的“进出”Fiddler 的数据（指 cookie,html,js,css 等文件，这些都可以让你胡乱修改的意思）。 Fiddler 要比其他的网络调试器要更加简单，因为它不仅仅暴露 http 通讯还提供了一个用户友好的格式。

利用 Fidder 可以方便的从数据连接中扒取链接的信息，即使是表单是通过_POST，我们也能够很方便的从数据流中截取用户发送的信息，从而将机器伪装成人肉对服务器进行请求，实现模拟登入。
## 通过 Fidder 查看图书馆登入过程
首先，打开 Fidder，开始对通信进行监听，登入我们的目标站点——福建农林大学图书馆的登入页面`http://210.34.85.114:8080/reader/login.php`,输入学号，密码和验证码（正不正确不重要，这里主要查看数据的发送格式），选择`证件号`，点击`登入`，打开 Fidder，选择验证服务器的 url`http://210.34.85.114:8080/reader/redr_verify.php`，在`Inspectors 下的 Raw 标签`下可以看到发送的信息，如下


	POST http://210.34.85.114:8080/reader/redr_verify.php HTTP/1.1
	Host: 210.34.85.114:8080
	User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3
	Accept-Encoding: gzip, deflate
	Referer: http://210.34.85.114:8080/reader/login.php
	Cookie: PHPSESSID=ordh88pv46g9jco23d32pvqer3
	Connection: keep-alive
	Content-Type: application/x-www-form-urlencoded
	Content-Length: 70

	number=3125002076&passwd=147qj9&captcha=1173&select=cert_no&returnUrl=

发送的目标地址

	POST http://210.34.85.114:8080/reader/redr_verify.php HTTP/1.1

用户进行浏览器的头文件为：
	
	User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0
	Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
	Accept-Language: zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3
	Accept-Encoding: gzip, deflate

用户发送的信息格式为：

	number=3125002076&passwd=147qj9&captcha=1173&select=cert_no&returnUrl=

后头我们将对这些信息进行封装，通过 curl POST 至目标地址`http://210.34.85.114:8080/reader/redr_verify.php HTTP/1.1`。
由于本文涉及到验证码的机器识别，所以我们还需要获取生成验证码的服务器地址，Fidder 中我们可以找到生成验证码的 url`http://210.34.85.114:8080/reader/captcha.php`，返回数据格式为`captcha.gif`

# 模拟图书馆登入系统过程

	
	//*-----------------------------------WXLibSearch.php--------------------------------------*//
	<?php
	include ('Valite.php');	//验证码识别类
	include ('u2utf82gb.php');	//NCR 转 GB2312类
	//获取用户键入的信息
	function getborrowedbookinfo ($keyword){

		/*
		*正则表达式检测用户输入信息是否为"学号（10位）+密码"
		*如果是则进行模拟登入获取借阅信息
		*如果输入错误则返回"请按照格式输入学号和密码"
		*/
	
		if (preg_match_all("/^(\d{10})\+\S+/",$keyword,$info")){
			
			//切割字符串，学号为前10位（0-9），密码为第11后（10-）
			$num=substr($info,0,9);	//学号
			$pass=substr($info,11);	//密码
			
			
			//获取验证码 url
			$url = "http://210.34.85.114:8080/reader/captcha.php";
			//保存 cookie 在当前路径下的"valid.tmp"文件之中，跨页面传递用户信息
			$cookie = dirname(__FILE__)."/valid.tmp";
			 
			//curl 访问验证码图片网址，把返回的 cookie 保存为 valid.tmp 文件
			$curl = curl_init($url);
			curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);	//设定返回 的数据是否自动显示
			curl_setopt($curl, CURLOPT_COOKIEFILE, $cookie);
			curl_setopt($curl, CURLOPT_COOKIEJAR, $cookie);	// 把返回来的 cookie 信息保存在$cookie 文件中
			$data = curl_exec($curl);
			curl_close($curl);
			 
			//保存验证码图片
			$fp = fopen("valid.gif","wb");
			fwrite($fp, $data);
			fclose($fp); 
			 
			//识别验证码图片
			$valid = new Valite();
			$valid->setImage("valid.gif");
			$valid->getHec();
			$validCode = $valid->run();
			
			
			//组装数据
			$data = "number=".$num."&passwd=".$pass."&captcha=".$validCode."&select=cert_no&returnUrl=";
			//头文件
			$headers = array(
				"User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0","Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
				"Accept-Language: zh-cn,zh;q=0.8,en-us;q=0.5,en;q=0.3",
				"Accept-Encoding: gzip, deflate",
				);
	
			//curl 发送数据进行登陆
			$url = "http://210.34.85.114:8080/reader/redr_verify.php";
			$curl = curl_init();
			curl_setopt($curl, CURLOPT_URL, $url);
			curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);	/设定是否显示头信息
			curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($curl, CURLOPT_COOKIEFILE, $cookie);
			curl_setopt($curl, CURLOPT_POST, 1);
			curl_setopt($curl,CURLOPT_POSTFIELDS,$data);
			$src = curl_exec($curl);
			curl_close($curl);
			
			//利用 cookie 进行借阅图书查询
			$url = "http://210.34.85.114:8080/reader/book_lst.php";
			$curl = curl_init();
			curl_setopt($curl, CURLOPT_URL, $url);
			curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
			curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
			curl_setopt($curl, CURLOPT_COOKIEFILE, $cookie);
			$src = curl_exec($curl);
			curl_close($curl);
			
			//截取出 html 中的书名
			
			$booknum = preg_match_all('/<a[^>]+>\S+<\/a>/',$src,$bookinfo);
			$hadborrowed = $booknum-23;	//当前借阅量
			$bookborrowed = "当前借阅量：$hadborrowed";
			$bookborrowed = iconv('UTF-8', 'GB2312', $bookborrowed);
			$books = "";	//建立容器储存书名
			$n = 0;	//用于为标题名前加上标号
			for ($i=23;$i<$booknum;$i++){
			$n = $n+1;	
			$str1 = "";	//建立容器用于储存转化为 GB2312的字符，每循环一次数据清空，储存到$books 容器之中
				//unicode 转化为 utf-8
				//通过 explode，标记为";"将字符串切割成单个 Uincode 编码字符
				$bookinf = explode(';',$bookinfo[0][$i]);
				foreach($bookinf as $bookinfx){	//遍历字符串字符进行转码
					$bookinfx = preg_replace('/<[^>]+>/','',$bookinfx);	//除去头尾字符的 html 标签
					$code = substr($bookinfx,3);	//除去16位 NCR 编码的表示部分"&# x"
					$unicode = hexdec($code);	//16进制转化我10进制
					$string = u2utf82gb($unicode);	//Unicode 转 GB2312
					$str1 = $str1.$string;	//将转码后的单个字符拼接起来
				}
				$books = $books.$n.".".$str1;	//将每个书名拼接起来并加上标号"$n."
			}
			
			echo $bookborrowed."\n".$books;	//输出借阅的图书量加上书名输出测试
		}
	}
			
	?>


## 机器识别验证码
验证码识别一般分为以下几个步骤：
一、取出字模
识别验证码，毕竟不是专业的 OCR 识别，并且，由于各个网站的验证码各不相同，所以，最常见的方法就是就是建立这个验证码的特征码库。去字模时，我们需要多下载几张图片，使这些图片中，包括所有的字符，我们这里的图片里只有数字，所以，只要收集到包括0-9的数字图片即可。

1、多刷新几次验证码，将验证码图片保存起来，要搜集齐0-9的图片，这里我收集了4张验证码才凑齐0~9 10位数字。

![验证码](http://picture-lotors.qiniudn.com/lotors%E9%AA%8C%E8%AF%81%E7%A0%812.JPG "验证码")

2、用图片处理软件打开图片，我用的是 Photoshop，按住 ctrl+可以将图片的视图放大，这样就能很清楚地观察到图片的每个像素。

![放大后的验证码](http://picture-lotors.qiniudn.com/lotors%E9%AA%8C%E8%AF%81%E7%A0%813.JPG "放大后的验证码")

可以发现，每个数字的宽是8px，高是10px，数字的间隔是4px，第一个数字左边偏移了6px，顶部偏移了16px。这些数字后面都是要用到的。
3、将每个数字截出来保存为图片，大小为8*10（虽然数字1的宽只占了7个像素，但是还是要截取8个像素的宽度）。

二、图片二值化
二值化就是把图片上的验证数字上每个象素用数字1表示，其它部分用0表示。把要识别的图片，进行二值化，将数据保存到二维数组里，得到图片特征数组。

1、首先要将数字和背景色和干扰色区分开来，用屏幕取色器观察颜色的规律。
可以得出一个结论：背景颜色的 R、G、B 值都是大于200的，而数字的颜色的 R、G、B 值的某一项有小于200，因此可以很容易区分。

2、下面的 php 代码只是为了演示二维数组，为了直观看出数字，所以把1和0改为了0和-：
```
echo '<br><img src="captcha.gif"><br><br>';
 
getHec("captcha.gif");	//"captcha.gif"中的值为8690
 
function getHec($imagePath) {
    $res = imagecreatefromjpeg($imagePath);
    $size = getimagesize($imagePath);
    
    for ($i = 0; $i < $size[1]; ++$i) {
        for ($j = 0; $j < $size[0]; ++$j) {
            $rgb = imagecolorat($res, $j, $i);
            $rgbarray = imagecolorsforindex($res, $rgb);
            if ($rgbarray['red'] < 200 || $rgbarray['green']<200 || $rgbarray['blue'] < 200) {
                echo "0";
            }else{
                echo "-";
            }
        }
        echo "<br>";
    }
} 
```

二值化的结果如下图所示：

```
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
--------0000--------0000--------0000---------00-------------
-------00--00------00--00------00--00-------0000------------
------00----00----00----0-----00----00-----00--00-----------
-------00--00-----00----------00----00----00----00----------
--------0000------00-000-------00--000----00----00----------
-------00--00-----000--00-------000-00----00----00----------
------00----00----00----00----------00----00----00----------
------00----00----00----00-----0----00-----00--00-----------
-------00--00------00--00------00--00-------0000------------
--------0000--------0000--------0000---------00-------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
```

如果图片的背景颜色比较复杂，处理方法也是一样的，总能找到临界值来区分，具体要靠自己观察了。

三、数字字模二值化
计算出每个数字字模的二值化的数据，记录下这些数据，当作 key 即可。

1、将0-9的数字字模图片进行二值化，逐个取出图片的像个像素的颜色，然后获取每个像素的 R、G、B 值，再进行判断，代码如下：
```
for ($i = 0; $i < 10; $i++) {
    echo "'$i'=>'";
    echo getHec("$i.jpg")."',<br>";
}
 
function getHec($imagePath) {
    $res = imagecreatefromjpeg($imagePath);
    $size = getimagesize($imagePath);
    
    for ($i = 0; $i < $size[1]; ++$i) {
        for ($j = 0; $j < $size[0]; ++$j) {
            $rgb = imagecolorat($res, $j, $i);
            $rgbarray = imagecolorsforindex($res, $rgb);
            if ($rgbarray['red'] < 200 || $rgbarray['green']<200 || $rgbarray['blue'] < 200) {
                echo "1";
            }else{
                echo "0";
            }
        }
    }
}
输出结果：
'0' => '011110100001100001100001100001100001100001100001100001011110',
'1' => '001000111000001000001000001000001000001000001000001000111110',
'2' => '011110100001100001000001000010000100001000010000110011111111',
'3' => '011110100001100001000010001100000010000001100001100001011110',
'4' => '000100000100001100010100100100100100111111000100001100001111',
'5' => '111111100000100000101110110001000001000001100001100001011110',
'6' => '001110010001100000100000101110110001100001100001100001011110',
'7' => '111111100010100010000100000100001000001000001000001000001000',
'8' => '011110100001100001100001011110010010100001100001100001011110',
'9' => '011100100010100001100001100011011101000001000001100010011100',
```

四、对照样本
把步骤二中的图片特征码和步骤三中的验证码的字模进行对比，得到验证图片上的数字。
算法过程：

1、将图片二值化后的值保存到二维数组里。

2、通过循环，求出每一个数字的位置，要用到前面得到的数字的宽、高、间隔、左边偏移、顶部偏移。
例如：第 i 个数字左边偏移 =（数字宽 + 间隔）* i + 左边偏移。

3、知道了数字的偏移位置，就可以计算出数字在二维数组里的位置，通过循环将数字的6*10=60个数据取出来拼接在一起，就形成了与数字字模类似的字符串。

4、将字符串与每一个字模的字符串比较，求其相似度，取最高的相似度对应的数字，或者相似度达到95%以上就可以断定是某个数字。

5、识别结果如下：

![识别结果](http://picture-lotors.qiniudn.com/lotors%E9%AA%8C%E8%AF%81%E7%A0%814.JPG "识别结果")

使用目前这种方法，对验证码的识别基本上可以做到100%。
通过以上步骤，您可能说了，并没有发现如何取出干扰素啊！其实取出干扰素的方法很简单，干扰素的一个重要特征是，不能影响验证码的显示效果，所以制作干扰素时它的 RGB 可能低于或者高于某个特定值，比如我给的例子中的图片，干扰素的 RGB 各项值是不会小于200的，所以，这样我们就很容易去掉干扰素了。
	
	//*-----------------------------------Valite.php--------------------------------------*//
	<?php
	
	define('WORD_WIDTH',8);	//定义每个字符的高度
	define('WORD_HIGHT',10);	//定义每个字的宽度
	define('OFFSET_X',6);	//左偏移的像素数
	define('OFFSET_Y',16);	//高偏移的像素数
	define('WORD_SPACING',4);	//字符间的像素数
	
	class valite
	{
		public function setImage($Image)	//打开二维码图片
		{
			$this->ImagePath = $Image;
		}
		
		public function getData()
		{
			return $data;
		}
		
		public function getResult()
		{
			return $DataArray;
		}
		
		public function getHec()	//除去干扰像素的方法
		{
			$res = imagecreatefromgif($this->ImagePath);		//创建图像
			$size = getimagesize($this->ImagePath);	//获取图像长宽
			$data = array();
			for($i = 0; $i < $size[1]; ++$i)	//循环遍历宽度
			{
				for($j = 0; $j < $size[0]; ++$j)	//循环遍历长度
				{
					$rgb = imagecolorat($res,$j,$i);	//取得某像素的颜色索引值
					$rgbarray = imagecolorsforindex($res, $rgb);	// 取出 red，green，blue 和 alpha 的键名的关联数组
					if($rgbarray['red'] < 160 || $rgbarray['green'] < 160 || $rgbarray['blue'] < 160)	//比较 RGB 值
					{
						$data[$i][$j]=1;	//数字输出1
					}else{
						$data[$i][$j]=0;	//背景输出0
					}
				}
			}
			$this->DataArray = $data;	//返回数据
			$this->ImageSize = $size;
		}
		
		public function run()	//获取每个像素点的坐标
		{
			$result = "";
			$data = array("","","","");
			
			for($i = 0; $i < 4; ++$i)	//循环,切割四个数字
			{
				$x = ($i * (WORD_WIDTH + WORD_SPACING)) + OFFSET_X;	//计算每个数字的起始水平像素的值 X
				$y = OFFSET_Y;
				for($h = $y; $h < (OFFSET_Y + WORD_HIGHT); ++$h)	//循环获取每个数字的竖直像素值
				{
					for($w = $x; $w < ($x + WORD_WIDTH); ++$w)	//循环获取水平像素值
					{
						$data[$i] .= $this->DataArray[$h][$w];	//储存每个数字的二值码
					}
				}
			}
	
	
			foreach($data as $numKey => $numString)	//循环每个数字的二值码
			{
				$max = 0.0;
				$num = 0;
				foreach($this->Keys as $key => $value)	//匹配每个的字模二值码
				{
					$percent = 0.0;	//初始相似百分比
					similar_text($value, $numString, $percent);	//计算两个字符串的相似度（以百分比计）
					if(intval($percent) > $max)	//当前相似度与前一次循环的相似度比较，如果比前一次相似度高则
					{
						$max = $percent;	//储存最大相似度百分比
						$num = $key;	//储存最大百分比的数字
						if(intval($percent) > 95)	//将字符串与每一个字模的字符串比较，求其相似度，取最高的相似度对应的数字，或者相似度达到95%以上就可以断定是某个数字
							break;
					}
				}
				$result .= $num;
			}
			$this->data = $result;
	
			return $result;
		}
	
		public function Draw()	//按顺序取出每个相熟点的值（0/1）
		{
			for($i = 0; $i < $this->ImageSize[1]; ++$i)
			{
		        for($j = 0; $j < $this->ImageSize[0]; ++$j)
			    {
				    echo $this->DataArray[$i][$j];
		        }
			    echo "\n";
			}
		}
		
		public function __construct()	//二值化的每个数字
		{
			$this->Keys = array(
				'0'=>'00011000001111000110011011000011110000111100001111000011011001100011110000011000',
				'1'=>'00011000001110000111100000011000000110000001100000011000000110000001100001111110',
				'2'=>'00111100011001101100001100000011000001100000110000011000001100000110000011111111',
				'3'=>'01111100110001100000001100000110000111000000011000000011000000111100011001111100',
				'4'=>'00000110000011100001111000110110011001101100011011111111000001100000011000000110',
				'5'=>'11111110110000001100000011011100111001100000001100000011110000110110011000111100',
				'6'=>'00111100011001101100001011000000110111001110011011000011110000110110011000111100',
				'7'=>'11111111000000110000001100000110000011000001100000110000011000001100000011000000',
				'8'=>'00111100011001101100001101100110001111000110011011000011110000110110011000111100',
				'9'=>'00111100011001101100001111000011011001110011101100000011010000110110011000111100',
			);
		}
		
		protected $ImagePath;
		protected $DataArray;
		protected $ImageSize;
		protected $data;
		protected $Keys;
		protected $NumStringArray;
	}
	?>

## NRC 转 UTF-8编码过程

UTF-8的编码规则很简单，只有二条： 

1）对于单字节的符号，字节的第一位设为0，后面7位为这个符号的 unicode 码。因此对于英语字母，UTF-8编码和 ASCII 码是相同的。 

2）对于 n 字节的符号（n>1），第一个字节的前 n 位都设为1，第 n+1位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的 unicode 码。 

下表总结了编码规则，字母 x 表示可用编码的位。
<table><tr><td>Unicode 符号范围(十六进制)</td><td>UTF-8编码方式（二进制）</td></tr>
<tr><td>0000 0000-0000 007F</td><td>0xxxxxxx</td></tr>
<tr><td>0000 0080-0000 07FF</td><td>110xxxxx 10xxxxxx</td></tr>
<tr><td>0000 0800-0000 FFFF</td><td>1110xxxx 10xxxxxx 10xxxxxx</td></tr>
<tr><td>0001 0000-0010 FFFF</td><td>11110xxx 10xxxxxx 10xxxxxx 10xxxxxx</td></tr></table>

下面，还是以汉字“严”为例，演示如何实现 UTF-8编码。 

已知“严”的 unicode 是4E25（100111000100101），根据上表，可以发现4E25处在第三行的范围内（0000 0800-0000 FFFF），因此“严”的 UTF-8编码需要三个字节，即格式是“1110xxxx 10xxxxxx 10xxxxxx”。然后，从“严”的最后一个二进制位开始，依次从后向前填入格式中的 x，多出的位补0。这样就得到了，“严”的 UTF-8编码是“11100100 10111000 10100101”，转换成十六进制就是 E4B8A5。 


	//*-----------------------------------u2utf82gb.php--------------------------------------*//
	<?php
	function u2utf82gb($c){
		$str="";
		if ($c < 0x80) {
			$str.=chr($c);	//当 NRC 编码位于0000 0000-0000 007F 区段时，UTF-8编码转换为0xxxxxxx
		} else if ($c < 0x800) {
			$str.=chr(0xC0 | $c>>6);	//当 NRC 编码位于0000 0080-0000 07FF 区段时，UTF-8编码转换为110xxxxx 10xxxxxx
	 		$str.=chr(0x80 | $c & 0x3F);
		} else if ($c < 0x10000) {
			$str.=chr(0xE0 | $c>>12);	//当 NRC 编码位于0000 0800-0000 FFFF 区段时，UTF-8编码转换为1110xxxx 10xxxxxx 10xxxxxx
			$str.=chr(0x80 | $c>>6 & 0x3F);
			$str.=chr(0x80 | $c & 0x3F);
		} else if ($c < 0x200000) {
			$str.=chr(0xF0 | $c>>18);	//当 NRC 编码位于0001 0000-0010 FFFF 区段时，UTF-8编码转换为11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
			$str.=chr(0x80 | $c>>12 & 0x3F);
			$str.=chr(0x80 | $c>>6 & 0x3F);
			$str.=chr(0x80 | $c & 0x3F);
		}
			return iconv('UTF-8', 'GB2312', $str);
	}
	?>
## 通过 QR 获取借阅信息
`<2014/7/25更新>`今天灵（nao）机（can）一动想到一个新想法，能否利用当前借阅最下边的二维码来获取图书的有关信息，于是用手机扫了一下二维码，发现真的可以获取图书的借阅信息。
![扫描结果](http://picture-lotors.qiniudn.com/lotorsQRcode.jpg "扫描结果")
于是查看了一下源码，找到了生成二维码的的地址。

```
  	<img src="../opac/ajax_qr.php?qrcode=6K%2B76ICF44CQMzEyNTAwMjA3NuOAkeeahOWcqOWAn%2BS5puWIiijpopjlkI0v5bqU6L%2BY5pel5pyfKe%2B8mg0KTHVh56iL5bqP6K6%2B6K6hIDIwMTQtMDktMTQgICAgICAgIA0KUEhQ5a6e5L6L57K%2B6YCaIDIwMTQtMDktMTQgICAgICAgIA0KQXJkdWlub%2BS4gOivleWwseS4iuaJiy4y54mILi4gMjAxNC0wOS0xNCAgICAgICAgDQrkuK3lm73lhpnmhI%2FnlLvlhaXpl6jovbvmnb7lraYs56u55a2QLi4gMjAxNC0wOS0xNCAgICAgICAgDQpBbmRyb2lk54Ot6Zeo5bqU55So5byA5Y%2BR6K%2Bm6KejLi4gMjAxNC0wOS0xNCAgICAgICAgDQrkuK3lm73lhpnmhI%2FnlLvlhaXpl6jovbvmnb7lraYs6I2J5pys6Iqx5Y2JLi4gMjAxNC0wOS0xNCAgICAgICAgDQrlkI3lrrblm73nlLvor77loIIu6Iqx6bif56%2BHLi4gMjAxNC0wOS0xNCAgICAgICAgDQpBcmR1aW5v5oqA5pyv5YaF5bmVIDIwMTQtMDktMTQgICAgICAgIA0K55av54uCQW5kcm9pZOiusuS5iS4y54mILi4gMjAxNC0wOS0xNCAgICAgICAgDQpqUXVlcnnlvIDlj5HmioDmnK%2For6bop6MuLiAyMDE0LTA5LTE0ICAgICAgICANCg%3D%3D" border="0" /></a></span>

```

这个二维码是由`http://210.34.85.114:8080/opac/ajax_qr.php`这个服务器生成的，后面的 qrcode 应该是 GB2312转 UTF-8后 url 编码产生，这里不去管它。
由于直接利用 PHP 无法解析二维码，所以调用`http://zxing.org/w/decode.jspx`这一个网页服务来对二维码解析。
用 Fidder 分析数据发送，该页面是通过_GET 方法`http://zxing.org/w/decode?u=`后接查询图片的 url 来传递数据，这里的 url 是`http://210.34.85.114:8080/opac/ajax_qr.php`这个服务器生成的二维码的地址。

```
GET http://zxing.org/w/decode?u=http%3A%2F%2F210.34.85.114%3A8080%2Fopac%2Fajax_qr.php%3Fqrcode%3D6K%252B76ICF44CQMzEyNTAwMjA3NuOAkeeahOWcqOWAn%252BS5puWIiijpopjlkI0v5bqU6L%252BY5pel5pyfKe%252B8mg0KTHVh56iL5bqP6K6%252B6K6hIDIwMTQtMDktMTQgICAgICAgIA0KUEhQ5a6e5L6L57K%252B6YCaIDIwMTQtMDktMTQgICAgICAgIA0KQXJkdWlub%252BS4gOivleWwseS4iuaJiy4y54mILi4gMjAxNC0wOS0xNCAgICAgICAgDQrkuK3lm73lhpnmhI%252FnlLvlhaXpl6jovbvmnb7lraYs56u55a2QLi4gMjAxNC0wOS0xNCAgICAgICAgDQpBbmRyb2lk54Ot6Zeo5bqU55So5byA5Y%252BR6K%252Bm6KejLi4gMjAxNC0wOS0xNCAgICAgICAgDQrkuK3lm73lhpnmhI%252FnlLvlhaXpl6jovbvmnb7lraYs6I2J5pys6Iqx5Y2JLi4gMjAxNC0wOS0xNCAgICAgICAgDQrlkI3lrrblm73nlLvor77loIIu6Iqx6bif56%252BHLi4gMjAxNC0wOS0xNCAgICAgICAgDQpBcmR1aW5v5oqA5pyv5YaF5bmVIDIwMTQtMDktMTQgICAgICAgIA0K55av54uCQW5kcm9pZOiusuS5iS4y54mILi4gMjAxNC0wOS0xNCAgICAgICAgDQpqUXVlcnnlvIDlj5HmioDmnK%252For6bop6MuLiAyMDE0LTA5LTE0ICAgICAgICANCg%253D%253D HTTP/1.1
Host: zxing.org
Connection: keep-alive
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1883.0 Safari/537.36
Referer: http://zxing.org/w/decode.jspx
Accept-Encoding: gzip,deflate,sdch
Accept-Language: en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4
```


```
//*-----------------------------------qrcode.php--------------------------------------*//
<?php
	function qrcode($src){	
		//切割 html 字符串获取图片 url	
		preg_match_all('/<img\ssrc=\"[^\"]+\"/',$src,$qr);
		//截取的""中间的 url
		preg_match_all('/\"[^\"]+/',$qr[0][0],$qrcode);
		//除去开头的 ..
		$qrcode = preg_replace('/\.\./','',$qrcode[0][0]);
		//除去开头的 "
		$qrcode = preg_replace('/\"/','',$qrcode);
		//拼接 url
		$qrcode = 'http://210.34.85.114:8080'.$qrcode;
		//url 编码
		$qrcode = urlencode($qrcode);
		//拼接查询 url
		$url = 'http://zxing.org/w/decode?u='.$qrcode;
		//获取数据
		$curl = curl_init($url);
		curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
		$infomation = curl_exec($curl);
		curl_close($curl);
		
		//获取 QRcode 信息
		preg_match_all('/读者[^<]+/',$infomation,$info);
		//UTF-8转 GB2312
		$info = iconv('UTF-8','GB2312',$info[0][0]);
		print_r($info);
	}
?>
```

