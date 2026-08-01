# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-01 09:04 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🚀 Release（版本发布）
近期无新版发布。

### 🐛 Issue（问题）
近期无新增 Issue 记录。

### 💡 Pull Request（拉取请求）
近期的 PR 主要集中在 **Bug 修复**、**ROCm/AMD 性能优化** 以及 **CI/测试稳定性提升**：

**1. 重要 Bug 修复**
*   **Elastic EP 权重传输修复** (#50641)：修复了非连续权重张量传输问题，此前 PyNccl 会将张量展平为连续内存而忽略了步长，导致非连续张量传输异常。
*   **多节点运行崩溃修复** (#50633)：修复了使用 `runai_streamer` 及对象存储（s3/gs/az）部署时，非主节点因拉取不到非张量模型文件而崩溃的问题。
*   **CUDA 初始化与 Fork 冲突修复** (#50639, #50638)：解决了 CI 和运行中因 fork 子进程导致 CUDA 重复初始化报错的问题（`Cannot re-initialize CUDA in forked subprocess`），通过阻止 common ops 导入时初始化 CUDA，以及在 CUDA fork 状态异常时强制使用 spawn 模式来规避。
*   **测试缓冲区溢出修复** (#50640)：修复了单体路由回放测试中，缓冲区分配容量（8 tokens）小于实际用例需求（16 tokens）导致的失败。

**2. 性能优化（ROCm / AMD）**
*   **Kimi-K3 模型专项优化** (#50637, #50634)：针对 AMD gfx950 架构，将 Kimi-K3 的 AttnRes 与 RMSNorm 融合进 Triton 内核；同时在 ROCm 上强制启用融合的 RMSNorm/sigmoid gate 自定义算子以提升解码性能。
*   **wvSplitK 性能退化规避** (#50636)：修复了 ROCm 未量化 GEMM 实现中因缺少成本模型，导致 `wvSplitK` 在特定形状下反而引起性能退化的问题，现已在退化的形状上跳过该策略。

**3. 内核与 CI 改进**
*   **Helion 内核基准测试** (#50635)：修复了基准测试重复运行时输入张量被修改的问题，现会在每次启动前重置被修改的参数，确保测试结果准确。
*   **新增精度测试** (#50632)：为基于 ROCm 的 AMD-Quark MXFP4 量化版 DeepSeek-V4-Flash 模型添加了 GSM8K 正确性测试。

---

## 🔀 Pull Requests

### #50641 — [[Elastic EP] Fix non-contiguous weight transfers](https://github.com/vllm-project/vllm/pull/50641)
- **作者**: itayalroy  **时间**: 2026-08-01 08:56 CST
- **摘要**: Elastic EP weight transfer passed each weight tensor directly to PyNccl send API, but PyNccl transfers each tensor as a flat contiguous memory and does not account for tensor strides. For non-contiguous tensor views, such as some DeepSeek-V3 weights, this caused new workers to receive incorrect weig…

### #50640 — [[Bugfix][Test] Fix monolithic routing replay test buffer capacity](https://github.com/vllm-project/vllm/pull/50640)
- **作者**: Amir-19  **时间**: 2026-08-01 08:32 CST
- **标签**: bug, ready
- **摘要**: ## Purpose  The test fixtures allocate routing replay buffers for 8 tokens, but some cases use up to 16 tokens. Set max_num_tokens=16 in all three fixtures to match the largest test case and prevent false failures.  ## Test Plan ```bash uv run --no-project python -m pytest \   tests/kernels/moe/test…

### #50639 — [[Bugfix][CI] Prevent common ops imports from initializing CUDA](https://github.com/vllm-project/vllm/pull/50639)
- **作者**: AndreasKaratzas  **时间**: 2026-08-01 08:01 CST
- **标签**: bug, nvidia, kimi, k3
- **摘要**: The [PyTorch Fullgraph job](https://buildkite.com/vllm/ci/builds/81692/summary?jid=019fb955-8175-4d2d-ac07-a66e527c95b1&tab=output) started failing with `Cannot re-initialize CUDA in forked subprocess`, while the [Extra Initialization job](https://buildkite.com/vllm/ci/builds/81692/summary?jid=019fb…

### #50638 — [[Bugfix] [CI Stabilization] Force spawn when CUDA is in a bad fork state](https://github.com/vllm-project/vllm/pull/50638)
- **作者**: taneem-ibrahim  **时间**: 2026-08-01 07:54 CST
- **标签**: bug, nvidia
- **摘要**: ## Purpose  A process forked from a CUDA-initialized parent cannot use CUDA — any initialization attempt raises `Cannot re-initialize CUDA in forked subprocess`. vLLM guards against this in `_maybe_force_spawn()`, but the guard never fires in that situation: `torch.cuda.is_initialized()` reports `Fa…

### #50637 — [perf(rocm): fuse Kimi-K3 AttnRes with RMSNorm](https://github.com/vllm-project/vllm/pull/50637)
- **作者**: JohnQinAMD  **时间**: 2026-08-01 07:04 CST
- **标签**: rocm, kimi, k3
- **摘要**: ## Summary  Fuse the RMSNorm immediately following AMD Kimi-K3 AttnRes into the existing gfx950 Triton kernel. The fused kernel preserves the intermediate BF16 materialization boundary before applying the second RMSNorm.  The model helper selects the fused path only for gfx950 with a following norm.…

### #50636 — [perf(rocm): skip wvSplitK on the shapes where it regresses (gfx950)](https://github.com/vllm-project/vllm/pull/50636)
- **作者**: JohnQinAMD  **时间**: 2026-08-01 07:03 CST
- **标签**: rocm
- **摘要**: ## Purpose  `rocm_unquantized_gemm_impl` selects `ops.wvSplitK` on shape admissibility alone — `m > 8 and 0 < n <= 5`, plus `k % 8 == 0` from `use_skinny`. There is no cost model, so every admissible shape is sent to the kernel whether or not it beats the GEMM the dispatch would otherwise reach.  Tw…

### #50635 — [[Kernel][Helion] Reset mutated inputs between benchmark repetitions](https://github.com/vllm-project/vllm/pull/50635)
- **作者**: yushangdi  **时间**: 2026-08-01 07:00 CST
- **标签**: needs-rebase
- **摘要**: ## Summary  - Snapshot tensor arguments declared in each Helion kernel's `mutates_args` metadata. - Restore those arguments before every warmup and measured launch so each repetition sees the same inputs. - Exclude reset and L2-cache-clear setup from event timings, and subtract the equivalent setup …

### #50634 — [perf(rocm): fuse Kimi-K3 KDA decode gate](https://github.com/vllm-project/vllm/pull/50634)
- **作者**: JohnQinAMD  **时间**: 2026-08-01 06:59 CST
- **标签**: rocm, kimi, k3
- **摘要**: ## Purpose  Force Kimi-K3's existing fused RMSNorm/sigmoid gate custom op on ROCm even when global custom ops are disabled. Other callers retain the default custom-op dispatch, and NVIDIA is unchanged because the override is selected only on ROCm.  The earlier #50054 was closed without merge. It tar…

### #50633 — [[Bugfix] Pull runai_streamer non-tensor model files on every node](https://github.com/vllm-project/vllm/pull/50633)
- **作者**: thegoldenflow  **时间**: 2026-08-01 06:45 CST
- **标签**: bug
- **摘要**: ## Purpose  With `--load-format runai_streamer` and a model hosted in object storage (`s3://`, `gs://`, `az://`), a multi-node deployment crashes on every node except the one that built the config, because the small non-tensor model files were only ever downloaded there.  Closes #50616  ## Root caus…

### #50632 — [[CI] Add GSM8K accuracy test for amd/DeepSeek-V4-Flash-MXFP4](https://github.com/vllm-project/vllm/pull/50632)
- **作者**: ColinZ22  **时间**: 2026-08-01 06:44 CST
- **标签**: rocm, ci/build, deepseek
- **摘要**: ## Purpose  Registers an `lm-eval-harness` GSM8K correctness test for the AMD-Quark **MXFP4** DeepSeek-V4-Flash checkpoint, which serves on ROCm via the `deepseek_v4_fp8` path with the AITER MXFP4 MoE backend and fused shared experts. Follow-up of #48044.  ### Accuracy threshold Thresholds are set t…
