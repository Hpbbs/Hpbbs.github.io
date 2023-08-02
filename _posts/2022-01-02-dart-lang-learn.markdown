---
layout: post
title: "Dart 基本使用和介绍"
published: false
categories: [ComputerScience]
tags: [Flutter]
---

# Flutter 端侧开发 学习笔记
{: toc }

1. 查看 flutter 在 github 的 sample
2. 快速过一遍 Dart 语言知识
3. 结合Sample 学习记忆 Flutter编写项目的最佳实践

基础学习资源：
1. https://github.com/Solido/awesome-flutter
2. 往前所做的Dart 以及 flutter的笔记

# Chapter ONE Dart

=> 是单行函数的简写 `void main() => runApp(MyApp());`

MyApp 继承自StatelessWidget，使得应用本身变成了一个widget，Flutter中，几乎所有都是widget，包括 align padding layout

Scaffold 是Material库提供的一个widget，组件，默认提供了 导航栏，标题和包含主屏幕widget树的body属性