# verl-project/verl — 动态追踪

> 生成时间: 2026-09-25 13:15 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue 动态
* **#8011 奖励计算进程池死亡导致后续评分全为 0**
  作者报告了 `math_verify.compute_score` 中的一个严重 Bug：当共享的单例 `ProcessPoolExecutor` 中某个 worker 进程意外死亡（如 OOM 被杀）后，后续所有正确答案的奖励分数都会被错误地记为 0.0，且仅打印一行日志。

### 🔧 PR 动态
本期 PR 主要集中在修复底层训练框架与计算模块的 Bug，并补充测试：
* **#8015 [修复] 恢复 math_verify 中崩溃/卡死的 worker 进程池**
  针对上述 Issue #8011 的直接修复。修复了 `math_verify.py` 中 worker 死亡后进程池无法恢复的问题，确保后续 `compute_score` 调用能正常运行。
* **#8016 [修复] 修复 FSDP checkpoint 转换时的 device mesh 检测问题**
  修复了 FSDP checkpoint 转换过程中的一个缺陷：当存在复制的 buffer（replicated buffer）排在分片参数之前时，`_extract_device_mesh_info` 仅检查了字母序的第一个 key，导致无法正确检测 sharded mesh。
* **#8014 [修复] 剔除 tied-embedding 模型中冗余的 lm_head.weight 传输**
  针对 tied-embedding 模型，修复了 `FSDPEngine` 在按张量进行权重同步时，将共享的 `lm_head.weight` 作为独立张量重复传输的问题，减少了不必要的开销。
* **#8013 [配置修复] 在 Qwen3-8B NPU 示例中启用 paged attention**
  将 paged attention 修复 Backport 到 v0.9.1 版本。通过设置 `pa_shape_list` 避免了 vllm-ascend 中的 FIA decode mask 回归 Bug。
* **#8012 [测试] 补充序列长度平衡分配的测试用例**
  为 `get_seqlen_balanced_partitions` 和 `karmarkar_karp` 算法添加测试覆盖，该算法主要用于在数据并行 (DP) 各 rank 之间平衡工作负载。

### 🚀 Release 动态
* 近期动态中暂无新的版本发布信息。

---

## 🐛 Issues

### #8011 — [[reward] math_verify.compute_score: after one worker process dies, every later correct answer scores 0.0 with only a print line](https://github.com/verl-project/verl/issues/8011)
- **作者**: shaurya416  **时间**: 2026-09-24 13:58 CST
- **摘要**: `verl/utils/reward_score/math_verify.py:29-39` and `:57-67` at `6093e007` (file blob `7071dac`, identical to current main):  ```python _pool = None _pool_lock = threading.Lock()  def _get_pool():     global _pool     if _pool is None:         with _pool_lock:             if _pool is None:           …

## 🔀 Pull Requests

### #8016 — [[ckpt, fsdp] fix: detect sharded mesh after replicated buffers](https://github.com/verl-project/verl/pull/8016)
- **作者**: DaoyuanLi2816  **时间**: 2026-09-25 11:54 CST
- **摘要**: ### What does this PR do?  Fix FSDP checkpoint conversion when a replicated buffer sorts before the sharded parameters.  `_extract_device_mesh_info` only inspects the alphabetically first key. If it is a plain tensor, the fallback mesh has shape `(1,)` and the merger loads only rank 0, even when lat…

### #8015 — [[reward] fix: recover from broken/stuck worker pool in math_verify compute_score](https://github.com/verl-project/verl/pull/8015)
- **作者**: ChdDongyang  **时间**: 2026-09-24 23:56 CST
- **摘要**: ### What does this PR do?  Fixes #8011: after one worker of the shared `ProcessPoolExecutor` singleton in `verl/utils/reward_score/math_verify.py` dies (e.g. OOM-kill), every later `compute_score` call raises `BrokenProcessPool`, which the broad `except Exception` swallows and returns as `0.0` — so …

### #8014 — [[fsdp, vllm] fix: drop redundant tied lm_head.weight from bucketed weight-transfer](https://github.com/verl-project/verl/pull/8014)
- **作者**: kahlun  **时间**: 2026-09-24 18:23 CST
- **摘要**: waiting copilot review in fork.  ### What does this PR do?  For a tied-embedding model, `FSDPEngine`'s per-tensor weight-sync stream sends `lm_head.weight` as a **separate tensor** even though it is byte-identical to (aliases) `model.embed_tokens.weight`. The vLLM colocated rollout uses a **bucketed…

### #8013 — [[recipe] fix: use paged attention in Qwen3-8B NPU example](https://github.com/verl-project/verl/pull/8013)
- **作者**: lxb007981  **时间**: 2026-09-24 15:32 CST
- **摘要**: ### What does this PR do?  Backport verl-project/verl#8006 to v0.9.1. Set pa_shape_list to avoid the FIA decode mask regression in vllm-ascend (vllm-project/vllm-ascend#16889).  Source-commit: 9e89d277db5c2e308f704f07365da9747020ce86  ### Checklist Before Starting  - [x] Search for similar PRs. Past…

### #8012 — [[tests] test: cover get_seqlen_balanced_partitions and karmarkar_karp](https://github.com/verl-project/verl/pull/8012)
- **作者**: linhongyu510  **时间**: 2026-09-24 14:46 CST
- **摘要**: ### What does this PR do?  `get_seqlen_balanced_partitions(..., equal_size=True)` is the routine that balances per-rank workload across data-parallel ranks — it's called with `k_partitions=dp_size` from three trainers:  - `verl/trainer/ppo/ray_trainer.py:1227` and `:1235` - `verl/trainer/ppo/v1/trai…
