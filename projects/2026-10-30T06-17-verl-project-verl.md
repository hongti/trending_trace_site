# verl-project/verl — 动态追踪

> 生成时间: 2026-10-30 14:17 CST

## AI 总结

# verl-project/verl 近期动态摘要

## Issue

- **#8452**：提出 RFC，建议新增 `data.custom_dataloader` 钩子，允许用户直接接入已有数据加载管线到 Verl，无需先转换为本地文件或额外封装。属于功能请求，尚未实现。
- **#8449**：反馈 `math_dapo` 奖励函数在处理 Minerva 答案时，对带 markdown 装饰的答案（如 `\boxed{}`）错误评分 -1.0，影响数学评测准确性。

## Pull Request

### 重要新特性
- **#8454**：为 V1 同步 PPO 实现自定义 dataloader 支持（POC），对应 #8388，允许更灵活的数据接入。
- **#8456**：新增 Qwen3-8B（2×v6e-8）和 Qwen3-32B（2×v6e-32）的 TPU GRPO 训练 recipe，扩展大模型 TPU 支持。
- **#8451**：新增 Qwen3-30B-A3B（MoE 架构）TPU GRPO recipe，在 GSM8K 上使用 TorchTitan + vLLM rollout。

### Bug 修复
- **#8450**：修复 `math_dapo` 评分器无法正确处理 markdown 装饰答案的问题，对应 Issue #8449。
- **#8457 / #8455**：修复 GRPO 训练在较新 TransferQueue 版本下的失败问题——当 critic 禁用时仍请求了不存在的 `values` 字段。
- **#8453**：修复 `concat_tensordict` 在 tensordict ≥ 0.11 下对 `NonTensorData` 的处理，确保保留首个实例。
- **#8446**：修复 Dockerfile.stable.vllm 中 transformers 版本（5.3.0）与 vLLM 0.24.0 不兼容的安装问题。

### 其他
- **#8458**（**BREAKING**）：对蒸馏 estimator 的 k1/k3 更新模式组合增加显式校验，防止用户误用 detached k3 reward 导致无法获得 reverse-KL 策略梯度。
- **#8448**：调试 Qwen3-2.5B OOM 问题，属调试性 PR。

## Release

> 本期无新版本发布。

---

## 🐛 Issues

### #8452 — [[RFC] Pluggable custom dataloaders for PPO](https://github.com/verl-project/verl/issues/8452)
- **作者**: Mecoli1219  **时间**: 2026-10-30 06:52 CST
- **摘要**: ### Feature request  Add an optional `data.custom_dataloader` hook so users can plug an existing data-loading pipeline into Verl without first converting its output into local files or wrapping it in another dataset and dataloader. For example: ```yaml data:   custom_dataloader:     _target_: my_pro…

### #8449 — [[reward] math_dapo: markdown-decorated Minerva answers are scored -1.0](https://github.com/verl-project/verl/issues/8449)
- **作者**: RerankerGuo  **时间**: 2026-10-30 00:51 CST
- **摘要**: ### System Info  - verl `main` @ a08ea02a - Python 3.12.13, CPU only (no model, worker or GPU is needed)  ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [x] An officially supported task in the `examples` folder  ### Reproduction  `verl/utils/reward_sc…

## 🔀 Pull Requests

### #8458 — [[BREAKING][algo, cfg] fix: validate distillation estimator update modes](https://github.com/verl-project/verl/pull/8458)
- **作者**: Eppiehu  **时间**: 2026-10-30 11:44 CST
- **摘要**: ### What does this PR do?  Proposes explicit validation of the k1/k3 estimator/update-mode combinations discussed in #8423. A detached k3 reward does not give the reverse-KL policy gradient, even at an exactly on-policy update with no clipping; it can ascend that KL. The loss kernels and PPO impleme…

### #8457 — [[fsdp, trainer] fix: request values from TransferQueue only when critic is enabled](https://github.com/verl-project/verl/pull/8457)
- **作者**: renjunying2  **时间**: 2026-10-30 11:38 CST
- **摘要**: ### What does this PR do?  - Fix GRPO training failures with newer TransferQueue versions that reject requests for missing fields. When the critic is disabled, is not produced but was still requested during advantage and metrics computation.values - Fix Unsupported dispatcher fused in the Qwen3.5-35…

### #8456 — [[hardware, recipe] feat: Qwen3-8B (2 x v6e-8) and Qwen3-32B (2 x v6e-32) TPU GRPO recipes with tpu-sync](https://github.com/verl-project/verl/pull/8456)
- **作者**: wenjung2007  **时间**: 2026-10-30 11:35 CST
- **摘要**: ### What does this PR do?  Adds two TPU GRPO recipes to `examples/tpu/grpo/`, for models larger than the existing 0.6B / 4B ones:  - `run_qwen3_8b_torchtitan.sh`: Qwen3-8B on 2 TPU `v6e-8` slices (FSDP8 TorchTitan trainer on one slice, one TP=8 vLLM server on the other). - `run_qwen3_32b_torchtitan.…

### #8455 — [[fsdp, trainer] fix: request values from TransferQueue only when critic is enabled](https://github.com/verl-project/verl/pull/8455)
- **作者**: renjunying2  **时间**: 2026-10-30 11:01 CST
- **摘要**: ### What does this PR do?  - Fix GRPO training failures with newer TransferQueue versions that reject requests for missing fields. When the critic is disabled, `values` is not produced but was still requested during advantage and metrics computation. - Fix Unsupported dispatcher fused in the Qwen3.5…

### #8454 — [ [data, trainer] feat: support custom dataloaders for V1 sync PPO](https://github.com/verl-project/verl/pull/8454)
- **作者**: Mecoli1219  **时间**: 2026-10-30 06:57 CST
- **摘要**: ### What does this PR do? First POC for #8388. > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: [dataloader](h…

### #8453 — [[data] fix: keep the first NonTensorData in concat_tensordict with tensordict >= 0.11](https://github.com/verl-project/verl/pull/8453)
- **作者**: jialei777  **时间**: 2026-10-30 06:52 CST
- **摘要**: ### What does this PR do?  `concat_tensordict` treats `NonTensorData` as per-batch metadata and keeps the first instance. `concat_tensordict_with_none_bsz` documents this ("metadata concatenation simply preserves the first instance"), and `test_concat_tensordict` asserts it (`output["temp"] == 1.0`)…

### #8451 — [[hardware, recipe] feat: Qwen3-30B-A3B (MoE) GRPO recipe for TPU](https://github.com/verl-project/verl/pull/8451)
- **作者**: wenjung2007  **时间**: 2026-10-30 01:30 CST
- **摘要**: Adds examples/tpu/grpo/run_qwen3_30b_a3b_torchtitan.sh: GRPO on GSM8K with Qwen3-30B-A3B (128 experts, 8 active), TorchTitan trainer and vLLM rollout on two TPU slices with tpu-sync weight sync. Defaults to TPU v7x 2x2x1 + 2x2x1, like run_qwen3_5_torchtitan.sh; the README documents TPU v5p 2x2x4 + 2…

### #8450 — [[reward] fix: accept markdown-decorated answers in math_dapo Minerva scoring](https://github.com/verl-project/verl/pull/8450)
- **作者**: RerankerGuo  **时间**: 2026-10-30 01:11 CST
- **摘要**: ### What does this PR do?  `verl/utils/reward_score/math_dapo.py` scores the Minerva `Answer: ...` format by extracting the text that follows the `Answer:` label and comparing it with the ground truth. The extracted text kept the markdown emphasis that models put around the answer label or around th…

### #8448 — [debug qwen3 2.5b oom](https://github.com/verl-project/verl/pull/8448)
- **作者**: lxb007981  **时间**: 2026-10-29 21:33 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8446 — [[docker] fix: install a transformers version vllm 0.24.0 accepts in Dockerfile.stable.vllm](https://github.com/verl-project/verl/pull/8446)
- **作者**: nataliekung  **时间**: 2026-10-29 21:13 CST
- **摘要**: ### What does this PR do?  `docker/Dockerfile.stable.vllm` installs `vllm==0.24.0` and later runs `pip install transformers==${TRANSFORMERS_VERSION}` with `TRANSFORMERS_VERSION=5.3.0`. That leaves the image with a transformers version below two lower bounds:  - `vllm==0.24.0` requires `transformers>…
