# verl-project/verl — 动态追踪

> 生成时间: 2026-10-17 12:52 CST

## AI 总结

以下是 **verl-project/verl** 仓库最近动态的中文摘要：

### 📌 Issue 动态
- **#8265 PyTorch 生态系统维护者问卷调查**
  PyTorch 基金会团队发起了一项简短调查，旨在了解生态系统维护者的参与体验，并收集改进建议。

### 🚀 PR 动态

#### ✨ 新特性与优化
- **支持昇腾 NPU (Ascend NPU) 增量权重同步 (#8272)**
  扩展了 `delta_sharded` 检查点引擎，通过 HCCL 传输支持在 Ascend NPU 上进行增量权重同步。
- **降低 TPU GRPO 示例硬件门槛 (#8266)**
  TPU GRPO Qwen3-0.6B 示例现可在 2 芯片（2x v6e-1）上运行（此前需 8 芯片），1 芯片跑 TorchTitan 训练，1 芯片跑 vLLM rollout。
- **跨 Reward Worker 共享异步评分并发度 (#8270)**
  新增 `reward.run_max_concurrent` 配置项，限制同一 RewardLoopManager 下的多个协同异步评分器的并发度。
- **Checkpoint 恢复前增加数据集校验 (#8269)**
  新增 `data.resume_check` 配置项，在恢复 v1 checkpoint 前通过版本化的 `data_manifest.json` 校验数据源和行身份等信息。

#### 🛠 Bug 修复
- **修复 TPU SFT 脚本环境变量报错 (#8271)**
  解决了 TPU SFT torchrun 模式下未设置 `TPU_WORKER_HOSTNAMES` 导致的脚本执行失败问题，并同步了 README。
- **修复 Megatron Checkpoint 保存时的参数同步问题 (#8264)**
  修复了在开启 `overlap_param_gather=true` 且优化器更新后保存 checkpoint 时，可能读取到尚未完成 all-gather 的模型参数的问题。
- **修复 vLLM 路由重放 (R3) 错误匹配问题 (#8263)**
  当 vLLM 运行了不同的 prompt 时，拒绝路由重放，避免将错误的 routed-experts 行映射到 trainer 上。
- **修复 PrimeRewardManager 处理字典格式分数崩溃的问题 (#8262)**
  解决了当奖励函数返回字典格式（如 `{"score": 1.0, "acc": true}`）时，`PrimeRewardManager` 抛出 `KeyError: 0` 的问题。

#### 🧪 测试与健壮性
- **验证 GSPO 梯度计算 (#8268)**
  新增测试验证 PPO loss wrapper 和 GSPO loss 的梯度计算是否符合独立标量和解析梯度期望，覆盖多种数据布局和精度。
- **增加 Hermes 和 Kimi 协议跨阶段测试 (#8267)**
  新增 CPU 端契约测试，覆盖 Hermes 和 Kimi 工具调用的解析、消息构造、工具调度和回复组装全流程。

*(注：近期无 Release 发版动态)*

---

## 🐛 Issues

### #8265 — [Quick survey: your experience as a PyTorch Ecosystem maintainer](https://github.com/verl-project/verl/issues/8265)
- **作者**: michellearoth  **时间**: 2026-10-16 22:38 CST
- **摘要**: Hi,  I'm reaching out from the PyTorch Foundation's Ecosystem team. We're running a short survey to understand how participation in the Ecosystem is going for maintainers like you, and where we can improve.  It should take about 5 minutes: https://linuxfoundation.research.net/r/K8H6FR3  Thanks for b…

## 🔀 Pull Requests

### #8272 — [[ckpt, rollout, vllm] feat: support delta weight sync on Ascend NPU (HCCL transport)](https://github.com/verl-project/verl/pull/8272)
- **作者**: HaoWang113  **时间**: 2026-10-17 11:29 CST
- **摘要**: ### What does this PR do?  Extends verl's delta weight sync (`delta_sharded` checkpoint engine, #6974 / #7227, roadmap #7060) to **Ascend NPU** hosts via an **HCCL transport**, so NPU training runs can skip full-weight resyncs between trainer and rollout the same way CUDA hosts already do. Design di…

### #8271 — [[hardware, doc] fix: handle unset TPU_WORKER_HOSTNAMES in TPU SFT torchrun mode and sync README](https://github.com/verl-project/verl/pull/8271)
- **作者**: jialei777  **时间**: 2026-10-17 10:21 CST
- **摘要**: ### What does this PR do?  Fixes two follow-up issues from #8233 in `examples/tpu/sft`:  1. **`examples/tpu/sft/run_qwen3_0_6b_torchtitan.sh`**: Because the script starts with `set -xeuo pipefail`, line 115 (`DEFAULT_MASTER_ADDR="${TPU_WORKER_HOSTNAMES%%,*}"`) aborts with `TPU_WORKER_HOSTNAMES: unbo…

### #8270 — [[reward, ray, cfg, doc] feat: share async scorer concurrency across reward workers](https://github.com/verl-project/verl/pull/8270)
- **作者**: NaCr05  **时间**: 2026-10-17 08:46 CST
- **摘要**: ### What does this PR do?  Add optional reward.run_max_concurrent to bound cooperative async scorers across the workers owned by one RewardLoopManager. Two workers with local max_concurrent=2 reach a peak of four without a shared limit and two with run_max_concurrent=2. An unnamed, non-restarting Ra…

### #8269 — [[data, ckpt, cfg, doc] feat: check dataset identity before v1 checkpoint restore](https://github.com/verl-project/verl/pull/8269)
- **作者**: NaCr05  **时间**: 2026-10-17 08:46 CST
- **摘要**: ### What does this PR do?  Add opt-in data.resume_check=off|warn|error (default off). A versioned data_manifest.json records source-byte and effective-row identities plus text preprocessing/tokenizer settings. The v1 trainer checks it before restoring actor, critic, dataloader or TransferQueue state…

### #8268 — [[algo, training_utils, doc] test: verify GSPO gradients through the PPO loss wrapper](https://github.com/verl-project/verl/pull/8268)
- **作者**: NaCr05  **时间**: 2026-10-17 08:43 CST
- **摘要**: ### What does this PR do?  Exercise the production ppo_loss wrapper and registered GSPO loss against independent scalar and analytic-gradient expectations. Cover float32/64, padded/nested layouts, sequence length, clipping, token weights, entropy/KL and equivalent microbatch/DP reductions.  ### Chec…

### #8267 — [[tool, rollout, doc] test: add cross-stage Hermes and Kimi protocol contracts](https://github.com/verl-project/verl/pull/8267)
- **作者**: NaCr05  **时间**: 2026-10-17 08:40 CST
- **摘要**: ### What does this PR do?  Add CPU contracts that follow Hermes and Kimi tool calls through the real parser, assistant-message construction, tool dispatch and reply assembly. Ten fixtures run through native tools and FunctionTool, checking IDs, invalid JSON, enabled/unknown tools, parallel limits, z…

### #8266 — [[doc, hardware] feat: run the TPU GRPO Qwen3-0.6B example on 2 chips (2x v6e-1)](https://github.com/verl-project/verl/pull/8266)
- **作者**: jialei777  **时间**: 2026-10-17 03:55 CST
- **摘要**: ### What does this PR do?  Lowers the hardware you need to try the TPU GRPO example from **8 chips (2x `v6e-4`) to 2 chips (2x `v6e-1`)**. One chip runs the TorchTitan trainer and one runs the vLLM rollout. Users with little TPU quota can now run the Qwen3-0.6B GSM8K example end to end. The Qwen3-4B…

### #8264 — [[megatron, ckpt] fix: synchronize overlapping parameters before checkpoint save](https://github.com/verl-project/verl/pull/8264)
- **作者**: ewan0x79  **时间**: 2026-10-16 21:54 CST
- **摘要**: ### What does this PR do?  Fix Megatron checkpoint saving after an optimizer update when `overlap_param_gather=true`: the save path can read model parameters whose all-gather has not yet completed.  The distributed optimizer updates the parameter shards owned by each rank. With parameter-gather over…

### #8263 — [[vllm, rollout] fix: reject routing replay when vLLM ran a different prompt](https://github.com/verl-project/verl/pull/8263)
- **作者**: aoshen02  **时间**: 2026-10-16 21:41 CST
- **摘要**: ### What does this PR do?  Rollout routing replay (R3) returns one routed-experts row per token vLLM ran. The agent loop places those rows on the trainer's prompt by position. Nothing checks that the two prompts are the same token sequence.  They can differ for multimodal prompts. vLLM's multimodal …

### #8262 — [fix(reward): accept dict-shaped scores in PrimeRewardManager](https://github.com/verl-project/verl/pull/8262)
- **作者**: Evolian-o  **时间**: 2026-10-16 18:27 CST
- **摘要**: ## Problem  `PrimeRewardManager` crashes the result-processing loop of `parallel_compute_score_async` with `KeyError: 0` whenever the reward function returns a dict such as `{"score": 1.0, "acc": true, "pred": "42"}` — exactly what `math_dapo.compute_score`, `rlla.compute_score` and several custom r…
