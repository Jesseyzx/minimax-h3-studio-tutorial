# MiniMax-H3 Studio 中文教程

本仓库是 [MiniMax-H3 Studio](https://github.com/zxcHolmes/minimax-h3-studio) 的简体中文教程转写。上游是一套自托管、面向单张 NVIDIA GPU 的影音工作室：把 MiniMax-H3（以及图片、音乐、语音、超分、图生 3D）包成常驻 HTTP 服务和浏览器界面，底层把 ComfyUI 当 Python 库调用。

正文在 `content/`，配图在 `content/assets/`。`diagrams/` 存放 D2 源文件，以及与配图对应的导出副本。

## 章节

按这个顺序读：

1. [教程首页](content/index.md)
2. [这是什么：单卡自托管的影音工作室](content/01-overview.md)
3. [环境与安装：ComfyUI、.env、模型下载](content/02-install.md)
4. [最小可跑通：deploy 与第一条 curl](content/03-quickstart.md)
5. [核心概念与架构](content/04-architecture.md)
6. [常见用法与进阶](content/05-usage.md)
7. [踩坑与限制](content/06-pitfalls.md)

## 两张架构图

[核心概念与架构](content/04-architecture.md) 使用了两张由 D2 渲染的图：

- **整体架构**：[content/assets/architecture.svg](content/assets/architecture.svg)（[手绘风](content/assets/architecture-sketch.svg)，[PNG](content/assets/architecture.png)）
- **任务数据流**：[content/assets/job-flow.svg](content/assets/job-flow.svg)（[手绘风](content/assets/job-flow-sketch.svg)，[PNG](content/assets/job-flow.png)）

源文件是 [diagrams/architecture.d2](diagrams/architecture.d2) 和 [diagrams/job-flow.d2](diagrams/job-flow.d2)。`diagrams/` 里的 SVG / PNG 是同一批导出副本。

## 编写范围

教程根据上游仓库的 README、源码和单元测试整理。编写环境没有 NVIDIA GPU、没有 ComfyUI 权重、也没有 Hugging Face Token，因此没有跑通端到端 GPU 生成。文中的命令、接口和数字转述自上游文档与源码。
