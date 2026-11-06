---
title: 最小可跑通
---

# 最小可跑通

目标：按仓库「Quick start」把服务装上，并用文档里的 curl 提交第一条任务。  
**说明**：编写本教程的沙箱没有 GPU、没有 `H3_DEPLOY_HOST`、没有模型权重，因此下列命令只作「照着仓库文档复述」；未能实测 `deploy.sh` 与真实生成。

## 1. 配置并部署

```bash
git clone https://github.com/zxcHolmes/minimax-h3-studio && cd minimax-h3-studio
cp .env.example .env
# 填 H3_DEPLOY_HOST、H3_COMFYUI，以及 conda/venv
./deploy.sh models            # 把配置到的 checkpoint 下到远端 ComfyUI/models
./deploy.sh                   # 构建 UI、安装 systemd、启动
```

浏览器打开：`http://<gpu-host>:8190`。首次启动会加载 text encoder 与 VAE；UI 会显示服务何时 ready。

`deploy.sh` 子命令：

| 命令 | 作用 |
|---|---|
| `./deploy.sh` / `all` | 前端 + 服务（会重启服务） |
| `./deploy.sh web` | 只更 UI，不重启（权重保持热） |
| `./deploy.sh server` | 只更服务并重启 |
| `./deploy.sh models` | 下载模型；额外参数传给 `prepare_models.py` |
| `FORCE=1 ./deploy.sh server` | 有排队/运行中任务也强制重启（任务会丢） |

GPU 机运维：

```bash
journalctl -u h3-server -f
sudo systemctl stop h3-server
sudo systemctl start h3-server
```

数据默认在 `$H3_DATA_DIR`（常为 `$H3_DEPLOY_DIR/data/`）：`jobs.db`、`videos/`、`pictures/`、`music/`、`thumbs/`、`uploads/` 等。

## 2. 先看健康与预设

```bash
export STUDIO_URL='http://<gpu-host>:8190'
curl --fail-with-body "$STUDIO_URL/api/health"
curl --fail-with-body "$STUDIO_URL/api/presets"
```

- `health.state`：`loading` / `ready` / `error`
- `ready` 表示 Worker 可以接活；`running` / `queued` 看当前占用
- `presets` 是分辨率、模型组、LoRA、图片尺寸、音乐时长、超分目标等的权威列表——**以线上返回为准**，不要死记本文快照

## 3. 提交一条文生视频（README 示例）

`POST /api/jobs` 成功返回 **201**，只表示入队，不表示生成完成。全站共用一条 FIFO；同一时刻只跑一个任务（语音可多实例）。

```bash
curl -F resolution=544p -F duration=5 -F aspect_ratio=16:9 \
  -F "prompt=integrated_multimodal_description: A red panda sips cocoa by a frosty window.

overall_soundscape: Crackling fireplace.

non_diegetic_music: N/A" "$STUDIO_URL/api/jobs"
```

记下返回的 `id`，轮询：

```bash
curl --fail-with-body "$STUDIO_URL/api/jobs/<id>"
```

`status` 变为 `done` 后，用返回的 `video_url`（形如 `/files/videos/...`）拼到 `STUDIO_URL` 下载。

提示：若 prompt 可能以 `@` 或 `<` 开头，curl 要用 `--form-string`，避免被当成文件。

## 4. 无头启动器（不经 HTTP）

在 GPU 机、ComfyUI 环境中：

```bash
python comfy_launcher/run_h3.py --comfyui /path/to/ComfyUI \
  --prompt-file prompt.txt \
  --lora minimax_h3_fl2v_turbo_8step_v1.0_comfyui_bf16.safetensors \
  --steps 8 --width 960 --height 544 --duration 5 \
  -- --enable-triton-backend --use-sage-attention
```

还支持 `--first-frame`、`--last-frame`、重复 `--ref-image`、`--jobs-json` 批跑等。详见 `python comfy_launcher/run_h3.py --help`。`comfy_launcher/bench.sh` 会包一层并记录 GPU 显存。

## 5. 前端本地开发

```bash
cd web && npm install && npm run dev
```

开发服务器读仓库根目录 `.env`，把 `/api` 与 `/files` 代理到已部署的服务（`H3_API` 或由 `H3_DEPLOY_HOST`+`H3_PORT` 推导）。

## 6. 本教程实际跑过什么

在编写机上已做：

- `git clone` 仓库
- `gitingest` 生成 `digest.txt`
- `PYTHONPATH=server` 下尝试 `python -m unittest discover -s tests`：装了 `aiohttp`、`pillow` 后大部分用例可导入；若干用例因缺 `av`（PyAV）等依赖失败；**没有**启动真实 `H3Engine` / ComfyUI

未做：SSH 部署、`deploy.sh models`、任何真实推理、需要 `HF_TOKEN` 的 gated 下载。

下一篇：[核心概念与架构](04-architecture.md)。上一篇：[环境与安装](02-install.md)。
