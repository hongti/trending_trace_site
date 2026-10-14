# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-14 13:16 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🚀 Release
**v0.30.0rc2**
*   **版本亮点**：本次发布**无代码逻辑变更**，主要针对 Dockerfile 进行了快速修复，以确保官方 Docker 镜像能够如期正常构建。

---

### 📝 Issue
**#18664 文档与实际不符及性能疑问**
*   **核心内容**：用户反馈 `0.30.0rc1` 版本存在多处文档与实际不符的问题：
    1. 文档称 DeepSeek V4.1 做了 TQ4 特性优化，但实际不支持；
    2. 镜像版本标注错误（文档写 9.2.0，实际为 9.1.0），且同配置下 Qwen3 27b 性能不如 0.23.0 版本；
    3. 该版本不支持 `fuse_muls_add=true` 配置，导致 GLM5.3 双机混部在关闭该选项后性能不佳。

---

### 🔧 Pull Request

**🚀 性能优化**
*   **#18671 GLM V1 SP 性能优化**：在 DSA-CP 阶段保留 GLM V1 序列并行 MoE 的 token 布局，避免了 attention 前冗余的 gather 和再次 shard 操作。
*   **#18666 MiniMax 长序列性能优化**：将融合 MiniMax 稀疏 prepare 路径的支持范围扩展至 256K tokens，避免了在处理长文本时回退到分离的 Q/K normalization、RoPE 和 cache-insertion 链路。
*   **#18667 KVPool 查询优化 (v0.30.0)**：在非分层 AscendStore 的 lookup 和 save 操作中避免全前缀扫描，剔除无法查询的历史 chunks，提升性能。

**✨ 新特性**
*   **#18665 DeepSeek-V4 KV Cache 分片**：在 A2 算力上为 DeepSeek-V4 引入配合 DCP 使用的 C4 KV cache 分片功能（为 RFC #17485 的第一阶段实现）。

**🐛 Bug 修复**
*   **#18669 CPP Profiling 路由修复 (v0.30.0)**：修复了启用 PCP 时，CPP 启动 profiling 使用单个合成请求导致 V2 dummy batch 路由异常的问题。
*   **#18663 RoPE 析构函数修复**：为 RoPE tiling 基类添加了虚析构函数，修复潜在的内存泄漏或未定义行为。

**📚 文档与常规合并**
*   **#18662 / #18661 DeepSeek V4 TurboQuant 文档**：在 DeepSeek-V4-Flash 部署指南中补充了 `turboquant_4bit_nc` 的使用说明，介绍其在超长上下文场景下的内存优势（含 main 和 v0.30.0 两版）。
*   **#18668 Kimi-K3 指南更新**：将 Kimi-K3 部署指南迁移至最新，并更新其支持矩阵中的版本号。
*   **#18670 常规 Cherry-pick**：合并 PR 18465 的相关更新。

---

## 🐛 Issues

### #18664 — [[Doc]: Questions for `/zh-cn/v0.30.0rc1/user_guide/release_notes.html`](https://github.com/vllm-project/vllm-ascend/issues/18664)
- **作者**: Sunnyday1998  **时间**: 2026-10-14 12:00 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  1.官方0.30.0rc1文档中写的dsv4.1做了tq4特性优化，实际保持不支持； 2. 0.30.0rc1镜像实际是9.1.0，文档上写的9.2.0，且同配置qwen3.8  27b性能不及0.23.0镜像（官方文档描述有优化）； 3. 0.30.0rc1不支持fuse_muls_add为true的配置，glm5.3 flash双机混部关闭该选项后性能也不及0.23.0镜像（支持为true）。  ### Suggest a potential alternative/fix  _No response_

## 🔀 Pull Requests

### #18671 — [[Performance][Model] Preserve GLM V1 SP token layout through DSA-CP](https://github.com/vllm-project/vllm-ascend/pull/18671)
- **作者**: wzx0726  **时间**: 2026-10-14 13:15 CST
- **摘要**: ### What this PR does / why we need it?  GLM with the V1 model runner, sequence-parallel MoE and DSA-CP currently gathers the local token shard before attention, only for DSA-CP to shard it again. The attention result then returns through a global token layout before the model scatters it back. This…

### #18670 — [Cherry pick pr 18465](https://github.com/vllm-project/vllm-ascend/pull/18670)
- **作者**: czydyy  **时间**: 2026-10-14 12:57 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/c4df37dfcc7f72ca41cd43801a468526ecf99a2b

### #18669 — [[Backport][v0.30.0][BugFix] Fix CPP profiling dummy prefill routing with PCP](https://github.com/vllm-project/vllm-ascend/pull/18669)
- **作者**: dianWang325  **时间**: 2026-10-14 12:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport of #18622 to `releases/v0.30.0`.  CPP startup profiling uses a single synthetic request. With PCP enabled, the generic V2 dummy batch can otherwise retain decode-only attention state and bypass PCP `partition_batch()`, allowing a large profiling samp…

### #18668 — [[Doc][Model] Migrate Kimi-K3 guide to latest](https://github.com/vllm-project/vllm-ascend/pull/18668)
- **作者**: MaybeChz  **时间**: 2026-10-14 12:37 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Migrate the Kimi-K3 guide from `MaybeChz:docs/kimi-k3-v0.27.1` (f916e3f7a) to `main`, including its Kimi-K3 support matrix entries.  Change the two `v0.27.1rc` references to `latest`, remove Experimental from the document title, and mark both Kimi-K3 support …

### #18667 — [[Performance][KVPool][v0.30.0] Avoid full-prefix scans during lookup and save](https://github.com/vllm-project/vllm-ascend/pull/18667)
- **作者**: bowgneo  **时间**: 2026-10-14 12:17 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Backport #18277 to `releases/v0.30.0`.  Non-layerwise AscendStore lookup and save walk historical chunks that cannot be queried or addressed. Lookup also expands the locally computed HBM prefix into per-group hit sets, repeating hash conversion and set constr…

### #18666 — [[Performance] Enable fused MiniMax sparse prepare through 256K tokens](https://github.com/vllm-project/vllm-ascend/pull/18666)
- **作者**: HaoxinZong  **时间**: 2026-10-14 12:03 CST
- **标签**: documentation, module:tests, module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it  Forward-port of #18660 to `main`.  MiniMax-M3's fused sparse-prepare path currently falls back to the separate Q/K normalization, RoPE, and cache-insertion chain whenever a batch contains more than 512 tokens. Raise the validated routing limit to 262144 tokens…

### #18665 — [[Feature][DSA] Add C4 KV cache sharding with DCP for DeepSeek-V4 on A2](https://github.com/vllm-project/vllm-ascend/pull/18665)
- **作者**: Dunnn-Smith  **时间**: 2026-10-14 12:02 CST
- **标签**: module:tests, module:ops, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  This is the first implementation step of [RFC #17485](https://github.com/vllm-project/vllm-ascend/issues/17485).  DeepSeek-V4's DSA KV caches are replicated across TP ranks, which uses extra HBM for long contexts. This PR starts reducing that duplication by s…

### #18663 — [[BugFix] Add a virtual destructor to the RoPE tiling base class](https://github.com/vllm-project/vllm-ascend/pull/18663)
- **作者**: 756522197  **时间**: 2026-10-14 11:57 CST
- **摘要**: - vLLM main: https://github.com/vllm-project/vllm/commit/c4df37dfcc7f72ca41cd43801a468526ecf99a2b

### #18662 — [[Doc]: Document DeepSeek V4 TurboQuant serving options](https://github.com/vllm-project/vllm-ascend/pull/18662)
- **作者**: qjgggsse  **时间**: 2026-10-14 11:54 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?    Document --kv-cache-dtype turboquant_4bit_nc in the DeepSeek-V4-Flash deployment guide.    Explain its memory benefits for ultra-long-context workloads such as AI coding, operator requirements, and independence from weight and indexer cache quantization. Ex…

### #18661 — [[v0.30.0][Doc]: Document DeepSeek V4 TurboQuant serving options](https://github.com/vllm-project/vllm-ascend/pull/18661)
- **作者**: qjgggsse  **时间**: 2026-10-14 11:53 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Cherry-pick of PR https://github.com/vllm-project/vllm-ascend/pull/18662    Document --kv-cache-dtype turboquant_4bit_nc in the DeepSeek-V4-Flash deployment guide.    Explain its memory benefits for ultra-long-context workloads such as AI coding, operator req…

## 🚀 Releases

### [v0.30.0rc2](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.30.0rc2)
- **作者**: wangxiyuan  **时间**: 2026-10-13 23:24 CST
- **摘要**: There is no code change in this release. It just contains a quick fix for Dockerfile to make the official Docker image build as expected. If you're using v0.30.0rc1 official docker image, please upgrade to v0.30.0rc2 instead. Otherwise, Please just ignore this release.  ### Highlights  - Upgrade CAN…
