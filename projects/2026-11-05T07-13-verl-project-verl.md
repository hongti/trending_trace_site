# verl-project/verl — 动态追踪

> 生成时间: 2026-11-05 15:13 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issues (问题反馈)
近期主要报告了 3 个 Bug，集中在工具解析与数据处理方面：
1. **工具解析报错**：Qwen3 XML 工具解析器在遇到参数值为 `...`（省略号/Ellipsis）时崩溃，因为 Ellipsis 无法被 JSON 序列化 (#8559)。
2. **数据集路径问题**：`RLHFDataset` 无法像 `AgentLoopWorker` 那样正确解析相对路径的 `tool_config_path`，导致在 Ray 分布式环境中出错 (#8554)。
3. **过滤器重复执行**：`overlong-prompt` 过滤器在每次启动时都会重新运行，原因是数据集指纹依赖于进程状态，导致缓存失效 (#8552)。

### 🚀 Pull Requests (代码合并)
PR 动态涵盖了重要 Bug 修复、新硬件支持及新训练后端引入：

**重要修复：**
- **工具与数据修复**：#8560 和 #8551 修复了上述 Qwen3 解析器遇到 `...` 报错的问题（将其保留为纯文本）；#8555 修复了 `RLHFDataset` 相对路径解析问题；#8553 修复了 `overlong-prompt` 过滤器重复运行的问题，实现跨启动复用缓存。
- **算法精度修复**：#8548 修复了在 BF16 logits 下，`use_chunked_topk=True` 时 `forward_kl_topk_tail` 可能计算出负值并反转梯度的严重 Bug，确保为 tail KL 保留 fp32 对数概率。

**新特性与硬件支持：**
- **新增 MaxText 训练后端**：#8547 引入 MaxText (JAX) actor 引擎，支持在 AMD MI300X (ROCm) 上使用 GRPO 训练 Qwen3-30B-A3B 模型，并由 vLLM 提供独立 rollout 服务。
- **TPU 硬件支持完善**：#8557 在 `_DEVICE_FLOPS` 中添加了 TPU 设备的算力（FLOPS）映射；#8558 升级了 TPU GRPO 镜像中的 `vllm-torchtpu` 依赖。
- **文档更新**：#8556 在 README 的硬件支持列表中明确添加了 Intel (XPU) 支持。

**依赖维护：**
- #8546 将依赖 `torchaudio` 从 2.11.0 升级至 2.11.1。

### 📦 Release (版本发布)
近期无新的 Release 发布。开发重点集中在修复核心数据处理 Bug、增强算法稳定性以及扩展底层硬件（AMD/TPU/Intel）生态支持。

---

## 🐛 Issues

### #8559 — [[tool] bug: Qwen3 XML tool parser raises on `...` argument values (Ellipsis is not JSON serializable)](https://github.com/verl-project/verl/issues/8559)
- **作者**: super-yuki  **时间**: 2026-11-05 14:57 CST
- **摘要**: ### System Info  ```bash ----------Python Info---------- Version      : 3.12.16 Compiler     : GCC 15.3.0 Build        : ('main', 'Oct 26 2026 11:41:19') Arch         : ('64bit', '') ------------Pip Info----------- Version      : 26.2.2 Directory    : /path/to/site-packages/pip vllm         : 0.27.1…

### #8554 — [[data] bug: RLHFDataset does not resolve relative tool_config_path (unlike AgentLoopWorker)](https://github.com/verl-project/verl/issues/8554)
- **作者**: super-yuki  **时间**: 2026-11-05 09:56 CST
- **摘要**: ### System Info  - verl: main @ `3c6d9b43` (the affected files are unchanged on `31cfccb4`) - Python 3.12, PyTorch 2.10, transformers 5.12.1 - Ray 2.x with actors whose working directory differs from the driver's (multi-node cluster or a cluster started from another directory) - Platform: Ascend NPU…

### #8552 — [[data] bug: overlong-prompt filter re-runs on every launch (datasets fingerprint depends on process state)](https://github.com/verl-project/verl/issues/8552)
- **作者**: super-yuki  **时间**: 2026-11-05 09:54 CST
- **摘要**: ### System Info  - verl: main @ `3c6d9b43` (the affected files are unchanged on `31cfccb4`) - Python 3.12, PyTorch 2.10, datasets 5.1.0, transformers 5.12.1 - `data.filter_overlong_prompts=True`, `data.filter_overlong_prompts_workers=96` - Platform: Ascend NPU 8 cards/node (the caching behavior is p…

## 🔀 Pull Requests

### #8560 — [[tool] fix: keep `...` arguments as text in Qwen3 XML tool parser](https://github.com/verl-project/verl/pull/8560)
- **作者**: super-yuki  **时间**: 2026-11-05 14:58 CST
- **摘要**: ### What does this PR do?  `Qwen3XMLToolParser` raises `TypeError: Object of type ellipsis is not JSON serializable` when a tool-call parameter value is literally `...`: `ast.literal_eval("...")` returns `Ellipsis`, and `json.dumps(param_dict)` cannot serialize it. The exception propagates out of `e…

### #8558 — [[hardware, vllm] chore: bump vllm-torchtpu to 281b5266 in the TPU GRPO image](https://github.com/verl-project/verl/pull/8558)
- **作者**: xiaoxlu  **时间**: 2026-11-05 13:22 CST
- **摘要**: ### What does this PR do?  Bumps `VLLM_TORCHTPU_COMMIT` in `examples/tpu/grpo/Dockerfile` from `732c4346` to `281b5266`, which includes vllm-project/vllm-torchtpu#2732 (`TPUWorker.reset_encoder_cache`). With it, verl-hardware-plugin can drop its local vLLM patch for that method.  The dependency pins…

### #8557 — [[hardware] fix: add TPU device FLOPS entries to _DEVICE_FLOPS](https://github.com/verl-project/verl/pull/8557)
- **作者**: askhat-g  **时间**: 2026-11-05 12:56 CST
- **摘要**: ## Summary On TPU workers, `torch.tpu.get_device_name()` queries `PjRtDevice::device_kind()` in `libtpu.so`, which returns the canonical hardware family strings (`"TPU v7"`, `"TPU v6 lite"`, `"TPU v5p"`, `"TPU v5 lite"`), while GKE / Ray node metadata uses SKU strings (`"TPU v7x"`, `"TPU7x"`, `"tpu7…

### #8556 — [[doc] feat: add intel as supported hardware in readme - Hardware Support](https://github.com/verl-project/verl/pull/8556)
- **作者**: kahlun  **时间**: 2026-11-05 12:07 CST
- **摘要**: ## Description This PR updates the main `README.md` to explicitly list Intel (XPU) hardware support.   While Intel support is currently provided via the `verl-hardware-plugin` extension, having it visible in the root README makes the project much more discoverable for first-time users or those speci…

### #8555 — [[data] fix: resolve relative tool config paths in RLHFDataset](https://github.com/verl-project/verl/pull/8555)
- **作者**: super-yuki  **时间**: 2026-11-05 09:56 CST
- **摘要**: Fixes #8554.  `RLHFDataset` passed `data.tool_config_path` to `load_all_tools` unchanged, while `AgentLoopWorker` resolves it with `resolve_config_path`. In a Ray setup where the actors' cwd differs from the driver's, the dataset failed to load the tools (`Failed to initialize tools ... [Errno 2]`) …

### #8553 — [[data] fix: reuse the overlong-prompt filter cache across launches](https://github.com/verl-project/verl/pull/8553)
- **作者**: super-yuki  **时间**: 2026-11-05 09:54 CST
- **摘要**: Fixes #8552.  `datasets.filter()` fingerprints the callable, and the filter lambda closes over the dataset object and processor, so the fingerprint changes between processes and every launch re-renders and re-tokenizes the full dataset (33k prompts, 13-16 min).  Pass an explicit stable `new_fingerpr…

### #8551 — [[tool] fix: keep `...` arguments as text in Qwen3 XML tool parser](https://github.com/verl-project/verl/pull/8551)
- **作者**: super-yuki  **时间**: 2026-11-05 09:52 CST
- **摘要**: Fixes #8550.  `ast.literal_eval("...")` returns `Ellipsis`, so `json.dumps(param_dict)` raised `TypeError` and the whole tool call was dropped. Keep the original string when the parsed literal is `Ellipsis`.  Test: `python -m pytest -q tests/experimental/agent_loop/test_qwen3_tool_parser_on_cpu.py` …

### #8548 — [[fsdp, algo] fix: preserve fp32 log probabilities for tail KL](https://github.com/verl-project/verl/pull/8548)
- **作者**: taking-lying-flat  **时间**: 2026-11-05 08:46 CST
- **摘要**: ### What does this PR do?  With BF16 logits, `forward_kl_topk_tail` can compute a negative student tail mass and reverse the logit gradient when `use_chunked_topk=True`. For logits `[0, 0, -9, -9]` and teacher probabilities `[0.45, 0.45, 0.05, 0.05]`, rounding the selected log-probabilities back to …

### #8547 — [[trainer, worker] feat: add MaxText (JAX) trainer backend for Qwen3-30B-A3B GRPO on ROCm](https://github.com/verl-project/verl/pull/8547)
- **作者**: yuankaichen-amd  **时间**: 2026-11-05 06:46 CST
- **摘要**: ### What does this PR do?  Adds a **MaxText (JAX) actor engine** to the v1 `separate_async` trainer, so Qwen3-30B-A3B can be trained with GRPO on AMD MI300X (ROCm) while vLLM serves rollouts from a separate node. The engine plugs in as `model_engine=maxtext`. It does not touch the FSDP or Megatron p…

### #8546 — [build(deps-dev): bump torchaudio from 2.11.0 to 2.11.1](https://github.com/verl-project/verl/pull/8546)
- **作者**: dependabot[bot]  **时间**: 2026-11-05 01:34 CST
- **标签**: dependencies, python
- **摘要**: Bumps [torchaudio](https://github.com/pytorch/audio) from 2.11.0 to 2.11.1. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/pytorch/audio/releases">torchaudio's releases</a>.</em></p> <blockquote> <h2>TorchAudio 2.11.1</h2> <p>This is a minor release with n…
