# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-27 13:38 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### Issue
近期暂无新增 Issue 动态。

### Release
近期无新版本发布。

### Pull Request (PR)
近期共合并/提交 10 个 PR，主要涉及投机解码支持、Bug 修复、性能优化及测试完善：

**1. 新特性**
*   **#17557**：在 Model Runner V2 上支持 GLM-5.3-Flash 与 DFlash2 的投机解码，针对其 mHC 模型架构提供了目标端支持。

**2. 性能优化**
*   **#17555**：缩小 `AscendStoreConnector` 中 KV save fence 的范围，使其仅针对有释放 block 的请求执行，避免无脑阻塞调度器，从而提升性能。

**3. Bug 修复**
*   **#17556**：修复 `NPUWorker` 未调用上游 `warmup_kernels` 导致 V2 Triton kernels 未编译的问题，通过 `enable_jit_warmup` 守卫移植了 V2 kernel 预热逻辑。
*   **#17554**：清理 Ascend DFlash 输入 kernel 中未使用的死代码 `BLOCK_SIZE` 常量。
*   **#17552**：修复 GLM-5.3-Flash NoPE 检查点在 `qk_rope_head_dim=0` 时，sparse indexer 仍构建零宽旋转嵌入导致的错误。
*   **#17551**：修复混合层加载失败时在 Worker 中引发异常的问题（因 group-local block ID 无法在调度器中安全识别失败请求）。
*   **#17549**：恢复 GLM5.2 DSpark 的 `seq_lens` 选择逻辑。

**4. 测试与 CI**
*   **#17553 & #17550**：针对 Qwen3.6-35B-A3B DSpark 投机解码补充测试，在 MT-Bench 上测量平均接受长度，并验证 graph 与 eager 模式下的接受率。
*   **#17548**：通过添加空行重新触发 Qwen3.6 DSpark E2E 测试的 CI 流程，以排查日常运行中的接受率失败问题。

---

## 🔀 Pull Requests

### #17557 — [[Feature][Model] Support GLM-5.3-Flash DFlash2 speculative decoding on model runner V2](https://github.com/vllm-project/vllm-ascend/pull/17557)
- **作者**: tanjiangshan  **时间**: 2026-09-27 11:46 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Enables GLM-5.3-Flash + DFlash2 speculative decoding on Model Runner V2 with vLLM 0.30.  **Target-side support**  GLM-5.3-Flash is an mHC model: between decoder layers flows the *deferred* `hc_post` state (raw FFN output + n residual streams + post/comb weigh…

### #17556 — [[BugFix][Worker] Port V2 kernel warmup with enable_jit_warmup guard](https://github.com/vllm-project/vllm-ascend/pull/17556)
- **作者**: mashuiping  **时间**: 2026-09-27 11:19 CST
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  `NPUWorker.compile_or_warm_up_model` never calls upstream `warmup_kernels`. That function runs dummy prefill and decode steps so the V2 Triton kernels compile at startup. Without it, both phases compile on the first real request.  Cold cache, 8 × 910B, DeepSee…

### #17555 — [[Performance]narrow the KV save fence to requests with released blocks](https://github.com/vllm-project/vllm-ascend/pull/17555)
- **作者**: luoxiaolin712  **时间**: 2026-09-27 11:15 CST
- **标签**: module:tests
- **摘要**: ## 1. Baseline and conclusion  - Baseline: the current save fence in `AscendStoreConnector`. `handle_preemptions()` calls `KVPoolWorker.wait_for_previous_save()`, which blocks the scheduler step until the **entire previous save batch** has drained — i.e. every in-flight put of every request, whether…

### #17554 — [[BugFix][SpecDecode] Remove dead BLOCK_SIZE constexpr from Ascend DFlash input kernel](https://github.com/vllm-project/vllm-ascend/pull/17554)
- **作者**: mashuiping  **时间**: 2026-09-27 11:06 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `_prepare_dflash_inputs_kernel_ascend` declares `BLOCK_SIZE: tl.constexpr` and never reads it. Block 0 walks tokens in a scalar loop; the runtime `block_size` is only the KV-cache block size. Other blocks return immediately.  Upstream still passes `BLOCK_SIZE…

### #17553 — [[Test] Measure Qwen3.6 DSpark acceptance length on MT-Bench prompts](https://github.com/vllm-project/vllm-ascend/pull/17553)
- **作者**: drslark  **时间**: 2026-09-27 10:51 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  1. Run the Qwen3.6-35B-A3B DSpark E2E with the shared 40 MT-Bench prompts through `_run_speculative_decoding`, and report mean acceptance length. 2. Print all seven per-position acceptance rates before the assertion. Keep the deterministic HCCL, LCCL, and mat…

### #17552 — [[BugFix][Model] Skip zero-width GLM5-Next indexer RoPE for NoPE](https://github.com/vllm-project/vllm-ascend/pull/17552)
- **作者**: Wyz-134  **时间**: 2026-09-27 09:20 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  GLM-5.3-Flash NoPE checkpoints set `qk_rope_head_dim=0`. The sparse indexer currently constructs a zero-width rotary embedding anyway, and its interleaved cache reshape aborts model initialization. Construct the indexer RoPE only when the rotary dimension is …

### #17551 — [[BugFix][KV Pool] Recompute failed hybrid layerwise loads](https://github.com/vllm-project/vllm-ascend/pull/17551)
- **作者**: bowgneo  **时间**: 2026-09-27 01:04 CST
- **标签**: module:tests, module:core
- **摘要**: ## Purpose A layerwise AscendStore load failure on a hybrid (multi-group) model currently raises in the worker because group-local block IDs cannot safely identify the failed request in the scheduler. Report the failed request instead, so it can be restarted from scratch rather than terminating the …

### #17550 — [[Test] Exercise Qwen3.6 DSpark graph and eager acceptance](https://github.com/vllm-project/vllm-ascend/pull/17550)
- **作者**: drslark  **时间**: 2026-09-26 21:53 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Keep the requested HCCL, LCCL, and matmul environment settings for `test_qwen36_35b_dspark_spec_decoding`. Apply the model chat template to all prompts and print every speculative-position acceptance rate before the baseline assertion.  Run the same test with…

### #17549 — [[BugFix] Restore GLM5.2 DSpark seq_lens selection](https://github.com/vllm-project/vllm-ascend/pull/17549)
- **作者**: drslark  **时间**: 2026-09-26 21:47 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Reverts the revert in #17525 and restores the GLM5.2 DSpark sequence-length selection from #17184. The change is limited to the five files modified by #17525. The restored CPU sequence-length branch applies to GLM-family DSpark models. Qwen3.6 DSpark continue…

### #17548 — [[CI] Retrigger Qwen3.6 DSpark E2E](https://github.com/vllm-project/vllm-ascend/pull/17548)
- **作者**: drslark  **时间**: 2026-09-26 21:16 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Add one blank line to the Qwen3.6 DSpark E2E test to retrigger CI for the acceptance rate failure seen in the daily run. The test parameters, golden values, and assertions are unchanged.  ### Does this PR introduce any user-facing change?  No.  ### How was th…
