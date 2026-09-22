# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-22 13:19 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🚀 Release 亮点
*   **vLLM Ascend v0.23.0.post1 发布** (2026-09-21)
    这是 v0.23.0 的首个修订版本，主要聚焦于提升稳定性与易用性。该版本包含了针对上一版的 Bug 修复、依赖项更新、CI 流水线优化以及文档改进。

### 🐛 Issue 动态
*   **DeepSeek-V4 兼容与性能问题**：
    *   在 `releases/v0.23.0` 分支上，DeepSeek-V4-pro 使用 dummy load 格式加载及 ACL-graph 权重更新时宣告失败（#17197）。
    *   有用户反馈在 A3 硬件上推理 DeepSeek-V4-Flash 时时延和吞吐表现较差，并询问是否必须启用 eager 模式（#17196）。
*   **多模态调度死循环 Bug**：
    *   在运行 Kimi K3 与 DSpark 多模态预填充时，Encoder lookahead 导致 encoder-cache 产生循环依赖，请求进度停滞，触发零进度的调度死循环（#17195）。

### 🔥 PR 更新
**新特性与优化**
*   **A5 设备算子支持**：为 A5 设备引入 Sink FlashAttention tiling 及元数据 AscendC 算子，并添加共享的 A5 MLA tiling 头文件与打包支持（#17198）。
*   **MLA 算子统一**：将 Kimi K3 的 MLA prolog 算子迁移至统一的 `mla_prolog_v3` 布局，并新增 DCP C8 MLA prolog 算子及更新内核/元数据路径（#17190）。
*   **KV Pool 安全传输**：支持针对循环状态的安全 Mooncake 逐层 PUTs 传输，在提升混合层传输能力的同时保留了缓存就绪边界（#17194）。
*   **量化性能优化**：在加载期间针对 block-FP8 标量进行对齐广播，避免显式实例化扩展的 FP32 标量，优化显存与速度（#17192）。

**Bug 修复**
*   **量化融合修复**：将 norm-quant 融合限制在 INT8 类型，防止 FP8 量化错误匹配 INT8 替换路径从而导致前向传播失败（#17191）。
*   **采样掩码内核修复**：修复并优化 Ascend 上的采样掩码打包，解决了因 `logits.stride(1) > 1` 导致 Triton-Ascend 出现 UB 溢出的问题（#17193）。

**CI 与测试改进**
*   优化 CI 触发逻辑：当仅 `test_selector.py` 变更时，不再强制运行全量测试套件（#17187）。
*   修复 Nightly 结果上传时因 `sys.path` 路径污染导致的 `bisect` 模块遮蔽问题（#17189）。
*   在单元测试中增加 `autouse=True` 的 pytest fixture，强制初始化 NPU 硬件属性（如 AI Core 数量）以防止报错（#17185）。

**文档**
*   自动翻译并同步了 96 个文档文件（#17199）。

---

## 🐛 Issues

### #17197 — [[Bug]: DeepSeek-V4 dummy load and ACL-graph weight update fail on releases/v0.23.0](https://github.com/vllm-project/vllm-ascend/issues/17197)
- **作者**: zyang6  **时间**: 2026-09-22 12:04 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  - Branch: `releases/v0.23.0` - Commit: `2c59ff104` (`[v0.23.0][Doc] Translated Doc files 2026-09-10 (#16202)`) - Model: DeepSeek-V4-pro - Launch flag: `--load-format=dummy` (bugs 1 and 2); ACL graph + RL `update_weights` (bug 3) - Hardware / CANN / torch_npu: to be prov…

### #17196 — [[Doc]: Feedback for `/zh-cn/main/tutorials/models/DeepSeek-V4-Flash.html`](https://github.com/vllm-project/vllm-ascend/issues/17196)
- **作者**: liuzhigang34394416  **时间**: 2026-09-22 11:50 CST
- **标签**: documentation, llm-model, deepseek
- **摘要**: ### 📚 The doc issue  在A3上推理deepseek v4 flash时，必须使能eager模式吗？按照脚本在A3上拉起模型后，时延和吞吐都比较差：  ### Suggest a potential alternative/fix  _No response_

### #17195 — [[Bug]: Encoder lookahead can cause a zero-progress scheduling loop with multimodal K3 DSpark](https://github.com/vllm-project/vllm-ascend/issues/17195)
- **作者**: stormchasingg  **时间**: 2026-09-22 11:48 CST
- **标签**: bug
- **摘要**: ## Summary  We observed requests making no further progress during multimodal prefill with Kimi K3 and DSpark on vLLM Ascend. Scheduler instrumentation shows a circular dependency between encoder-cache reclamation and the shifted encoder scheduling window:  1. A request reaches the end of a previous…

## 🔀 Pull Requests

### #17199 — [[Doc] Translated Doc files 2026-09-22](https://github.com/vllm-project/vllm-ascend/pull/17199)
- **作者**: vllm-ascend-ci  **时间**: 2026-09-22 12:51 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **96** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/user_stories/index.po<…

### #17198 — [[Feat][Attention] Sink FlashAttention tiling to A5 device kernels](https://github.com/vllm-project/vllm-ascend/pull/17198)
- **作者**: Dawn952  **时间**: 2026-09-22 12:25 CST
- **标签**: documentation
- **摘要**: ## Summary`n- add the A5 FlashAttention and metadata AscendC operators`n- add shared A5 MLA tiling headers and packaging support`n- register FlashAttention and metadata in the A5 custom-op build list`n`n## Validation`n- git diff --check`n- conflict-marker scan`n`nThe operator implementation is carri…

### #17194 — [[Feature][KV Pool] Support safe Mooncake layerwise PUTs for recurrent state](https://github.com/vllm-project/vllm-ascend/pull/17194)
- **作者**: Pz1116  **时间**: 2026-09-22 11:42 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Enable Mooncake hybrid layerwise transfer for aligned recurrent-state groups while preserving the cache-readiness boundary introduced by #16747.  Main currently rejects hybrid layouts containing recurrent state. Simply removing that restriction is unsafe: GDN…

### #17193 — [[BugFix][Kernel] Fix and optimize sampling mask packing on Ascend](https://github.com/vllm-project/vllm-ascend/pull/17193)
- **作者**: Liamup777  **时间**: 2026-09-22 11:40 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  The sampling-mask kernel can overflow UB on Triton-Ascend when `logits.stride(1) > 1`. For a batch-32, vocabulary-151936, stride-2 input, the original BLOCK_SIZE=8192 kernel fails compilation with:  `ub overflow, requires 12583424 bits while 1572864 bits avai…

### #17192 — [[Perf][Quantization] Broadcast block-FP8 scales during loading](https://github.com/vllm-project/vllm-ascend/pull/17192)
- **作者**: Oseltamivir  **时间**: 2026-09-22 11:35 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  Broadcast block scales over aligned weight tiles instead of materializing expanded FP32 scales in `resolve_block_scales`. Partial tiles retain the existing path. Complements #16922.  ### Does this PR introduce _any_ user-facing change?  Lower checkpoint conve…

### #17191 — [[BugFix][Quantization] Restrict norm-quant fusion to INT8](https://github.com/vllm-project/vllm-ascend/pull/17191)
- **作者**: Oseltamivir  **时间**: 2026-09-22 11:34 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Constrain both INT8 norm-quant fusion patterns with `dst_type=torch.int8`. Otherwise, FP8 quantization can match an INT8-only replacement and fail during the following FP8 matmul.  ### Does this PR introduce _any_ user-facing change?  FP8 models can start wit…

### #17190 — [[Feat][Attention] Migrate MLA prolog operators to unified layout](https://github.com/vllm-project/vllm-ascend/pull/17190)
- **作者**: Dawn952  **时间**: 2026-09-22 11:32 CST
- **标签**: documentation, module:tests
- **摘要**: ## Summary - migrate the Kimi K3 MLA prolog operators to the unified `mla_prolog_v3` layout - add the DCP C8 MLA prolog operator and register the updated kernels and metadata paths - update the Python dispatch and focused MLA tests for the renamed operator layout  ## Scope The `csrc` change is a dir…

### #17189 — [[Test][CI] Run nightly result upload via python -m to avoid bisect shadowing](https://github.com/vllm-project/vllm-ascend/pull/17189)
- **作者**: jiangyunfan1  **时间**: 2026-09-22 11:25 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Nightly result postprocess launches `tools/upload_to_openlibing.py` as a script path. That puts `tools/` on `sys.path`, so `import bisect` resolves to `tools/bisect` instead of the stdlib and upload fails with:  ``` ImportError: cannot import name 'bisect' fr…

### #17187 — [[CI] Do not force full suite when only test_selector.py changes](https://github.com/vllm-project/vllm-ascend/pull/17187)
- **作者**: shiqiangA  **时间**: 2026-09-22 11:12 CST
- **标签**: ci/build, ready-precise
- **摘要**: Remove test_selector.py from the ci_pipeline path filter so updates to the selector script no longer invalidate precise test selection.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm…

### #17185 — [[Test][Ops] Initialize NPU to prevent errors.](https://github.com/vllm-project/vllm-ascend/pull/17185)
- **作者**: Night-lbk  **时间**: 2026-09-22 10:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  This PR adds an `autouse=True` pytest fixture `setup_device_properties_for_ut` to ensure NPU hardware properties (e.g., number of AI Cores) are initialized via `init_device_properties_triton()` before running unit tests.  **Why we need it:** When running Trit…

## 🚀 Releases

### [vLLM Ascend v0.23.0.post1](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.23.0.post1)
- **作者**: yiz-liu  **时间**: 2026-09-22 10:47 CST
- **摘要**: ## v0.23.0.post1 - 2026.09.21  This is the first post release of vLLM Ascend v0.23.0. It includes the fixes, dependency updates, CI changes, and documentation updates merged into the v0.23.0 release branch after the v0.23.0 tag. Please follow the [official documentation](https://docs.vllm.ai/project…
