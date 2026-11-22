# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-22 15:20 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题反馈)
*   **#22766 引擎在长上下文下卡死**：在 TP8 且 `max_model_len=32768` 的长上下文环境下，使用 verl 的 `bypass_mode` 请求 `calculate_log_probs` 时，vllm-ascend V1 引擎会出现挂起（卡死且无报错回溯）现象。

### 🔧 Pull Request (代码合并)
**新特性与硬件支持**
*   **#22772 支持 GLM-5.3-Flash**：在 Ascend A5 (950DT) 芯片上为 GLM-5.3-Flash 模型启用 HiSparse 优化（交错使用 DSA 和 KDA，并通过 FP8 保持压缩 KeyPool）。
*   **#22773 支持 Qwen3.5-397B-A17B**：为 `Qwen/Qwen3.5-397B-A17B` 添加华为昇腾硬件的 Overlay 配置，包含 NPU 专属的启动参数、环境与镜像配置。
*   **#22769 GDN 性能提升**：优化 Model Runner V2 (MRV2)，使其能使用显式的 Mamba 检查点块 ID，并在现有模型前向传播期间将 GDN 状态导出到专用页面，保留暖预填充检查点。

**Bug 修复**
*   **#22774 KV 分配修复**：修复 Granite Swash 3B 模型（64维头、可学习注意力 sinks）因通用全注意力 KV 分配导致的层块状条纹问题，改为为其分配连续 KV。
*   **#22770 KV Pool 检测修复**：修复 AscendStore 调度器在 SSD 描述符存在时，可能遗漏可读 MemCache 前缀副本的问题。
*   **#22767 MRV2 行读取修复**：跳过 MRV2 对 `max_num_seqs` 填充行的 Engram 回溯读取，避免不必要的数据读取。

**维护与测试**
*   **#22768 vLLM 版本升级**：将底层 vLLM 版本从 **v0.33.0 升级至 v0.34.0**。
*   **#22764 CANN 更新**：同步 CANN 1122 相关更新。
*   **#22765 测试维护**：在 DeepSeek V4 Flash A5 nightly 配置中添加空行以重新触发夜间测试。
*   **#22771 文档翻译**：自动化翻译了 59 个中文文档文件。

### 🚀 Release (版本发布)
*   本期动态中未包含正式的 Release 发布信息，但需重点关注 **PR #22768**，该项目正在将底层依赖的 vLLM 核心版本升级对齐至 **v0.34.0**。

---

## 🐛 Issues

### #22766 — [Engine wedge (idle, no traceback) when verl bypass_mode requests calculate_log_probs on TP8 long-context](https://github.com/vllm-project/vllm-ascend/issues/22766)
- **作者**: dahai80  **时间**: 2026-11-22 11:22 CST
- **摘要**: ## Summary  With verl's `algorithm.rollout_correction.bypass_mode=True` + `rollout.calculate_log_probs=True`, the vllm-ascend V1 engine (TP8, `max_model_len=32768`) wedges during the generation long tail. Full symptoms:  - generation stalls with a partial finish count (e.g. 473/512 sequences finishe…

## 🔀 Pull Requests

### #22774 — [[BugFix][Attention] Allocate contiguous KV for sink attention graphs](https://github.com/vllm-project/vllm-ascend/pull/22774)
- **作者**: YujianGuo  **时间**: 2026-11-22 14:50 CST
- **标签**: module:tests, ci-run
- **摘要**: ### What this PR does / why we need it?  Fixes #22616.  Granite Swash 3B uses learnable attention sinks with 64-dimensional heads. The generic full-attention KV allocation can give its layers block-strided K/V views. When the model captures an ACL graph, the FIA V2 max-workspace call passes those vi…

### #22773 — [[Ops][Feature] Add Ascend overlay for Qwen/Qwen3.5-397B-A17B](https://github.com/vllm-project/vllm-ascend/pull/22773)
- **作者**: zhangxiaoqing667  **时间**: 2026-11-22 14:44 CST
- **标签**: recipe
- **摘要**: ### What this PR does / why we need it?  Add the Huawei Ascend hardware overlay for `Qwen/Qwen3.5-397B-A17B` under `recipes/overlays/`, carrying the NPU-specific launch arguments, environment, image pin and strategy support captured from a live vLLM-Ascend deployment.  ### Does this PR introduce _an…

### #22772 — [[Feature][HiSparse] Support GLM-5.3-Flash on Ascend A5](https://github.com/vllm-project/vllm-ascend/pull/22772)
- **作者**: LZZhang-HW  **时间**: 2026-11-22 14:39 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Enable opt-in HiSparse for GLM-5.3-Flash on Ascend 950DT (A5). Flash interleaves DSA and KDA. DSA keeps a compressed KeyPool for TopK selection and uses FP8 NoPE pages for attention. KDA keeps separate convolution and recurrent state.  ```mermaid flowchart LR…

### #22771 — [[Doc] Translated Doc files 2026-11-22](https://github.com/vllm-project/vllm-ascend/pull/22771)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-22 14:31 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **59** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents…

### #22770 — [[BugFix][KV Pool] Detect readable MemCache replicas after SSD descriptors](https://github.com/vllm-project/vllm-ascend/pull/22770)
- **作者**: tomleeai  **时间**: 2026-11-22 12:04 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The AscendStore scheduler can miss a readable MemCache prefix when the key metadata lists an SSD descriptor before a memory replica. For example, `gva_list() == [0, 4096]` has a resident copy that the worker can load, but the scheduler currently checks only t…

### #22769 — [[Performance][GDN] Preserve warm prefill checkpoints in dedicated pages on MRV2](https://github.com/vllm-project/vllm-ascend/pull/22769)
- **作者**: Dwight2458  **时间**: 2026-11-22 11:54 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Enable Model Runner V2 to consume an explicit Mamba checkpoint block ID and export the GDN state into a dedicated page during an existing model forward. The Ascend fused prefill path runs the recurrence on both sides of the checkpoint and retains the intermed…

### #22768 — [[Misc] Main2Main v0.34.0 (supersedes #22198, includes nightly fixes)](https://github.com/vllm-project/vllm-ascend/pull/22768)
- **作者**: jixuanlu  **时间**: 2026-11-22 11:44 CST
- **标签**: documentation, module:tests, module:ops, module:core, ci-run
- **摘要**: ## Summary  Upgrade the vLLM release tag from **v0.33.0** to **v0.34.0** on top of the already-merged #21469.  The main-pin upgrade landed with #21469, so this PR leaves it untouched and only completes the release-lane tag migration plus the adaptations for the differences between the two pins (veri…

### #22767 — [[BugFix] Skip MRV2 Engram lookback reads for padded rows](https://github.com/vllm-project/vllm-ascend/pull/22767)
- **作者**: brandonRong  **时间**: 2026-11-22 11:42 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  MRV2 launches one lookback program per row of a persistent window sized to `max_num_seqs`. The request-to-slot mapping contains only the active requests for the current step. When a batch shrinks, the old kernel still forms a pointer beyond that mapping for e…

### #22765 — [[Test] Add whitespace for DeepSeek V4 Flash A5 nightly](https://github.com/vllm-project/vllm-ascend/pull/22765)
- **作者**: LXX1122  **时间**: 2026-11-22 11:01 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Add one blank line to the shared environment block in the `DeepSeek_V4_Flash_A5` nightly config on `releases/v0.30.0`. This whitespace-only change creates a new PR revision for the requested nightly verification and does not set or change `HCCL_OP_EXPANSION_M…

### #22764 — [Cann 1122](https://github.com/vllm-project/vllm-ascend/pull/22764)
- **作者**: xqchen7  **时间**: 2026-11-22 10:43 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…
