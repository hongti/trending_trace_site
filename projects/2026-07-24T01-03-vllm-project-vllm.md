# vllm-project/vllm — 动态追踪

> 生成时间: 2026-07-24 09:03 CST

## AI 总结

## vllm-project/vllm 近期动态摘要

---

### 📌 Issue（3 项）

| # | 标题 | 亮点 |
|---|------|------|
| **#49649** | ROCm Sparse-MLA persistent-kernel 在 `gqa_ratio=64` fp8 下不安全 | 当前 `build()` 始终预计算持久化元数据并走 persistent work-stealing 路径，缺乏非持久化 kernel 回退，导致特定配置下行为不安全 |
| **#49643** | RFC：为 MoRIIO READ 模式通过异步 KV-load gating 启用完整 CUDA graphs | MoRIIO 的 P/D 分离中 WRITE 模式已支持完整 CUDA graphs，但 READ 模式（decode 拉取 KV）尚未支持，提议通过异步 gating 机制补齐 |
| **#49640** | Bug：CPU GDN attention 在非 AMX 的 AVX-512BF16 CPU 上回退到慢速 torch conv1d | AMD EPYC Turin (Zen5) 等仅有 AVX-512BF16而无 AMX 的 CPU 上，GDN attention 性能大幅退化 |

---

### 🔀 Pull Request（10 项）

#### 🐛 Bugfix（6 项）

| # | 标题 | 亮点 |
|---|------|------|
| **#49648** | Granite Tool Parser：提取被环绕文本包裹的 tool calls | Granite 模型的 chat template 会用 prose 包裹 `<|tool_call|>`，非流式 parser 仅做前缀剥离导致含前导文本的调用被丢弃 |
| **#49646** | Spec Decode：将 Qwen3DSparkModel 架构的 dflash 正确路由到 dspark | DeepSeek dflash draft 携带 `markov_rank=0` 的 Qwen3DSparkModel，之前路由不正确 |
| **#49645** | Core：将 DBO padding 传播穿过 grouped top-k routing | 修复 ROCm DBO 分布式测试失败——DBO slicing 在 grouped top-k routing 中丢失 schedule padding |
| **#49642** | step3.7-flash：为 MTP 层增加 per-layer config 查找的边界检查 | MTP draft 层 `layer_idx == num_hidden_layers` 越界，导致配置查找崩溃 |
| **#49641** | 修复 HF benchmark 未传 `--hf-split` 时崩溃 | `vllm bench serve` 在缺少该参数时抛出 TypeError |
| **#49639** | 恢复 RMSNorm CUDA kernel 中间计算的四舍五入边界 | 先前提交将中间值保持为单精度 float，改变了原有舍入行为，现已恢复 |

#### ✨ 新特性（3 项）

| # | 标题 | 亮点 |
|---|------|------|
| **#49647** | **Rubin (SM107)：启用 NVLink all-reduce 路径** | 为下一代 Rubin 架构 `sm_107` 补齐集体通信选择表中的 NVLink 优化路径，延续 Rubin 支持上线工作 |
| **#49644** | **Core：为 SimpleCPUOffloadConnector 添加磁盘卸载支持** | 实现 RFC #19854 的可插拔卸载后端扩展，在 Mooncake Store 外部连接器之外增加内置 SSD/disk offload 后端 |
| **#49636** | **DeepSeek-V4：添加可选 FlashInfer moe_ep expert 后端** | 为 DeepSeek-V4 mega-MoE 专家新增 FlashInfer `moe_ep` 计算路径，可与现有 `deep_gemm_mega_moe` 后端运行时切换 |

#### 📦 依赖更新（1 项）

| # | 标题 | 亮点 |
|---|------|------|
| **#49637** | Dependabot：minor-update 组批量更新 | 跨 1 个目录、168 个包的小版本升级（如 regex 2026.2.28→2026.7.19 等） |

---

### 🚀 Release

**本期无新版本发布。**

---

### 一句话总结

本期重点：**Rubin SM107 NVLink 支持、DeepSeek-V4 FlashInfer MoE 后端、磁盘卸载后端**三大新特性落地；多项 bugfix 覆盖 Granite tool parser、Spec Decoding 路由、MTP 边界检查、RMSNorm 舍入等；Issue 侧聚焦 ROCm MLA kernel 安全性、CUDA graphs READ 模式缺失、非 AMX CPU 性能退化三个架构级问题。

---

## 🐛 Issues

### #49649 — [[ROCm] Sparse-MLA persistent-kernel gate is unsafe for gqa_ratio=64 fp8 (no non-persistent kernel)](https://github.com/vllm-project/vllm/issues/49649)
- **作者**: MohitAMD  **时间**: 2026-07-24 09:01 CST
- **标签**: rocm
- **摘要**: ## Summary  `main`'s `vllm/v1/attention/backends/mla/rocm_aiter_mla_sparse.py::build()` currently **always** precomputes persistent metadata and passes `work_meta_data` (always the persistent work-stealing path). That is **correct** for GLM-5.1-FP8 DSA served at `data_parallel_size=8, tensor_paralle…

### #49643 — [[RFC]: Enable full CUDA graphs for MoRIIO READ mode via async KV-load gating](https://github.com/vllm-project/vllm/issues/49643)
- **作者**: edwinlim0919  **时间**: 2026-07-24 06:14 CST
- **标签**: RFC
- **摘要**: ### Motivation.  MoRIIO supports two KV-transfer modes for P/D disaggregation: - WRITE (producer pushes KV to the decode node): decode runs with FULL CUDA graphs. - READ (decode pulls KV from the prefill node): decode runs with PIECEWISE CUDA graphs with #48534.  READ is forced to piecewise because …

### #49640 — [[Bug]: [CPU] GDN attention falls back to slow torch conv1d on non-AMX AVX-512BF16 CPUs](https://github.com/vllm-project/vllm/issues/49640)
- **作者**: dineshchitlangia  **时间**: 2026-07-24 04:05 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>collect_env.py not applicable as the pods were removed</summary> </details> Related Environment:  - CPU: AMD EPYC Turin (Zen5, AVX-512BF16, no AMX); also verified on Intel Granite Rapids (AMX).  - Model: Qwen3.5-9B, BF16.   - vLLM: `main`.   - python:…

## 🔀 Pull Requests

### #49648 — [[Bugfix][Tool Parser] Granite: extract tool calls wrapped in surrounding text](https://github.com/vllm-project/vllm/pull/49648)
- **作者**: nikhilkulkarni1755  **时间**: 2026-07-24 08:20 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Granite wraps tool calls in surrounding prose per its chat template. The non-streaming parser drops those calls. It strips the `<|tool_call|>` marker only as a prefix, so leading text hides it. It also decodes with `json.loads`, which rejects trailing prose after the array. This fix loca…

### #49647 — [[Rubin] Enable NVLink all-reduce paths on SM107](https://github.com/vllm-project/vllm/pull/49647)
- **作者**: zaristei  **时间**: 2026-07-24 08:18 CST
- **摘要**: ## Purpose  Part of the ongoing effort to bring up **Rubin (sm_107)** support in vLLM (similar to #49387).  sm_107 has no entries in vLLM's collective-communication selection tables, so the optimized NVLink all-reduce paths are all skipped and traffic silently falls back to NCCL. This PR wires sm_10…

### #49646 — [[Bugfix][Spec Decode] Route dflash on Qwen3DSparkModel arch to dspark](https://github.com/vllm-project/vllm/pull/49646)
- **作者**: DiegoCao  **时间**: 2026-07-24 07:54 CST
- **标签**: bug, qwen
- **摘要**: DeepSeek's DFlash Qwen3 drafts (e.g. `deepseek-ai/dflash_qwen3_4b_block7`) ship the `Qwen3DSparkModel` arch with `markov_rank=0`. With `method="dflash"` — explicit or inferred from the `dflash_*` model name — vLLM EAGLE-renames the arch to the unregistered `DFlashQwen3DSparkModel` and crashes at loa…

### #49645 — [[Bugfix][Core] Propagate DBO padding through grouped top-k routing](https://github.com/vllm-project/vllm/pull/49645)
- **作者**: AndreasKaratzas  **时间**: 2026-07-24 07:48 CST
- **标签**: bug, v1
- **摘要**: The Distributed Tests / DBO group on ROCm fails in [build 11147](https://buildkite.com/vllm/amd-ci/builds/11147/list?jid=019f8bae-0610-4768-8be3-65983033438f&tab=output) when DBO slicing loses the scheduler padding metadata. Grouped top-k can consequently route scheduler-added dummy rows as real exp…

### #49644 — [[Feat][Core] Add disk offloading support to SimpleCPUOffloadConnector](https://github.com/vllm-project/vllm/pull/49644)
- **作者**: chengy-sysu  **时间**: 2026-07-24 07:14 CST
- **标签**: v1, kv-connector
- **摘要**: ## Summary  Implements the disk backend extension envisioned in RFC #19854 (pluggable offloading backends). While #45036 provides SSD offload via the external Mooncake Store connector, this PR adds a **native vLLM disk offloading path** with zero external dependencies — targeting scenarios where hos…

### #49642 — [[Model][Bugfix] step3.7-flash: bounds-check per-layer config lookups for MTP layers](https://github.com/vllm-project/vllm/pull/49642)
- **作者**: chanh  **时间**: 2026-07-24 06:13 CST
- **标签**: bug
- **摘要**: ## Purpose  Step3p5 multi-token-prediction (MTP) draft layers are constructed with `layer_idx == num_hidden_layers` — one past the base decoder layers. See `Step3p5AMultiTokenPredictor` in `step3p5_mtp.py`, which builds the draft blocks over `range(num_hidden_layers, num_hidden_layers + num_nextn_pr…

### #49641 — [[Bugfix] Fix HF benchmark datasets crashing when --hf-split is omitted](https://github.com/vllm-project/vllm/pull/49641)
- **作者**: Shubham-S-Bhatt  **时间**: 2026-07-24 05:56 CST
- **标签**: bug, performance
- **摘要**: ## Purpose  `vllm bench serve` crashes with `TypeError: string indices must be integers` when benchmarking a HuggingFace dataset and `--hf-split` is not passed.  Fixes #49480.  ### Root cause  `--hf-split` defaults to `None`, so `HuggingFaceDataset.load_data` calls `load_dataset(..., split=None, str…

### #49639 — [fix(kernel): restore scalar_t RMSNorm intermediate rounding boundary (#49616)](https://github.com/vllm-project/vllm/pull/49639)
- **作者**: Hasnaathussain  **时间**: 2026-07-24 03:55 CST
- **摘要**: ## Purpose  Fixes #49616  ### Problem & Root Cause In commit `225936a`, intermediate RMSNorm calculation in CUDA kernels was modified to keep normalized intermediate values in single-precision `float` through weight multiplication: `dst.val[j] = static_cast<scalar_t>(x * s_variance * w);`  Because f…

### #49637 — [Bump the minor-update group across 1 directory with 168 updates](https://github.com/vllm-project/vllm/pull/49637)
- **作者**: dependabot[bot]  **时间**: 2026-07-24 03:04 CST
- **标签**: rocm, ci/build, cpu, nvidia, dependencies
- **摘要**: Bumps the minor-update group with 168 updates in the / directory:  | Package | From | To | | --- | --- | --- | | [regex](https://github.com/mrabarnett/mrab-regex) | `2026.2.28` | `2026.7.19` | | [requests](https://github.com/psf/requests) | `2.32.3` | `2.34.2` | | [tqdm](https://github.com/tqdm/tqdm…

### #49636 — [[Model][MoE] DeepSeek-V4: add opt-in FlashInfer moe_ep expert backend](https://github.com/vllm-project/vllm/pull/49636)
- **作者**: mhoqueanik  **时间**: 2026-07-24 02:50 CST
- **标签**: deepseek
- **摘要**: Add a FlashInfer `moe_ep` compute path for the DeepSeek-V4 mega-MoE experts, selectable at runtime alongside the existing native `deep_gemm_mega_moe` backend.  - New module `fi_utils.py`: backend predicates, flashinfer moe_ep runtime bootstrap/finalize, NVFP4 prequant weight packing + epilogue alpha…
