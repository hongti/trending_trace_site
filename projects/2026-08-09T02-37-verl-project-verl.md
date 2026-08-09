# verl-project/verl — 动态追踪

> 生成时间: 2026-08-09 10:37 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📌 Issue
*   **#7321 [讨论]**：关于 `agg_loss` 中当 `batch_num_tokens` 为 None 时 micro-batch fallback 逻辑的探讨。
*   **#7320 [报错]**：用户反馈更新至最新代码后出现大量报错，涉及运行环境及数据/模型配置。

### 📌 Pull Request
**🔧 Megatron 核心修复**
*   **#7328**：修复 mHC（流形超连接）+ MTP 训练时的崩溃问题（针对 DeepSeek-V4），将 `mhc_multistream` 转发至 MTP，并跳过 MTP 检查点的激活回收。
*   **#7326**：修复 MTP（多 Token 预测）在 megatron-core 0.18.x 上与 `recompute_granularity='full'` 不兼容的问题。

**🤖 模型与数据修复（Qwen 系列）**
*   **#7323**：修复 Qwen3.5 视觉模型在 FSDP2 动态多模态批次下跨 ranks 前向调用不一致的问题，确保参数正确 unshard。
*   **#7322**：修复 Qwen3.5/3.6 多轮 SFT 渲染丢失上下文的问题，使其正确支持邻居感知的聊天模板。

**🛠️ vLLM 与引擎新特性**
*   **#7327**：修复 vLLM 接收端在非合并 LoRA 同步时无法解析 `.base_layer` 的问题。
*   **#7324**：**[新特性]** 为 TorchTitan 训练引擎新增分片 delta 权重同步支持。

**🧪 测试增强**
*   **#7325**：为 `core_algos.py` 中 14 个优势估计器里缺乏测试的 9 个补充测试，固定其策略梯度的预期值，提升算法鲁棒性。

### 📌 Release
*   近期无新版本发布。

---

## 🐛 Issues

### #7321 — [[Discussion] Understanding the micro-batch fallback in agg_loss when batch_num_tokens is None](https://github.com/verl-project/verl/issues/7321)
- **作者**: zyang6  **时间**: 2026-08-08 13:09 CST
- **摘要**: Thanks for the great work on verl — really helpful for our experiments. We @alwaysyiyu just wanted to ask about a small point in `agg_loss`.  While reading `agg_loss` (`verl/trainer/ppo/core_algos.py`), we noticed that `token-mean` uses different denominators depending on whether `batch_num_tokens` …

### #7320 — [after update to latest code ,  Many errors occurred.](https://github.com/verl-project/verl/issues/7320)
- **作者**: cqray1990  **时间**: 2026-08-08 10:07 CST
- **标签**: bug
- **摘要**: ### System Info                set -x                              export VLLM_USE_V1=1                              # ================= data/model/tool =================               HDFS_ROOT=${HDFS_ROOT:-$PWD}               DATA_ROOT=${DATA_ROOT:-$PWD}                              #dapo_math_17k…

## 🔀 Pull Requests

### #7328 — [[megatron] fix: forward mhc_multistream to MTP and skip activation reclaim for MTP checkpoints](https://github.com/verl-project/verl/pull/7328)
- **作者**: HollowMan6  **时间**: 2026-08-09 05:48 CST
- **摘要**: ### What does this PR do?  Fixes two crashes that block mHC (manifold hyper-connections) + MTP training on Megatron-core (DeepSeek-V4, `use_fused_mhc=False` + `mtp.enable=True`). Both stem from verl's MTP postprocess / recomputation patches not accounting for the mHC multi-stream tensor.  **Backgrou…

### #7327 — [[vllm] fix: resolve .base_layer on the vLLM receiver for non-merged LoRA sync](https://github.com/verl-project/verl/pull/7327)
- **作者**: HollowMan6  **时间**: 2026-08-09 05:22 CST
- **摘要**: ### What does this PR do?  Reland of #5599   With LoRA enabled, vLLM wraps *every* linear layer (exposing base weights under `.base_layer`), while Megatron only LoRA-wraps the user's `target_modules` and exports plain HF names for the rest. The old sender-side `add_base_layer_suffix` relied on hard-…

### #7326 — [[megatron] fix: make MTP work with recompute_granularity=full on megatron-core 0.18.x](https://github.com/verl-project/verl/pull/7326)
- **作者**: gaohongkui  **时间**: 2026-08-08 21:39 CST
- **摘要**: ## What does this PR do?  Makes Multi-Token Prediction usable together with `recompute_granularity='full'` on every released megatron-core, and makes the existing signature-based branch in `patch_mtp_layer_checkpointed_forward` visible instead of silent.  ### The bug  megatron-core 0.18.x is interna…

### #7325 — [[algo] test: pin the expected policy gradient of each advantage estimator](https://github.com/verl-project/verl/pull/7325)
- **作者**: tianyi-zhang-02  **时间**: 2026-08-08 19:04 CST
- **摘要**: ### What does this PR do?  `core_algos.py` registers 14 advantage estimators. Nine of them appear in no test at all, and the tests that do exist check shapes, registry mechanics, KL-penalty values, and that a vectorized path matches its reference on random tensors — never the gradient the estimator …

### #7324 — [[ckpt, fsdp]  feat: sharded delta weight sync for the TorchTitan engine](https://github.com/verl-project/verl/pull/7324)
- **作者**: attack204  **时间**: 2026-08-08 18:15 CST
- **摘要**: **STATUS: I am running more tests, 2 days needs**   ### What does this PR do?  Adds sharded delta weight sync support to the TorchTitan training engine.  ### Test  **Results** (GSM8K GRPO, `one_step_off_policy` disaggregated, TorchTitan FSDP2 trainer → SGLang rollout, A100/A800 80GB, 20 steps per ba…

### #7323 — [[model, fsdp] fix: align Qwen3.5 vision calls across ranks](https://github.com/verl-project/verl/pull/7323)
- **作者**: yangzhou24  **时间**: 2026-08-08 17:32 CST
- **摘要**: ### What does this PR do?  FSDP2 records each wrapped-module forward use with multiplicity for pre-backward parameter unsharding. With dynamic multimodal batching, a rank containing both images and videos enters `model.visual` twice, while an image-only, video-only, or text-only rank enters it once.…

### #7322 — [[data] fix: preserve context in multi-turn SFT rendering for Qwen3.5/3.6](https://github.com/verl-project/verl/pull/7322)
- **作者**: yph22  **时间**: 2026-08-08 17:04 CST
- **摘要**: ### What does this PR do?  `MultiTurnSFTDataset` currently renders every message in isolation and concatenates the results. That assumption does not hold for neighbor-aware chat templates such as Qwen3.5/3.6:  - rendering an initial `system` message alone is invalid because the template requires a u…
