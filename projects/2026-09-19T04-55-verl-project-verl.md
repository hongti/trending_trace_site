# verl-project/verl — 动态追踪

> 生成时间: 2026-09-19 12:55 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### Issue 动态
本次提供的数据中未包含 Issue 相关动态。

### Release 动态
本次提供的数据中未包含 Release 相关动态。

### Pull Request 动态

**1. 新特性与性能优化**
* **#7930 Value rlvr patches**：引入 Value rlvr 相关补丁。
* **#7928 支持 OPD 性能分析**：新增 OPD 性能分析工具支持。
* **#7927 分离 teacher rollout lanes**：在工具层面将 teacher rollout lanes 进行分离，优化执行流程。

**2. Bug 修复**
* **#7929 修复 IPv6 地址解析问题**：修复了 `get_free_port()` 因无法识别带括号的 IPv6 地址（如 `[::1]`）导致将其误判为 IPv4 并引发绑定失败的问题。
* **#7926 修复 V1 独立 rollout 显存预算问题**：修复了 V1 `separate_async` 模式下未正确应用独立显存预算（`standalone_gpu_memory_utilization`）的 Bug。
* **#7924 拦截非正数配置项**：在配置阶段直接拒绝 batch size、并行度和 GPU 数量等参数的零或负数值，避免后续出现难以定位的报错。
* **#7923 校验梯度裁剪参数**：增加对 `optim.clip_grad` 参数的校验，并将已废弃的 `grad_clip` 路由至新参数。

**3. CI 与文档改进**
* **#7922 文档路径校验**：新增 pre-commit 检查机制，确保文档引用的文件路径真实存在，并修复了 4 处过时路径。
* **#7925 修复 SFT 工作流触发**：修正了 SFT 端到端测试中因路径不存在导致无法正确触发的配置问题。
* **#7921 修复 Ascend 端到端测试**：为满足 `vllm-ascend` 融合内核的限制，将 `qwen3_5-2b_ascend` 测试中的 LoRA rank 降至 8，修复了该环境下的测试失败问题。

---

## 🔀 Pull Requests

### #7930 — [Value rlvr patches](https://github.com/verl-project/verl/pull/7930)
- **作者**: minhphucng30-UND  **时间**: 2026-09-19 11:32 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7929 — [[utils] fix: accept bracketed IPv6 addresses in get_free_port](https://github.com/verl-project/verl/pull/7929)
- **作者**: Redemption-ZTX  **时间**: 2026-09-19 10:16 CST
- **摘要**: ## What  `get_free_port()` picks the socket family with `is_valid_ipv6_address(address)`. `ipaddress.IPv6Address` rejects the bracketed form, so `[::1]` is classified as IPv4 and the bind fails:  ```python get_free_port("[::1]") # socket.gaierror: [Errno 8] nodename nor servname provided, or not kno…

### #7928 — [[perf, tool] feat: support opd profiling](https://github.com/verl-project/verl/pull/7928)
- **作者**: tardis-key  **时间**: 2026-09-19 09:18 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7927 — [[tool] fix: separate teacher rollout lanes](https://github.com/verl-project/verl/pull/7927)
- **作者**: tardis-key  **时间**: 2026-09-19 09:18 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7926 — [[trainer, rollout] fix: honor V1 standalone rollout memory budget](https://github.com/verl-project/verl/pull/7926)
- **作者**: zupengwang  **时间**: 2026-09-19 09:10 CST
- **摘要**: ### What does this PR do?  V1 `separate_async` currently passes the same rollout memory budget to hybrid and standalone managers, ignoring the existing `standalone_gpu_memory_utilization` option. This change applies a non-null standalone override to a copied config so dedicated rollout GPUs can use …

### #7925 — [[ci] fix: trigger the SFT e2e workflows on the config they actually load](https://github.com/verl-project/verl/pull/7925)
- **作者**: linhongyu510  **时间**: 2026-09-18 23:37 CST
- **摘要**: ### What does this PR do?  The three SFT e2e workflows list two entrypoint paths that no longer exist:  ```yaml       - "verl/trainer/fsdp_sft_trainer.py"       - "verl/trainer/config/sft_trainer.yaml" ```  A `paths:` entry naming a missing file still parses and GitHub reports nothing — it simply ne…

### #7924 — [[cfg, worker, trainer] fix: reject non-positive batch, parallelism and GPU-count values at config time](https://github.com/verl-project/verl/pull/7924)
- **作者**: brentgryffindor  **时间**: 2026-09-18 23:25 CST
- **摘要**: ### What does this PR do?  Zero (or negative) values for a handful of integer knobs are currently accepted by config validation and fail later in ways that do not name the knob:  | override | current main (3efe38c) | |---|---| | `trainer.n_gpus_per_node=0` | `ZeroDivisionError` *inside* `validate_co…

### #7923 — [[cfg, worker] fix: validate optim.clip_grad and route deprecated grad_clip to it](https://github.com/verl-project/verl/pull/7923)
- **作者**: brentgryffindor  **时间**: 2026-09-18 23:25 CST
- **摘要**: ### What does this PR do?  `optim.clip_grad` is passed straight to `clip_grad_norm_(max_norm=...)` by the FSDP engines (`engine/fsdp/transformer_impl.py`, `fsdp_turbo_impl.py`) without validation. `clip_grad_norm_` scales gradients by `max_norm / total_norm`, so `0` zeroes every gradient, a negative…

### #7922 — [[ci, doc] feat: check that docs reference existing files, fix 4 stale paths](https://github.com/verl-project/verl/pull/7922)
- **作者**: linhongyu510  **时间**: 2026-09-18 21:54 CST
- **摘要**: ### What does this PR do?  Adds a pre-commit guard so stale file paths in the docs can't rot silently, plus the 4 stale references it currently finds.  **This supersedes #7765**, rebased onto current `main`. #7765 was opened before #7876 landed and its branch still carried the `npu_unit_test.yml` → …

### #7921 — [[ci] fix: lower LoRA rank to 8 in qwen3_5-2b Ascend e2e to satisfy vllm-ascend fused kernel limit](https://github.com/verl-project/verl/pull/7921)
- **作者**: aass-79  **时间**: 2026-09-18 21:47 CST
- **摘要**: ### What does this PR do?  Fixes the `e2e_ppo_trainer_fsdp-qwen3_5-2b_ascend` job in `e2e_ppo_trainer_megatron_vllm_2_ascend`, which fails on every run on the current Ascend image stack. Example: [run 35234324254 / job 105246291821](https://github.com/verl-project/verl/actions/runs/35234324254/job/1…
