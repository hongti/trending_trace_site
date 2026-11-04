# verl-project/verl — 动态追踪

> 生成时间: 2026-11-04 14:55 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🚀 Pull Requests (PR)
本期 PR 主要集中在**新特性引入**、**Bug 修复**以及针对**Ascend（昇腾）硬件的适配与优化**。

**🌟 重要新特性与变更**
1. **支持 Ascend 平台训练 Kimi K3**（#8534）
   - 新增基于 FSDPTurbo 或 Megatron/MindSpeed 结合 vLLM-Ascend rollout 的 Kimi K3 GRPO 训练支持，涵盖 checkpoint 加载、权重同步及 ID-only R3。
2. **引入 `decoupled_async` Trainer 模式**（#8526）
   - V1 trainer 新增解耦异步模式：trainer 不再直接驱动 rollout，仅发布权重版本并将 prompts 放入 TransferQueue，由各 vLLM worker 主动拉取权重。
3. **支持后台权重更新等待**（#8530）
   - 新增 `CheckpointEngineManager.wait_for_weight_update()` 接口，允许 trainer 在验证或训练循环退出前等待后台权重更新完成。
4. **Ascend vLLM 软件栈升级**（#8525）
   - 将 Ascend vLLM 栈整体升级至 0.30.0 版本。

**🛠️ 重要修复**
1. **修复 vLLM 服务器启动报错**（#8529）
   - 修复了当 rollout worker 未分配到加速器时，`vLLMReplica.launch_servers` 因获取不到 accelerator ID 而抛出 `IndexError` 的问题。
2. **修复 Rollout 诊断数据污染**（#8528）
   - 修复了在多次本地更新期间，空响应批次可能导致 NaN 占位符污染 V1 rollout 诊断信息的问题。
3. **修复奖励模型 HTTP 请求无超时的问题**（#8527）
   - 为奖励模型的 HTTP 请求增加了可配置的超时时间，防止后端接受连接但不返回响应时陷入无限等待。
4. **Ascend CI 流水线优化**（#8531, #8532, #8533）
   - 为解决主机内存溢出（OOM）问题，将 Qwen3-8B PPO Ascend nightly CI 扩展至 16 个 NPU runner，并移除了该 workflow 的 PR 触发器。

### 📝 Issues
本期数据中无相关 Issue 动态。

### 📦 Release
本期数据中无相关 Release 动态。

---

## 🔀 Pull Requests

### #8534 — [[fsdp, megatron, vllm] feat: support Kimi K3 training on Ascend](https://github.com/verl-project/verl/pull/8534)
- **作者**: wangdongleix  **时间**: 2026-11-04 12:19 CST
- **摘要**: ### What does this PR do?  Support Kimi K3 GRPO training on Ascend with FSDPTurbo or Megatron/MindSpeed and vLLM-Ascend rollout, including checkpoint loading, weight synchronization, and ID-only R3.  Related: [FSDPTurbo integration #7362](https://github.com/verl-project/verl/pull/7362).  This extend…

### #8533 — [test ascend 16 runner](https://github.com/verl-project/verl/pull/8533)
- **作者**: lxb007981  **时间**: 2026-11-04 10:09 CST
- **摘要**: ### What does this PR do?  test ascend 16 runner

### #8532 — [[ci] fix: remove pr trigger for nightly Ascend workflow](https://github.com/verl-project/verl/pull/8532)
- **作者**: lxb007981  **时间**: 2026-11-04 10:04 CST
- **摘要**: ### What does this PR do?  Remove pr trigger for nightly Ascend workflow  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fs…

### #8531 — [[ci] fix: Scale Qwen3-8B PPO Ascend nightly CI to 16 NPUs](https://github.com/verl-project/verl/pull/8531)
- **作者**: lxb007981  **时间**: 2026-11-04 09:53 CST
- **摘要**: ### What does this PR do?  Switch to 16 NPU runners to work around host oom.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include…

### #8530 — [[ckpt, trainer] feat: let the trainer wait for a weight update that finishes in the background](https://github.com/verl-project/verl/pull/8530)
- **作者**: xiaoxlu  **时间**: 2026-11-04 07:38 CST
- **摘要**: ### What does this PR do?  Adds `CheckpointEngineManager.wait_for_weight_update()`, a no-op by default, and has the v1 trainer call it before validation and before the training loop exits.  Today `update_weights` returns only once the rollout holds the new weights. Some backends do not need to block…

### #8529 — [[rollout, vllm] fix: launch vLLM servers when rollout workers hold no accelerator](https://github.com/verl-project/verl/pull/8529)
- **作者**: xiaoxlu  **时间**: 2026-11-04 04:38 CST
- **摘要**: ### What does this PR do?  `vLLMReplica.launch_servers` takes the first accelerator id of every rollout worker (`get_accelerator_ids()[get_resource_name()][0]`). This raises `IndexError` when a worker holds no accelerator. On TPU, verl's rollout workers get no TPU, because a chip can belong to only …

### #8528 — [[trainer] fix: align V1 rollout diagnostics with token availability](https://github.com/verl-project/verl/pull/8528)
- **作者**: hectopascal  **时间**: 2026-11-04 04:30 CST
- **摘要**: ### What does this PR do?  Fixes #8504. With multiple local updates per parameter synchronization, an empty-response batch can contaminate rollout diagnostics with a placeholder NaN, while the final batch's validity flag can hide an earlier available nonfinite measurement.  Exclude unavailable value…

### #8527 — [[reward] fix: bound reward model HTTP requests with configurable timeouts](https://github.com/verl-project/verl/pull/8527)
- **作者**: joshuaburgen  **时间**: 2026-11-04 03:08 CST
- **摘要**: ### What does this PR do?  When a discriminative reward backend accepts a connection but never completes its response, `RewardLoopWorker._post_request()` waits indefinitely because it explicitly passes `ClientTimeout(total=None)`. Its timeout retry handler is never reached, so batch reward computati…

### #8526 — [[ckpt, trainer, rollout, vllm, doc] feat: decoupled_async trainer with rollout-pulled weights over one-sided reads](https://github.com/verl-project/verl/pull/8526)
- **作者**: Luosuu  **时间**: 2026-11-04 03:04 CST
- **摘要**: ### What does this PR do?  Adds `decoupled_async`, a v1 trainer mode in which the trainer never drives rollout. The trainer only publishes weight versions and enqueues prompts in TransferQueue. Each vLLM replica pulls new versions with one-sided reads and decides on its own when to load them, and a …

### #8525 — [[hardware, vllm, doc] chore: upgrade Ascend vLLM stack to 0.30.0](https://github.com/verl-project/verl/pull/8525)
- **作者**: renjunying2  **时间**: 2026-11-03 23:01 CST
- **摘要**: ### What does this PR do?  upgrade Ascend vLLM stack to 0.30.0  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `mega…
