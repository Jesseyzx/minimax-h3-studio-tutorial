---
title: 这是什么
---

# 这是什么：单卡自托管的影音工作室

仓库：<https://github.com/zxcHolmes/minimax-h3-studio>  
许可证：代码 Apache-2.0；ComfyUI（GPL-3.0）与模型权重各自有独立许可，需自行遵守。

## 它解决什么问题

你想在**自己的 GPU 机器**上做：

- 文生视频 / 图生视频（首帧，或首+尾帧）/ 参考生视频（最多 6 张图 + 3 段视频）
- 配音轨引导（guide audio）：让画面跟着歌或说话走
- 出图与修图（MOSS-Image、FLUX.2 Klein 4B、Qwen Image 2.1 Turbo）
- 图生带贴图的 GLB（Pixal3D）
- 语音合成 / 克隆 / 设计，以及上传音视频转写（audio.cpp）
- 写歌（MiniMax Music 3，最长约 5 分钟）
- 成品视频 2×/4× 或拉到 1080p/1440p 超分

同时不想每次手拖 ComfyUI 工作流 JSON，也不想把权重拆成「每次冷启动」。Studio 做的是：

1. 把官方工作流的节点顺序写成 Python（`comfy_launcher` / `server/pipelines`）
2. 权重常驻，任务进 FIFO 队列，一次只跑一个（语音可多实例池化）
3. 前面挂 aiohttp API + Vue 浏览器 UI
4. 一条 `./deploy.sh` 把代码和 UI 同步到 GPU 机并装成 systemd 服务

在 README 给出的 RTX 4090 参考机上，一段带声音的 5 秒片大约：544p 约 30–45 秒，768p 约 65–70 秒（具体随 LoRA / FastH3 / Pika 选择变化）。

## 和「直接用 ComfyUI」有什么不同

| 点 | 直接 ComfyUI | MiniMax-H3 Studio |
|---|---|---|
| 界面 | ComfyUI 画布 / 模板 | 任务导航 + 历史卡片 |
| 调用方式 | 工作流 JSON | Python 调节点；像素级对齐官方模板 |
| 部署 | 你自己起进程 | `deploy.sh` + systemd `h3-server` |
| 队列 | 依赖 Comfy 自身或外部编排 | SQLite + 单线程 Worker |
| 语音 ASR | 另找节点 | 独立 audio.cpp CUDA 子进程 |
| LLM 批 | 无 | 可选 Strata 子进程（`kind=llm`） |

## 仓库目录怎么分

```
minimax-h3-studio/
├── server/           # HTTP 服务、引擎、管线、任务表
├── web/              # Vue 3 + Vite 前端
├── comfy_launcher/   # 无服务的命令行生成器 run_h3.py
├── scripts/          # 下模型、准备 audio.cpp、拆书等
├── doc/              # API 与生产向工作流文档
├── tests/            # 用 stub engine 的回归测试
├── deploy.sh         # SSH 部署入口
└── .env.example      # 机器相关配置模板
```

入口：

- 服务：`server/app.py`（`python server/app.py`，需 `H3_COMFYUI`）
- 前端构建：`web/` → `npm run build`，产物由服务静态托管
- 无头单次生成：`comfy_launcher/run_h3.py`

## 谁适合读

按 README 定位：已经有（或准备有）Linux + NVIDIA GPU + 自装 ComfyUI 的开发者 / 制作者。没有 GPU 也能读懂架构和 API，但**没法完成本地生成实测**。

下一篇：[环境与安装](02-install.md)。
