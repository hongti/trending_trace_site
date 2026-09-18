# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-18 13:04 CST

## AI 总结

## vLLM 近期动态摘要（2026-09-18）

---

### 🐛 Issue（3 项）

1. **#57494 — ROCm gfx1151 编译失败**：`q_gemm.cu` 缺少 `half`/`half2` 的 `atomicAdd` 重载，导致源码编译中断。
2. **#57493 — ROCm gfx1151 注意力输出不稳定**：相同的贪婪解码请求在混入其他请求后，`ROCM_ATTN` 路径返回不同输出，存在正确性隐患。
3. **#57490 — InternLM 流式工具调用丢失**：当 `<|action_start|>` 标记被切分到不同流式分块时，解析器将标记片段当作普通内容输出，工具调用未被正确识别。

> 共性问题集中在 **ROCm gfx1151 平台兼容性**（编译 + 数值一致性）和 **流式解析鲁棒性**。

---

### 🔀 Pull Request（10 项）

**ROCm / AMD 优化（4 项，本批次重点）：**

- **#57497** — Qwen4Exp PLE n-gram 表 CPU offload：将 PLE n-gram 表驻留 pinned host memory，通过 UVA 在 GPU 读取，释放显存以支持更大模型。
- **#57491** — DeepSeek-V4.1 Engram 表移至主机内存：47.2 GiB/rank 的 n-gram 表从显存迁至 pinned host memory，大幅降低 VRAM 占用。
- **#57489** — DSv4.1 gfx950 FP4 压缩 KV 分页注意力：将压缩 KV cache 扩展至 ROCm gfx950，并新增 gfx950 原生 MXFP4 选项。
- **#57492** — gfx950（MI350X）W8A8 block-FP8 GEMM 调优配置：针对 GLM-5.3-Flash 的 MLA 投影形状提供 tuned Triton kernel。

**Bug 修复（4 项）：**

- **#57495 / #57487 / #57484**（三者关联，均修复 #57473）：Aria MoE expert 权重加载失败——fused mapping 生成了多余的 `.weight` 后缀，导致参数名不匹配，现已修正。
- **#57485** — XPU sleep 模式下 KV cache 释放测试修复：planner 预算值与实际分配（受 cache layout / 整块对齐约束）存在差异，测试断言已校正。

**新功能 / 前端（2 项）：**

- **#57488** — Muse Glimmer 统一解析器（Rust 前端）：支持整代结构化标签输出，替代此前的分片解析方案。
- **#57496** — GLM5Next CPU KDA 后端（Draft）：为 GLM-5.3-Flash 在 CPU 路径规划 KDA 后端，目前为骨架 PR。

---

### 📦 Release

本批次无新版本发布。

---

### 📌 总结

本轮活动的核心主题是 **ROCm 平台深度优化**：多项 PR 将大容量 n-gram 表从显存迁移至主机内存（PLE / Engram），并扩展 FP4 压缩 KV 和 block-FP8 GEMM 调优至 gfx950（MI350X），显著降低显存占用并提升 AMD 平台支持。同时修复了 Aria MoE 权重加载、InternLM 流式工具调用解析等问题。gfx1151 平台仍存在编译和数值一致性 Bug，值得关注。

---

## 🐛 Issues

### #57494 — [[Installation]: [ROCm][gfx1151] q_gemm.cu fails to compile: missing half and half2 atomicAdd overloads](https://github.com/vllm-project/vllm/issues/57494)
- **作者**: kvcache670  **时间**: 2026-09-18 12:59 CST
- **标签**: installation, rocm, quantization
- **摘要**: ### Your current environment   ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : AOSC OS (x86_64) GCC version                  : (GCC) 15.3.0 20260612 (AOSC OS) Clang version       …

### #57493 — [[Bug]: [ROCm][gfx1151] ROCM_ATTN returns different outputs for the same greedy request after other requests](https://github.com/vllm-project/vllm/issues/57493)
- **作者**: kvcache670  **时间**: 2026-09-18 12:57 CST
- **标签**: bug, rocm
- **摘要**: ### Your current environment   <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : AOSC OS (x86_64) GCC …

### #57490 — [[Bug]: internlm loses tool calls when the opening marker is split across chunks](https://github.com/vllm-project/vllm/issues/57490)
- **作者**: kvcache670  **时间**: 2026-09-18 12:23 CST
- **标签**: bug, tool-calling
- **摘要**: ### Your current environment   <details><summary><code>vllm collect-env</code></summary>  ```text Collecting environment information... uv is set ==============================         System Info ============================== OS                           : Ubuntu 24.04.4 LTS (x86_64) GCC version  …

## 🔀 Pull Requests

### #57497 — [[Qwen4Exp][ROCm] PLE n-gram table CPU offload + async prefetch](https://github.com/vllm-project/vllm/pull/57497)
- **作者**: mrodden  **时间**: 2026-09-18 13:04 CST
- **摘要**: <!-- markdownlint-disable -->  ## Purpose - Enable `VLLM_PLE_CPU_OFFLOAD` on the AMD/ROCm path: keep the Qwen4Exp PLE n-gram table in pinned host memory, read through a UVA view on the GPU. - Free VRAM so `Qwen/Qwen3.8-Flash-Next-FP8` (FP8 table ~51 GiB) / `Inferact/Qwen3.8-Flash-Next-NVFP4` (bf16 t…

### #57496 — [[Draft][CPU] Add KDA backend for GLM5Next](https://github.com/vllm-project/vllm/pull/57496)
- **作者**: kunkunblueberry  **时间**: 2026-09-18 13:03 CST
- **摘要**: ## Summary  This draft PR tracks the planned CPU KDA backend for GLM-5.3-Flash / GLM5Next in #57346.  No implementation code is included yet. The branch and upstream synchronization are complete so the implementation can proceed from the reviewed baseline.  Refs #57346  ## Scope  The first implement…

### #57495 — [fix: resolve Aria expert checkpoint loading failure](https://github.com/vllm-project/vllm/pull/57495)
- **作者**: debanshd  **时间**: 2026-09-18 13:03 CST
- **摘要**: Fixes #57473  The fused mapping in `routed_experts.py` previously replaced the `experts.gate_up_proj` string with `experts.routed_experts.w13_weight`, which caused the `.weight` suffix to be retained incorrectly for checkpoint keys that already included it (like `experts.fc1.weight` mapped to `exper…

### #57492 — [[Kernel] Add tuned W8A8 block-FP8 GEMM configs for gfx950 (MI350X)](https://github.com/vllm-project/vllm/pull/57492)
- **作者**: mustafayildirim  **时间**: 2026-09-18 12:44 CST
- **标签**: quantization
- **摘要**: ## Purpose  Add tuned W8A8 block-FP8 triton GEMM configs for gfx950 (AMD Instinct MI350X, `device_name=0x75b0`). These cover the four MLA projection GEMM shapes exercised by GLM-5.3-Flash (`Glm5NextForConditionalGeneration`); the configs directory currently has zero entries for gfx950, so the kernel…

### #57491 — [[ROCm][DSv4.1] Keep the Engram tables in host memory on ROCm](https://github.com/vllm-project/vllm/pull/57491)
- **作者**: JohnQinAMD  **时间**: 2026-09-18 12:43 CST
- **标签**: rocm, deepseek, DSv4.1
- **摘要**: ## Purpose  DeepSeek-V4.1-Flash's Engram n-gram tables are the largest single consumer of device memory — 47.2 GiB per rank at TP=4, about a third of everything the weights hold. CUDA moves them to pinned host memory and reads them over UVA (#56512); on ROCm two `is_cuda()` gates turn that off, so `…

### #57489 — [[ROCm][DSv4.1] Add gfx950 FP4 compressed-KV paged attention](https://github.com/vllm-project/vllm/pull/57489)
- **作者**: LiuYinfeng01  **时间**: 2026-09-18 12:22 CST
- **标签**: performance, rocm, deepseek, DSv4, DSv4.1
- **摘要**: ## Purpose  Extend the DeepSeek V4.1 compressed-KV cache introduced by #56935 to ROCm gfx950 and add a gfx950-native MXFP4 option.  - Keep the sliding-window cache in the existing FP8 DS-MLA record. - Store only compressed KV as either 288-byte NVFP4 (256-byte E2M1 payload + 32 E4M3 scales) or 272-b…

### #57488 — [[Frontend] Add Muse Glimmer unified parser with whole-generation structural tags](https://github.com/vllm-project/vllm/pull/57488)
- **作者**: abmfy  **时间**: 2026-09-18 11:58 CST
- **标签**: needs-rebase, rust, kimi, k3
- **摘要**: ## Purpose  Supersedes #52390 (thanks @sid-rp; the author stepped back, and this implements the review direction from that thread in the Rust frontend). For Muse Glimmer on the Rust frontend, `response_format: json_schema` (and any structured-outputs request) was silently ignored: the engine-side re…

### #57487 — [[Bugfix][Model] Fix Aria expert weight names and layout](https://github.com/vllm-project/vllm/pull/57487)
- **作者**: JiangLLM  **时间**: 2026-09-18 11:47 CST
- **标签**: bug
- **摘要**: ## Purpose  Fixes #57473.  Aria cannot load its expert weights. The name conversion leaves an extra `.weight` suffix, so the loader cannot find the right parameter. After that suffix is removed, the tensor shapes still do not match because HF and vLLM use different layouts.  This PR fixes both issue…

### #57485 — [[XPU] sleep mode: fix KV cache release test](https://github.com/vllm-project/vllm/pull/57485)
- **作者**: yma11  **时间**: 2026-09-18 11:40 CST
- **标签**: intel-gpu
- **摘要**: ## Purpose  kv_cache_memory_bytes is a planner budget. The final KV-cache allocation is constrained by cache layout and whole-block rounding, so it can be smaller than the configured value even when release works correctly.  1. Fix test_release_kv_cache_memory_preserves_generation to measure the act…

### #57484 — [[Bugfix][MoE] Fix fused expert mapping producing double .weight suffix](https://github.com/vllm-project/vllm/pull/57484)
- **作者**: zimingttkx  **时间**: 2026-09-18 11:33 CST
- **标签**: bug
- **摘要**: ## What  Fixes #57473.  The fused mapping entries in `build_expert_params_mapping` produce parameter names with a double `.weight` suffix (`w13_weight.weight` instead of `w13_weight`), causing `AttributeError` when loading Aria's pre-fused expert checkpoints.  ## Why  The fused `weight_name` (e.g. `…
