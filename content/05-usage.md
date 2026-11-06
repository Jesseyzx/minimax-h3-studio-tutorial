---
title: 常见用法与进阶
---

# 常见用法与进阶

完整参数与 curl 见仓库 [doc/api.md](https://github.com/zxcHolmes/minimax-h3-studio/blob/master/doc/api.md)。下面按任务族列出「表单/API 怎么选」以及生产文档入口。

## 视频

| 模式 | 底模族 | 输入 |
|---|---|---|
| Text | fl2va | prompt |
| Frames | fl2va | 首帧，可选尾帧 |
| Reference | OmniRef 或 ref2va+Pika | prompt；最多 6 图、3 视频 |

常用表单字段：`resolution`（`544p`/`768p`）、`duration`（2–15 s）、`aspect_ratio`（`16:9` 等或 `image`）、`lora`（模型选择）、可选 `lora_strength`、`sampler`、`steps`（部分选择项会忽略覆盖）、`seed`，文件字段 `image` / `last_image` / 重复 `ref_image` / `ref_video`。

**Guide audio**：任意视频任务可带 `audio` + `audio_start`，从第 0 帧锚定一段配乐/人声（`MiniMaxH3AddGuide`）。成品歌的 **Video** 按钮会把它塞进表单。

参考模式前端会按官方 ref2va 布局拼 prompt（subject_definitions / summary / retention…）；主体用 `<Subject 1>` 等引用。参考视频会显著拉长采样时间（README 测过：5 s 544p 参考视频采样约 100 s vs 无参考约 26 s）。

## 图片

`kind=image`，`image_model`：

| 值 | 能力 |
|---|---|
| `moss` | 仅文生图（模板约 25 step） |
| `flux-klein-4b` | 4-step 蒸馏；可带最多 4 张参考修图 |
| `qwen-image-21-turbo` | 8-step；可修图；编辑时 `aspect_ratio=image` |

成品图可一键进参考 / 首帧 / 3D。

## 音乐

`kind=music`：`prompt`（caption）、可选 `lyrics`、`duration` 上限 10–300 s（默认 120）、`seed`。caption 建议三段：Global Metadata / Vocal Details / Arrangement；歌词可用 `[Verse]`、`[Chorus]` 等。空歌词 ≈ 纯器乐。实际长度可能短于上限。

## 图生 3D

`kind=3d`，必填 `image`；可选 `mesh_resolution`（1024/1536）、`texture_size`（1024/2048/4096）、`remove_background`。输出 GLB，`model_3d_url`。需要 ComfyUI 已带 Trellis2 / MoGe / 抠图等原生节点。

## 语音与转写

依赖已准备的 audio.cpp runtime + GGUF（`./deploy.sh models --only audio`）。

- `kind=speech`：`audio_model`=`qwen3_tts`|`voxcpm2`|`omnivoice`；`speech_mode`=`generate`|`clone`|`design`
- `kind=asr`：`audio_model`=`qwen3_asr`；上传音视频；导出 TXT/JSON，有对齐时才有 SRT/VTT

子进程只绑 loopback；Studio 代管启停与懒加载。

## 超分

对已完成视频：`POST /api/jobs/{id}/upscale`

| upscaler | target |
|---|---|
| `relayvsr` | `2x` / `4x`（上限短边 2176） |
| `flashvsr` | `1080p` / `1440p` |
| `seedvr2` | `1080p` / `1440p` |

音轨与帧率从源片拷贝。H3 原生就能出 768p；超分主要加锐利，不创造新内容。

## LLM 批（可选）

配置 `H3_STRATA_DIR` + `H3_STRATA_CONFIG` 后：`kind=llm`，上传 JSONL（每行 `{"id","body":{OpenAI chat body}}`，1–5000 行）。与图/视频/语音共用 FIFO；进入 LLM 会卸 Comfy / 停 audio.cpp。Web UI 无 LLM 页，仅 API。

## HTTP 速查

| 方法 | 路径 |
|---|---|
| GET | `/api/health` |
| GET | `/api/presets` |
| GET/POST | `/api/jobs` |
| GET | `/api/jobs/{id}` |
| POST | `/api/jobs/{id}/cancel` |
| POST | `/api/jobs/{id}/upscale` |
| DELETE | `/api/jobs/{id}` |
| POST | `/api/unload` |

当前服务**无内置鉴权**；公网暴露需自建入口鉴权。请求体上限 **256 MiB**。

## 生产向工作流文档（仓库自带）

- [narrated-image-workflow.md](https://github.com/zxcHolmes/minimax-h3-studio/blob/master/doc/narrated-image-workflow.md)：解说图文频道——大纲、脚本、静帧、旁白、Ken Burns
- [directing-workflow.md](https://github.com/zxcHolmes/minimax-h3-studio/blob/master/doc/directing-workflow.md)：用 Studio API 导戏——定妆参考、参考生视频、连续性、剪辑字幕
- [novel-adaptation-workflow.md](https://github.com/zxcHolmes/minimax-h3-studio/blob/master/doc/novel-adaptation-workflow.md)：小说分章改编；`scripts/split_book.py` 拆 epub/txt

## 加一个新 LoRA

在 `server/model_catalog.py` 的 `LORAS` 加条目（文件、源仓库、步数、sigma shift、分辨率），然后：

```bash
./deploy.sh models
./deploy.sh server
```

shift 必须与训练一致。前端通过 `/api/presets` 自动出现。

下一篇：[踩坑与限制](06-pitfalls.md)。上一篇：[核心概念与架构](04-architecture.md)。
