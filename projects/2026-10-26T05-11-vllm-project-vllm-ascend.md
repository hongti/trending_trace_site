# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-26 13:11 CST

## AI 总结

# vllm-ascend 仓库近期动态摘要

## Issue
本次提供的动态中未包含 Issue 相关记录。

## Pull Request

共 10 个 PR，主要集中在以下方向：

### 🐛 Bug 修复
- **#19802**：修复 CI 中 AISBench 进程生命周期管理问题。原 `AisbenchRunner` 的 `Popen` 句柄仅观察 shell 而非实际 benchmark 进程，导致可执行文件缺失或提前退出时无法正确追踪，现予以修正。

### ✨ 新特性
- **#19798**：将 A5 MiniMax DSpark 的 CANN FlashAttn 选择扩展到原生 `DSparkMiniMaxDraftModel` 及旧架构，支持 eager-only、PP/PCP 等场景。
- **#19797**：集成自包含的 CANN 派生 GenericBlockSparseAttention（GBSA）算子包，包含 attention/metadata kernel、PyTorch/Meta 注册、custom-op 安装等，并修复了精度和因果性问题。
- **#19796**：支持从父 checkpoint 的 `dspark/` 目录直接加载原生 MiniMax M3.1 Flash BF16 或 MXFP8 草稿权重，无需预转换，并启用 FA3 attention。

### 🔧 架构对齐与优化（MRV2 / PCP）
- **#19801**：将 PCP 投机解码与上游 draft 路由对齐，移除 Ascend 维护的复制式 PCP draft 旁路（`pcp_utils`、runner 预恢复等），改用上游更简洁的 `PCPManager` hook 模型。
- **#19800**：MRV2 PCP 下，由 attention builder 和 decode 实现来决定是否构建 PCP 分片布局，而非由 runner 直接填充元数据。

### 🐳 Docker / CI
- **#19803**：在 Docker 镜像中预装 flash-attention-npu 依赖，免去手动安装。
- **#19795**：为 A2/A3（Ubuntu 和 openEuler）构建固定版本的 FlashAttention NPU。
- **#19794**：Docker 构建支持 `PIP_INDEX_URL`、`PIP_TRUSTED_HOST`、`GIT_PROXY` 等构建参数，方便使用 pip 镜像和 Git 代理。

### 📚 文档
- **#19799**：自动翻译 26 个文档文件至中文（zh_CN）。

## Release
本次提供的动态中未包含 Release 记录。

---

**总结**：本期 PR 活动重点围绕 **MiniMax DSpark 草稿模型的原生支持**、**PCP 投机解码与上游架构对齐**、**GBSA 稀疏注意力算子集成**以及 **Docker/CI 构建流程优化**展开，整体方向是降低部署复杂度并提升与 vLLM 上游的一致性。

---

## 🔀 Pull Requests

### #19803 — [[Misc] Install flash-attention-npu in Docker images](https://github.com/vllm-project/vllm-ascend/pull/19803)
- **作者**: xujiaz2000  **时间**: 2026-10-26 12:15 CST
- **摘要**: ### What this PR does / why we need it?  Preinstall flash-attention-npu in the Docker images so the dependency is available without a separate manual installation.  - Update Dockerfile, Dockerfile.a3, Dockerfile.openEuler, and Dockerfile.a3.openEuler. - Build from the MinghuasLab/flash-attention-npu…

### #19802 — [[BugFix][CI] Track and clean up the AISBench process lifecycle](https://github.com/vllm-project/vllm-ascend/pull/19802)
- **作者**: specetator  **时间**: 2026-10-26 12:11 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  `AisbenchRunner` currently launches a background shell command. Its `Popen` handle observes the shell, not the benchmark: a missing executable or an early exit without an uppercase `ERROR` log can leave it polling forever, and cleanup can leave the actual ben…

### #19801 — [[MRV2][PCP][SpecDecode] Align PCP speculative decoding with upstream draft routes](https://github.com/vllm-project/vllm-ascend/pull/19801)
- **作者**: recky-c  **时间**: 2026-10-26 12:09 CST
- **标签**: module:tests
- **摘要**: ## Purpose  Ascend maintained a replicated-PCP-draft bypass (`pcp_utils`, runner pre-restore, etc.). Upstream's model is simpler: MTP/Eagle3 shard draft prefill with the target through `PCPManager` hooks; DSpark/DFlash draft the global batch replicated. Each method should have one fixed route chosen…

### #19800 — [[MRV2][PCP][Attention] Decide PCP layout from builder outputs](https://github.com/vllm-project/vllm-ascend/pull/19800)
- **作者**: recky-c  **时间**: 2026-10-26 12:09 CST
- **标签**: module:tests
- **摘要**: ## Purpose  Under MRV2 PCP the runner fills `common_attn_metadata.pcp_context` only for target-shaped metadata. Attention builders and decode impls should decide whether to build the PCP-sharded layout from that **produced** context (and from the slot-mapping shape), not from the PCP config alone. T…

### #19799 — [[v0.30.0][Doc] Translated Doc files 2026-10-26](https://github.com/vllm-project/vllm-ascend/pull/19799)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-26 12:04 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **26** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/faqs.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/features/pd_colocated_mooncake_multi_insta…

### #19798 — [[Feature][Attention] Support native MiniMax DSpark drafts with CANN FlashAttn](https://github.com/vllm-project/vllm-ascend/pull/19798)
- **作者**: yingyingzizi  **时间**: 2026-10-26 11:57 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Extend the A5 MiniMax DSpark CANN FlashAttn selection to native `DSparkMiniMaxDraftModel` and the legacy prepared architecture. Preserve the eager-only, PP/PCP/DCP and adaptive-verification restrictions for these native/prepared drafts; keep the Qwen3 draft e…

### #19797 — [[Feature][Attention] Bundle GBSA operators with precision and causality fixes](https://github.com/vllm-project/vllm-ascend/pull/19797)
- **作者**: yingyingzizi  **时间**: 2026-10-26 11:57 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Provide a self-contained CANN-derived GenericBlockSparseAttention operator bundle, including attention/metadata kernels, PyTorch/Meta registration, custom-op initialization and regression coverage. Select the supported softmax precision consistently for FP8 i…

### #19796 — [[Feature][Model] Load native MiniMax DSpark drafts and enable FA3 attention](https://github.com/vllm-project/vllm-ascend/pull/19796)
- **作者**: yingyingzizi  **时间**: 2026-10-26 11:57 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Load native MiniMax M3.1 Flash BF16 or MXFP8 draft weights from the parent checkpoint's `dspark/` directory without preparing a converted checkpoint. Validate the complete tensor header/schema, map native names into a registered draft subclass, and reuse the …

### #19795 — [[CI][Docker] Build pinned FlashAttention NPU for A2 and A3](https://github.com/vllm-project/vllm-ascend/pull/19795)
- **作者**: yingyingzizi  **时间**: 2026-10-26 11:57 CST
- **摘要**: ### What this PR does / why we need it?  Install the optional ecosystem FlashAttention NPU implementation from pinned revision `9c9f00648fa150e84228a9b757990485fc670df3` in the A2/A3 Ubuntu and openEuler images. Source the CANN/ATB environment and resolve the installed architecture's development-lib…

### #19794 — [[CI][Docker] Honor pip mirror and Git proxy build arguments](https://github.com/vllm-project/vllm-ascend/pull/19794)
- **作者**: yingyingzizi  **时间**: 2026-10-26 11:57 CST
- **摘要**: ### What this PR does / why we need it?  Declare the optional `PIP_INDEX_URL`, `PIP_TRUSTED_HOST`, and `GIT_PROXY` Docker build arguments and forward the pip settings into the installation step across the eight image variants. Existing GitHub download expressions can then use the supplied proxy pref…
