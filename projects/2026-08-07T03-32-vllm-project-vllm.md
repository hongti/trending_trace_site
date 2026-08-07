# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-07 11:32 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue (问题)
1. **Qwen3 Omni MoE 模型不支持 LoRA** (#51347)：用户在使用 ms-swift 训练 Qwen3-Omni 模型并进行 GRPO 推理时，因该架构暂未适配 LoRA 导致报错。
2. **多节点部署启动挂起** (#51340)：`kernel_warmup()` 阶段缺乏跨 rank 同步机制，导致在多节点部署下发起了真实的 TP/EP 集合通信，进而引发启动卡死。

---

### 🔧 Pull Request (拉取请求)

**✨ 新特性与性能优化**
1. **多模态张量 IPC 分页共享内存** (#51349)：为多模态张量的跨进程通信（IPC）引入了分页共享内存存储机制。
2. **DeepSeek V4 XPU 序列并行** (#51346)：为 XPU 上的 DeepSeek V4 模型添加了序列并行支持，沿序列维度保持 Attention 激活分片，降低单卡显存占用。
3. **VirtioFS 检查点自动预取** (#51339)：当检测到文件系统为 `virtiofs` 时，自动启用受 RAM 保护的 safetensors 检查点预取，提升模型加载性能。
4. **KV Cache 淘汰策略优化** (#51348)：优化核心缓存块管理，优先淘汰从未命中过的缓存块，避免其与高频命中块混合影响性能。

**🛠️ Bug 修复**
1. **投机解码方法识别修复** (#51338)：修复了推测性解码在未显式指定 `method` 时，错误依赖路径名启发式推断，导致草稿模型静默回退到普通自回归模式的问题。
2. **Prompt Token 计数修正** (#51343)：前端修复，从 `usage.prompt_tokens` 统计中排除强制添加的生成前缀，确保返回给用户的 token 用量准确。

**🏗️ CI 与文档**
1. **CI 依赖与构建修复** (#51345, #51342, #51341)：修复了 Buildkite 过期的源依赖（MiniMax, DeepSeek V4, Mamba 等），解决了 `mkdocs` 严格构建失败及 pre-commit 挂钩失效的问题。
2. **文档格式修复** (#51344)：修复了 `committers.md` 中违反 markdownlint 规范的空行问题。

---

### 🚀 Release (发布)
本次动态周期内**无新版本发布**。

---

## 🐛 Issues

### #51347 — [[Bug]: Qwen3OmniMoeThinkerForConditionalGeneration does not support LoRA yet](https://github.com/vllm-project/vllm/issues/51347)
- **作者**: zhao-huan  **时间**: 2026-08-07 10:38 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 22.04.5 LTS (x86_64) GCC …

### #51340 — [[Bug]: kernel_warmup() has no inter-rank synchronization between stages that issue real TP/EP collectives, causing startup hangs on multi-node deployments](https://github.com/vllm-project/vllm/issues/51340)
- **作者**: b0bh00d  **时间**: 2026-08-07 09:05 CST
- **摘要**: ## Summary  `kernel_warmup()` (`vllm/model_executor/warmup/kernel_warmup.py`) runs a sequence of ~10 warmup stages, several of which internally call `runner._dummy_run(...)` and thereby issue real cross-rank TP/EP collectives (all-reduce, all-gather, etc.) against the model. There is no inter-rank s…

## 🔀 Pull Requests

### #51349 — [[Render][1/n] Paged shared memory storage for mm tensor ipc.](https://github.com/vllm-project/vllm/pull/51349)
- **作者**: noooop  **时间**: 2026-08-07 11:14 CST
- **标签**: nvidia
- **摘要**: <!-- markdownlint-disable --> PLEASE FILL IN THE PR DESCRIPTION HERE ENSURING ALL CHECKLIST ITEMS (AT THE BOTTOM) HAVE BEEN CONSIDERED.  ## Purpose  Paged shared memory storage for mm tensor ipc.  0. This compares two approaches for IPC of mm tensors:    - **Paged shared memory (pshm):** mm tensor →…

### #51348 — [[Core] cached blocks that never hit should be evicted first](https://github.com/vllm-project/vllm/pull/51348)
- **作者**: shanrow-amd  **时间**: 2026-08-07 10:56 CST
- **摘要**: ## Purpose  Most cached blocks with block hash that never hit. Now they are appended to the free list mixed with fewer hit blocks. We should differentiate the both cases because of the non-hit blocks amount that is huge and far greater than hit blocks. So we should append the non-hit blocks first an…

### #51346 — [[XPU] Add sequence parallelism support for DeepSeek V4](https://github.com/vllm-project/vllm/pull/51346)
- **作者**: majian4work  **时间**: 2026-08-07 10:31 CST
- **标签**: intel-gpu, deepseek
- **摘要**: ## Purpose  Add sequence parallelism (SP) support to the XPU DeepSeek V4 model path.  Attention activations are kept sharded along the sequence dimension across TP ranks, so each rank only materializes `num_tokens / tp_size` rows for the MoE and hyper-connection stages. This reduces activation memor…

### #51345 — [[CI] Fix stale Buildkite source dependencies](https://github.com/vllm-project/vllm/pull/51345)
- **作者**: khluu  **时间**: 2026-08-07 09:33 CST
- **标签**: ci/build
- **摘要**: ## Summary  - update MiniMax, DeepSeek V4, and Mamba source dependencies after their native sources moved under `csrc/libtorch_stable/` - point MiniMax and sampler dependencies at their current Python modules, and remove the deleted Engine request-logger path - remove the two FlashMLA backend trigge…

### #51344 — [[Docs] Fix markdownlint blank-line violations in committers.md](https://github.com/vllm-project/vllm/pull/51344)
- **作者**: chaojun-zhang  **时间**: 2026-08-07 09:25 CST
- **标签**: documentation
- **摘要**: ## Why `docs/governance/committers.md` is currently missing blank lines after two list headers, which fails `markdownlint-cli2` under `pre-commit --all-files`. This causes the pre-commit CI check to fail on unrelated PRs (any PR triggers a full-repo lint run).  ## What Add the two missing blank line…

### #51343 — [[Frontend] Exclude forced generation prefix from usage.prompt_tokens](https://github.com/vllm-project/vllm/pull/51343)
- **作者**: smurthy024  **时间**: 2026-08-07 09:20 CST
- **标签**: frontend, kimi, k3
- **摘要**: Hey everyone this is my first every commit so pretty excited! hopefully i didn't miss any key steps during this process. thank you!  Tl;dr Engine prompt_token_ids may include a trailing decode stub (e.g. Kimi K3 channel-open markers). Add optional generation_prefix_len so OpenAI usage.prompt_tokens …

### #51342 — [[CI/Build] Fix strict docs build: annotate _to_serve_args parameter](https://github.com/vllm-project/vllm/pull/51342)
- **作者**: harjothkhara  **时间**: 2026-08-07 09:13 CST
- **标签**: performance
- **摘要**: ## Purpose  `mkdocs build --strict` is currently failing on `main`:  ``` WARNING - griffe: vllm/benchmarks/throughput.py:535: No type or annotation for parameter 'args' Aborted with 1 warnings in strict mode! ```  `_to_serve_args` documents `args` in its `Args:` section but leaves the parameter unan…

### #51341 — [fix pre-commit broken](https://github.com/vllm-project/vllm/pull/51341)
- **作者**: jikunshang  **时间**: 2026-08-07 09:07 CST
- **标签**: documentation
- **摘要**: ## Purpose  ## Test Plan  ## Test Result  --- <details> <summary> Essential Elements of an Effective PR Description Checklist </summary>  - [ ] The purpose of the PR, such as "Fix some issue (link existing issues this PR will resolve)". - [ ] The test plan, such as providing test command. - [ ] The …

### #51339 — [[Model Loader][Perf] Auto-prefetch VirtioFS checkpoints](https://github.com/vllm-project/vllm/pull/51339)
- **作者**: bvolpato  **时间**: 2026-08-07 09:02 CST
- **摘要**: ## Purpose  Enable RAM-guarded safetensors auto-prefetch when the checkpoint filesystem is reported as `virtiofs`.  The default strategy currently detects `VIRTIOFS` but leaves prefetch disabled because the automatic allowlist contains only NFS, NFS4, and Lustre. With a 1403.19 GiB GLM-5.2 checkpoin…

### #51338 — [[Bugfix][Spec Decode] Trust checkpoint-declared method over path-name heuristics](https://github.com/vllm-project/vllm/pull/51338)
- **作者**: WindChimeRan  **时间**: 2026-08-07 08:53 CST
- **标签**: bug, rust
- **摘要**: ## Problem  A speculators-format draft checkpoint passed via `--speculative-config` without an explicit `"method"` is silently run as a plain autoregressive draft model:  ```bash vllm serve Qwen/Qwen3-8B \   --speculative-config '{"model": "/path/to/checkpoints/6", "num_speculative_tokens": 15}' ```…
