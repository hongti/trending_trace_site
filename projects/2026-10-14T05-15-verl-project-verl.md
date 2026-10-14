# verl-project/verl — 动态追踪

> 生成时间: 2026-10-14 13:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue
* **#8220 A5一键安装脚本执行失败**：用户反馈在执行 DeepSeek-V4-Flash 的一键安装脚本时失败，原因是缺少 `megatron`、`mbridge` 与 `vllm-ascend` 相关的 patch 文件。

---

### 🔀 Pull Requests

**1. 核心架构与训练优化**
* **#8227 训练阶段 SkipManager 配置扩展**：将原本仅用于 rollout 阶段的 SkipManager 协议推广到整个 RL 训练循环（包括 reward、log-probs、values、actor-update、weight-sync 等阶段），支持各阶段独立配置跳过策略。

**2. 硬件支持与性能加速 (TPU)**
* **#8226 加速 Qwen3-4B TPU GRPO 示例**：通过调整默认配置（如并发 rollout 数提至 256、micro batch 设为 4、使用 1024-token buckets），将该示例的单步训练耗时大幅缩减约一半。
* **#8225 修复 TPU GRPO 示例可跑性**：使 `examples/tpu/grpo/` 能在 main 分支正常运行，并新增了 v6e-4 RayCluster 配置和 Dockerfile，实现从零开始的便捷复现。

**3. Bug 修复与健壮性提升**
* **#8224 保护 delta-loader 的 masked-copy 操作**：为模型参数/缓冲存储的 masked-copy 增加保护机制，修复低精度相关的已知问题。
* **#8221 修复超长 prompt 过滤器静默丢数据问题**：当缺少多模态可选依赖（如 `qwen_vl_utils`）时，原逻辑会静默丢弃所有数据行，现修正为直接抛出 `ImportError`，避免数据无谓丢失。

**4. 文档与 CI 维护**
* **#8223 澄清 GRPO 批次大小文档**：详细补充了 GRPO/PPO 中 train、mini、micro batch sizes 的单位与算术逻辑说明，消除文档歧义。
* **#8222 优化 Ascend CI 流水线**：在 Ascend CI 任务中配置集群本地 PyPI 缓存以加速依赖安装，并移除了 VeOmni 夜间版安装的显式 PyPI 源覆盖。

---

### 🚀 Release
* 本统计周期内无新版本发布。

---

## 🐛 Issues

### #8220 — [【Q3众测】A5一键安装脚本执行失败](https://github.com/verl-project/verl/issues/8220)
- **作者**: jialixin00295144  **时间**: 2026-10-13 15:19 CST
- **标签**: bug
- **摘要**: ### System Info  脚本路径：bash verl-ascend-recipe/DeepSeek-V4-Flash/install_A5.sh 执行一键安装脚本失败，缺少megatron、mbridge与vllm-ascend的patch文件  ### Information  - [ ] The official example scripts - [ ] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQu…

## 🔀 Pull Requests

### #8227 — [[training_utils, cfg] feat: per-stage SkipManager configs for reward/log-probs/values/actor-update/weight-sync](https://github.com/verl-project/verl/pull/8227)
- **作者**: knlnguyen1802  **时间**: 2026-10-14 13:13 CST
- **摘要**: ### What does this PR do?  Generalizes the SkipManager protocol from rollout-only to training-loop stages, implementing PR 1 of the RFC in verl-omni#716 ([SkipManager for every RL pipeline stage](https://github.com/verl-project/verl-omni/issues/716)).  Six new stage roles — `reward`, `old_log_prob`,…

### #8226 — [[hardware, doc] feat: speed up the Qwen3-4B TPU GRPO example (256 concurrent rollouts, micro batch 4, 1024-token buckets)](https://github.com/verl-project/verl/pull/8226)
- **作者**: jialei777  **时间**: 2026-10-14 10:01 CST
- **摘要**: ### What does this PR do?  Roughly halves the step time of the Qwen3-4B TPU GRPO example (`examples/tpu/grpo/run_qwen3_4b_torchtitan.sh`, added in #8225) with three defaults:  - `MAX_NUM_SEQS` 32 -> 256: vLLM decodes all 32 x 8 rollouts of a batch at once. With 32 concurrent sequences the trainer wa…

### #8225 — [[hardware, doc] fix: make examples/tpu/grpo run on main and add a v6e-4 RayCluster and Dockerfile](https://github.com/verl-project/verl/pull/8225)
- **作者**: jialei777  **时间**: 2026-10-14 07:05 CST
- **摘要**: ### What does this PR do?  Makes the TPU GRPO examples in `examples/tpu/grpo/` run on `main` with `verl-hardware-plugin` and makes them reproducible from scratch: a `Dockerfile`, a KubeRay `RayCluster` for 2 TPU v6e-4 slices (8 chips), and a step-by-step README. Only `examples/tpu/grpo/` changes.  R…

### #8224 — [[rollout] guard delta-loader masked copy to model param/buffer storage (part of #7060 known issue)](https://github.com/verl-project/verl/pull/8224)
- **作者**: Virtuoso461  **时间**: 2026-10-13 21:49 CST
- **摘要**: Part of the masked-copy guards known issue in #7060 (also relates to the low-precision scope of #5972 / the 26Q4 roadmap #8186).  ## What does this PR do?  **Guard (a):** the delta loader's masked-copy context patched `Tensor.copy_` globally — any same-shape floating `copy_` inside the window (tempo…

### #8223 — [[doc] fix: clarify GRPO batch sizes and optimizer updates](https://github.com/verl-project/verl/pull/8223)
- **作者**: bianyh  **时间**: 2026-10-13 21:20 CST
- **摘要**: ### What does this PR do?  Closes #1524.  Clarify the units and arithmetic behind GRPO/PPO train, mini and micro batch sizes. The configured actor and critic `ppo_mini_batch_size` values are prompt-equivalent global sizes; the trainers multiply them by `rollout.n` before partitioning trajectories ac…

### #8222 — [[ci] chore: use Ascend cluster-local PyPI cache](https://github.com/verl-project/verl/pull/8222)
- **作者**: lxb007981  **时间**: 2026-10-13 18:02 CST
- **摘要**: ### What does this PR do?  Set PIP_INDEX_URL and PIP_TRUSTED_HOST in Ascend CI jobs to use the cluster-local PyPI cache. Remove the explicit PyPI index override from the nightly VeOmni installation.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ...…

### #8221 — [[data] fix: re-raise ImportError in overlong prompt filter instead of dropping all rows](https://github.com/verl-project/verl/pull/8221)
- **作者**: yweiii  **时间**: 2026-10-13 17:54 CST
- **摘要**: ### What does this PR do?  When an optional dependency is missing (for example `qwen_vl_utils` for multimodal data), `_process_multi_modal_info` raises `ModuleNotFoundError` inside `doc2len` for every sample. The catch-all `except Exception` in `maybe_filter_out_long_prompts`, added in #3814 to drop…
