# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-13 15:07 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题反馈)
近期共收到 2 个 Issue，主要涉及性能与文档反馈：
*   **DFlash 吞吐量问题** (#21807)：用户反馈在 v0.30.0 版本中，当仅输入单个请求时，DFlash 的草稿吞吐量异常偏低。
*   **DeepSeek-V4.1-Flash 文档疑问** (#21804)：用户对 A2 系列 1p1d PD 分离部署教程中的 `--additional-config` 参数（特别是 `enable_fused_mc2` 的配置）提出疑问。

---

### 🔧 Pull Request (代码合并)
近期共有 11 个 PR，涵盖性能优化、新特性、Bug 修复及大量文档更新：

**1. 新特性与性能优化**
*   **MoE 通信量化默认启用** (#21812)：为 W4A8MXFP 专家默认启用 MXFP Combine 通信量化（设置 `comm_quant_mode=4`），此前该模式需手动覆盖，此更新提升了默认情况下的通信效率。
*   **Mooncake Push 支持 Prefill KVPP** (#21805)：为支持非混合 MLA/SFA 模型（如 GLM-5.3 W8A8C8），在 `MooncakePushConnector` 中引入了 prefill 侧的 KVPP 支持，优化了生产者副本的选择。

**2. Bug 修复**
*   **修复 Ascend recipe TP/DP 配置丢失** (#21808)：修复了在选择 Data + Expert Parallel 或多节点环境下继承单节点 DP 设置时，张量并行（TP）设置可能会丢失的问题。
*   **修复 SuperKernel 下的 MoE 专家重叠** (#21802)：通过为现有的 MoE 阶段引入显式的融合边界，修复了在 SuperKernel 作用域下共享/路由专家重叠失效的问题（仅靠不同的流不足以解决）。

**3. 文档与工程维护**
*   **DeepSeek-V4.1-Flash 部署文档完善** (#21803)：为 A3 添加 `hccs:device` 资源配置，标记 A2 专属的 RoCE 变量，并同步 `scheduler_config` 键。
*   **清理过时配置说明** (#21809)：移除迁移指南中关于已废弃环境变量的过时过渡期声明。
*   **修复教程参数与标签** (#21811, #21810)：修正 GLM-5.1 教程表中的 `Max Num Seqs` 值；修复 Kimi-K2.6 教程中引用的不存在的 release 标签。
*   **文档自动翻译** (#21806)：自动翻译了 10 个开发者指南相关的文档文件。

---

### 🚀 Release (版本发布)
*   本期动态中**无正式的 Release 发布**。
*   **Note**：从 PR #21801（将 DeepSeek V4.1 共置 replay 设置对齐）和 Issue 反馈可以看出，仓库目前正在积极筹备和打磨 **v0.32.0rc** 版本的文档与配置细节。

---

## 🐛 Issues

### #21807 — [[Bug]: v0.30.0 DFlash low draft throughput with single request input](https://github.com/vllm-project/vllm-ascend/issues/21807)
- **作者**: OfferPlease  **时间**: 2026-11-13 14:31 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: openEuler 22.03 LTS (aarch64) GCC version: (GCC) 10.3.1 Clang version: Could not collect CMake versio…

### #21804 — [[Doc]: Feedback for `/zh-cn/v0.32.0rc/tutorials/models/DeepSeek-V4.1-Flash.html`](https://github.com/vllm-project/vllm-ascend/issues/21804)
- **作者**: wangyuanfei-9527  **时间**: 2026-11-13 14:08 CST
- **标签**: documentation, llm-model, deepseek
- **摘要**: ### 📚 The doc issue  https://docs.vllm.ai/projects/ascend/zh-cn/v0.32.0rc/tutorials/models/DeepSeek-V4.1-Flash.html#52-1p1d-pd A2系列1p1d pd分离部署中   --additional-config  参数加了  "enable_fused_mc2":1,意思是在0.32版本中A2也支持这个算子了吗?看了代码目前是不支持的，请问笔误还是什么  <!-- Failed to upload "image.png" -->  ### Suggest a potentia…

## 🔀 Pull Requests

### #21812 — [[Performance][MoE] Enable MXFP Combine communication quantization by default for W4A8](https://github.com/vllm-project/vllm-ascend/pull/21812)
- **作者**: Karryking3  **时间**: 2026-11-13 14:52 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  W4A8MXFP experts currently fall back to unquantized MC2 Combine communication unless an override is supplied. Select `comm_quant_mode=4` by default for this quantization type, using the existing MC2 operator to quantize expert outputs for communication and re…

### #21811 — [[Doc] Fix Max Num Seqs values in GLM-5.1 tutorial table](https://github.com/vllm-project/vllm-ascend/pull/21811)
- **作者**: Sunfanxi0828  **时间**: 2026-11-13 14:48 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Table 9.1.1 in `docs/source/tutorials/models/GLM5.md` introduces itself with "The TP/DP columns show the values **per node** as configured in the Deployment scripts", but the two 950DT Products `PD — Server-P Node` rows (PD 135K High Throughput and PD 135K Lo…

### #21810 — [[Doc] Fix nonexistent release tags in Kimi-K2.6 tutorial](https://github.com/vllm-project/vllm-ascend/pull/21810)
- **作者**: Sunfanxi0828  **时间**: 2026-11-13 14:48 CST
- **标签**: documentation, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  `docs/source/tutorials/models/Kimi-K2.6.md` references release tags that do not exist in this repository:  - "Kimi-K2.6 was first supported in vLLM-Ascend v0.20.0rc1" - there is no `v0.20.0rc1` tag (`git tag -l "v0.20*"` returns only `v0.20.2rc1`). The tutori…

### #21809 — [[Doc] Remove stale transition-period claims for removed env vars](https://github.com/vllm-project/vllm-ascend/pull/21809)
- **作者**: Sunfanxi0828  **时间**: 2026-11-13 14:47 CST
- **标签**: documentation, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  The Migration Guide in `docs/source/user_guide/configuration/additional_config.md` still tells users that "Both environment variables and `--additional-config` are supported during the transition period" and provides a Quick Reference mapping 7 legacy environ…

### #21808 — [[BugFix][Recipes] Fix Ascend recipe TP/DP configuration](https://github.com/vllm-project/vllm-ascend/pull/21808)
- **作者**: SOMEONEUNSEEN  **时间**: 2026-11-13 14:33 CST
- **标签**: recipe
- **摘要**: ### What this PR does / why we need it?  Generated Ascend commands can lose tensor-parallel settings when selecting Data + Expert Parallel, or inherit single-node data-parallel settings in a multi-node TP deployment. These mismatches can cause incorrect device allocation and OOM during startup.  Thi…

### #21806 — [[Doc] Translated Doc files 2026-11-13](https://github.com/vllm-project/vllm-ascend/pull/21806)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-13 14:29 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **10** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/contribution/doc_writing.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/…

### #21805 — [[Feature] Support prefill KVPP in Mooncake Push](https://github.com/vllm-project/vllm-ascend/pull/21805)
- **作者**: Yuli-yx  **时间**: 2026-11-13 14:13 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Enable prefill-side KVPP with `MooncakePushConnector` for supported non-hybrid MLA/SFA models, including GLM-5.3 W8A8C8. Push previously chose a producer replica by destination rank. With KVPP, that rank can lack the layer's persistent cache, leaving a decode…

### #21803 — [[Doc][Model] DeepSeek-V4.1-Flash: add hccs:device resource config for A3, mark A2-only RoCE vars, sync scheduler_config key](https://github.com/vllm-project/vllm-ascend/pull/21803)
- **作者**: Mr-qiji  **时间**: 2026-11-13 13:48 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Fixes #19881  Related: kvcache-ai/Mooncake#4872, kvcache-ai/Mooncake#4363, kvcache-ai/Mooncake#4892, vllm-project/vllm-ascend#19551  The DeepSeek-V4.1-Flash tutorial (section 5.2, "A3 series") instructs readers to launch on Ascend 910C (A3) with plain default…

### #21802 — [[BugFix] Preserve MoE expert overlap with SuperKernel phase scopes](https://github.com/vllm-project/vllm-ascend/pull/21802)
- **作者**: Jingluo-nan  **时间**: 2026-11-13 12:54 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Preserve shared/routed expert overlap under SuperKernel by giving the existing MoE phases explicit fusion boundaries. Different streams alone were insufficient: fusion across branch/resource boundaries extended shared quantization or GEMM task lifetimes and d…

### #21801 — [[Cherry-pick][releases/v0.32.0rc][Doc] Align DeepSeek V4.1 colocated replay settings (from #21800)](https://github.com/vllm-project/vllm-ascend/pull/21801)
- **作者**: xingcanlee  **时间**: 2026-11-13 12:28 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Backport #21800 onto `releases/v0.32.0rc`.  Update Section 5.1, `Single-Node Colocated Deployment`, in the DeepSeek-V4.1-Flash documentation:  - Change the 950DT (A5) and A3 examples from `--swa-bounded-replay` to `--no-swa-bounded-replay`. - Add `--speculati…
