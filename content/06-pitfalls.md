---
title: 踩坑与限制
---

# 踩坑与限制

全部来自仓库 README / CLAUDE.md / 代码注释中的实测结论与明确限制；不是臆测。

## 部署与环境

- **ComfyUI 必须自装**，本仓库不顺带 clone。版本过旧会缺 RelayVSR / Trellis 等节点。
- **comfy_kitchen CUDA**：cu128 + 旧驱动上官方 cu13 wheel 常加载失败；需源码重建，并给 `quant_ops.py` 打「仅不可用才 disable」补丁。更新 ComfyUI 可能冲掉补丁。
- 没装 triton / sageattention 时，把 `H3_COMFY_ARGS` 清空，否则启动可能报错或无效。
- `deploy.sh` 默认**拒绝**在有 queued/running 任务时重启；要用 `FORCE=1`，并接受丢任务。
- 数据目录只存文件名引用：搬家要停服务后**整目录迁移**，再改 `H3_DATA_DIR`。

## 模型与质量

- lightx2v fl2v / ref2v turbo LoRA 已从产品选择中移除（日期见 README）：快速动作会出现梳状拖影；参考路线改推 OmniRef / Pika。
- larryvrh 在参考模式下主体一致性差，因此只提供给 fl2va（文本/图生视频）。
- FastH3 的 VSA 依赖其训练时的粗注意力分支，**不能**套到官方底模上；官方底上的 sparse 用的是 Sol-Attn。
- Pika 的 sparse 是训练无关方法；与 dense 结果会分叉，属预期。
- MOSS 官方模板里的 LLM prompt 扩写被刻意关掉（作者测约 45 s 且效果更差）。
- 参考视频会进每次 attention 的精确前缀，时间成本陡增；长参考 + 长镜头要做好排队心理预期。
- RelayVSR 4× 峰值系统内存可到数十 GB（README：15 s 544p 4× 约 54 GB）；短边上限 2176。

## API / 产品边界

- 无鉴权：勿裸奔公网。
- 201 ≠ 生成完成；务必轮询。
- 上传字段是**字节**，不是服务器本地路径；复用成品需先下载再上传。
- ASR 只支持上传文件，不做实时麦克风流；最长约 2 小时 / 256 MB。
- LLM UI 不存在；Strata 未配置时 `kind=llm` 直接拒绝。
- 服务进程内 OOM 回收失败后 Worker 可能停在 `error`，需要 `systemctl restart`；库里的排队任务还在。

## 本教程环境额外限制

编写机：**无 NVIDIA GPU、无 ComfyUI、无权重、无 HF Token、无 SSH GPU 主机**。因此：

- 未执行 `./deploy.sh` / `models`
- 未跑通真实文生视频 / 出图 / TTS
- 单元测试缺 `av` 等依赖时有失败；即使全绿也只是 stub engine，不等于推理正确

若你本地补齐 GPU 与 `.env`，请以 `GET /api/health` → `ready` 和一条最短 544p 任务作为验收，而不是以本专栏截图为准。

## 许可提醒

仓库代码 Apache-2.0。ComfyUI GPL-3.0 不内嵌。权重各有许可证（含 MiniMax-H3 Community License）。商用前自行核对。

返回目录：[教程首页](index.md)。上一篇：[常见用法与进阶](05-usage.md)。
