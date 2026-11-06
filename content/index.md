---
title: MiniMax-H3 Studio 中文教程
---

# MiniMax-H3 Studio 中文教程

本专栏把开源仓库 [zxcHolmes/minimax-h3-studio](https://github.com/zxcHolmes/minimax-h3-studio) 转写成一套可跟着做的简体中文技术教程。内容以仓库 README、源码与本机可读的单元测试为准；**没有在真实 GPU 上跑通端到端生成**（本环境无 NVIDIA GPU、无 ComfyUI 权重、无 Hugging Face Token）。

官网样例画廊：<https://h3-gallery.aibit.im>

## 阅读顺序

1. [这是什么：单卡自托管的影音工作室](01-overview.md)
2. [环境与安装：ComfyUI、.env、模型下载](02-install.md)
3. [最小可跑通：deploy 与第一条 curl](03-quickstart.md)
4. [核心概念与架构](04-architecture.md)（含 D2 图）
5. [常见用法与进阶](05-usage.md)
6. [踩坑与限制](06-pitfalls.md)

## 项目一句话

在单张 NVIDIA GPU 上，把 MiniMax-H3（以及图片、音乐、语音、超分、图生 3D）包成常驻 HTTP 服务 + 浏览器 UI；底层把 ComfyUI 当 Python 库调用，不启 ComfyUI 官方界面。

## 阅读提示

章节之间使用相对 Markdown 链接，在 GitHub 上可直接打开。
