# verl-project/verl — 动态追踪

> 生成时间: 2026-11-16 15:08 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📝 Issue (议题 / RFC)
本期议题主要为架构与算法优化的征求意见稿（RFC），聚焦低精度计算与显存优化：
*   **NVFP4 MoE 配方低精度优化 (#8697, #8699)**：提议在现有的原生 NVFP4 MoE 配方中进一步降低精度。一是将非专家层（non-expert layers）当前的 BF16 计算替换为 MXFP8；二是为反向传播 GEMMs 新增 MXFP8 模式（支持可选的随机舍入），以提升计算效率。
*   **共享主干网络 PPO (#8696)**：提议引入一种配置控制模式，使 PPO 的 Critic 与 Actor 共享冻结的主干网络，Critic 仅作为一个小型的 value head（包含 3 层 Transformer + MLP）进行独立训练，旨在大幅降低显存和计算开销。

### 🔧 Pull Request (拉取请求)
近期 PR 涵盖了重要 Bug 修复、硬件适配及性能提升：
*   **[新特性] 多 LoRA 教师模型共享底座 (#8694)**：在多教师在线策略蒸馏（OPD）场景下，支持在同一个基础模型上挂载并提供多个 LoRA 适配器作为教师模型。这打破了原本“一个教师需要独占一套 GPU”的限制，大幅节省了 GPU 资源。
*   **[修复] 保证 R3 路由一致性 (#8701)**：修复了在部分 Rollout 权重同步时 R3 路由一致性被破坏的问题，确保 R3 算法所依赖的路由保证在同步过程中依然有效并可测量。
*   **[修复] 轨迹回放索引错误 (#8695)**：修复了 V1 trainer 缓存命中回放路径中的 Bug。原逻辑硬编码了 `index=0` 后缀，现已更正为使用已保存的 session 和 index 正确重建轨迹。
*   **[升级] ROCm 镜像升级至 vLLM v0.32.0 (#8700)**：将 ROCm 的 Docker 镜像及 e2e 工作流升级至 vLLM v0.32.0，使其与 verl 的 CUDA 技术栈版本保持一致。
*   **[性能] DeepSeek V4 Flash NPU 性能优化 (#8698)**：针对 NPU（神经网络处理器）硬件环境，调整并优化了 DeepSeek V4 flash 训练的性能方案。

### 🚀 Release (版本发布)
*   近期无新的 Release 版本发布。

---

## 🐛 Issues

### #8699 — [[RFC][megatron] MXFP8 backward GEMMs (optionally with stochastic rounding) for the NVFP4 MoE recipe](https://github.com/verl-project/verl/issues/8699)
- **作者**: BeichenHuang  **时间**: 2026-11-16 11:09 CST
- **摘要**: ### Feature request  Add a third backward mode to the native NVFP4 MoE recipe ([#7971](https://github.com/verl-project/verl/pull/7971), `docs/low_precision/nvfp4.md`). The new mode runs the routed experts' backward GEMMs in **MXFP8**, as in Thinking Machines' [End-to-End 4-Bit Training](https://thin…

### #8697 — [[RFC][megatron, vllm] MXFP8 instead of BF16 for the non-expert layers of the NVFP4 MoE recipe](https://github.com/verl-project/verl/issues/8697)
- **作者**: BeichenHuang  **时间**: 2026-11-16 10:37 CST
- **摘要**: ### Feature request  Add an option to the native NVFP4 MoE recipe ([#7971](https://github.com/verl-project/verl/pull/7971), `docs/low_precision/nvfp4.md`) that runs the GEMMs it currently keeps in BF16 (attention projections, dense MLPs, shared experts) in **MXFP8** instead, in both training and rol…

### #8696 — [[RFC] Shared-backbone PPO — train a small value head on the frozen actor backbone](https://github.com/verl-project/verl/issues/8696)
- **作者**: w82318029  **时间**: 2026-11-15 21:35 CST
- **摘要**: ### Feature request  ## Summary  Add a config-gated mode where the PPO critic shares the actor's backbone: the critic becomes a small value head (3 transformer layers + MLP) trained by an independent optimizer while the backbone stays frozen (updated only by the actor loss). No paper exists; all evi…

## 🔀 Pull Requests

### #8701 — [[rollout, fully_async, trainer] fix: keep R3 routing consistent across partial rollout weight syncs](https://github.com/verl-project/verl/pull/8701)
- **作者**: OutstanderWang  **时间**: 2026-11-16 12:47 CST
- **摘要**: ### What does this PR do?  Make partial rollout preserve the routing-consistency guarantee that R3 depends on, and measure that it does.  **Why this matters (R3, [arXiv:2510.11370](https://arxiv.org/abs/2510.11370)).** In MoE RL the rollout engine and the training engine route the same token to diff…

### #8700 — [[docker, ci, doc] feat: upgrade the ROCm image to vLLM v0.32.0](https://github.com/verl-project/verl/pull/8700)
- **作者**: PeterYang12  **时间**: 2026-11-16 11:56 CST
- **摘要**: ### What does this PR do?  Upgrades the ROCm image (`docker/rocm/Dockerfile.rocm`) to vLLM v0.32.0, the version verl's CUDA stack already uses, and points the ROCm e2e workflow at the image built from it (`verlai/verl:rocm7.14_torch2.12_release_1112`).  It also sets `HSA_TOOLS_DISABLE_REGISTER=1` fo…

### #8698 — [[perf][fsdp] chore: deepseek v4 flash performance optimization solution for NPU](https://github.com/verl-project/verl/pull/8698)
- **作者**: zhouhengan1211  **时间**: 2026-11-16 10:49 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review. - Adjust the DeepSeek V4 flash training policy to optimize its performance. - Redundant code is deleted. This function is controlled…

### #8695 — [[tool, rollout] fix: replay every cached trajectory with its saved session and index](https://github.com/verl-project/verl/pull/8695)
- **作者**: RerankerGuo  **时间**: 2026-11-15 21:22 CST
- **摘要**: ### What does this PR do?  Fixes #8651.  `RolloutTqSkip.load_dump_data` (the cache-hit replay path of the V1 trainer) rebuilt trajectory keys with a hardcoded `index=0` suffix and forcibly sized the replay with `for session_id in range(n)`:  - cached rows whose key index was `> 0` were dropped — a s…

### #8694 — [[rollout, vllm, cfg] feat: serve multiple LoRA distillation teachers on one base model](https://github.com/verl-project/verl/pull/8694)
- **作者**: VladKozlovskiy  **时间**: 2026-11-15 19:26 CST
- **摘要**: ### What does this PR do?  In multi-teacher OPD every teacher gets its own vLLM replicas, so K teachers need K GPUs even when they are LoRA fine-tunes of one base. This PR adds `lora_adapters` to a teacher entry: the base is loaded once and the adapter is picked per request via vLLM multi-LoRA.  ```…
