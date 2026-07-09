# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-09 09:11 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文简洁摘要：

### 📘 Issue
- **#11658 [Bug][Upstream]**：上游测试报错 `TypeError: argument of type 'Mock' is not iterable`。该问题出现在 `test_eagle_quantization.py` 测试用例中，涉及 Mock 对象的不可迭代类型错误。

### 🚀 Pull Request (PR)
**🔥 重要新特性**
- **#11656 [Feature]**：新增对 **NVIDIA Nemotron 3 Super** 模型（BF16格式）在 Ascend 端的适配与支持，已通过 OpenAI 兼容服务验证。
- **#11649 [Feature]**：增加 Trace（追踪）能力，便于问题排查与性能分析。

**🛠️ 关键修复 (BugFix)**
- **#11659 [v0.23.0]**：修复 **Block Table 溢出**问题。将 Mamba KV cache 组的推测性 block 计算移至最大 block 计算之后，此修复合入到了 `v0.23.0` 版本。
- **#11653 [v0.23.0]**：回退此前加入的 `allgatherEP MXFP4 quantization` (#11287)，该 PR 因缺少 topk 权重导致 **w4a8mxfp 量化功能中断**。
- **#11650 [BugFix]**：修复 **vLLM Ascend 在新版本 CANN 下的编译错误**。

**📝 文档与杂项 (Doc / Misc)**
- **#11657 [Misc]**：适配 vLLM 上游最新代码 (commit `37995019`)，当前无需修改 vllm-ascend 自身代码。
- **#11655 / #11651 [Doc]**：自动批量翻译文档，共更新约 185 个中文翻译文件。
- **#11652 [Doc]**：修复部分中文翻译的错误。
- **#11654 [Misc]**：清理 po 翻译文件中无用的 babel 提示信息。

### 📦 Release
- 本统计周期内**无新版本发布**。

---

## 🐛 Issues

### #11658 — [[Bug][Upstream]: TypeError: argument of type 'Mock' is not iterable](https://github.com/vllm-project/vllm-ascend/issues/11658)
- **作者**: jiangyunfan1  **时间**: 2026-07-09 08:59 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  tests/model_executor/test_eagle_quantization.py  https://github.com/vllm-project/vllm-ascend/actions/runs/28737304823…

## 🔀 Pull Requests

### #11659 — [[v0.23.0][BugFix] Resolve block table overflow](https://github.com/vllm-project/vllm-ascend/pull/11659)
- **作者**: Pz1116  **时间**: 2026-07-09 09:09 CST
- **摘要**: ### What this PR does / why we need it?  Backport #11466 to `releases/v0.23.0`.  This moves speculative block accounting for Mamba KV cache groups after the max block calculation to avoid block table overflow.  ### Does this PR introduce _any_ user-facing change?  No.  ### How was this patch tested?…

### #11657 — [[Misc]feat: adapt to vLLM main (37995019)](https://github.com/vllm-project/vllm-ascend/pull/11657)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-09 01:53 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...37995019` (1/1 steps).  No vllm-ascend changes needed.  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11656 — [[Feature][Ascend] Add NVIDIA Nemotron 3 Super support](https://github.com/vllm-project/vllm-ascend/pull/11656)
- **作者**: yooooo00  **时间**: 2026-07-09 01:49 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ## Summary  This draft PR ports NVIDIA Nemotron 3 Super BF16 support onto the current `vllm-ascend` main branch and validates it on Ascend with real OpenAI-compatible serving.  The changes are scoped to Ascend-specific runtime compatibility:  - support Nemotron-H non-gated latent MoE construction on…

### #11655 — [[Doc] Translated Doc files 2026-07-08](https://github.com/vllm-project/vllm-ascend/pull/11655)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-09 00:31 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **52** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents…

### #11654 — [[MISC] Clean up uesless babel tip in po](https://github.com/vllm-project/vllm-ascend/pull/11654)
- **作者**: wangxiyuan  **时间**: 2026-07-09 00:19 CST
- **标签**: documentation
- **摘要**: Clean up uesless babel tip in po  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11653 — [[v0.23.0][BugFix] Revert Add allgatherEP MXFP4 quantization (#11287) to fix w4a8mxfp break](https://github.com/vllm-project/vllm-ascend/pull/11653)
- **作者**: hust17yixuan  **时间**: 2026-07-09 00:15 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it? The #11287 break the w4a8mxfp quantization due to the missing of topk weight.   ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9…

### #11652 — [[DOC] Fix wrong CN translate](https://github.com/vllm-project/vllm-ascend/pull/11652)
- **作者**: wangxiyuan  **时间**: 2026-07-08 23:59 CST
- **标签**: documentation
- **摘要**: Fix wrong CN translate  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11651 — [[Doc] Translated Doc files 2026-07-08](https://github.com/vllm-project/vllm-ascend/pull/11651)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-08 23:19 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **133** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/governance.po</code> - …

### #11650 — [[BugFix] Fixing compilation errors in the new version of cann vLLM Ascend](https://github.com/vllm-project/vllm-ascend/pull/11650)
- **作者**: ZT-AIA  **时间**: 2026-07-08 23:12 CST
- **摘要**: ### What this PR does / why we need it? Fixing compilation errors in the new version of cann  vLLM Ascend.  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? pip install   - vLLM version:  - vLLM main: https://github.com/vllm-project/vllm/commit/

### #11649 — [Add trace capability](https://github.com/vllm-project/vllm-ascend/pull/11649)
- **作者**: qigaiwangguilai  **时间**: 2026-07-08 23:01 CST
- **标签**: documentation, module:tests, module:core, module:tools
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446
