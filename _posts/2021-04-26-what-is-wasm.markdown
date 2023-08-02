---
layout: post
title: WebAssemble 概览
author: Hpbbs
categories: [ComputerScience]
published: true
---

* toc
{:toc}

## What

Wasm 是浏览器提供的一个在沙箱中运行的Environment，包括图灵完备的汇编语言指令支持的VM（Runtime）以及提供的统一的能力和资源接口（Context）

- 主要是为了让本源只考虑在 os platform的Native代码可以很方便的运行在 浏览器web环境中
- 其次，让很多对性能要求的高的程序可以跑在wasm中
- 当然，wasm具有一定的跨架构的能力（x86 arm x64）

可以说：Wasm不是一个技术实践，而是一个标准，在标准层次提供的解决方案
从代码运行阶段角度而言：WebAssembly as an intermediary compiler target，是一种中间代码

中间代码猜测拥有的特点：
- 易于在 http 协议之上传输
  - 容错校验
  - 压缩紧凑
  - 代码安全性要高
- 支持流式执行的优化
  - 可能代码执行每次只能执行一个切片
  - 可能上下文会随时裁剪 丢失 丢弃
- 所提供的能力是受限的
  - 能力接口权限是受限的
  - 接口是裁剪的

- 与JSruntime vm之间的Context共享

## 摘录
When the browser downloads the WebAssembly code it can quickly turn it to any machine’s assembly.
有两种格式: 开发者友好的 wat 格式 和 bin格式 wasm

Wasm bring：
- speed: download decode execute
- protable
- flexible: 除了JS，我们可以选择其他语言了（但是其他语言没有意义，学习broswer提供的能力Context和规则才是重要的）

## 官网 self-intro

https://webassembly.org/

- Efficient and Fast

WASM Stack machine : size and load-time-effcient
执行于于公共硬件的本地能力

- Safe

memory-safe, sandboxed execution environment
支持网站的同源安全策略

- open & debuggable

开发者友好

- Part of the open web platform

和JS VM 互通，和Broswer给JS暴露的能力互通