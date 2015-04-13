title: 微信公众平台实时监控室内温度和湿度
date:
categories: Arduino开发记录
tags: [微信公众平台,温度,湿度,传感器,DHT11,Arduino,乐联网,物联网]
description: 这是一个记录软硬件结合的物联网项目，目的是打造微信实时监控室内环境，由于传感器简陋，这篇文章只介绍温度和湿度的监控，使用的是DHT11温湿度传感器，主机为Arduino UNO+W5100 Ethernet/SD扩展版，路由器为HG255D Pandorabox by lintel,物联网服务由乐联网提供，本篇文章主要内容有（1）DHT11硬件基础（2）Arduino读取DHT11的温湿度数据（3）LCD1602_I2C硬件基础及在LCD模块上显示DHT11获取的温湿度数据（4）将数据上传到乐联网（5）微信查询传感器数据
---
**摘要**：*这是一个记录软硬件结合的物联网项目，目的是打造微信实时监控室内环境，由于传感器简陋，这篇文章只介绍温度和湿度的监控，使用的是DHT11温湿度传感器，主机为Arduino UNO+W5100 Ethernet/SD扩展版，路由器为HG255D Pandorabox by lintel,物联网服务由乐联网提供，本篇文章主要内容有（1）DHT11硬件基础（2）Arduino读取DHT11的温湿度数据（3）LCD1602_I2C硬件基础及在LCD模块上显示DHT11获取的温湿度数据（4）将数据上传到乐联网（5）微信查询传感器数据*
 <!--more-->
#DHT11硬件基础
DHT11是一款性价比极高的含有已校准数字信号输出的温湿度复合传感器，传感器包括一个电阻式感湿元件和一个NTC测温元件并与一个高性能8位单片机相连接。
##引脚说明
DHT11有四个引脚，一个引脚为空脚不连接元器件（所以一些采用DHT11传感器的封装好的温湿度传感器只能看到3个引脚），其中正极接VCC（工作电压为3－5.5VDC），负极接GND，由于DHT11是以串行接口以高低电平传递数据，所以数据总线（SDATA）引脚连接单片机或Arduino的数字信号（Digital）引脚，通常在元器件上正极标有（+），负极标有（—），数据引脚标有（S），我用的是KEYES的元器件，左边标有（S）为数据引脚，右边标有（-）为负极引脚，中间则为正极引脚，不同厂家的元器件引脚顺序不同，连接时要找准引脚，否者单片机则读取不到数据。
![引脚说明](http://picture-lotors.qiniudn.com/lotorspin.JPG)
![应用电路](http://picture-lotors.qiniudn.com/lotors%E5%BA%94%E7%94%A8%E7%94%B5%E8%B7%AF.JPG)

##数据传输原理
`MCU发送开始信号（+-+）->触发DHT响应（-+-）数据采集->{[准备发送（-）+发送数据（+）]*5}->机械拉低(-)->MCU拉高（+）`
	

DHT11采用的是串行接口(单线双向)的通讯方式，DATA用于微处理器与DHT11之间的通讯和同步,采用单总线数据格式,一次通讯时间4ms左右,数据分小数部分和整数部分,具体格式在下面说明,当前小数部分用于以后扩展,现读出为零.操作流程如下:一次完整的数据传输为40bit,高位先出。

###数据格式:
`40bit=8bit湿度整数数据+8bit湿度小数数据+8bi温度整数数据+8bit温度小数数据+8bit校验和数据`

传送正确时校验和数据等于“8bit湿度整数数据+8bit湿度小数数据+8bi温度整数数据+8bit温度小数数据”所得结果的末8位。

###数据传输过程
用户MCU发送一次开始信号后,DHT11从低功耗模式转换到高速模式,等待主机开始信号结束后,DHT11发送响应信号,送出40bit的数据,并触发一次信号采集,用户可选择读取部分数据.从模式下,DHT11接收到开始信号触发一次温湿度采集,如果没有接收到主机发送开始信号,DHT11不会主动进行温湿度采集.采集数据后转换到低速模式。
通讯过程如图所示
![通讯过程](http://picture-lotors.qiniudn.com/lotors%E9%80%9A%E8%AE%AF%E8%BF%87%E7%A8%8B.JPG)
总线空闲状态为高电平,主机把总线拉低等待DHT11响应,主机把总线拉低必须大于18毫秒,保证DHT11能检测到起始信号。DHT11接收到主机的开始信号后,等待主机开始信号结束,然后发送80us低电平响应信号.主机发送开始信号结束后,延时等待20-40us后, 读取DHT11的响应信号,主机发送开始信号后,可以切换到输入模式,或者输出高电平均可, 总线由上拉电阻拉高。

总线为低电平,说明DHT11发送响应信号,DHT11发送响应信号后,再把总线拉高80us,准备发送数据,每一bit数据都以50us低电平时隙开始,高电平的长短定了数据位是0还是1.格式见下面图示.如果读取响应信号为高电平,则DHT11没有响应,请检查线路是否连接正常.当最后一bit数据传送完毕后，DHT11拉低总线50us,随后总线由上拉电阻拉高进入空闲状态。
数字0信号表示方法如图所示
![数字0信号表示方法](http://picture-lotors.qiniudn.com/lotors0.JPG)
数字1信号表示方法如图所示
![数字1信号表示方法](http://picture-lotors.qiniudn.com/lotors1.JPG)

##Arduino读取DHT11的温湿度数据

	int DHpin=8;	//定义数字引脚8为DHT11数据接收/发送引脚
	byte dat[5];	//声明数组，用于储存5组8bit的数据

	void setup()	//只执行一次
	{
	  Serial.begin(9600);	//串口通信波特率（和电脑）
	  pinMode(DHpin,OUTPUT);	//输出模式
	  digitalWrite(DHpin,HIGH);	//静息电平，空闲状态
	}
	
	void start_test()	//MCU发送开始信号
	{
	  digitalWrite(DHpin,LOW);	//拉低电平发送信号
	  delay(30);
	  digitalWrite(DHpin,HIGH);	//拉高电平，发送信号结束
	  delayMicroseconds(40);
	  pinMode(DHpin,INPUT);	//输入模式，开始检测DHT发来的信号
	  while(digitalRead(DHpin)==LOW);	//DHT发来响应信号，低电平80us
	  delayMicroseconds(80);
	  while(digitalRead(DHpin)==HIGH);	//高电平80us，响应信号结束
	  delayMicroseconds(80);
	  for(int i=0;i<4;i++)	//开始读取数据
	  dat[i]=read_data();
	  pinMode(DHpin,OUTPUT);	//读取数据结束
	  digitalWrite(DHpin,HIGH);	//拉高电平，空闲状态
	}
	
	byte read_data()	//读取数据的方法
	{
	  byte data;
	  for(int i=0;i<8;i++)	//1bit的读取数据
	  {
	    if(digitalRead(DHpin)==HIGH)	
	    {
	      while(digitalRead(DHpin)==HIGH);	//检测到高电平开始,延迟30us
	      delayMicroseconds(30);
	      if(digitalRead(DHpin)==HIGH)	//如果30us后,电平拉高，则为1，否者则为0
	      data|=(1<<(7-i));
	    }
	  }
	return data;
	}
	
	void loop()	//串口输出数据
	{
	  start_test(); 
	  Serial.print("t1:"); 
	  Serial.print(dat[0],DEC); //显示湿度的整数位； 
	  Serial.print('.'); 
	  Serial.print(dat[1],DEC); //显示湿度的小数位； 
	  Serial.print('%');
	  Serial.print("t2:"); 
	  Serial.print(dat[2],DEC); //显示温度的整数位； 
	  Serial.print('.'); 
	  Serial.print(dat[3],DEC); //显示温度的小数位； 
	  Serial.print('C');
	  delay（5000）;		//延迟5s
	
	}

#LCD1602_I2C硬件基础
LCD1602_I2C由两部分组成，LCD1602和一个I2C并行串口电路构成，LCD提供液晶屏显示，I2C并行串口电路为单片机提供并行串口通讯，减少LCD1602所需的引脚。
##LCD1602硬件基础
LCD：英文全称为Liquid Crystal Display，即为液态晶体显示，也就是我们常说的液晶显示了。1602则是表示这个液晶一共能显示2行数据，每一行显示16个字符。这个就是LCD1602的全部来由。

我们首先来看1602的引脚定义,1602的引脚是很整齐的SIP单列直插封装，所以器件手册只给出了引脚的功能数据表：
![LCD1602引脚定义](http://picture-lotors.qiniudn.com/1602%E5%BC%95%E8%84%9A.JPG)
 
我们只需要关注以下几个管脚：
3脚：VL，液晶显示偏压信号，用于调整LCD1602的显示对比度，一般会外接电位器用以调整偏压信号，注意此脚电压为0时可以得到最强的对比度。
4脚：RS，数据/命令选择端，当此脚为高电平时，可以对1602进行数据字节的传输操作，而为低电平时，则是进行命令字节的传输操作。命令字节，即是用来对LCD1602的一些工作方式作设置的字节；数据字节，即使用以在1602上显示的字节。这是一个选择数据寄存器还是选择指令寄存器的过程，值得一提的是，LCD1602的数据是8位的。
5脚：R/W，读写选择端。当此脚为高电平可对LCD1602进行读数据操作，反之进行写数据操作。笔者认为，此脚其实用处不大，直接接地永久置为低电平也不会影响其正常工作。但是尚未经过复杂系统验证，保留此意见。
6脚：E，使能信号，其实是LCD1602的数据控制时钟信号，利用该信号的上升沿实现对LCD1602的数据传输。
7~14脚：8位并行数据口，使得对LCD1602的数据读写大为方便。


###LCD1602通讯过程
现在来看LCD1602的操作时序：

在此，我们可以先不读出它的数据的状态或者数据本身。所以只需要看两个写时序：

当我们要写入指令字节，设置LCD1602的工作方式时：需要把RS置为低电平，RW置为低电平，然后将数据送到数据口D0~D7，最后E引脚一个高脉冲将数据写入。
当我们要写入数据字节，设置LCD1602的工作方式时：需要把RS置为高电平，RW置为低电平，然后将数据送到数据口D0~D7，最后E引脚一个高脉冲将数据写入。
发现了么，写指令和写数据，差别仅仅在于RS的电平不一样而已。以下是LCD1602的读时序图：

![读指令](http://picture-lotors.qiniudn.com/1602%E8%AF%BB.JPG)

当要写命令字节的时候，时间由左往右，RS变为低电平，R/W变为低电平，注意看是RS的状态先变化完成。然后这时，DB0~DB7上数据进入有效阶段，接着E引脚有一个整脉冲的跳变，接着要维持时间最小值为tpw=400ns的E脉冲宽度。然后E引脚负跳变，RS电平变化，R/W电平变化。这样便是一个完整的LCD1602写命令的时序。
![读写指令](http://picture-lotors.qiniudn.com/%E5%9F%BA%E6%9C%AC%E6%97%B6%E5%BA%8F.JPG)

###LCD1602基本指令
![基本指令](http://picture-lotors.qiniudn.com/%E6%8C%87%E4%BB%A4.jpg)
***
###Arduino与LCD1602的通讯
8针接法，数据输出口：DB0-DB7。
```
// 8pinlcd1602.ino
int DI = 12;	//定义数据/命令针脚
int RW = 11;	//定义读/写针脚
int DB[] = {3, 4, 5, 6, 7, 8, 9, 10};//使用数组来定义总线需要的管脚
int Enable = 2;	//定义使能管脚
 
//LCD写命令的方法
void LcdCommandWrite(int value) {
 // 定义所有引脚
 int i = 0;
digitalWrite(DI, LOW);	//DI为O，RW为0，写入命令
 digitalWrite(RW, LOW);
 for (i=DB[0]; i <= DI; i++) //总线赋值
{
   digitalWrite(i,value & 01);	//获取每个字节的第一位
   value >>= 1;	//2进制移位，将要获取的位移至第一位
 }
 digitalWrite(Enable,LOW);	//使能拉低电平，准备触发高脉冲
 delayMicroseconds(1);
 digitalWrite(Enable,HIGH);	//Arduino拉高电平，高脉冲将数据泵入指令寄存器
 delayMicroseconds(1);  
 digitalWrite(Enable,LOW);	//使能恢复低电平，静息电平
 delayMicroseconds(1);  
}
 
void LcdDataWrite(int value) {
 // 定义所有引脚
 int i = 0;
 digitalWrite(DI, HIGH);	//DI为1，RW为0，写入数据
 digitalWrite(RW, LOW);
 for (i=DB[0]; i <= DB[7]; i++) {
   digitalWrite(i,value & 01);	//获取每个字节的第一位，0xXX & 01 =0 | 1
   value >>= 1;
 }
 digitalWrite(Enable,LOW);
 delayMicroseconds(1);
 digitalWrite(Enable,HIGH);	//高脉冲泵入数据，写入数据寄存器
 delayMicroseconds(1);
 digitalWrite(Enable,LOW);
 delayMicroseconds(1);  
}
 
void setup (void) {	//起始函数，只执行一次
 int i = 0;
 for (i=Enable; i <= DI; i++) {	//将所有管脚设置为输出
   pinMode(i,OUTPUT);
 }
 delay(100);
 // 短暂的停顿后初始化LCD
 // 用于LCD控制需要
 LcdCommandWrite(0x38);  // 设置为8-bit接口，2行显示，5x7文字大小                     
 delay(64);                      
 LcdCommandWrite(0x38);  // 设置为8-bit接口，2行显示，5x7文字大小                        
 delay(50);                      
 LcdCommandWrite(0x38);  // 设置为8-bit接口，2行显示，5x7文字大小                        
 delay(20);                      
 LcdCommandWrite(0x06);  // 输入方式设定
                         // 自动增量，没有显示移位
 delay(20);                      
 LcdCommandWrite(0x0E);  // 显示设置
                         // 开启显示屏，光标显示，无闪烁
 delay(20);                      
 LcdCommandWrite(0x01);  // 屏幕清空，光标位置归零  
 delay(100);                      
 LcdCommandWrite(0x80);  // 显示设置
                         // 开启显示屏，光标显示，无闪烁
 delay(20);                      
}
 
void loop (void) {
  LcdCommandWrite(0x01);  // 屏幕清空，光标位置归零  
  delay(10); 
  LcdCommandWrite(0x80+3); //0x80+为第一行命令地址，0x80+3表示第1行第4列的字符
  delay(10);                     
  // 写入欢迎信息 
  LcdDataWrite('W');
  LcdDataWrite('e');
  LcdDataWrite('l');
  LcdDataWrite('c');
  LcdDataWrite('o');
  LcdDataWrite('m');
  LcdDataWrite('e');
  LcdDataWrite(' ');
  LcdDataWrite('t');
  LcdDataWrite('o');
  delay(10);
  LcdCommandWrite(0xc0+1);  // 定义光标位置为第二行第二个位置  
  delay(10); 
  LcdDataWrite('M');
  LcdDataWrite('a');
  LcdDataWrite('k');
  LcdDataWrite('e');
  LcdDataWrite('B');
  LcdDataWrite('l');
  LcdDataWrite('a');
  LcdDataWrite('z');
  LcdDataWrite('e');
  delay(5000);
  LcdCommandWrite(0x01);  // 屏幕清空，光标位置归零  
  delay(10);
  LcdDataWrite('I');
  LcdDataWrite(' ');
  LcdDataWrite('a');
  LcdDataWrite('m');
  LcdDataWrite(' ');
  LcdDataWrite('R');
  LcdDataWrite('i');
  LcdDataWrite('c');
  LcdDataWrite('e');
  LcdDataWrite('L');
  LcdDataWrite('y');
  LcdDataWrite('n');
  delay(3000);
  LcdCommandWrite(0x02); //设置模式为新文字替换老文字，无新文字的地方显示不变。
  delay(10);
  LcdCommandWrite(0x80+5); //定义光标位置为第一行第六个位置
  delay(10);  
  LcdDataWrite('t');
  LcdDataWrite('h');
  LcdDataWrite('e');
  LcdDataWrite(' ');
  LcdDataWrite('a');
  LcdDataWrite('d');
  LcdDataWrite('m');
  LcdDataWrite('i');
  LcdDataWrite('n');
  delay(5000);
}
```
4针接法，数据输出口：DB4-DB7，传输一个字节分两次传输，每次传输4bit，需要2倍的时间。
```
// 4pinlcd1602.ino

int LCD1602_RS=12;   
int LCD1602_RW=11;   
int LCD1602_EN=10;   
int DB[] = { 6, 7, 8, 9};	//定义4位总线
char str1[]="Welcome to";	//自定义字符，数据格式为字符（一个数组的值等于一个字符）
char str2[]="MakeBlaze";
char str3[]="this is the";
char str4[]="4-bit interface";
 
void LCD_Command_Write(int command)	//LCD写命令函数
{
 int i,temp;
 digitalWrite( LCD1602_RS,LOW);
 digitalWrite( LCD1602_RW,LOW);
 digitalWrite( LCD1602_EN,LOW);
 
 temp=command & 0xf0;	//切割字节，获取高4位，0xff=11110000
 for (i=DB[0]; i <= 9; i++)
 {
   digitalWrite(i,temp & 0x80);	//获取最高位，0x80=10000000
   temp <<= 1;
 }
 
 digitalWrite( LCD1602_EN,HIGH);	//高脉冲泵入指令寄存器，位于栈底，D7位为0
 delayMicroseconds(1);
 digitalWrite( LCD1602_EN,LOW);
 
 temp=(command & 0x0f)<<4;	////切割字节，获取低4位，0xff=11110000
 for (i=DB[0]; i <= 9; i++)
 {
   digitalWrite(i,temp & 0x80);	//获取最高位，0x80=10000000
   temp <<= 1;
 }
 
 digitalWrite( LCD1602_EN,HIGH);	//高脉冲泵入指令寄存器，位于栈顶
 delayMicroseconds(1); 
 digitalWrite( LCD1602_EN,LOW);
}
 
void LCD_Data_Write(int dat)	//LCD写数据的函数
{
 int i=0,temp;
 digitalWrite( LCD1602_RS,HIGH);
 digitalWrite( LCD1602_RW,LOW);
 digitalWrite( LCD1602_EN,LOW);
 
 temp=dat & 0xf0;
 for (i=DB[0]; i <= 9; i++)
 {
   digitalWrite(i,temp & 0x80);
   temp <<= 1;
 }
 
 digitalWrite( LCD1602_EN,HIGH);
 delayMicroseconds(1);
 digitalWrite( LCD1602_EN,LOW);
 
 temp=(dat & 0x0f)<<4;
 for (i=DB[0]; i <= 9; i++)
 {
   digitalWrite(i,temp & 0x80);
   temp <<= 1;
 }
 
 digitalWrite( LCD1602_EN,HIGH);
 delayMicroseconds(1); 
 digitalWrite( LCD1602_EN,LOW);
}
 
void LCD_SET_XY( int x, int y )	//设置字符位置
{
  int address;
  if (y ==0)    address = 0x80 + x;	//如果是第一行，则地址设置为0x80+
  else          address = 0xC0 + x;	//如果是第二行，则地址设置为0xC0+
  LCD_Command_Write(address); //写入地址寄存器
}
 
void LCD_Write_Char( int x,int y,int dat)	//设置LCD显示数据
{
  LCD_SET_XY( x, y ); 	//设置LCD位置
  LCD_Data_Write(dat);	//将字模对应的地址编号写入数据寄存器
}
 
void LCD_Write_String(int X,int Y,char *s)
{
    LCD_SET_XY( X, Y );    //设置地址 
    while (*s)             //写字符串
    {
      LCD_Data_Write(*s);   
      s ++;
    }
}
 
void setup (void) 
{
  int i = 0;	//设置所有管脚为输出
  for (i=6; i <= 12; i++) 
   {
     pinMode(i,OUTPUT);
   }
  delay(100);
  LCD_Command_Write(0x28);	//4线 2行 5x7，8线我0x38
  delay(50); 
  LCD_Command_Write(0x06);	//写一个字符后地址指针加1，光标向后移一位
  delay(50); 
  LCD_Command_Write(0x0c);	//打开显示，不显示光标
  delay(50); 
  LCD_Command_Write(0x80);	// 显示设置
                         	// 开启显示屏，光标显示，无闪烁
  delay(50); 
  LCD_Command_Write(0x01);	//显示清屏
  delay(50); 
 
}
 
void loop (void)
{
   LCD_Command_Write(0x01);
   delay(50);
   LCD_Write_String(3,0,str1);//第1行，第4个地址起
   delay(50);
   LCD_Write_String(1,1,str2);//第2行，第2个地址起
   delay(5000);
   LCD_Command_Write(0x01);
   delay(50);
   LCD_Write_String(0,0,str3);
   delay(50);
   LCD_Write_String(0,1,str4);
   delay(5000);
 
}
```
***
###LCD1602内部构造
![LCD1602内部构造](http://picture-lotors.qiniudn.com/lcd1602.JPG)
LCD1602模块内部由一块LCD显示屏（LCDpanel），控制器（controller）,列驱动器（segment driver）和偏压产生电路构成。
控制器接收来自MPU（这里为Arduino）的指令和数据，控制着整个模块的运行，控制器主要指令寄存器(IR),数据寄存器（DR），忙标志（BF），地址计数器（AC），显示数据寄存器（DDRAM），字符发生器（CGROM（内置字符库），CGRAM（自定义字符库））构成。

Arduino传输过来的指令储存在指令寄存器之中，内部存储着DDRAM和CGRAM中数据显示的指令代码或地址信息，然后主控芯片（SoC）读取数据，储存DDRAM中根据指令代码执行具体的命令，如字符的位置，光标的开关，字符和光标的移位等等。
而Arduino传输过来的数据则储存在数据寄存器之中，从CGROM中查找到想要显示的字符的字符码，送入DDRAM之中，在LCD显示屏上与DDRAM存储单元对应的规定位置显示出该字符。

主控芯片根据DDRAM中储存的信息，从数字引脚中发出40SEG的扫描信号外加将信息传输到Segment Driver，由Segment Driver构成的40SEG扫描信号，构成LCD显示屏的列，行由公共电极（ROM），原理类似扫描键盘，从而控制整个LCD显示屏的输出。
***
##I2C并口扩展电路
我这里用的是一颗PCF8574T芯片，其他型号的芯片应该工作原理相同，它通过两条双向总线（I2C）可以使大多数的MCU（这里是Arduino）实现远程I/O口扩展，当然要以牺牲部分性能为代价，不过这里我们传递的数据量小，所以性能的丢失可以忽略不计，该器件包含一个8位准双向口（这里用于连接LCD1602，采用4位接法，加上RS,RW,E共占用7个I/O，当然也可以使用I/O口更多的芯片实现8位接法）和一个I2C总线接口（这里连接Arduino，占用2个数字引脚）。
![PCF8574T针脚定义](http://picture-lotors.qiniudn.com/i2c%E7%AE%A1%E8%84%9A.JPG)
###I2C通讯过程
I2C总线由两条线组成，一条串行数据中线（SDA）和一条串行时钟总线（SCL），当总线空闲时，SDA和SCL均保持高电平，这时如果，SDA电平由高电平变化为低电平，标志着起始信号的开始，随后SCL电平由高变低开始位传输，SDA开始传递数据（电平拉高或拉低），SCL随后由低变高，并保持一段时间，若SDA线上的电平保持稳定，则认为SDA是在传输数据bit，此时SDA上高电平表示1，低电平表示0，SLC由高变低，改变数据，准备检测下一个bit，连续传递8个bit（7个数据位加一个R/W操作位1bit），等待PCF8574T应答，第9个clock，若从IC发ACK，SDA会被拉低，若没有ACK，SDA会被置高，这会引起Master发生RESTART或STOP流程，一个字节的写入完成。电平由低至高变化定义为总线的停止信号。
![I2C时序图](http://picture-lotors.qiniudn.com/I2C%E6%97%B6%E5%BA%8F%E5%9B%BE.JPG)
###PCF8574T内部结构
![](http://picture-lotors.qiniudn.com/I2c%E5%8A%9F%E8%83%BD.JPG)
由MCU传递过来的数据存储在每个引脚的寄存器中，然后写入LCD1602。完成通过I2C控制LCD1602的过程。
##I2C驱动
```
********************************************************************/
#include <reg51.h>
#include <intrins.h>
#define uchar unsigned char /*宏定义*/
#define uint unsigned int
#define _Nop() _nop_()        /*定义空指令*/

sbit SDA=P3^4;            /*模拟I2C数据传送位*/
sbit SCL=P3^5;            /*模拟I2C时钟控制位*/
bit ack;          /*应答标志位*/
/*******************************************************************
                     起动总线函数               
函数原型: void Start_I2c(); 
功能:     启动I2C总线,即发送I2C起始条件. 
********************************************************************/
void Start_I2c()
{
SDA=1;   /*发送起始条件的数据信号*/
_Nop();
SCL=1;
_Nop();    /*起始条件建立时间大于4.7us,延时*/
_Nop();
_Nop();
_Nop();
_Nop();     SDA=0;   /*发送起始信号*/
_Nop();    /* 起始条件锁定时间大于4μs*/
_Nop();
_Nop();
_Nop();
_Nop();       
SCL=0;   /*钳住I2C总线，准备发送或接收数据 */
_Nop();
_Nop();
}
/*******************************************************************
                      结束总线函数               
函数原型: void Stop_I2c(); 
功能:     结束I2C总线,即发送I2C结束条件. 
********************************************************************/
void Stop_I2c()
{
SDA=0; /*发送结束条件的数据信号*/
_Nop();   /*发送结束条件的时钟信号*/
SCL=1; /*结束条件建立时间大于4μs*/
_Nop();
_Nop();
_Nop();
_Nop();
_Nop();
SDA=1; /*发送I2C总线结束信号*/
_Nop();
_Nop();
_Nop();
_Nop();
}
/*******************************************************************
                 字节数据发送函数               
函数原型: void SendByte(uchar c);
功能:     将数据c发送出去,可以是地址,也可以是数据,发完后等待应答,并

对
          此状态位进行操作.(不应答或非应答都使ack=0)     
        发送数据正常，ack=1; ack=0表示被控器无应答或损坏。
********************************************************************/
void SendByte(uchar c) {
uchar BitCnt;

for(BitCnt=0;BitCnt<8;BitCnt++) /*要传送的数据长度为8位*/
    {
     if((c<<BitCnt)&0x80)SDA=1;   /*判断发送位*/
       else SDA=0;                
     _Nop();
     SCL=1;               /*置时钟线为高，通知被控器开始接收数据位*/
      _Nop(); 
      _Nop();             /*保证时钟高电平周期大于4μs*/
      _Nop();
      _Nop();
      _Nop();         
     SCL=0; 
    }
    
    _Nop();
    _Nop();
    SDA=1;                /*8位发送完后释放数据线，准备接收应答位*/
    _Nop();
    _Nop();   
    SCL=1;
    _Nop();
    _Nop();
    _Nop();
    if(SDA==1)ack=0;     
       else ack=1;        /*判断是否接收到应答信号*/
    SCL=0;
    _Nop();
    _Nop();
}
/*******************************************************************
                 字节数据接收函数               
函数原型: uchar RcvByte();
功能:    用来接收从器件传来的数据,并判断总线错误(不发应答信号)，
          发完后请用应答函数应答从机。 
********************************************************************/ 
uchar RcvByte()
{
uchar retc;
uchar BitCnt; 
retc=0; 
SDA=1;                /*置数据线为输入方式*/
for(BitCnt=0;BitCnt<8;BitCnt++)
      {
        _Nop();           
        SCL=0;                  /*置时钟线为低，准备接收数据位*/
        _Nop();
        _Nop();                 /*时钟低电平周期大于4.7μs*/
        _Nop();
        _Nop();
        _Nop();
        SCL=1;                  /*置时钟线为高使数据线上数据有效*/
        _Nop();
        _Nop();
        retc=retc<<1;
        if(SDA==1)retc=retc+1; /*读数据位,接收的数据位放入retc中 */
        _Nop();
        _Nop(); 
      }
SCL=0;    
_Nop();
_Nop();
return(retc);
}
/********************************************************************
                     应答子函数
函数原型: void Ack_I2c(bit a);
功能:      主控器进行应答信号(可以是应答或非应答信号，由位参数a决定)
********************************************************************/
void Ack_I2c(bit a)
{

if(a==0)SDA=0;           /*在此发出应答或非应答信号 */
        else SDA=1;
_Nop();
_Nop();
_Nop();      
SCL=1;
_Nop();
_Nop();                    /*时钟低电平周期大于4μs*/
_Nop(); _Nop();
_Nop(); 
SCL=0;                     /*清时钟线，钳住I2C总线以便继续接收*/
_Nop();
_Nop();    
}
/******************************************************************* 
```
##I2C&LCD连接电路
![I2C&LCD连接电路](http://picture-lotors.qiniudn.com/i2c%E7%94%B5%E8%B7%AF.JPG)
