title: "Material Design设计参考"
date: 2014-10-16
category: Material Design
tags: [Material Design]
---
**摘要**：*Google I/O 2014 发布的 Material Design 势必将会成为统一 Android Mobile、Android Table、Desktop Chrome 等全平台设计语言规范，对从业人员意义重大，我们已经通过互联网的方式将其翻译成中文，希望能帮到大家！*
<!--more-->


# 真实的动作（Authentic Motion）

感知一个物体有形的部分可以帮助我们理解如何去控制它。观察一个物理的运动可以告诉我们它轻还是重，柔性还是刚性，小还是大。在`material design`设计规范中，动作不止是呈现着它美丽的一面，它还意味着在空间中的关系、功能以及在整个系统中的趋势。


## 体积和重量（Mass and Weight）

物理世界中物体拥有质量，所以只有当施加给它们力量的时候才会移动，因此，物体没法在瞬间开始或者结束动作。动画突然开始或者停止，或者在运动时突兀的变化方向，都会使用户感到意外和不和谐的干扰。

### 最佳实践（Best Practices）

`material design`规范中，一个重要方面在于如何在“动作”完整的展现物体的各个真实的特性，譬如优雅、简约、美观和神奇的无缝的用户体验，下面的动画将帮助大家理解这些理念。

<video crossorigin="anonymous"   loop controls width="740" height="270">
<source src="http://materialdesign.qiniudn.com/videos/animation-authentic-motion-authenticMotion_massAndWeight_ex1_large_xhdpi.webm" type="video/webm">
</video>

要：迅速的加速和平滑的减速会感到自然和愉快

<video crossorigin="anonymous"  loop  controls width="740" height="270">
<source src="http://materialdesign.qiniudn.com/videos/animation-authentic-motion-authenticMotion_massAndWeight_ex2_large_xhdpi.webm" type="video/webm">
</video>

不要：线性动作会感到机械，在结束和开始的时候生硬的速度变化意味着物体突然开始运动或停止，这是不贴合现实的。


#### 特殊情况：进入和退出的场景

当一个物体进入这个场景时，请确保它在最高速度下移动，这个行为模拟了自然移动：一个人进入场景的时候，并不是从场景的边缘开始走入的，而是从更远的地方。当然，一个物体退出这个场景时，需要维持它的速度，缓慢的离开场景，逐渐的进入和缓慢的离开会把用户的注意力吸引到这个动作上，在大多数情况下，这是你希望的效果。

<video crossorigin="anonymous"  loop  controls width="740" height="270">
<source src="http://materialdesign.qiniudn.com/videos/videos-authenticMotion_massAndWeight_ex3_do_large_xhdpi.webm" type="video/webm">
</video>

要：小球在最大的速度下进入和退出场景，创造了一个确信的过渡效果。

<video crossorigin="anonymous"  loop  controls width="740" height="270">
<source src="http://materialdesign.qiniudn.com/videos/animation-authenticmotion-massandweight-authenticmotion_massandweight_example6_large_xhdpi.webm" type="video/webm">
</video>

不要：快速进入和缓慢离开，不要让用户因速度变化转移注意力。

### 需要做的调整（Making adjustments）

不是所有物体的移动方式是相同的，轻的/小的物体可能会更快的加速和减速，因为它们质量比较小，所以只需要施加给他们较少的力就可以。大的/重的物体可能花需要更多的时间来到达他的最高速度或者回到停止状态。仔细琢磨如何将他们的动作应用到你的应用的UI元素中。

# 打动用户的细节

动画可以存在于应用程序的所有组件和扩展中，从细小的图标到核心的场景转换和动作，所有元素共同构建出一个拥有无缝体验、美观且功能强大的应用。

动画最基本的使用场景是过度效果，但哪怕是最基本的动画，只要恰到好处并足够出色，同样能打动用户。例如一个菜单图标变成一个箭头或者是播放控制按钮，这种服务间的无缝切换不仅仅能让用户感知，更是让完美的细节和精湛的设计充满你的应用。用户真的会感受到这些小细节。

细节动画的演示：

<video crossorigin="anonymous" loop controls width="360" height="285">
<source src="http://materialdesign.qiniudn.com/videos/DelightfulDetails_WellCrafted_v01_large_xhdpi.webm" type="video/webm">
</video>

# 工具提示(Tooltips)

## 用法

对同时满足以下条件的元素使用工具提示：

1. 具有交互性
2. 主要是图形而非文本

![](../../../../images/components-tooltips-usage-tooltips_06a_large_mdpi.png)  
`要(Do)`

![](../../../../images/components-tooltips-usage-tooltips_06b_large_mdpi.png)  
`不要(Don't)`

工具提示不同于悬浮卡片，后者用来显示图片和格式化的文本等更为丰富的信息。

工具提示也不同于`ALT`属性，后者用来提示静态图片的主旨。


![](../../../../images/components-tooltips-usage-tooltips_13a_large_mdpi.png)  
`要(Do)`

![](../../../../images/components-tooltips-usage-tooltips_13b_large_mdpi.png)  
`不要(Don't)`

## 光标和键盘的工具提示

文本：Roboto Medium 10 sp

背景填充：90%不透明度

![](../../../../images/components-tooltips-cursorkeyboardtooltips-tooltips_09_large_mdpi.png)

![](../../../../images/components-tooltips-cursorkeyboardtooltips-tooltips_03_large_mdpi.png)

工具提示动画

<video width="720" height="270" loop="true" controls="controls"
src="http://materialdesign.qiniudn.com/videos/components-tooltips-cursorkeyboardtooltips-tooltips_005_large_xhdpi.webm" ></video>

## 触摸屏UI的工具提示

文本：Roboto Medium 14 sp

背景填充：90%不透明度

![](../../../../images/components-tooltips-touchuitooltips-tooltips_16_large_mdpi.png)

![](../../../../images/components-tooltips-touchuitooltips-tooltips_15a_large_mdpi.png)

![](../../../../images/components-tooltips-touchuitooltips-tooltips_15b_large_mdpi.png)

![](../../../../images/components-tooltips-touchuitooltips-tooltips_19a_large_mdpi.png)

![](../../../../images/components-tooltips-touchuitooltips-tooltips_19b_large_mdpi.png)

# 按钮(Buttons)

按钮由文字和/或图标组成，文字及图标必须能让人轻易地和点击后展示的内容联系起来。  
主要的按钮有三种:

* 悬浮响应按钮(Floating action button)， 点击后会产生墨水扩散效果的圆形按钮。
* 浮动按钮(Raised button)， 常见的方形纸片按钮，点击后会产生墨水扩散效果。
* 扁平按钮(Flat button)， 点击后产生墨水扩散效果，和浮动按钮的区别是没有浮起的效果。

颜色饱满的图标应当是功能性的，尽量避免把他们作为纯粹装饰用的元素。

按钮的设计应当和应用的颜色主题保持一致。

## 用法

![p0](images/components-buttons-usage-01_intro_large_mdpi.png)  
悬浮响应按钮  

![p1](images/components-buttons-usage-02_intro_large_mdpi.png)  
浮动按钮  

![p2](images/components-buttons-usage-03_intro_large_mdpi.png)  
扁平按钮  

### 主按钮

按钮类型应该基于主按钮、屏幕上容器的数量以及整体布局来进行选择。

首先，审视一遍你的按钮功能: 它是不是非常重要而且应用广泛到需要用上悬浮响应按钮?

然后，基于放置按钮的容器以及屏幕上层次堆叠的数量来选择使用浮动按钮还是扁平按钮。而且应该避免过多的层叠。

最后，检查你的布局。 一个容器应该只使用一种类型的按钮。 只在比较特殊的情况下（比如需要强调一个浮起的效果）才应该混合使用多种类型的按钮。

![p3](../../../../images/components-buttons-usage-05_chart_large_mdpi.png)  

### 对话框中的按钮

对话框中使用扁平按钮作为主要按钮类型以避免过多的层次叠加。

![p4](../../../../images/components-buttons-dialog_usage_1_large_mdpi.png)

![p5](../../../../images/components-buttons-dialog_usage_2_large_mdpi.png)

### 按钮内边距

根据特定的布局来选择使用扁平按钮或者浮动按钮。对于扁平按钮，应该在内部四周留出足够的空间（内边距）以使按钮清晰可见。

![p6](../../../../images/components-buttons-usage-1_usage_padding_large_mdpi.png)

![p7](../../../../images/components-buttons-usage-2_usage_padding_large_mdpi.png)

### 底部固定按钮

如果需要一个对用户持续可见的功能按钮，应该首先考虑使用悬浮响应按钮。如果需要一个非主要、但是能快速定位到的按钮，则可以使用底部固定按钮。

![p8](../../../../images/components-buttons-usage-1_buttons_usage_19_large_mdpi.png)

![p9](../../../../images/components-buttons-usage-2_buttons_usage_19_large_mdpi.png)

![p10](../../../../images/components-buttons-persistant-footer-1_large_mdpi.png)

不可在底部固定按钮的区域内使用浮动按钮。

![p11](../../../../images/components-buttons-usage-usage_persistent_large_mdpi.png)

底部固定按钮也可以用在内容可拉动的对话框中，前提是要加上divider。

![p12](../../../../images/components-buttons-persistant-footer-4a_large_mdpi.png)

![p13](../../../../images/components-buttons-persistant-footer-4b_large_mdpi.png)

## 主按钮

### 悬浮响应按钮

悬浮响应按钮是促进动作里的特殊类型。 是一个圆形的漂浮在界面之上的、拥有一系列特殊动作的按钮，这些动作通常和变换、启动、以及它本身的转换锚点相关。



![p14](../../../../images/components-buttons-floating-actions-1a_large_mdpi.png)

![p15](../../../../images/components-buttons-floating-actions-1b_large_mdpi.png)

悬浮响应按钮有两种尺寸: 默认尺寸和迷你尺寸。 迷你尺寸仅仅用于配合屏幕上的其他元素制造视觉上的连续性。

![p16](../../../../images/patterns-promotedactions-floatingactionbuttonFAB3_large_mdpi.png)

![p17](../../../../images/patterns-promotedactions-floatingactionbuttonFAB4_large_mdpi.png)

### 浮动按钮

浮动按钮使按钮在比较拥挤的界面上更清晰可见。能给大多数扁平的布局带来层次感。

![p18](../../../../images/components-buttons-usage-raised-do_large_mdpi.png)

![p19](../../../../images/components-buttons-usage-raised-dont_large_mdpi.png)

![p20](../../../../images/components-buttons-usage-raised-1a_large_mdpi.png)  
要  
正确使用浮动按钮。  

![p21](../../../../images/components-buttons-usage-raised-1b_large_mdpi.png)  
不要  
按钮不明显。  

### 扁平按钮

扁平按钮一般用于对话框或者工具栏， 可避免页面上过多无意义的层叠。

![p22](../../../../images/components-buttons-usage-flat-1a_large_mdpi.png)  

![p23](../../../../images/components-buttons-usage-flat-1b_large_mdpi.png)  


![p24](../../../../images/components-buttons-usage-flat-do_large_mdpi.png)  
要  
正确使用扁平按钮。  

![p25]../../../../images/components-buttons-usage-flat-dont_large_mdpi.png)  
不要  
层次感太重。  

### 扁平和浮动按钮的状态

按钮状态模拟

浮动按钮看起来像一张放在页面上的纸片，点击后会浮起来并表现出色彩。

扁平按钮会一直保持和页面贴合的状态，点击后会填充颜色。

墨水效果会跟着焦点的改变从一个按钮转换到另一个按钮。聚焦状态的动画会表现出正常状态和点击状态间来回切换的过渡效果。

模拟按钮状态的时候， 可以使用图形轮换来表现动画。注意聚焦状态会一直处于动画的状态。 (下面这些图并没有显示出真实的聚焦状态。)

![p26](../../../../images/components-buttons-states-for-mocks-1a_large_mdpi.png)  
Flat Light/Light color  
最小宽度: 88 dp， 高度: 36 dp  
覆盖状态: 20% #999， 点击状态: 40% #999， 不可用状态: 10% #999  

![p27](../../../../images/components-buttons-states-for-mocks-1b_large_mdpi.png)  
Flat Dark/Dark Color  
最小宽度: 88 dp， 高度: 36 dp  
覆盖状态: 15% #ccc， 点击状态: 25% #ccc， 不可用状态: 10% #ccc  

![p28](../../../../images/components-buttons-states-for-mocks-2a_large_mdpi.png)  
Raised Light/Light Color  
最小宽度: 88 dp， 高度: 36 dp  

![p29](../../../../images/components-buttons-states-for-mocks-2b_large_mdpi.png)  
Raised Dark/Dark Color  
最小宽度: 88 dp， 高度: 36 do  
正常状态: Color 500， 覆盖状态: Color 600， 点击状态: Color 700，
不可用状态: 10% #ccc  

### 按钮动态效果

<video src="http://materialdesign.qiniudn.com/videos/components-buttons-mainbuttons-buttons-motion1_large_xhdpi.webm" controls="controls" width="360" loop height="450"></video>  
扁平按钮  

<video src="http://materialdesign.qiniudn.com/videos/components-buttons-mainbuttons-buttons-motionraised_large_xhdpi.webm" controls="controls" width="360" loop height="450"></video>  
浮动按钮  

## 其他类型的按钮

### 图标开关

图标适合用在应用导航条或者工具条上，作为动作按钮或者开关。

图标开关可以在它的范围内呈现弹性或者非弹性的墨水扩散涟漪效果。 更多信息请参考： [面响应](../animation/responsive-interaction.html#responsive-interaction-ink-reactions)

![p30](../../../../images/components-buttons-icon-toggles-1_large_mdpi.png)

![p31](../../../../images/components-buttons-icon-toggles-2_large_mdpi.png)

![p32](../../../../images/components-buttons-icon-toggles-3_large_mdpi.png)

### 移动端下拉菜单按钮

**下拉菜单按钮**

下拉菜单按钮可以用来控制对象状态; 一般会有两个甚至更多的状态。 按钮会显示当前状态以及一个向下的箭头—当按钮触发后， 一个包含所有状态的菜单会在按钮周围弹出（通常都是在下方）。 菜单中的状态通常会以字符、调色板、图标或者其他的形式呈现出来。点击任意一个状态将会改变按钮的状态显示。这展示的是一个常见的带有列表式菜单的下拉菜单按钮。

![p33](../../../../images/components-buttons-mobile-dropdowns-1a_large_mdpi.png)

![p34](../../../../images/components-buttons-mobile-dropdowns-1b_large_mdpi.png)

**溢出下拉菜单按钮**

这种类型的下拉菜单按钮不会显示当前状态，而是显示一个向下箭头或者一个默认菜单图标。点击后会弹出菜单。点击菜单中的任意一个选项将会引导到对应的设置页面。

**分段式下拉菜单按钮**

分段式下拉菜单按钮有两个区域: 当前状态和下拉箭头。点击当前状态会触发状态相应的动作。点击下拉箭头则会弹出所有状态菜单; 点击任意一个状态会改变当前的状态。

**可编辑分段式下拉菜单按钮**

可编辑分段式下拉菜单按钮的当前状态位置是可编辑的（例如用来选择文字大小的下拉菜单）。 点击当前状态位置会触发相应的动作并且当前状态会变成可编辑。点击下拉箭头会显示所有状态。

![p35](../../../../images/components-buttons-mobile-dropdowns-3_large_mdpi.png)

<video crossorigin="anonymous"  loop  controls width="359" height="401">
<source src="http://materialdesign.qiniudn.com/videos/components-menus-menus-textfield_dropdown_spec_large_xhdpi.webm" type="video/webm">
</video>  

### 桌面下拉相关

**桌面应用工具栏**

![p36](../../../../images/components-buttons-desktop-dropdowns_large_mdpi.png)

## 底部动作条(Bottom Sheets)

底部动作条(Bottom Sheets)是一个从屏幕底部边缘向上滑出的一个面板，使用这种方式向用户呈现一组功能。底部动作条呈现了简单、清晰、无需额外解释的一组操作。


## 使用

底部动作条(Bottom Sheets)特别适合有三个或者三个以上的操作需要提供给用户选择、并且不需要对操作有额外解释的情景。如果只有两个或者更少的操作，或者需要详加描述的，可以考虑使用菜单(Menu)或者对话框替代。

底部动作条(Bottom Sheets)可以是列表样式的也可以是宫格样式的。宫格布局可以增加视觉的清晰度。

你可以使用底部动作条(Bottom Sheets)展示和其他app相关的操作，比如做为进入其他app的入口(通过app的icon进入)。

## 内容

在一个标准的列表样式的底部动作条(Bottom Sheets)中，每一个操作应该有一句描述和一个左对齐的icon。如果需要的话，也可以使用分隔符对这些操作进行逻辑分组，也可以为分组添加标题或者副标题。

一个可以滚动的宫格样式的底部动作条，可以用来包含标准的分享操作。

![图1](../../../../images/components-bottomsheet-for-mobile-1a_large_mdpi.png)
![图2](../../../../../../../../images/components-bottomsheet-for-mobile-1b_large_mdpi.png)

## 行为

显示底部动作条的时候，动画应该从屏幕底部边缘向上展开。根据上一步的内容，向用户展示用户上一步的操作之后能够继续操作的内容，并提供模态[1]的选择。点击其他区域会使得底部动作条伴随下滑的动画关闭掉。如果这个窗口包含的操作超出了默认的显示区域，这个窗口需要可以滑动。滑动操作应当向上拉起这个动作条的内容，甚至可以覆盖整个屏幕。当窗口覆盖整个屏幕的时候，需要在上部的标题栏左侧增加一个收起按钮。

## 规格

下面的字体、颜色和区域规格都是提供给手机app使用的。

![图1](../../../../images/components-bottomsheets-content-actionsheet_08_large_mdpi.png)
![图2](../../../../images/components-bottomsheets-content-actionsheet_08b_large_mdpi.png)

(上图)列表样式的底部动作条规格设计

![图1](../../../../images/components-bottomsheets-content-actionsheet_12_large_mdpi.png)
![图2](../../../../images/components-bottomsheets-content-actionsheet_12b_large_mdpi.png)

(上图)带头部的列表样式的底部动作条规格设计

![图1](../../../../images/components-bottomsheets-content-bottomsheet_10a_large_mdpi.png)
![图2](../../../../images/components-bottomsheets-content-bottomsheet_10b_large_mdpi.png)

![图1](../../../../images/components-bottomsheets-content-actionsheet_20_large_mdpi.png)
![图2](../../../../images/components-bottomsheets-content-actionsheet_20b_large_mdpi.png)

(上图)包含跳转到其他程序入口的标准宫格样式的底部动作条规格设计


[1]模态：模态的对话框需要用户必须选择一项操作后才会消失，比如Alert确认等；而非模态的对话框并不需要用户必须选择一项操作才会消失，比如页面上弹出的Toast提示。

# 卡片

卡片是包含一组特定数据集的纸片，数据集含有各种相关信息，例如，关于单一主题的照片，文本，和链接。卡片通常是通往更详细复杂信息的入口。卡片有固定的宽度和可变的高度。最大高度限制于可适应平台上单一视图的内容，但如果需要它可以临时扩展（例如，显示评论栏）。卡片不会翻转以展示其背后的信息。

## 用途

卡片是用来显示由不同种类对象组成的内容的便捷途径。它们也适用于展示尺寸或操作相当不同的相似对象，像带有不同长度标题的照片。

**注意：**即使外观相近，Now卡片是卡片的一个带有唯一表现和格式要求的独特子集。

![](../../../../images/components-cards-usage-card_single_large_mdpi.png)  

**卡片集**是**卡片**的一个平面布局。

![](../../../../images/components-cards-usage-card_travel_large_mdpi.png)     

![](../../../../images/components-cards-content-card_books_large_mdpi.png)  

这些卡片每张包含一组特定数据集：带操作的确认表，带操作的笔记，带照片的笔记。

![](../../../../images/components-cards-content-card_notes_large_mdpi.png)  

显示这些内容时使用卡片布局：

* 作为一个集合，由多种数据类型组成（例如，卡片集包含照片，电影，文本，图像）
* 不要求直接比较（用户不直接与图像或字符串比较）
* 包含可变长度内容，例如评论
* 由富内容或互动操作组成，例如+1按钮，滑块，或评论
* 如果使用列表需要显示超过三行文本
* 如果使用网格列表需要显示更多文本来补充图像

![](../../../../images/components-cards-usage-cardvstilea_large_mdpi.png)    
要       
1. 卡片带圆角。    
2. 卡片带多种操作。   
3. 卡片可以忽略和重排。    

![](../../../../images/components-cards-usage-cardvstileb_large_mdpi.png)  
不要     
这是瓷砖，不是卡片。    
1. 瓷砖带直角。    
2. 瓷砖少于两种操作。   

![](../../../../images/components-cards-usage-card_noa_large_mdpi.png)  
要   
可快速扫描的列表，用来代替卡片，是表现没有许多操作的同类内容的合适方法。

![](../../../../images/components-cards-usage-card_nob_large_mdpi.png)  
不要   
这里卡片的使用分散了用户注意力，不能快速扫描。这些列表项也不能忽略，所以把它们放在不同的卡片上是难以理解的。

![](../../../../images/components-cards-usage-card_no2a_large_mdpi.png)  
要   
网格瓷砖是表现图片库的干净轻量的方法。

![](../../../../images/components-cards-usage-card_no2b_large_mdpi.png)  
不要   
卡片在图片库中没有必要（同类内容）。

### 卡片布局准则

**字体设计** 

正文：14 sp 或 16 sp  

标题：24 sp 或更大   

扁平按钮：Roboto Medium, 14 sp, 10 sp 字间距   

**移动设备上的卡片间距** 

屏幕边界与卡片间留白：8 dp  

卡片间留白：8 dp   

**内容留白**  

16 dp   

![](../../../../images/components-cards-usage-cards_guidelines_large_mdpi.png)   

![](../../../../images/components-cards-11_large_mdpi.png)  

![](../../../../images/components-cards-13_large_mdpi.png)  

![](../../../../images/components-cards-15_large_mdpi.png)  

## 内容

卡片内容类型和数量可以很大程度上根据传递的内容变化。卡片提供上下文及通往更复杂信息与视图的入口；确保不要滥用带有无用信息或操作的卡片。

![](../../../../images/components-cards-content-card_books_large_mdpi.png)  

![](../../../../images/components-cards-content-card_discover_large_mdpi.png)  

放置主要内容在卡片顶部。使用层级结构来引导用户注意到卡片上最重要的信息。

![](../../../../images/components-cards-usage-card_travel_large_mdpi.png)  

![](../../../../images/components-cards-content-card_notes_large_mdpi.png)  

## 操作

卡片中的主要操作通常是卡片本身。

追加操作可以在一组卡片间根据内容类型和期望结果变化，例如，播放电影和打开书籍。一组卡片中，始终有定位操作。

**追加操作**   

卡片的追加操作通过图标，文本，和UI控制准确地呼出，这些通常放置在卡片底部。

放置在主要内容中的行内UI控制可以调整主要内容的外观，例如，滑块来选择日期，星星来给内容评分，或者分段的按钮来选择日期范围。

除弹出菜单外，限制追加操作在两项。

**弹出菜单**   

弹出菜单（可选）通常放置在卡片的右上角，但它也可以放置在右下方，如果这样安排改善内容布局和易读性。

注意不要滥用带过多操作的弹出菜单。

**注意事项**    

强烈不推荐文本内容的行内链接。

尽管卡片可以提供多种操作，UI控制，和弹出菜单，谨慎使用并且记得卡片是通往更复杂详细信息的入口。

![](../../../../images\components-cards-actions-card_actionsa_large_mdpi.png)  

![](../../../../images\components-cards-actions-card_actionsb_large_mdpi.png)  

![](../../../../images\components-cards-actions-card_actionsc_large_mdpi.png)  

![](../../../../images\components-cards-actions-card_actionsd_large_mdpi.png)  

## 表现

### 手势

支持单张卡片基准上的滑动手势。卡片手势表现应该始终在卡片组中实现。

按住并拖动手势可行。然而，考虑对用户能够在集合中排序卡片是否重要，或者如果按要求筛选/排序内容可以提供更好的体验。

### 卡片集筛选，排序，和重组

卡片集可以按要求排序或按日期，文件大小，字母表顺序，或其他参数筛选。集合中的第一项定位于集合的左上角，其余的从左至右从上至下延续。

### 滚动

卡片集只会竖直滚动。超过最大卡片高度的卡片内容将被截断且不可滚动。

带截断内容的卡片可以扩展，这样卡片高度可以超过视图的最大值。这种情况下，卡片将与卡片集一起滚动。

### 卡片焦点

对于取决于连续焦点遍历用于导航的界面（方向键，键盘），单张卡片应该仅有基本操作或打开一个带基本和可用追加操作的新视图。

#纸片(Chips)

Chips(我们暂时叫他纸片视图)是一种小块的用来呈现复杂实体的块，比如说日历的事件或联系人。它可以包含一张图片，一个短字符串(必要时可能被截取的字符串)，或者是其它的一些与实体对象有关的简洁的信息。Chips可以非常方便的通过托拽来操作。通过按压动作可以触发悬浮卡片(或者是全屏视图)中的Chip对应实体的视图，或者是弹出与Chip实体相关的操作菜单。   

##联系人纸片


联系人的纸片视图用于呈现联系人的信息。当用户在输入框(收件人一栏)中输入一个联系人的名字时，联系人纸片视图就会被触发，用于展示联系人的地址以供用户进行选择。而且联系人的纸片可以被直接添加到收件人一栏中去。    

联系人的纸片视图主要用于帮助用户高效的选择正确的收件人。     

![contact chips](../../../../images/components-chips-contactchips-chips_03a_large_mdpi.png)      
![contact chips](../../../../images/components-chips-contactchips-chips_03b_large_mdpi.png)    

**关闭状态的联系人纸片视图**

* 联系人名字使用的字体为：Roboto，常规，14sp

* 当点击关闭状态下的联系人纸片视图，它就会展开并且显示出联系人的地址


**打开状态的联系人纸片视图**

* 默认状态下，最顶的联系方式被激活并选中

* 联系人名字的字体为：Roboto，常规，16sp

* 地址文本的字体为： Roboto，常规，14sp

* 当用户选择后，纸片视图被关闭   

![关闭状态的联系人纸片视图](../../../../images/components-chips-contactchips-chips_08_large_mdpi.png)   

![打开状态的联系人纸片视图](../../../../images/components-chips-contactchips-chips_11_large_mdpi.png)   

![打开状态的联系人纸片视图各种状态](../../../../images/components-chips-contactchips-chips_06_large_mdpi.png) 

# 提示框(Dialogs)


Dialogs提示框)用于提示用户作一些决定，或者是完成某个任务时需要的一些其它额外的信息。 Dialog可以是用一种 取消/确定 的简单应答模式，也可以是自定义布局的复杂模式，比如说一些文本设置或者是文本输入 。

##用途

Dialog最典型的应用场景是提示用户去做一个些被安排好的决定 ，而这些决定可能是当前任务的一部分或者是前至条件。 Dialog可以用于告知用户具体的问题以便他们作用重要的决定(话外音：起到一个确认作用)，或者是用于解释
接下来的动作的重要性及后果 。(话外音：起到一个警示作用)。


一些复杂的操作，尤其是每个决策都需要相关解释说明的情况下是不适合使用Dialog形式的。


![contact chips](../../../../images/components-dialogs-usage-dialog_03_large_mdpi.png)    


Dialog包含了一个标题(可选)，内容 ，事件。

**标题**：主要是用于简单描述下选择类型。它是可选的，要需要的时候赋值即可。

**内容**：主要是描述要作出一个什么样的决定 。

**事件**：主要是允许用户通过确认一个具体操作来继续下一步活动。


![contact chips](../../../../images/components-dialogs-usage-dialogs_07_large_mdpi.png)    

###按钮的宽度及边框示例

![contact chips](../../../../images/components-buttons-buttonsindialogs_large_mdpi.png)    

![contact chips](../../../../images/components-dialogs-usage-dialogs_07a_large_mdpi.png)    

![contact chips](../../../../images/components-dialogs-usage-dialogs_07b_large_mdpi.png)    


###加宽型竖排按钮（Stacked full-width buttons）


当按钮的文本超过了通常的按钮宽度时，你就可以使用这种竖向叠模式来呈现我们的按钮文字信息。

![contact chips](../../../../images/components-dialogs-usage-stackedfullwidthbuttonsa_large_mdpi.png)    



###并排按钮（Side-by-side buttons）
在每个按钮的文本信息都没有超过通常的按钮宽度时，推荐使用并排模式。比如说最常用的 确定/取消 按钮 

![contact chips](../../../../images/components-dialogs-usage-sidebysidebuttonsa_large_mdpi.png)    

![contact chips](../../../../images/components-dialogs-usage-sidebysidebuttonsb_large_mdpi.png)    



##内容

###提示框标题
提示框的标题是可选的，用于说明提示的类型。可以是与之相关的程序名，或者是选择后会影响到的内容 。例如：设置

提示框标题应该作为提示框的一部分被整体地显示出来。


###提示框内容

提示框的内容是变化多样的。但是通常情况下由文本 和(或) 其它UI元素组成的，并且主要是用于聚焦于某个任务或者是某个步骤。比如说"确认"、"删除"或选择某个选项。

![contact chips](../../../../images/components-dialogs-content-dialogs_03a_large_mdpi.png)    
![contact chips](../../../../images/components-dialogs-content-dialogs_03b_large_mdpi.png)    



##事件

###提示框事件

提示框呈现的是一组聚焦和有限的事件，通常是一个肯定的事件和否定(与肯定的事件对立)的事件组成。

肯定的事件是放于提示框的右边并且可以继续接下来的步骤。肯定的事件可以是据有破坏性的，比如:"删除"，"移除"。(话外音：肯定的事件主要是指产品期望用户的一个决策。与按钮文字呈现的语意无关)

否定的事件是放于提示框的左边。用于返回用户原始的屏幕或者是步骤。(话外音：一般就是关闭提示框作用)

事件的按钮排列类型可以是并列的，也可以是竖向叠加加宽型的。这取决于事件按钮里面的文字长短。


肯定事件和否定事件除了可以使用"确认"/"取消"外，也可以是其它一些动词或者是动词短语来表明决策后的结果。

![contact chips](../../../../images/components-dialogs-actions-dialogs_11_large_mdpi.png)    



##表现(Behavior)

###滚动


提示框是与父视图是分隔开的。不会随着父视图滚动。


如果可以，请尽量保持提示框里面的内容不需要滚动 。如果滚动的内容太多了，那么可以考虑使用其它的容器或者是呈现方式。然而，如果内容是滚动的，那么请使用较明显的方式来提示用户。比如说被让文字或者是控件露一截出来。
![contact chips](../../../../images/components-dialogs-behavior-dialogs_12_large_mdpi.png)    

###手势


触摸提示框外面的区域可以关闭提示框 


###提示框焦点
 
 提示框的焦点是整个屏幕。提示框在关闭前或者是用户选择了一个事件(比如说选择了一个选项)前都会持有焦点。

#分隔线(Dividers)

分隔线(Dividers) 主要用于管理和分隔列表和页面布局内的内容，以便让内容生成更好的视觉效果及空间感。示例中呈现的分隔线是一种弱规则，弱到不会去打扰到用户对内容的关注。

##用途

##没有锚点的项（Items without anchors）


当在列表中没有像头像或者是图标之类的锚点元素时，单靠空格并不足以用于区分每个数据项(原文中使用的是“瓦片”)。这种情况下使用一个等屏宽(full-bleed)的分隔线就会帮助区别开每个数据项目。使其它看起来更独立和更有韵味。


![contact chips](../../../../images/components-dividers-items-without-anchor-1a_large_mdpi.png)
![contact chips](../../../../images/components-dividers-items-without-anchor-1b_large_mdpi.png)

###基于图片的内容

由于网格列表（grid）本身属性而造成的视觉效果，这就导致在网格列表中是不需要分隔线来区别子标题与内容的。在这种情况下，子标题与内容间的空白区域就可以分隔每块的内容了。


![contact chips](../../../../images/components-dividers-image-based-1a_large_mdpi.png)

##分隔线的类型

###等屏宽分隔线（Full-bleed dividers）


等屏宽分隔线或以用于分隔列表中的每个数据项或者是页面布局中的不同类型的内容。


![contact chips](../../../../images/components-dividers-full-bleed-1a_large_mdpi.png)
![contact chips](../../../../images/components-dividers-full-bleed-1b2_large_mdpi.png)

###内凹分隔线（Inset dividers）

在有锚点元素（头像或者是图标）并且有关键字的标题列中，我们可以使用内凹分隔线。


![contact chips](../../../../images/components-dividers-inset-1a_large_mdpi.png)
![contact chips](../../../../images/components-dividers-inset-1b_large_mdpi.png)

###子标题和分隔线

在使用分隔的子标题时，可以将分隔线置于子标题之上，可以加强子标题与内容关联度。


![contact chips](../../../../images/components-dividers-subheaders-1a_large_mdpi.png)
![contact chips](../../../../images/components-dividers-subheaders-1b_large_mdpi.png)

# 网格 #

网格列表是一种标准列表视图的可选组件。网格列表与应用于布局和其他可视视图中的网格有着明显的区别。



## 用法 ##

网格列表最适合用于同类数据（homogeneous data type），典型的如图片，并且对可视化理解（visual comprehension ）和相似数据类型的区别进行了优化。

![](../../../../images/components-grids-usage-spec_grid_drawings_01_large_mdpi.png)  

网格列表是一个连续元素（continuous element），该元素由棋盘式、规律性的小格子构成，通常称这些格子为单元格（cells）,单元格中包含有瓦片（tiles）。

单元格在网格中以垂直和水平的方式排列。

瓦片用以存放内容，并且可以跨越一个或者多个垂直或者水平的单元格。

![](../../../../../../../../images/components-grids-usage-spec_grid_drawings_02a_large_mdpi.png)  


![](../../../../images/components-grids-usage-spec_grid_drawings_02b_large_mdpi.png)  


如果瓦片中的文本需要与其他主要内容有着足够显著的区别，可以考虑使用一个容器，比如列表（Lists）或者卡片（Cards）。这样可以优化文本显示、增强阅读理解的便利性。

Lists：增强阅读理解的便利性，尤其是在比较一组具有多种数据类型的数据时。

Cards：用于不同格式的内容，比如带有不同长度标题的图片;用于不同类内容的数据集合中，比如具有图片、视频和图书的混合式数据集。


## 内容 ##

### 瓦片中的内容 ###

瓦片内容包括主要内容（primary content）和次要内容(secondary content)。主要内容是有着重要区别的内容，典型的如图片。次要内容可以是一个动作按钮或者文本。

为瓦片内容提供一个默认图片。


![](../../../../images/components-grids-content-spec_grid_drawings_03_large_mdpi.png)  


### 瓦片中的动作 ###

主要内容和次要内容中的动作--比如播放、放大、删除或者选择--是一种瞬时性动作，通常不会在网格列表中弹出选项子菜单（动作溢出列表，action voerflow）。

动作可以打开一个随后的视图，比如卡片或者悬浮卡片（hovercard）。

**主要动作**

- 充满整个瓦片，因此不会通过图标或者文本呈现。
- 在指定的网格中，所有瓦片的动作是一致的。比如，在单个网格中，对于所有瓦片的主要动作可以用于查看图片的详细信息。

**次要动作或者内容**

- 通过图标或者文本呈现出来。
- 在指定的网格中，所有瓦片的动作是一致的。
- 在指定网格的瓦片中放置的位置是一致的，但是位置可能会在不同的网格（边角或者边界， corners or edges）间有变化。比如，所有网格中的标题可以放置在左下角。


![](../../../../images/components-grids-content-spec_grid_drawings_04_large_mdpi.png)  


## 行为 ##

### 滚动 ###

网格典型的滚动只有垂直滚动。

水平滚动的网格是不鼓励使用的，这通常与用户的阅读习惯有关，影响阅读上的理解。

砍去网格瓦片来通知内容未结束。

![](../../../../images/components-grids-behavior-spec_grid_drawings_06_large_mdpi.png)    
要  
（图片用来）说明砍去网格瓦片提示内容未结束   

![](../../../../images/components-grids-behavior-spec_grid_drawings_05_large_mdpi.png)  
不要   

### 手势 ###

不允许使用轻扫(swipe)手势。选中并移动(pick-up-and-move)动作不鼓励使用。

### 瓦片过滤与排序 ###

网格列表中的内容可以编程实现其过滤和排序，比如通过数据类型、文件大小、字母顺序或者其他参数等。

网格中的第一个条目置于网格的左上角，并且其顺序为从左到右，自上而下。

### 维度与重置尺寸 ###

重置网格列表的尺寸会导致瓦片在有水平空间可用时重新排序。但是瓦片并不会缩放以填充可用的水平空间。

当水平空间受限时，网格列表不会转换为列表。网格列表与列表在强调不同数据类型的不同结构：图片优于文本与文本优于图片的区别。   

## 边框 ##

### 网格列表表头/表尾（header/footers）###

**单行表头/表尾**

高： 48dp

文本内边距： 16dp

默认字体大小： 16sp

次要动作与尾右对齐

**两行表头/表尾**

高： 68dp

文本内边距： 16dp

每行的默认字体大小： 16sp/12sp或者14sp/14sp


![](../../../../images/components-grids-keylines-Grid_footer_overview_01a_large_mdpi.png)  


![](../../../../images/components-grids-keylines-Grid_footer_overview_01b_large_mdpi.png)  


### 仅有图片的网格列表 ###

网格内边距： 4dp

网格列表中的瓦片可以跨多列。

仔细考虑网格列表中的次要文本是否需要使用多列瓦片，因为大的瓦片可能会造成很大的空间浪费。


![](../../../../images/components-grids-keylines-imageOnlyGrid_01_large_mdpi.png)  
元素    

![](../../../../images/components-grids-keylines-imageOnlyGrid_03_large_mdpi.png)  
  

### 单行网格列表 ###

**仅有文本**

高: 48dp

文本内边距： 16dp

默认字体大小： 16sp

网格内边距： 4dp


![](../../../../images/components-grids-keylines-SingleLineGrid_01a_large_mdpi.png)  


![](../../../../images/components-grids-keylines-SingleLineGrid_01b_large_mdpi.png)  
元素   


![](../../../../images/components-grids-keylines-SingleLineGrid_02_large_mdpi.png)  


**带图标的文本**

高： 48dp

文本内边距： 16dp

默认字体大小： 16sp

网格内边距： 4dp

网格列表表尾或者表头的中的次要文本可以右对齐或左对齐。


![](../../../../images/components-grids-keylines-SingleLineGrid_03a_large_mdpi.png)  


![](../../../../images/components-grids-keylines-SingleLineGrid_03b_large_mdpi.png)   
元素    


![](../../../../images/components-grids-keylines-SingleLineGrid_03c_large_mdpi.png)  


![](../../../../images/components-grids-keylines-SingleLineGrid_03d_large_mdpi.png)  


![](../../../../images/components-grids-keylines-SingleLineGrid_04_large_mdpi.png)  


### 两行网格列表 ###

**仅有文本**

高： 68dp

文本内边距： 16dp

每行的默认字体大小： 16sp/12sp或14sp/14sp

网格内边距： 4dp


![](../../../../images/components-grids-keylines-TwoLineGrid_01a_large_mdpi.png)  


![](../../../../images/components-grids-keylines-TwoLineGrid_01b_large_mdpi.png)  
元素    


![](../../../../images/components-grids-keylines-TwoLineGrid_02_large_mdpi.png)    
内容     

**带有图标的文本**

高： 68dp

文本内边距： 16dp

每行的默认字体大小： 16sp/12sp或14sp/14sp

网格列表表尾或者表头中的次要文本可以右对齐或左对齐。

网格内边距是4dp


![](../../../../images/components-grids-keylines-TwoLineGrid_03a_large_mdpi.png)  


![](../../../../images/components-grids-keylines-TwoLineGrid_03b_large_mdpi.png)     
元素     

![](../../../../images/components-grids-keylines-TwoLineGrid_03c_large_mdpi.png)    


![](../../../../images/components-grids-keylines-TwoLineGrid_03d_large_mdpi.png)   


![](../../../../images/components-grids-keylines-TwoLineGrid_04_large_mdpi.png)   
内容   


