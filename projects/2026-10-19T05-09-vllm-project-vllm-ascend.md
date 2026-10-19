# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-19 13:09 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期（2026年10月19日）动态的中文摘要：

### 📋 Issue 动态
本期共有 2 个 RFC（征求意见稿），均聚焦于多模态模型在 Ascend 平台的 EAGLE 投机解码适配：
* **Qwen3-ASR EAGLE3 支持** (#19149)：探讨为 Ascend 上的 Qwen3-ASR 模型启用 EAGLE3 投机解码。
* **多模态嵌入兼容性** (#19146)：指出 Llama 4 和 Mistral Large 3 的 EAGLE drafters 在与多模态目标模型配合时存在属性报错，需解决多模态嵌入的兼容问题。

### 🔧 Pull Request 动态
本期共有 9 个 PR，涵盖版本适配、新特性、性能优化及 Bug 修复，重点如下：

**1. 重要适配与新特性**
* **适配 vLLM v0.32.0** (#19148)：将 NPU 技术栈适配至最新的 vLLM v0.32.0 版本，更新镜像构建与 CI 配置。
* **Qwen3-ASR EAGLE3 支持** (#19150)：呼应上述 Issue，在 Qwen3-ASR 封装中暴露 EAGLE3 能力，修复能力检查被拒的问题。
* **新增 AntAngelMed 部署教程** (#19152)：补充了 AntAngelMed 模型的部署指南。

**2. Bug 修复**
* **修复多模态嵌入合并报错** (#19147)：解决 Llama 4 和 Mistral Large 3 EAGLE drafters 中多模态嵌入合并导致的 `AttributeError`。
* **修复 Beam drafts 路由错误** (#19144)：将 `DFlash2BeamDraftModel` 正确路由至 V2 DFlash2 投机器，避免其落入普通 DFlash 逻辑。

**3. 性能优化与重构**
* **MLA 缓存读取性能优化** (#19151)：针对步幅式 MLA 历史缓存读取，使用原生 `GatherPa` 替代繁复的索引构建与打包操作。
* **Triton 内核补丁重构** (#19153)：将 v2 手动 Triton 内核 monkey patch 迁移至官方内核调度器注册。
* **对齐上游投机解码路由** (#19145)：使 Ascend MRV2 PCP 投机解码与上游 draft 路由对齐，MTP/Eagle3 直接复用上游实现。

**4. 文档与 CI**
* **文档翻译** (#19154)：自动翻译了 11 个文档文件。
* **完善 CI 测试产物** (#19142)：在夜间基准测试中上传完整的 AISBench `outputs/` 目录，便于检查结果与排查问题。

### 🚀 Release 动态
本期无新的 Release 发布。

---

## 🐛 Issues

### #19149 — [[RFC]: Enable EAGLE3 speculative decoding for Qwen3-ASR on Ascend](https://github.com/vllm-project/vllm-ascend/issues/19149)
- **作者**: drslark  **时间**: 2026-10-19 11:52 CST
- **标签**: RFC
- **摘要**: ### Motivation  Upstream [vllm-project/vllm#52824](https://github.com/vllm-project/vllm/pull/52824) declares `SupportsEagle3` on `Qwen3ASRForConditionalGeneration`. The audio wrapper already contains a Qwen3 language model capable of exposing EAGLE3 auxiliary hidden states, but without this interfac…

### #19146 — [[RFC]: Multimodal embedding compatibility for Llama 4 and Mistral Large 3 EAGLE on Ascend](https://github.com/vllm-project/vllm-ascend/issues/19146)
- **作者**: drslark  **时间**: 2026-10-19 11:46 CST
- **标签**: RFC
- **摘要**: ### Motivation  Llama 4 and Mistral Large 3 EAGLE drafters can fail during draft prefill with `AttributeError: ... '_embed_text_input_ids'` when paired with a multimodal target. Their `embed_input_ids` aliases `SupportsMultiModal.embed_input_ids`, which expects helper methods and state that these dr…

## 🔀 Pull Requests

### #19154 — [[Doc] Translated Doc files 2026-10-19](https://github.com/vllm-project/vllm-ascend/pull/19154)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-19 12:32 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **11** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/disaggregated_prefill.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/de…

### #19153 — [[Refactor][Worker] Migrate v2 Triton kernel patches to the official dispatcher](https://github.com/vllm-project/vllm-ascend/pull/19153)
- **作者**: xiaohaifeng1999  **时间**: 2026-10-19 12:25 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Replaces the v2 manual Triton kernel monkey patch with registration through the official kernel dispatcher.  `patch/worker/patch_v2/patch_triton.py` applied 27 manual assignments unconditionally at import time. This PR deletes it and registers the same kernel…

### #19152 — [[Doc] Add AntAngelMed deployment tutorial](https://github.com/vllm-project/vllm-ascend/pull/19152)
- **作者**: cywang250805  **时间**: 2026-10-19 12:19 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Add an AntAngelMed model deployment tutorial and register it in the Model Tutorials navigation so it is discoverable on the documentation website.  The guide covers the pinned v0.23.0 image and package versions, eight-device Docker setup, BF16 TP4 + DP2 + EP …

### #19151 — [[Performance][Attention] Use native GatherPa for strided MLA cache reads (v0.30.0)](https://github.com/vllm-project/vllm-ascend/pull/19151)
- **作者**: HackClawMxw  **时间**: 2026-10-19 12:18 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Strided MLA history-cache reads currently build per-token request/slot indices with `repeat_interleave`, `index_select`, and advanced indexing, then pack and copy the gathered KV. This adds substantial indexing overhead for long cached prefixes.  Use the exis…

### #19150 — [[Feature] Expose EAGLE3 support for Qwen3-ASR](https://github.com/vllm-project/vllm-ascend/pull/19150)
- **作者**: drslark  **时间**: 2026-10-19 11:53 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The Qwen3-ASR wrapper in the currently verified vLLM ref (`c4df37dfcc7f72ca41cd43801a468526ecf99a2b`) lacks `SupportsEagle3`, so capability checks reject an EAGLE3 draft even though the inner Qwen3 decoder implements auxiliary hidden-state capture. Upstream […

### #19148 — [[Misc] Adapt to vLLM v0.32.0](https://github.com/vllm-project/vllm-ascend/pull/19148)
- **作者**: FayeSpica  **时间**: 2026-10-19 11:49 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Adapt the NPU stack to the vLLM v0.32.0 release so the upgraded vLLM-Omni NPU CI image can run existing functional tests.  Update the image build defaults, CI refs and dependency bounds, align the V1/V2 runner and worker interfaces, preserve layout-aware KV p…

### #19147 — [[BugFix] Fix multimodal embedding merge in Llama 4 and Mistral Large 3 EAGLE drafters](https://github.com/vllm-project/vllm-ascend/pull/19147)
- **作者**: drslark  **时间**: 2026-10-19 11:47 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Llama 4 and Mistral Large 3 EAGLE drafters in the currently verified vLLM ref (`c4df37dfcc7f72ca41cd43801a468526ecf99a2b`) still alias `SupportsMultiModal.embed_input_ids` without inheriting its helper methods. Draft prefill can raise `AttributeError` for `_e…

### #19145 — [[Refactor][MRV2][PCP] Align speculative decoding with upstream draft routes](https://github.com/vllm-project/vllm-ascend/pull/19145)
- **作者**: recky-c  **时间**: 2026-10-19 11:33 CST
- **标签**: module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Align Ascend MRV2 PCP speculative decoding with the upstream draft routes instead of maintaining a separate replicated-draft handoff. MTP/Eagle3 reuse upstream PCP-sharded draft prefill and replicated subsequent draft decode; DSpark follows the upstream PCP=1…

### #19144 — [[BugFix] Route Beam drafts to the V2 DFlash2 speculator](https://github.com/vllm-project/vllm-ascend/pull/19144)
- **作者**: drslark  **时间**: 2026-10-19 11:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  `DFlash2BeamDraftModel`, added by [vllm-project/vllm#62074](https://github.com/vllm-project/vllm/pull/62074), falls through to ordinary DFlash in Ascend's V2 speculator factory. It needs the DFlash2 candidate-selection and sampling path.  Recognize both `DFla…

### #19142 — [[CI] Upload complete AISBench outputs from nightly tests](https://github.com/vllm-project/vllm-ascend/pull/19142)
- **作者**: MrZ20  **时间**: 2026-10-19 11:28 CST
- **标签**: documentation, ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  Upload the complete AISBench `outputs/` directory as a GitHub Artifact for YAML-driven nightly tests.  The benchmark summary is useful for checking results, while investigating failures often requires the generated configs, logs, detailed request data, and re…
