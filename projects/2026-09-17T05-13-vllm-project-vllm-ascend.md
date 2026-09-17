# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-17 13:13 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 动态
近期收到 2 个关于 **GLM-5.3-Flash 量化模型**的严重 Bug 报告：
*   **#16754**：在麒麟 OS 和 HDK 环境下，部署 `GLM5.3-flash-w8a8` 0day 镜像时，服务在 KV cache 计算阶段卡死，部分卡死锁，AI Core 利用率为 0%，堆栈卡在 `_dummy_run` 的 `all_gather` 阶段。
*   **#16746**：在 950PR 上部署 `GLM-5.3-Flash-w8a8-mxfp8` 时，模型语言能力严重退化，对简单请求（如“你好”）陷入无限生成循环，不断重复输出陈词滥调。

---

### 🔧 Pull Request 动态

#### 1. 新特性与功能支持
*   **#16743 [FusedMoE]**：在 Ascend FusedMoE 路径中支持 **Zero-Compute Expert (ZCE)** "zero" 路由类型，并修复了 KV-delivery 抢占机制中过时的 `should_advance()` 调用。

#### 2. 性能优化
*   **#16753 [Performance]**：将 MLA DCP decode merge 路由至融合算子 `sfa_dcp_a2a_fused`，替代了原本冗长的 fp32 cast -> cat -> permute -> all_to_all 链路，提升执行效率。
*   **#16751 [Performance][Triton]**：在 Ascend A5 上对 SFA DCP8 pack 和 combine 操作进行批处理（一次处理 8 行），大幅减少了标量和内存传输开销。
*   **#16747 [Performance][KV Pool]**：为混合注意力 cache 布局增加 Mooncake 层级 KV pool 支持，并针对长共享前缀（long shared-prefix）工作负载优化了层级数据路径。

#### 3. 重要 Bug 修复
*   **#16745 [Bugfix][SpecDecode]**：修复了在 Ascend V2 上使用混合 Full/SWA DFlash cache 时，Qwen3.6 类模型投机解码输出损坏及 draft acceptance 崩溃的问题。
*   **#16750 [BugFix]**：修复了在 Model Runner v2 中启用 `PIECEWISE` 时，`num_input_tokens` 计算错误的问题。
*   **#16752 [BugFix][KV Pool]**：修复了 `touch_sending_mamba_blocks` 在保存时错误固定 mamba 投机解码 scratch blocks 的问题。

#### 4. CI 与代码清理
*   **#16749**：为 MRV2 新增 eplb nightly v2 CI 测试。
*   **#16748**：恢复 v0.27.1 外部 Qwen MoE gate 路径用于验收控制诊断（注：仅为诊断草稿，不合并）。
*   **#16744**：清理了 `ScoreEncoderCacheManager.reset()` 中继承自父类但已失效的 `freeable` 清理逻辑。

---

### 🚀 Release 动态
本期未发布正式 Release 版本。但通过 PR #16752 可以看出，团队正在积极维护 **v0.27.1rc** 候选版本，并将针对 Mamba 投机解码的 Bug 修复 Backport 到该分支，预示着 v0.27.1 稳定版即将推出。

---

## 🐛 Issues

### #16754 — [[Bug]: GLM5.3 flash 0day镜像部署GLM5.3-flash-w8a8，服务在kv cache计算阶段卡死](https://github.com/vllm-project/vllm-ascend/issues/16754)
- **作者**: delwen123  **时间**: 2026-09-17 12:56 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  麒麟操作系统（4.19.90-52.22.v2207.ky10.aarch64） HDK（26.0.rc1 / 25.5.6） GLM5.3flash 0day镜像  ### 🐛 Describe the bug  模型卡在kv cache计算阶段，部分卡卡死，且观察到aicore利用率为0，堆栈卡在_dummy_run的all_gather阶段，其余卡aicore利用率很高，堆栈卡在_dummy_sampler_run，最后出现allgather等超时告警，服务始终无法拉起。

### #16746 — [[Bug]: GLM-5.3-Flash-w8a8-mxfp8 on 950PR language capability degraded, outputting repetitive clichés in a loop.](https://github.com/vllm-project/vllm-ascend/issues/16746)
- **作者**: meehom  **时间**: 2026-09-17 11:40 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: openEuler 24.03 (LTS-SP3) (aarch64) GCC version: (GCC) 12.3.1 (openEuler 12.3.1-105.oe2403sp3) Clang …

## 🔀 Pull Requests

### #16753 — [[Performance] Route MLA DCP decode merge through fused sfa_dcp_a2a_fused](https://github.com/vllm-project/vllm-ascend/pull/16753)
- **作者**: Levi-JQ  **时间**: 2026-09-17 12:52 CST
- **摘要**: ### What this PR does / why we need it? The unsplit decode path in `AscendMlaDCPImpl._forward_decode` currently merges DCP shards with a long legacy chain: fp32 cast → cat → permute → `all_to_all_single` → permute → split/unbind → `npu_attention_update`. This launches ~20 small kernels per attention…

### #16752 — [[BugFix][KV Pool] Don't pin mamba speculative scratch blocks on save](https://github.com/vllm-project/vllm-ascend/pull/16752)
- **作者**: bowgneo  **时间**: 2026-09-17 12:49 CST
- **标签**: module:tests
- **摘要**: Backport of #16329 to `releases/v0.27.1rc`.  ## Problem  `touch_sending_mamba_blocks` pins every non-null block of the mamba group's block table for async sends — including the trailing speculative scratch blocks. After vllm PR #51358, `_relocate_speculative_block` requires speculative blocks to be …

### #16751 — [[Performance][Triton] Batch SFA DCP8 pack and combine rows on A5](https://github.com/vllm-project/vllm-ascend/pull/16751)
- **作者**: pisceskkk  **时间**: 2026-09-17 12:44 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  SFA DCP8 pack and combine process one row per loop iteration, leaving substantial scalar and memory-transfer overhead on Ascend A5. This change batches eight independent rows while streaming DCP ranks sequentially, preserving the FP32 LSE encoding and accumul…

### #16750 — [[BugFix] Correct num_input_tokens when using PIECEWISE](https://github.com/vllm-project/vllm-ascend/pull/16750)
- **作者**: slippersss  **时间**: 2026-09-17 12:28 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? This PR aims to fix `num_input_tokens` when enabling `PIECEWISE` in model runner v2 for sfa. Since sfa regards `num_input_tokens` as a padded one while it is actually not under `PIECEWISE`, it incurs shape mismatch in ops.  ### Does this PR introduce _any_ use…

### #16749 — [[CI][MRV2] eplb nightly v2](https://github.com/vllm-project/vllm-ascend/pull/16749)
- **作者**: Spicy-Stick  **时间**: 2026-09-17 12:04 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c

### #16748 — [[Test][MoE] Restore v0.27.1 external Qwen gate for acceptance control](https://github.com/vllm-project/vllm-ascend/pull/16748)
- **作者**: zhao-stack  **时间**: 2026-09-17 11:49 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  **Draft diagnostic only. Do not merge.** Restore the vLLM v0.27.1 Qwen MoE gate path on the current Ascend baseline to check whether the original external/native gate also fails the unchanged DSpark acceptance E2E. Related experiments: #16715 (FP32 router con…

### #16747 — [[Performance][KV Pool] Optimize Mooncake hybrid layerwise transfer](https://github.com/vllm-project/vllm-ascend/pull/16747)
- **作者**: Eric-dot  **时间**: 2026-09-17 11:46 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  This PR adds Mooncake layerwise KV pool support for hybrid-attention cache layouts and optimizes the layerwise data path for long shared-prefix workloads such as DeepSeek-V4 Flash.  - Build group-aware Mooncake range sessions for multiple KV cache groups, inc…

### #16745 — [[Bugfix][SpecDecode] Fix corrupted output and draft acceptance collapse with mixed Full/SWA DFlash cache on Ascend V2](https://github.com/vllm-project/vllm-ascend/pull/16745)
- **作者**: sunny-rain-63  **时间**: 2026-09-17 11:22 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  **Bug description.** When running Qwen3.6-style models with DFlash speculative decoding whose draft config mixes `full_attention` and `sliding_attention` layers on the Ascend V2 runner, generation degrades into corrupted (garbled) text and the draft acceptanc…

### #16744 — [[Ops][Misc] Remove dead legacy freeable clear inherited from parent in reset()](https://github.com/vllm-project/vllm-ascend/pull/16744)
- **作者**: joeqth  **时间**: 2026-09-17 11:20 CST
- **摘要**: ### What this PR does / why we need it?  `ScoreEncoderCacheManager.reset()` clears `self.freeable`, an attribute inherited from upstream `EncoderCacheManager` (`vllm/v1/core/encoder_cache_manager.py`). In this subclass the inherited single-level `freeable` dict is never populated:  - every method th…

### #16743 — [[FusedMoE] Support zero expert type and fix structured output call](https://github.com/vllm-project/vllm-ascend/pull/16743)
- **作者**: ninghuang00  **时间**: 2026-09-17 10:43 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it? Enables **Zero-Compute Expert (ZCE)** "zero" routing type in the Ascend fused-MoE path and fixes a stale `should_advance()` call in the KV-delivery preemption scheduler. Together with the vLLM-side model patch for ZEDA-GLM-4.7-Flash-Dynamic (`Glm4MoeLitePlusPl…
