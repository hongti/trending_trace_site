# vllm-project/vllm — 动态追踪

> 生成时间: 2026-10-15 13:02 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 📌 Issue 动态
本次更新周期内，未提供相关的 Issue 动态。

### 🚀 Release 动态
本次更新周期内，未提供新的版本发布（Release）信息。

### 🔧 Pull Request (PR) 动态
本次 PR 更新主要涵盖新特性、性能优化、重要 Bug 修复以及基础设施维护：

**1. 新特性与功能**
* **vLLM 硬件感知模拟器** (#61747)：新增 `tools/vllm-simulator`，可在仅 CPU 的主机上运行真实的 vLLM v1 引擎并绕过模型前向传播，通过模拟时钟推进来评估硬件感知服务。
* **DeepSeek-V4.1 NIXL P/D 支持** (#61746)：支持在 NIXL Prefill 生产者和 PCP=1 Decode 上使用 PCP Prefill，实现纯文本 DeepSeek-V4.1-Flash 的分离式推理（P/D）服务。

**2. 性能优化**
* **Kernel 性能提升** (#61753)：为 `rms_norm_dynamic_per_token_quant` 引入寄存器驻留路径。原先该操作需固定线程块读取 token 三次，优化后显著提升了 prefill 阶段的性能。

**3. 重要 Bug 修复**
* **前端 API 修复** (#61752)：修复了在 Responses API (`/v1/responses`) 中，同时使用 `structured_outputs` 和 `text.format: {"type": "text"}` 时被错误拒绝的问题。
* **多模态异常处理** (#61751)：当设置了本地媒体路径，若 `file://` URL 指向缺失文件或目录时，错误状态码从 500 修正为合理的 400。
* **MoE 算子输出乱码修复** (#61749)：修复了 `torch.compile` 下 Nemotron-3-Nano-30B-A3B-NVFP4 输出乱码的问题，要求 `moe_forward` 自定义算子必须使用精确步长。
* **监控指标修复** (#61750, #61748)：修复了衍生 API worker 退出时未能清理 Prometheus 实时 gauges 的问题；并修复了快照中 Prometheus 计数器小数值被截断的问题（避免 KV 卸载时间等指标丢失精度）。

**4. 基础设施与依赖**
* **CI 维护** (#61745)：增加 Intel XPU 关闭测试的超时时间，解决 `tensor_parallel_size=2` 场景下间歇性超时失败的问题。
* **依赖更新** (#61744)：更新 DeepGEMM 依赖版本。

---

## 🔀 Pull Requests

### #61753 — [[Kernel][Perf] Register-resident path for rms_norm_dynamic_per_token_quant](https://github.com/vllm-project/vllm/pull/61753)
- **作者**: samridh10exe  **时间**: 2026-10-15 12:40 CST
- **标签**: quantization
- **摘要**: ## Overview  `rms_norm_dynamic_per_token_quant` reads each token three times (RMS, scale, quantize) with a fixed `min(hidden_size, 1024)`-thread block, so at prefill sizes it is slower than `fused_add_rms_norm` + `scaled_int8_quant` (#60499). With at least two tokens per SM, launch a new kernel that…

### #61752 — [[Bugfix][Frontend] Allow structured_outputs with text.format type "text" in Responses API](https://github.com/vllm-project/vllm/pull/61752)
- **作者**: sungbin1015  **时间**: 2026-10-15 12:16 CST
- **标签**: bug, frontend
- **摘要**: ## Overview  On `/v1/responses`, sending vLLM's `structured_outputs` (choice, regex, grammar) together with `text: {"format": {"type": "text"}}` is rejected with "Cannot specify both structured_outputs and text.format". `{"type": "text"}` is the plain, unconstrained format, and Chat Completions alre…

### #61751 — [[Bugfix][Multimodal] Return 400 for file URLs that do not point to a file](https://github.com/vllm-project/vllm/pull/61751)
- **作者**: sungbin1015  **时间**: 2026-10-15 12:16 CST
- **标签**: bug, multi-modality
- **摘要**: ## Overview  With `--allowed-local-media-path` set, a `file://` URL inside the allowed directory that points to a missing file (or to a directory) fails the request with a 500. The path comes straight from the request, so it should be a client error like the "must be a subpath" check right above it.…

### #61750 — [[Bugfix][Metrics] Clean up live gauges in spawned API workers](https://github.com/vllm-project/vllm/pull/61750)
- **作者**: bojiang-li  **时间**: 2026-10-15 11:55 CST
- **标签**: bug
- **摘要**: ## Overview  Clean up the exiting API worker's live Prometheus gauges using `PROMETHEUS_MULTIPROC_DIR`. Spawned workers inherit this path but not the parent's `TemporaryDirectory` object, so their current shutdown returns without marking the process dead.  ## Claims  - Remove only the current proces…

### #61749 — [[Bugfix][MoE] Require exact strides for moe_forward custom ops](https://github.com/vllm-project/vllm/pull/61749)
- **作者**: ldkhang1201  **时间**: 2026-10-15 11:43 CST
- **标签**: bug, nvidia
- **摘要**: ## Overview  Fixes #52308: `nvidia/NVIDIA-Nemotron-3-Nano-30B-A3B-NVFP4` outputs garbage under `torch.compile` with the FlashInfer TRTLLM NVFP4 MoE backend. `--enforce-eager` works. The fix changes the tag on `vllm.moe_forward` and `vllm.moe_forward_shared` from `needs_fixed_stride_order` to `needs_…

### #61748 — [[Metrics] Preserve fractional counter values in snapshots](https://github.com/vllm-project/vllm/pull/61748)
- **作者**: bojiang-li  **时间**: 2026-10-15 11:30 CST
- **摘要**: ## Overview  Preserve fractional Prometheus counter values in `get_metrics_snapshot()`. The current integer conversion truncates duration counters such as KV offload load and store times, making a value of 0.125 seconds appear as zero through the metrics reader.  ## Claims  - Return the original cou…

### #61747 — [[Simulator] Add vLLM Simulator for hardware-aware serving simulation on CPU](https://github.com/vllm-project/vllm/pull/61747)
- **作者**: zhuzi-z  **时间**: 2026-10-15 11:30 CST
- **摘要**: ## Overview  Adds `tools/vllm-simulator`, the simulator half of RFC #61406. It runs the real vLLM v1 engine on a CPU-only host and bypasses model forward. A simulation clock then advances by a hardware-aware per-step latency prediction. The output is the same set of metrics `vllm bench serve` report…

### #61746 — [[Feature][DSv4.1][NIXL] Support PCP Prefill with NIXL P/D](https://github.com/vllm-project/vllm/pull/61746)
- **作者**: wangyicong52  **时间**: 2026-10-15 11:28 CST
- **标签**: documentation, ci/build, deepseek, kv-connector, kv-cache-manager, DSv4.1, mooncake
- **摘要**: ## Overview  Enable text-only DeepSeek-V4.1-Flash P/D serving with PCP on the NIXL Prefill producer and PCP=1 on Decode.  Stacked on #59857, #59325 and #59330. Their commits are included as attributed prerequisites; only the final commit, `a888e22d83`, implements this feature's model capability gate…

### #61745 — [[CI][XPU] Increase shutdown test timeout for XPU](https://github.com/vllm-project/vllm/pull/61745)
- **作者**: chaojun-zhang  **时间**: 2026-10-15 11:23 CST
- **标签**: rocm, intel-gpu
- **摘要**: ## Purpose  `tests/v1/shutdown` intermittently fails on Intel CI (XPU) with `Failed: Timeout (>120.0s) from pytest-timeout`, e.g. in various `tensor_parallel_size=2` cases of `test_forward_error.py`/`test_startup_error.py` (see https://buildkite.com/vllm/intel-ci/builds/14158, 14130, 14120).  XPU fo…

### #61744 — [Update DeepGEMM commit](https://github.com/vllm-project/vllm/pull/61744)
- **作者**: jeejeelee  **时间**: 2026-10-15 11:16 CST
- **标签**: ci/build
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…
