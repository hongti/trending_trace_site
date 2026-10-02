# verl-project/verl — 动态追踪

> 生成时间: 2026-10-02 14:02 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 🐛 Issue
近期主要报告了 2 个与 Triton 融合内核相关的 Bug：
*   **Triton 融合内核崩溃**：当设置 `use_remove_padding=False` 时，Triton 后端输出的 log 概率和熵被展平为 `[B*S]` 而非预期的 `[B,S]` 形状，导致崩溃（#8089）。
*   **绕过 FSDP2 钩子**：在使用 untied weights（非共享权重）时，融合内核会绕过 FSDP2 的 LM-head 钩子（#8091）。

### 🔧 Pull Request
#### ✨ 新特性
*   **引入 FusedWorker 与拓扑架构 RFC**：作为拓扑声明式 API RFC (#7269) 的首个 PR，在 v1 trainer 中启用了 `FusedWorker` 容器，并支持 await 异步子 worker（#8092）。
*   **NCCL M2N 检查点传输机制**：新增基于 NCCL M2N（多对多）的权重加载与导出功能。包含为 Megatron 添加的 producer 端（导出本地权重分片，#8088）和为 vLLM 添加的 receiver 端（加载目标节点本地权重，#8087）。

#### 🛠 修复与优化
*   **修复 Triton 融合输出形状**：将 Triton 后端的输出形状从展平的 `[B*S]` 修正为 `[B,S]`，解决了上述 #8089 的崩溃问题（#8090）。
*   **修复多轮对话数据集 Bug**：修正了 `enable_thinking_default` 的 YAML 配置错误，并修复了当 chat template 在思考开启/关闭时生成 token 数量不一致导致的 loss mask 不匹配问题（#8086）。
*   **优化 Agent 循环上下文传递**：通过构造函数注入 `RolloutContext`，向自定义 agent loop 暴露每次 rollout 的运行时元数据，避免将运行时字段混入数据集参数中（#8085）。
*   **修复拼写错误**：修正了奖励循环注释和安装脚本中的拼写错误，纯文本修改无行为变更（#8084）。

### 🚀 Release
近期无新版本发布。

---

## 🐛 Issues

### #8091 — [[Bug] Fused kernels bypass FSDP2 LM-head hooks with untied weights](https://github.com/verl-project/verl/issues/8091)
- **作者**: kylemontgomery1  **时间**: 2026-10-02 09:06 CST
- **标签**: bug
- **摘要**: ### System Info  n/a  ### Information  - [x] The official example scripts - [ ] My own modified scripts  ### Tasks  - [x] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ ] My own task or dataset (give details below)  ### Reproduction  With `use_fused_kernels=True`…

### #8089 — [[Bug] Triton fused kernels crash with use_remove_padding=False due to flattened outputs](https://github.com/verl-project/verl/issues/8089)
- **作者**: kylemontgomery1  **时间**: 2026-10-02 07:26 CST
- **标签**: bug
- **摘要**: ### System Info  n/a  ### Information  - [x] The official example scripts - [ ] My own modified scripts  ### Tasks  - [x] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [ ] My own task or dataset (give details below)  ### Reproduction  `use_fused_kernels=True` with…

## 🔀 Pull Requests

### #8092 — [[single_controller] feat: enable FusedWorker in v1 trainer and await async sub-workers](https://github.com/verl-project/verl/pull/8092)
- **作者**: ETOgaosion  **时间**: 2026-10-02 12:46 CST
- **摘要**: > This is the **first PR** of the topology RFC [#7269](https://github.com/verl-project/verl/issues/7269) (Declarative Model-Topology API). It enables the FusedWorker container in the default v1 path as a prerequisite; follow-up PRs will introduce atomic RL workers and the declarative `topology:` pla…

### #8090 — [[model] fix: triton fused output shape](https://github.com/verl-project/verl/pull/8090)
- **作者**: kylemontgomery1  **时间**: 2026-10-02 08:45 CST
- **摘要**: ### What does this PR do?  When `use_fused_kernels=True`, the Torch backend returns log probabilities and entropy as `[B,S]`, while the Triton backend leaves them flattened as `[B*S]`. With `use_remove_padding=False`, this flat output causes an `IndexError` at `output.log_probs.shape[1]` in `FSDPEng…

### #8088 — [[ckpt, megatron] feat: export local shards through NCCL M2N](https://github.com/verl-project/verl/pull/8088)
- **作者**: dongha-yoon  **时间**: 2026-10-02 03:30 CST
- **摘要**: ### What does this PR do?  Adds the Megatron producer for the NCCL M2N checkpoint backend. It exports HF-named tensors from their owning training ranks with global shape, shard geometry, and expert-ownership metadata, avoiding TP/EP tensor-value gathers during conversion. The backend transfers these…

### #8087 — [[ckpt, rollout, vllm] feat: load destination-local NCCL M2N weights](https://github.com/verl-project/verl/pull/8087)
- **作者**: dongha-yoon  **时间**: 2026-10-02 02:36 CST
- **摘要**: ### What does this PR do?  Adds the vLLM receiver for the NCCL M2N backend's `rank_local_named_tensors` wire format. M2N has already partitioned these tensors for each destination worker, so this path loads them into local parameter storage without applying vLLM's normal TP slicing a second time. It…

### #8086 — [[data] fix: correct enable_thinking_default YAML value and loss mask mismatch](https://github.com/verl-project/verl/pull/8086)
- **作者**: ZhiliangWu  **时间**: 2026-10-02 02:23 CST
- **摘要**: ### What does this PR do?  Two bugs in `MultiTurnSFTDataset` cause a mismatch between tokenization and loss masking when the chat template produces different token counts for thinking ON vs OFF.  1. **Wrong YAML type**: `enable_thinking_default: none` in `sft_trainer_engine.yaml` is the string `"non…

### #8085 — [[rollout] fix: supply runtime context to agent loop constructors](https://github.com/verl-project/verl/pull/8085)
- **作者**: yueyiming2009  **时间**: 2026-10-02 01:18 CST
- **摘要**: ### What does this PR do?  Expose per-rollout runtime metadata to custom agent loops through constructor-supplied `RolloutContext`, without mixing runtime fields into dataset kwargs.  `AgentLoopWorker._run_agent_loop()` already knows the validation mode and dispatch step, but custom loops have no ex…

### #8084 — [[trainer, data, tool] fix: correct misspellings in reward loop comments and install script](https://github.com/verl-project/verl/pull/8084)
- **作者**: li-lizhe  **时间**: 2026-10-01 13:38 CST
- **摘要**: ### What does this PR do?  Fix misspellings in two source comments and two user-visible strings. Text-only change, no behavior change.  | File | Line | Was | Now | |---|---|---|---| | `verl/trainer/ppo/ray_trainer.py` | 912 | `initalize` | `initialize` | | `verl/experimental/separation/ray_trainer.p…
