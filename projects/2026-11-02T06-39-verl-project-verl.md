# verl-project/verl — 动态追踪

> 生成时间: 2026-11-02 14:39 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 🐛 Issue
* **#8488 VeOmni 后端 R3 router replay 推断异常**：作者 horcham 反馈，在使用 VeOmni 后端时，R3 router replay 机制由于使用重复的 top-k 结果来推断无效行，而不是使用显式的 replay mask，导致推理出无效行数据。

### 🔧 Pull Requests (PR)
**重要新特性：**
* **#8482 引入 MoRI-IO checkpoint 引擎**：为 AMD GPU 新增了 `mori` checkpoint 引擎，支持通过单边 RDMA WRITE 同步权重，具备 NIC 条带化和 XGMI 扇出功能。
* **#8480 支持单个 Ray 任务初始化多个 Trainer**：新增 `instance.id` 配置项，允许在同一个 Ray 作业中初始化多个 `PPOTrainer` 实例。
* **#8486 支持 vLLM 权重检查器**：集成了 vLLM 的 weight checker 功能。

**关键修复：**
* **#8483 修复量化模型的 FP8 KV Cache 异常**：解决量化重置后 KV cache scales 未校准的问题，使量化 rollout 能够正确结合 FP8 KV cache 运行。
* **#8489 修复 SandboxFusion 过载导致的基础设施错误**：当 SandboxFusion 服务器出现过载（如 EMFILE 或 502/503）时，增加重试机制并将其正确标记为基础设施错误，避免程序因非代码原因误判失败。
* **#8485 修复 math_dapo Minerva 答案提取正则问题**：解决模型输出包含 Markdown 粗体/斜体修饰符（如 `**Answer:**`）时，正则表达式错误提取星号的问题。
* **#8481 修复 Windows 兼容性问题**：在 `setup.py` 中显式指定 UTF-8 编码，解决 Windows 系统（默认 CP1252 编码）下安装失败报 `UnicodeDecodeError` 的问题。

**文档与 CI 维护：**
* **#8484 新增 FP8 性能文档**：在 FP8 指南中补充了 H20 GPU 结合 FP8 KV cache 的端到端测试结果，并增加了开启该功能的说明。
* **#8487 Ascend A3 CI 调整**：测试在 Ascend A3 主机上丢弃文件缓存的 CI 操作。

### 🚀 Release
* 近期无新版本发布。

---

## 🐛 Issues

### #8488 — [[veomni, rollout] R3 router replay infers invalid rows from duplicate top-k instead of an explicit replay mask](https://github.com/verl-project/verl/issues/8488)
- **作者**: horcham  **时间**: 2026-11-02 11:25 CST
- **摘要**: ### System Info  verl main @ 8e48d72d (VeOmni backend, FSDP2). ### Information  - [ ] The official example scripts - [x] My own modified scripts  ### Tasks  - [ ] An officially supported task in the `examples` folder (such as GLUE/SQuAD, ...) - [x] My own task or dataset (give details below)  ### Re…

## 🔀 Pull Requests

### #8489 — [[reward] fix: retry sandbox_fusion infra failures and mark them as infra errors](https://github.com/verl-project/verl/pull/8489)
- **作者**: daegyu94  **时间**: 2026-11-02 12:16 CST
- **摘要**: ### What does this PR do?  When the SandboxFusion server is overloaded (e.g. EMFILE reported as `SandboxError`, or 502/503 from a gateway), test cases fail for infrastructure reasons and correct programs are silently scored 0.0. Today `call_sandbox_api` retries only HTTP 504.  - Also retry HTTP 500/…

### #8487 — [[ci] chore: test drop file cache on Ascend A3 hosts](https://github.com/verl-project/verl/pull/8487)
- **作者**: lxb007981  **时间**: 2026-11-02 11:23 CST
- **摘要**: ### What does this PR do?  test  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sglang`, `vll…

### #8486 — [support vllm feature of weight checker](https://github.com/verl-project/verl/pull/8486)
- **作者**: shiyuan680  **时间**: 2026-11-02 09:41 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8485 — [Fix math_dapo Minerva extraction for markdown-decorated answers](https://github.com/verl-project/verl/pull/8485)
- **作者**: siddhant-rajhans  **时间**: 2026-11-02 08:24 CST
- **摘要**: ## Bug  Models commonly wrap the Minerva `Answer:` label in markdown bold/italic (`**Answer:** 42`, `Answer: **42**`, `**Answer:**\n\n42`). The existing regex captures the `*` markers as part of the answer, so correct answers score -1.0.  Fixes #8449  ## Root cause  The Minerva answer pattern `(?i)A…

### #8484 — [[doc] feat: add H20 results with an FP8 KV cache to the FP8 guide](https://github.com/verl-project/verl/pull/8484)
- **作者**: BeichenHuang  **时间**: 2026-11-02 03:45 CST
- **摘要**: ### What does this PR do?  Adds FP8 E2E results on H20 with and without an FP8 rollout KV cache to `docs/low_precision/fp8.md`, plus a short section on enabling the FP8 KV cache. Depends on #8483: without it, FP8_BLOCK rollout combined with an FP8 KV cache generates garbage.  Results (Qwen3-30B-A3B-…

### #8483 — [[vllm, rollout] fix: reset uncalibrated KV cache scales after a quantized refit](https://github.com/verl-project/verl/pull/8483)
- **作者**: BeichenHuang  **时间**: 2026-11-02 03:22 CST
- **摘要**: ### What does this PR do?  Quantized rollout (`rollout.quantization.linear/moe=FP8_BLOCK`) combined with an FP8 KV cache (`+actor_rollout_ref.rollout.engine_kwargs.vllm.kv_cache_dtype=fp8_e4m3`) makes vLLM generate garbage. This PR resets the attention q/k/v/prob scales to 1.0 after every quantized …

### #8482 — [[ckpt] feat: MoRI-IO checkpoint engine for AMD GPUs (NIC-striped, XGMI fan-out, optional sharded export)](https://github.com/verl-project/verl/pull/8482)
- **作者**: lixiufei-leo  **时间**: 2026-11-02 01:57 CST
- **摘要**: ### What does this PR do?  Adds `mori`, a checkpoint engine that syncs weights from trainer to rollout with one-sided RDMA WRITE through [MoRI-IO](https://github.com/ROCm/mori) (`pip install amd-mori`), for AMD GPUs with RDMA NICs. It was tested on MI355X with Pollara 400G NICs. It implements the ex…

### #8481 — [[misc] fix: specify utf-8 encoding in setup.py for Windows compatibility](https://github.com/verl-project/verl/pull/8481)
- **作者**: kkdoestech  **时间**: 2026-11-01 21:44 CST
- **摘要**: Signed-off-by: kkdoestech <letuankiet5939@gmail.com> ### What does this PR do? On Windows systems where the locale encoding defaults to CP1252/ANSI, running `pip install -e .` fails with a `UnicodeDecodeError` when `setup.py` reads `README.md` and `version` without explicit encoding. This PR specifi…

### #8480 — [[ray, rollout, trainer] feat: add an instance.id config so multiple trainers can initialize in one Ray job](https://github.com/verl-project/verl/pull/8480)
- **作者**: zehanzhu  **时间**: 2026-11-01 21:44 CST
- **摘要**: ### What does this PR do?  This PR adds an optional `instance.id` config so that more than one `PPOTrainer` can initialize in the same Ray job. When it is unset (the default), the generated actor, placement-group and socket names are unchanged.  Every runtime resource name in VERL — Ray actor names,…
