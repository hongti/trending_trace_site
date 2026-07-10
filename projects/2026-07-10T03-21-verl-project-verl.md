# verl-project/verl — 动态追踪

> 生成时间: 2026-07-10 11:21 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 最近动态的中文摘要：

### 📌 Issue 概览
- **Ascend 平台 2026Q3 路线图 (#6995)**：规划了 verl 在昇腾平台下一季度的重点演进方向。计划支持新模型（GLM5.2、DeepSeek v4）、集成 **FSDP-Turbo** 与 **MindSpeed-Bridge**、全面兼容 **megatron-bridge**，并引入 uv package pro。

### 🚀 PR 概览

**1. 新特性与功能增强**
- **新增 HY v3 Megatron GRPO 示例 (#7003)**：添加了适配腾讯 Hy3 模型的 DAPO 风格 GRPO Megatron 训练脚本，并启用 mBridge 支持。
- **新增 Qwen3.5 397B 支持 (#6994)**：添加了该大参数变体的训练脚本，新增 A2/A3 架构的 Dockerfile，并绕过了 verl 0.8.0 分支中 vLLM 的多模态缓存问题。

**2. 核心架构与 Bug 修复**
- **恢复 RayPPOTrainer 并移除 TransferQueue (#6999, #7000)**：移除了 TransferQueue 运行时路径，将默认 PPO 入口切回 `RayPPOTrainer`。此变更为后续引入 NeoProto 数据平面架构铺路（#7000 主要用于触发 CI 验证，后续从 neo 分支合并至 master）。
- **修复 FSDP 异步加载权重问题 (#6996)**：确保 FSDP 初始权重仅由 rank0 加载后再通过网络广播，修复了此前所有 rank 重复加载权重的资源浪费与潜在问题。
- **修复 veomni MoE 参数处理逻辑 (#6993)**：因 veomni 已将 `gate_proj` 和 `up_proj` 融合为 `gate_up_proj`，统一改用 `default_moe_param_handler` 进行参数拆分。

**3. 文档与配置修正**
- **更新 Atlas 950DT A5 安装指南 (#7001, #7002)**：完善了 torch_npu、MindSpeed 及 Megatron 的安装说明。
- **修正 vllm-ascend commit ID (#6997, #6998)**：修复了 A5 安装文档中编译 vllm-ascend 所需的 commit ID 错误。

### 📦 Release 概览
- 近期暂无新版本发布记录。

---

## 🐛 Issues

### #6995 — [[roadmap][ascend] verl on ascend 2026Q3 roadmap](https://github.com/verl-project/verl/issues/6995)
- **作者**: wucong25  **时间**: 2026-07-09 19:33 CST
- **摘要**: ### model  ---  - [ ] GLM5.2 - [ ] deepseekv4  ### verl features support ---   - [ ] **FSDP-Turbo** and **MindSpeed-Bridge** integration   - [ ] Fully compatible with **megatron-bridge** - [ ] uv package process management  ### advanced features ---    ##### training stablization technique - [ ] Ful…

## 🔀 Pull Requests

### #7003 — [[megatron, trainer] feat: add HY v3 Megatron GRPO example](https://github.com/verl-project/verl/pull/7003)
- **作者**: xhx1022  **时间**: 2026-07-10 10:44 CST
- **摘要**: ### What does this PR do?  Adds HY v3 Megatron GRPO support pieces:  - Add `examples/grpo_trainer/run_hy_v3_megatron.sh`, a DAPO-style GRPO Megatron training script for `tencent/Hy3` with mBridge enabled. - Add HY v3 FLOPs estimation support in `verl/utils/flops_counter.py`.  Related Megatron-Bridge…

### #7002 — [[doc] feat: update installation instructions for Atlas 950DT A5](https://github.com/verl-project/verl/pull/7002)
- **作者**: fh188  **时间**: 2026-07-10 10:20 CST
- **摘要**: Updated the installation guidance for torch_npu and clarified the installation instructions for MindSpeed and Megatron.  ### What does this PR do?  update installation instructions for Atlas 950DT A5  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ..…

### #7001 — [[doc] feat: update installation instructions for Atlas 950DT A5 ](https://github.com/verl-project/verl/pull/7001)
- **作者**: fh188  **时间**: 2026-07-10 10:18 CST
- **摘要**: Updated the installation guidance for torch_npu and MindSpeed.  ### What does this PR do?  update installation instructions for Atlas 950DT A5   ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {des…

### #7000 — [[single_controller] fix: remove transfer queue and restore RayPPOTrainer](https://github.com/verl-project/verl/pull/7000)
- **作者**: zw0610  **时间**: 2026-07-10 02:12 CST
- **摘要**: **Note: This PR is only opened to trigger CI and validate correctness; NeoProto will be landed on the `neo` branch first, and `master` will be updated by merging from `neo` afterward.** Therefore, it's duplicated with #6999  ### What does this PR do?    This PR removes the TransferQueue runtime path…

### #6999 — [[single_controller] fix: remove transfer queue and restore RayPPOTrainer](https://github.com/verl-project/verl/pull/6999)
- **作者**: zw0610  **时间**: 2026-07-10 02:02 CST
- **摘要**: ### What does this PR do?    This PR removes the TransferQueue runtime path and switches the default PPO entrypoint back to `RayPPOTrainer`, paving the way for the NeoProto data-plane work proposed in #6988.    NeoProto aims to replace TransferQueue as the RL data-plane substrate with a ref/index-on…

### #6998 — [[doc] fix: correct vllm-ascend commit id in A5 install doc](https://github.com/verl-project/verl/pull/6998)
- **作者**: zjchenn  **时间**: 2026-07-09 20:44 CST
- **摘要**: ### What does this PR do?  as title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #6997 — [[doc] fix: fix commit id for vllm-ascend compile in a5 install doc](https://github.com/verl-project/verl/pull/6997)
- **作者**: zjchenn  **时间**: 2026-07-09 20:27 CST
- **摘要**: ### What does this PR do?  as title  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, …

### #6996 — [[fsdp] fix: Fix fsdp model loading on async](https://github.com/verl-project/verl/pull/6996)
- **作者**: theely  **时间**: 2026-07-09 20:16 CST
- **摘要**: ### What does this PR do?  This fix ensures that FSDP initial weights are indeed loaded only by rank0 and subsequently broadcast over the network.  ### Test  Logs before the fix (weights are loaded by all ranks): ``` Loading weights:   0%|          | 0/451 [00:00<?, ?it/s] [repeated 31x across clust…

### #6994 — [[misc] chore: add Qwen3.5 397B support, workaround vLLM cache, add A2/A3 Dockerfiles](https://github.com/verl-project/verl/pull/6994)
- **作者**: ruanhao566  **时间**: 2026-07-09 17:23 CST
- **摘要**: ### What does this PR do?  Modify the Qwen3.5 GRPO script to circumvent the vLLM multimodal cache issue in the verl 0.8.0 branch, add a training script for the Qwen3.5 397B variant, update the accompanying documentation, and add Dockerfiles to support Qwen3.5 training in A2 and A3 environments.  ###…

### #6993 — [[veomni] fix: use default moe param handler](https://github.com/verl-project/verl/pull/6993)
- **作者**: wuxibin89  **时间**: 2026-07-09 17:06 CST
- **摘要**: ### What does this PR do?  veomni fuse all model's `gate_proj` and `up_proj` into `gate_up_proj`, so all models should use `default_moe_param_handler` to split gate_up_proj.
