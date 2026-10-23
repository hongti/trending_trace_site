# verl-project/verl — 动态追踪

> 生成时间: 2026-10-23 13:13 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue
1. **#8346 `prime_math` 奖励函数评分逻辑错误**：作者 jimmy-holdout 指出，该函数对错误的区间括号、缩放数值（如 100 倍、1/100 倍）及差一（off-by-one）数字错误地给予满分，却对许多正确的方程答案给出 0 分。
2. **#8342 [RFC] VeOmni 端到端低精度训练**：作者 amenomori 提议实现 VeOmni 路线，采用 FSDP2 + torchao，在模型中分片 FP32 主权重，并使用标准优化器。

### 🔧 Pull Request (PR)
**奖励与特性**
* **#8347 修复 `prime_math` 评分错误**：针对 Issue #8346，修正了评分逻辑，拒绝错误的区间括号和缩放数字。
* **#8345 增强沙箱灵活性**：新增特性，允许其他包为 `sandbox_fusion` 奖励代码注册并提供沙箱环境来运行测试用例（不再局限于 Sandbox Fusion 服务自身）。

**Bug 修复**
* **#8341 修复异步客户端字段丢失**：修复了 `FullyAsyncLLMServerClient` 在恢复运行时丢失 `backend extra_fields` 的问题。
* **#8340 动态调整 TransferQueue 存储单元**：修复了存储单元数量固定为 8 导致的问题，现根据 `trainer.nnodes` 动态调整大小。
* **#8338 修复 Megatron 内存管理**：修复了重计算反向传播中由于错误释放其他张量仍在使用的 GPU 存储导致的非法内存访问崩溃问题。

**CI 与 NPU/Ascend 适配**
* **#8336 解除 NPU 单元测试阻塞**：修复 `MegatronAdaptor` 误读 pytest CLI 标志并生成无效 dataclass 的问题，在保留 GPU 测试行为的同时解除 NPU CI 阻塞。
* **#8337 清理 Ascend 测试残留进程**：在 GSM8K 和 GEO3K 测试（包括失败情况）后清理孤立的 vLLM 进程并停止 Ray，以释放 NPU 资源。
* **#8344 临时调整 CI 参数**：因 fsdp-turbo 后端代码重构，临时将 `SP_SIZE` 设为 1。
* **#8343 记录 Ascend 环境信息**：在 FSDP2 模型任务中增加驱动和 CANN 版本信息的日志记录。
* **#8339 添加 Ascend 探针**：添加探针以调试主机 OOM（内存溢出）问题。

### 🚀 Release
* 近期无新版本发布。

---

## 🐛 Issues

### #8346 — [prime_math reward gives full credit to wrong intervals and to 100x, 1/100x and off-by-one numbers, and gives 0 to many correct equation answers](https://github.com/verl-project/verl/issues/8346)
- **作者**: jimmy-holdout  **时间**: 2026-10-23 13:05 CST
- **摘要**: ### System Info  - verl `main` at `d430157d15516d89e0cc853fd10fcd21d9234c44` (`verl.__version__` = `0.10.0.dev`). The counts below were first measured at `dc709afb7a74fd2645640a4e5f2fdf40166b4fbb`. `verl/utils/reward_score/prime_math/` has not changed between the two commits, and the results are the…

### #8342 — [[RFC][VeOmni] End-to-end low-precision training](https://github.com/verl-project/verl/issues/8342)
- **作者**: amenomori  **时间**: 2026-10-23 09:56 CST
- **摘要**: ## Feature request  Implement the VeOmni route in [#8186](https://github.com/verl-project/verl/issues/8186): **FSDP2 + torchao, sharded FP32 master weights in the model, and standard optimizers.**  | Scope | Delivery | | --- | --- | | Native VeOmni QAT | DeepSeek-V4/V4.1 training → export/refit → ro…

## 🔀 Pull Requests

### #8347 — [[reward] fix: reject wrong interval brackets and scaled numbers in prime_math](https://github.com/verl-project/verl/pull/8347)
- **作者**: jimmy-holdout  **时间**: 2026-10-23 13:06 CST
- **摘要**: ### What does this PR do?  Fixes #8346.  This PR fixes grading errors in `verl/utils/reward_score/prime_math`, which scores the `numina_*` math rows of Eurus-2-RL-Data. Exactly these behaviors change:  1. **Interval brackets.** An interval with a wrong endpoint bracket no longer gets credit. For exa…

### #8345 — [[reward] feat: run sandbox_fusion test cases through sandboxes other packages register](https://github.com/verl-project/verl/pull/8345)
- **作者**: danbugs  **时间**: 2026-10-23 11:02 CST
- **摘要**: ### What does this PR do?  Lets another package provide the sandbox that `sandbox_fusion` code rewards run their test cases in. Today `reward.sandbox_fusion.url` can only name a Sandbox Fusion service over HTTP. With this PR, a URL with another scheme (`<scheme>://...`) goes to a runner that a packa…

### #8344 — [[ci] fix: set SP_SIZE=1 temporarily due to the code refactoring of fsdp-turbo backend](https://github.com/verl-project/verl/pull/8344)
- **作者**: pengnuoheng  **时间**: 2026-10-23 10:39 CST
- **摘要**: ### What does this PR do?  set SP_SIZE=1 temporarily due to the code refactoring of fsdp-turbo backend  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the C…

### #8343 — [[ci] chore: log driver and CANN info in Ascend FSDP2 model job](https://github.com/verl-project/verl/pull/8343)
- **作者**: lxb007981  **时间**: 2026-10-23 10:00 CST
- **摘要**: ### What does this PR do?  log driver and CANN info in FSDP2 model job.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsd…

### #8341 — [[rollout, fully_async] fix: keep backend extra_fields across FullyAsyncLLMServerClient resumes](https://github.com/verl-project/verl/pull/8341)
- **作者**: linyueqian  **时间**: 2026-10-23 08:21 CST
- **摘要**: ### What does this PR do?  `FullyAsyncLLMServerClient.generate` rebuilds every response into a fresh `TokenOutput`. It copies back only the keys it merges itself: the top-k heads, `num_cached_tokens` and the `*_global_steps` keys. Every other key that the backend put in `extra_fields` is dropped on …

### #8340 — [[trainer, cfg] fix: size TransferQueue storage units from trainer.nnodes](https://github.com/verl-project/verl/pull/8340)
- **作者**: nataliekung  **时间**: 2026-10-23 02:51 CST
- **摘要**: ### What does this PR do?  Fixes #7398. `transfer_queue.backend.SimpleStorage.num_data_storage_units` defaulted to a fixed 8. Each `SimpleStorageUnit` and the `TransferQueueController` reserve 1 CPU, and `TaskRunnerV1.run` always sets `transfer_queue.enable = True`, so every default PPO run reserves…

### #8339 — [[ci] chore: add Ascend reproduction probe](https://github.com/verl-project/verl/pull/8339)
- **作者**: lxb007981  **时间**: 2026-10-22 21:26 CST
- **摘要**: ### What does this PR do?  debug host oom issue  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`…

### #8338 — [[megatron] fix memory management in overrided recompute backward](https://github.com/verl-project/verl/pull/8338)
- **作者**: HydrogenSulfate  **时间**: 2026-10-22 19:15 CST
- **摘要**: ## What does this PR do?  Stops verl's recompute-backward patch from freeing GPU storage that other tensors still use. Today that makes `recompute_granularity='full'` crash with an illegal memory access whenever a tensor is fed to more than one checkpointed chunk. The fix keeps the MoE memory reclai…

### #8337 — [[ci] fix: clean up orphaned vLLM processes after Ascend E2E tests](https://github.com/verl-project/verl/pull/8337)
- **作者**: lxb007981  **时间**: 2026-10-22 17:44 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Stop Ray and kill remaining VLLM processes after both GSM8K and GEO3K runs, including on failure, to release NPU resources for subsequent tests. Log NPU status after cleanup.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: …

### #8336 — [[ci] fix: unblock NPU unit tests](https://github.com/verl-project/verl/pull/8336)
- **作者**: dodatboii  **时间**: 2026-10-22 17:25 CST
- **摘要**: ### What does this PR do?  Unblocks NPU CI while preserving GPU test behavior. MegatronAdaptor misinterprets pytest CLI flags and generates an invalid dataclass constructor; the NPU image has a newer ModelOpt version than GPU CI and exposes profiler enum rebinding.  ### Checklist Before Starting  - …
