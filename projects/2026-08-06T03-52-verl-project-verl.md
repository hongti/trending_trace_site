# verl-project/verl — 动态追踪

> 生成时间: 2026-08-06 11:52 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 📌 Issue 动态
近期共提交 4 个 Issue，主要涉及算法逻辑缺陷、配置失效及使用疑问：
*   **REINFORCE++ 多轮奖励丢失** (#7278)：在多轮工具调用中，REINFORCE++ 会在观察跨度前丢失结果奖励。
*   **Batch balancing 性能问题** (#7279)：在 Python 分区中，批平衡机制重复调度标量张量操作，存在冗余。
*   **HFRollout 配置失效** (#7270)：`HFRollout` 忽略了 `repetition_penalty` 配置，错误地继承了检查点中的默认值，而 vLLM 路径无此问题。
*   **Ascend 分布式检查点疑问** (#7276)：用户询问在 Ascend 环境下 `dist_checkpointing_path` 的相关配置问题。

---

### 🔀 Pull Request 动态
近期合并/提交 10 个 PR，重点集中在**算法与奖励机制修复**、**性能优化**及**底层硬件支持**：

**🛠️ 算法与奖励修复**
*   **修复 REINFORCE++ 返回值传播** (#7280)：解决上述 Issue #7278，保留多轮工具观察轨迹中的 REINFORCE++ returns。
*   **修复 GSM8k 答案截断问题** (#7274)：修复 `extract_solution()` 截断响应导致正确答案后若跟有额外文本时被错误丢弃的 Bug。
*   **修复 PRIME 评分事件循环** (#7266)：修复 PRIME 评分后关闭临时 asyncio 事件循环，导致后续调用失败的问题。
*   **新增批处理 RewardManager** (#7271)：为实验性 `reward_loop` 引入支持批处理的能力，替代原先只能单条处理的 `run_single`。

**🚀 性能优化与新特性**
*   **FlexKV KV Cache 卸载** (#7277)：为动态资源转换引入后端无关的 KV cache 复用生命周期，在混合 rollout 副本停用前保留 KV 缓存。
*   **TPU SFT 原生 NestedTensor 支持** (#7275)：为 TPU 上的 SFT 训练器启用原生 NestedTensor 支持。
*   **减少 JIT 编译时间** (#7272)：在 FSDP/veomni 中支持 `pad_to_length`，解决动态 batch size 下形状特化内核（torch.compile / Triton 等）频繁重编译的问题。

**🧹 杂项、CI 与文档**
*   **强制回收超时进程** (#7267)：修复子进程忽略 `SIGTERM` 导致僵尸进程存活的问题，确保彻底清理。
*   **更新 NPU/Ascend CI 与文档** (#7268, #7265)：将 CI 环境升级至 Python 3.12 与 CANN 9.0.1，清理无效文件，并更新 NPU 安装指南与多节点实践文档。

---

### 🚀 Release 动态
*   近期**无**新版本发布。

---

## 🐛 Issues

### #7279 — [Batch balancing repeatedly dispatches scalar tensor operations inside Python partitioning](https://github.com/verl-project/verl/issues/7279)
- **作者**: ai-yang  **时间**: 2026-08-06 11:26 CST
- **摘要**: ## System Info  - verl main commit: `2781c1e405fa80871a838bfdf9495b412df19c35` - Intel Xeon Gold 6133, Linux x86_64 - Python 3.12.13 - PyTorch 2.13.0+cpu, one Torch thread, process pinned to CPU 2 - CUDA was unavailable, so the measurements below are CPU-only.  ## Problem  PPO, V1 PPO, and SFT batch…

### #7278 — [REINFORCE++ drops outcome rewards before multi-turn observation spans](https://github.com/verl-project/verl/issues/7278)
- **作者**: ai-yang  **时间**: 2026-08-06 11:26 CST
- **摘要**: ## System Info  - verl main commit: `2781c1e405fa80871a838bfdf9495b412df19c35` - Linux x86_64, kernel 5.15 - Python 3.12.13 - PyTorch 2.13.0+cpu - tensordict 0.10.0 - NumPy 2.5.1 - CUDA was unavailable; this reproducer exercises the device-independent advantage calculation on CPU.  ## Information  -…

### #7276 — [Some questions about dist_checkpointing_path](https://github.com/verl-project/verl/issues/7276)
- **作者**: RichardHoOoOo  **时间**: 2026-08-06 10:14 CST
- **摘要**: Hi all,  I worked on Ascend and I followed some of the scripts under [examples/ascend_extras/grpo_trainer](https://github.com/verl-project/verl/tree/main/examples/ascend_extras/grpo_trainer) to set `dist_checkpointing_path`, `use_dist_checkpointing`, and `use_mbridge` as follows:  ``` # Actor config…

### #7270 — [HFRollout ignores rollout.repetition_penalty, inheriting the checkpoint's value instead](https://github.com/verl-project/verl/issues/7270)
- **作者**: benjamin05wilson  **时间**: 2026-08-05 16:56 CST
- **摘要**: ## Summary  `RolloutConfig` defines `repetition_penalty: float = 1.0` and the vLLM rollout path honours it, but `HFRollout` never reads it when building its `GenerationConfig`. Because `transformers` back-fills any unset sampling field from the model's `generation_config.json`, HF rollouts silently …

## 🔀 Pull Requests

### #7280 — [[algo] fix: preserve REINFORCE++ returns across observations](https://github.com/verl-project/verl/pull/7280)
- **作者**: ai-yang  **时间**: 2026-08-06 11:49 CST
- **摘要**: ### What does this PR do?  Fixes REINFORCE++ return propagation for multi-turn trajectories containing tool observations.  `response_mask=0` represents both internal observation tokens and trailing padding. The previous reverse scan reset its running return at every zero, so an outcome reward reache…

### #7277 — [[rollout] FlexKV KV cache offload for dynamic resource](https://github.com/verl-project/verl/pull/7277)
- **作者**: DylanChen-NV  **时间**: 2026-08-06 10:48 CST
- **摘要**: ### What does this PR do?  Add a backend-neutral lifecycle for reusing KV cache from requests aborted during fully asynchronous dynamic resource transitions.  Before a hybrid rollout replica is deactivated, VERL now:  1. removes the replica from the load balancer; 2. aborts in-flight requests and as…

### #7275 — [enable native NestedTensor support for TPU SFT trainer](https://github.com/verl-project/verl/pull/7275)
- **作者**: jialei777  **时间**: 2026-08-06 02:55 CST

### #7274 — [[reward] fix: do not drop a GSM8k answer that is followed by more text](https://github.com/verl-project/verl/pull/7274)
- **作者**: adityasingh2400  **时间**: 2026-08-06 01:54 CST
- **摘要**: ### What does this PR do?  `extract_solution()` in `verl/utils/reward_score/gsm8k.py` truncates the response to its last 300 characters before searching for the answer:  ```python if len(solution_str) > _SOLUTION_CLIP_CHARS:     solution_str = solution_str[-_SOLUTION_CLIP_CHARS:] ```  A rollout does…

### #7272 — [[fsdp,veomni] feat: support pad_to_length to reduce jit compile time](https://github.com/verl-project/verl/pull/7272)
- **作者**: wuxibin89  **时间**: 2026-08-05 23:09 CST
- **摘要**: ### What does this PR do?  With `use_remove_padding=True` + `use_dynamic_bsz=True`, every packed micro-batch has a different token count, so shape-specialized kernels (torch.compile, Triton/DeepGEMM autotune) recompile or re-autotune continuously throughout training. This PR adds a `pad_to_length` f…

### #7271 — [[reward] feat: batch-capable RewardManager for experimental reward_loop](https://github.com/verl-project/verl/pull/7271)
- **作者**: Junxiao-Zhao  **时间**: 2026-08-05 19:48 CST
- **摘要**: ### What does this PR do?  Existing reward managers under verl/experimental/reward_loop only expose run_single, so RewardLoopWorker.compute_score_batch has to fan out N tasks per chunk and calls the user compute_score N times. For rule-based rewards that can be evaluated in one shot (e.g. batched to…

### #7268 — [[ci] chore: Update python&cann version, delete non-existent file](https://github.com/verl-project/verl/pull/7268)
- **作者**: LeoYao123  **时间**: 2026-08-05 14:23 CST
- **摘要**: ### What does this PR do? update to python 3.12 & cann 9.0.1，delete non-existent file > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at le…

### #7267 — [[misc] fix: reap stubborn timeout processes](https://github.com/verl-project/verl/pull/7267)
- **作者**: RerankerGuo  **时间**: 2026-08-05 13:59 CST
- **摘要**: ### What does this PR do?  The multiprocessing timeout path currently stops cleanup after `terminate()` and a short join. A child that ignores `SIGTERM` therefore survives after the decorated call raises `TimeoutError`.  This change escalates cleanup to `Process.kill()` when graceful termination fai…

### #7266 — [[reward] fix: restore PRIME scoring event loop](https://github.com/verl-project/verl/pull/7266)
- **作者**: RerankerGuo  **时间**: 2026-08-05 13:58 CST
- **摘要**: ### What does this PR do?  `run_reward_scoring` installs a temporary asyncio event loop and closes it after PRIME scoring, but it leaves that closed loop as the thread's current loop. Callers that previously owned an open loop then observe the wrong, closed loop after scoring.  This change captures …

### #7265 — [[doc] chore: Modified some guide documents for NPU](https://github.com/verl-project/verl/pull/7265)
- **作者**: zhouhengan1211  **时间**: 2026-08-05 11:45 CST
- **摘要**: ### What does this PR do?  - Modified the install guidance document - Modified the multi-node practice document  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search f…
