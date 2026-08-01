# verl-project/verl — 动态追踪

> 生成时间: 2026-08-01 09:04 CST

## AI 总结

# verl-project/verl 近期动态摘要

---

## 🐛 Issue

| 编号 | 标题 | 要点 |
|------|------|------|
| #7216 | E2E 确定性训练测试失败 | 新引入的"全确定性可复现 RL 训练"特性在端到端测试中未实现逐位对齐（bitwise-aligned），与预期效果不符 |
| #7213 | `del_local_ckpt_after_load` 未删除已加载的检查点 | FSDP 和 Megatron 检查点管理器在共享本地路径下加载后未清理文件，导致存储空间持续占用 |

---

## 🔀 Pull Request

### 🚀 新特性
- **#7212** — **新增 TPU 设备的 SFT 支持**：基于 TorchTitan 引擎 + 静态序列打包（static sequence packing），使 verl 在 Google TPU 上可运行监督微调
- **#7218** — **Kimi k2.5 模型适配**：更新 Megatron-Bridge / Megatron-LM / MindSpeed 子模块，并移除模型转换中的冗余逻辑

### ⚠️ 破坏性变更
- **#7215** — **[BREAKING] FSDP 优化器选择性权重衰减**：修复此前对所有可训练参数统一施加全局 weight decay 的问题，改为按参数选择性应用，默认策略为 `standard`。涉及 FSDP + trainer 路径，**可能影响已有训练收敛行为**

### 🔧 Bug 修复
- **#7214 / #7209** — **检查点清理修复**（对应 #7213）：确保所有 rank 完成加载后，由 rank 0 删除共享角色检查点目录，再进行二次同步清理；修复 FSDP 和 Megatron 两条路径
- **#7217** — **Agent Loop 修复**：将 `validate` 标志正确传播至 agent-loop 的两条后处理路径，并附加到异步奖励的 `DataProto.meta_info`，新增 CPU 回归测试
- **#7220** — **Tokenizer/Processor 构建修复**：PPO 入口改为从 `HFModelConfig` 构建 tokenizer/processor，与 SFT trainer 保持一致
- **#7211** — **多进程大结果超时修复**：当被装饰函数返回值超过管道缓冲区大小时会产生虚假超时，现改为在 join 子进程前先排空结果队列
- **#7210** — **HDFS 拷贝超时强制执行**：HDFS 命令在 POSIX 系统上运行于独立进程组，超时/中断时正确终止进程组

### 🧹 维护
- **#7219** — 移除 Gemini Code Review CI：因 GitHub 上的 Gemini Code Assist 消费版已下线

---

## 📦 Release

> 本周期内无新版本发布。

---

**总结**：本期重点在于**基础设施稳定性**——检查点清理（3 个 PR）、多进程/HDFS 超时处理等可靠性修复占据主导；同时 **TPU SFT 支持**和 **Kimi k2.5 适配**是重要的平台/模型扩展；**FSDP 选择性权重衰减**为破坏性变更，升级需留意收敛行为变化。

---

## 🐛 Issues

### #7216 — [[Bug] E2E determinism training test is failing (not bitwise-aligned)](https://github.com/verl-project/verl/issues/7216)
- **作者**: JoshuaL3000  **时间**: 2026-07-31 14:15 CST
- **标签**: bug
- **摘要**: Hi, we were very excited to hear of the new feature to enable full determinism for reproducible RL training and decided to try it out. We installed from source and tried out the tests scripts but we noticed that the e2e tests fails because the results were not bitwise-aligned from step 2 onwards. We…

### #7213 — [[Bug] del_local_ckpt_after_load does not remove loaded checkpoints](https://github.com/verl-project/verl/issues/7213)
- **作者**: yuyz-cyber  **时间**: 2026-07-31 13:15 CST
- **摘要**: ### System Info  - verl: current `main` (`aebd1f8a`) - Affected code: FSDP and Megatron checkpoint managers - Storage topology: the shared local checkpoint path supported by trainer resume - This is filesystem- and accelerator-independent.  ### Information  - [x] The official trainer checkpoint flow…

## 🔀 Pull Requests

### #7220 — [[trainer] fix: build RL tokenizer/processor from HFModelConfig](https://github.com/verl-project/verl/pull/7220)
- **作者**: ZhiliangWu  **时间**: 2026-07-31 19:25 CST
- **摘要**: ### What does this PR do?  Make the PPO entrypoints build the dataset's tokenizer/processor from `HFModelConfig`, the same way the SFT trainer ([`sft_trainer.py:141`](https://github.com/verl-project/verl/blob/76324f1de4aabad06441a01fc17b605d9b3e564a/verl/trainer/sft_trainer.py#L141)) and the engine …

### #7219 — [[ci] chore: remove gemini code review](https://github.com/verl-project/verl/pull/7219)
- **作者**: wuxibin89  **时间**: 2026-07-31 16:21 CST
- **摘要**: ### What does this PR do?  The consumer version of Gemini Code Assist on GitHub has been sunset. All code review activity has officially ceased.

### #7218 — [Kimi k2.5](https://github.com/verl-project/verl/pull/7218)
- **作者**: Feng0w0  **时间**: 2026-07-31 15:54 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do? Megatron-Bridge:de93536 Megatron-LM:3bec9aa MindSpeed:16b9b91  We also need to delete `if task is None: continue` in Megatron-Bridge/src/megatron/bridge/models/conversion/model_bridge.py > Add **concise** overview of what this PR aims to achieve or accomplish. Reference rel…

### #7217 — [fix(agent_loop): propagate validation metadata to async rewards](https://github.com/verl-project/verl/pull/7217)
- **作者**: YAO-001  **时间**: 2026-07-31 14:38 CST
- **摘要**: ## Summary  - propagate the existing `validate` flag through both agent-loop postprocess paths - attach `validate` to the async reward `DataProto.meta_info` - add a CPU-only regression test covering training and validation values  ## Root cause  `AgentLoopWorker._compute_score` constructed a new rew…

### #7215 — [[BREAKING][fsdp, trainer] fix: apply selective weight decay](https://github.com/verl-project/verl/pull/7215)
- **作者**: YAO-001  **时间**: 2026-07-31 14:12 CST
- **摘要**: ### What does this PR do?  Fixes #5070 by applying weight decay selectively in the FSDP optimizer path instead of applying one global value to every trainable parameter.  - Add a default `standard` policy that excludes bias and normalization parameters while continuing to decay embeddings, matching …

### #7214 — [[ckpt] fix: remove loaded local checkpoints](https://github.com/verl-project/verl/pull/7214)
- **作者**: yuyz-cyber  **时间**: 2026-07-31 13:54 CST
- **摘要**: ### What does this PR do?  Fixes #7213.  Make `trainer.del_local_ckpt_after_load` remove the loaded actor or critic role directory on the trainer's shared checkpoint filesystem. All ranks finish loading before rank 0 removes the directory, and all ranks wait for deletion to complete before returning…

### #7212 — [ [doc, hardware, worker, trainer] feat: add SFT support for TPU using TorchTitan engine ](https://github.com/verl-project/verl/pull/7212)
- **作者**: jialei777  **时间**: 2026-07-31 12:59 CST
- **摘要**: ### What does this PR do?  This PR adds Supervised Fine-Tuning (SFT) support for Google TPU devices in `verl` using the TorchTitan execution engine and static sequence packing.      ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the …

### #7211 — [[misc] fix: handle large timeout results](https://github.com/verl-project/verl/pull/7211)
- **作者**: yuyz-cyber  **时间**: 2026-07-31 10:14 CST
- **标签**: wontfix
- **摘要**: ### What does this PR do?  Fixes false multiprocessing timeouts when decorated functions return values larger than the queue pipe buffer. The parent now drains the result queue before joining the child, keeps one deadline across result delivery and process exit, and rejects results whose deserializa…

### #7210 — [[misc] fix: enforce HDFS copy timeout](https://github.com/verl-project/verl/pull/7210)
- **作者**: yuyz-cyber  **时间**: 2026-07-31 09:49 CST
- **标签**: wontfix
- **摘要**: ### What does this PR do?  Honor the existing `timeout` keyword for HDFS copies. HDFS commands now run in an isolated process group on POSIX systems; timeout and interruption paths terminate the process tree and wait for cleanup so a failed copy cannot continue modifying its destination in the backg…

### #7209 — [[ckpt] fix: clean up local checkpoints after load](https://github.com/verl-project/verl/pull/7209)
- **作者**: yuyz-cyber  **时间**: 2026-07-31 09:20 CST
- **摘要**: ### What does this PR do?  Fix `del_local_after_load` for FSDP and Megatron checkpoints. All ranks now finish loading before rank 0 removes the shared role checkpoint directory, followed by a second synchronization so no rank races with cleanup. Disabled cleanup continues to preserve the checkpoint.…
