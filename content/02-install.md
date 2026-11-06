---
title: 环境与安装
---

# 环境与安装

这一篇只整理**仓库文档里写明的前置条件与配置**。本教程编写环境没有 GPU / ComfyUI，因此没有执行 `./deploy.sh` 或下载权重；命令原文来自 README 与 `.env.example`。

## 两台角色

1. **GPU 机器**（真正跑生成）
2. **你的机器**（跑 `deploy.sh`、构建前端；也可以就是同一台）

### GPU 机器

- Linux + NVIDIA GPU。作者在 RTX 4090 24GB、约 125GB 系统内存上开发；权重走 memory-map，建议 64GB+ 内存和快 SSD。
- 自行安装 [ComfyUI](https://github.com/comfyanonymous/ComfyUI)，提交 **`2f894332` 或更新**（v0.44.0 + 14 commits，首个带 RelayVSR）。`git pull` 后按其 `requirements.txt` 安装，文档写明该提交需要 `comfy-aimdo==0.6.0`、`comfy-kitchen==0.2.41`。
- systemd；部署用户对 systemd 相关命令能无密码 `sudo`（脚本用非交互 SSH 装单元）。
- `python3`、`curl`、`rsync`。
- 可选加速：ComfyUI 环境里装 `triton`、`sageattention`，通过 `H3_COMFY_ARGS` 打开。
- 可选自定义节点：
  - larryvrh LoRA：`git clone https://github.com/larryvrh/ComfyUI-MiniMax-H3-Turbo <ComfyUI>/custom_nodes/ComfyUI-MiniMax-H3-Turbo`
  - FlashVSR：`git clone https://github.com/T8mars/Comfyui-Flashvsr-Plus-T8 <ComfyUI>/custom_nodes/Comfyui-Flashvsr-Plus-T8`（文档对照 commit `ac60d1e`）

### 部署机

- Node.js 18+（前端构建）
- 到 GPU 机的 SSH 公钥登录、`rsync`、`scp`

## 克隆与配置

```bash
git clone https://github.com/zxcHolmes/minimax-h3-studio
cd minimax-h3-studio
cp .env.example .env
# 编辑 .env —— 绝不要提交 .env
```

### 必填

| 变量 | 含义 |
|---|---|
| `H3_DEPLOY_HOST` | GPU 机 SSH 目标，如 `me@gpu-host` |
| `H3_COMFYUI` | GPU 机上 ComfyUI 检出路径 |

### 常用可选项（完整表见 README / `.env.example`）

| 变量 | 默认 | 含义 |
|---|---|---|
| `H3_CONDA_ENV` | `comfyui` | Conda 环境名 |
| `H3_CONDA_SH` | `~/miniconda3/etc/profile.d/conda.sh` | conda 初始化脚本 |
| `H3_PYTHON` | 空 | 若设绝对路径 Python，则忽略 conda 两项 |
| `H3_DEPLOY_DIR` | `~/minimax-h3-studio` | 远端安装目录 |
| `H3_DATA_DIR` | `$H3_DEPLOY_DIR/data` | `jobs.db` 与全部生成文件；建议放到大盘 |
| `H3_HOST` / `H3_PORT` | `0.0.0.0` / `8190` | 监听地址 |
| `H3_COMFY_ARGS` | `--enable-triton-backend --use-sage-attention` | 传给 ComfyUI；没装就清空 |
| `HF_ENDPOINT` | `https://huggingface.co` | 镜像，国内常用 `https://hf-mirror.com` |
| `HF_TOKEN` | 空 | 仅 gated/私有仓需要；**不要写进教程或提交到 Git** |
| `H3_AUDIOCPP_*` | 见 `.env.example` | 原生语音运行时 |
| `H3_STRATA_*` | 空则禁用 LLM | Strata 批处理 |

Shell 里已导出的 `H3_*` 会覆盖 `.env` 同名项（`deploy.sh` 会保留预先 export 的值）。

## 模型下载

服务需要的文件清单由 `server/model_catalog.py`、`server/image_models.py`、`server/audio_catalog.py` 决定。在配置好 `.env` 后：

```bash
./deploy.sh models
# 等价于在 GPU 机上跑 prepare_models.py
python3 scripts/prepare_models.py --comfyui /path/to/ComfyUI
python3 scripts/prepare_models.py --list
python3 scripts/prepare_models.py --comfyui ... --skip ref2va fasth3
```

`--skip` / `--only` 分组包括：`shared`、`fl2va`、`ref2va`、`larryvrh`、`pika`、`fasth3`、`image`、`klein`、`qwen-image`、`audio`、`audio-runtime`、`music`、`upscale`、`flashvsr`、`relayvsr`、`pixal3d` 等。

示例（README）：

```bash
HF_ENDPOINT=https://hf-mirror.com ./deploy.sh models --only klein qwen-image --workers 4
HF_ENDPOINT=https://hf-mirror.com ./deploy.sh models --only audio --workers 4 --connections 8
```

体积很大（仅视频/共享部分就数十 GB；Klein+Qwen 约再加 31 GiB；语音约 15 GiB + runtime）。磁盘与带宽要提前留够。模型有各自许可证（含 MiniMax-H3 Community License），使用前自行阅读。

## PyTorch 低于 cu130 时的 comfy_kitchen

README 专节说明：官方 comfy_kitchen CUDA wheel 常要求 CUDA 13 / 驱动 r580+。在 cu128（例如 torch 2.11+cu128、驱动 570）上需要：

1. 按 GPU 架构从源码重建 comfy_kitchen（设 `COMFY_CUDA_ARCHS`，如 4090 → `89`）
2. 给 ComfyUI 的 `comfy/quant_ops.py` 打补丁：仅当 CUDA backend 真的不可用时才 `disable("cuda")`，而不是「凡是 < cu130 一律关」

补丁在 **ComfyUI 检出里**，不在本仓库；更新 ComfyUI 后可能要重打。本教程未在本机执行重建或补丁。

## 不部署、本机直接起服务（文档写法）

```bash
cd web && npm install && npm run build
# 在 ComfyUI 的 Python 环境里：
H3_COMFYUI=/path/to/ComfyUI H3_WEB_DIST=web/dist python server/app.py
```

下一篇：[最小可跑通](03-quickstart.md)。上一篇：[这是什么](01-overview.md)。
