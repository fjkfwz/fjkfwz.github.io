title: Android 开发新技能
date: 2014-12-4
categories: Android 学习笔记
tags:
description:
---
**摘要**：从 Google 发布 Android 1.5（Cupcake 纸杯蛋糕）已经7年过去了，当初 Geek 们的玩具已经成为大众消费产品的一员，移动应用的开发已成为不可阻挡的趋势，那么近些年来 Android 开发又有哪些新的技术出现，这些改变会给我们的开发带了怎样的便利？
<!--more-->
>知乎吴晶的回答：
http://www.zhihu.com/question/32037895/answer/54486823

# 开发工具
## Android Studio
Google 官方放弃 Eclipse 和 Android Studio 普及。AS 虽然不算新，但是对 Android Studio 这个软件的更新速度快的惊人，有大量的新功能发布。例如支持很多注解代码提示注解、Live code template、支持自动生成 Parcelable 实现等等，作为开发者，持续关注这个更新列表 [Recent Changes](http://tools.android.com/recent) ，一定会让你的写代码的生活更加美好。

***
# 编程“语言”

## Kotlin
作为 Android 领域的 Swift，绝对让你如沐新风。抛弃沉重的 Java 语法，Kotlin 融入了很多现代编程语言的思想，作为开发者，接受新的语言，了解新语言的发展趋势，更有利于开阔你的思路和加深对语言的理解。在 Android 开发上，使用 Kotlin 并不会让你付出什么代价，为什么不来试试？
[使用 Kotlin 进行 Android 开发](http://ragnraok.github.io/using-kotlin-to-write-android-app.html)

## React Native
跨平台一直是程序员的梦想，而且移动应用的跨平台解决方案也很多，因为 Facebook 的参与和力推，让这个解决方案带上了光环。第一个用 React Native 开发的 App 已经在 Google Play 上架 [Facebook 广告管理工具](https://play.google.com/store/apps/details?id=com.facebook.adsmanager)，听说 Android 的 SDK 也马上会到来，[React Native](https://facebook.github.io/react-native/)。
Sky: 与 React Native 类似，使用 Web 开发语言来做移动平台的开发，虽然这个只是一个尝试，但是这是 Google 自身推出的，特别是在 Java 语言的使用上败诉之后，这可能会有一些作为呢?
[domokit/sky_sdk · GitHub](https://github.com/domokit/sky_sdk)

***
# 开发模式
Dagger 2：依赖注入并不是什么新技术，但是使用在 Android 确实一个新的尝试。Android App 越来越被当成严肃的大型项目来构建，很多在以前大型服务器开发上使用的技术都被应用到了移动开发。Android 开发分模块开发，使用 Dagger 来松耦合模块。特别值得一体的是，Dagger 2 现在由 Google 亲自接管。 
[Dagger ‡ A fast dependency injector for Android and Java](http://google.github.io/dagger/)

## MVP
因为 Android 并没有严格的业务和界面区分，项目一复杂，就很容易使代码陷入混乱。现在 Android 开发社区对 MVP 模式讨论越来越热，觉得 MVP 是非常适合 Android APP 开发。[MVP for Android: how to organize the presentation layer](http://antonioleiva.com/mvp-android/)

## RxAndroid
函数响应式编程（Functional Reactive Programming）也不是新内容，RxAndroid 把 RxJava 带到 Android 环境中。很多时候，编写 Android 程序，你也可以看成是数据的处理和流动，换一种思想编程，曾经看起来很棘手的问题，瞬间就很优雅的解决了：[ReactiveX/RxAndroid · GitHub](https://github.com/ReactiveX/RxAndroid)

## MVVM
这是因为开始官方支持 DataBinding，把 MVVM 直接带到 Android 中。数据绑定在 Windows WPF 和 Web 已经非常常见，它非常高效的开发效率，让你只关心你的数据和业务。这也对 Android 开发来说，无疑是一个非常重大的影响：[android UI 设计 MVVM 设计模式讨论？ - M.A.G.I 的回答](http://www.zhihu.com/question/30976423/answer/50224601)

## 插件化
针对大型 Android 项目，很多 App 开始使用插件来分模块构建相对独立的功能。
Hybrid：完全使用 HTML 5 开发 App，目前还不成熟。但是折中方案在很多情况下是非常适合的，典型的就是微信，大部分信息展示都是通过 H5 来完成，同时通过 Hybird 方式，把 Web 和 Native 打通，提供给网页访问本地资源的能力。
***
# UI 设计
## Material Design
已经红遍了大江南北，这方面的讨论实在太多了，而且各种支持库都有了，特别是 Google 官方出了一个支持库 [Android Design Support Library}(http://android-developers.blogspot.hk/2015/05/android-design-support-library.html)。

## Sketch 3
这是一个专为设计移动端 UI 的设计工具，作为开发者，不用懂那么复杂的 PS 使用，也可以做非常专业设计：https://designcode.io/sketch
