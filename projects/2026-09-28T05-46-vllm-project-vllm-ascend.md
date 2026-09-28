# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-28 13:46 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📌 Issue (问题反馈)
本期共收到 2 个高权重模型的 Bug 反馈，主要集中在特定硬件与量化配置下的输出异常：
*   **Kimi-K2.6 结构化输出死循环 (#17621)**：在 Ascend A3 硬件上使用 W4A8 量化部署 Kimi-K2.6 时，结合 `json_schema` 进行结构化输出，模型偶尔会陷入重复循环。
*   **DeepSeek-V4-Flash 贪心解码输出损坏 (#17616)**：在 Atlas A2 (910B3) 部署 DeepSeek-V4-Flash W8A8 模型时，开启 `enable_dsa_cp` 或移除 dspark 投机解码后，贪心解码输出出现乱码损坏。

### 🔧 Pull Request (代码合并)
本期共有 8 个 PR，涵盖重要算子迁移、性能优化、Bug 修复及 CI 调整：
*   **重要特性与性能优化**：
    *   **#17617 (Kimi-K3 算子迁移)**：将 Kimi-K3 的注意力残差算子迁移至 AscendC（针对 A3 硬件），提升算子执行效率。
    *   **#17615 & #17610 (MLA prolog 统一)**：基于 CANN 算子统一 A3 和 A5 硬件上的 MLA (Multi-head Latent Attention) 预处理逻辑，支持带/不带 RoPE 的调用，并连接 DCP 当前 KV。
    *   **#17614 (KDA 性能优化)**：在服务启动前预先准备 Triton 预填充状态复制计划，优化 Kimi 模型的 prefill state gather/clear/scatter 性能。
*   **Bug 修复**：
    *   **#17613 (GLM MRV2 修复)**：修复启用 Model Runner V2 时 GLM 模型的验证报错问题，修正了 `enable_fa_quant` 函数的适用范围。
*   **CI 与文档维护**：
    *   **#17619 & #17618 (EPLB 夜间测试)**：调整夜间测试用例，部分切换至 Model Runner V1 运行 EPLB，部分移除 EPLB 设置。
    *   **#17612**：CI 流水线调试。
    *   **#17620 & #17611 (文档更新)**：自动翻译 13 个文档文件；并在 DeepSeek V4 Pro 部署指南中省略了 Node0 DP 的启动 rank 说明。

### 🚀 Release (版本发布)
*   本期暂无新的 Release 版本发布。

---

## 🐛 Issues

### #17621 — [[Bug]: Kimi-K2.6 sometimes gets stuck in a repetition loop with json_schema](https://github.com/vllm-project/vllm-ascend/issues/17621)
- **作者**: kuan27285  **时间**: 2026-09-28 13:41 CST
- **标签**: bug, llm-model, kimi-k2
- **摘要**: ### Your current environment  Our diagnostic setup: - Hardware: Ascend A3 - Model: Kimi-K2.6, ModelSlim W4A8 deployment - Deployment: disaggregated prefill/decode, 2P + 2D - Structured output backend: XGrammar - Sampling: temperature=0, top_p=1, streaming enabled - Community vLLM / vLLM-Ascend versi…

### #17616 — [[Bug] DeepSeek-V4-Flash-0731-w8a8 on Atlas A2 (910B3): corrupted greedy output with enable_dsa_cp, and when dspark speculative decoding is removed; working config enclosed](https://github.com/vllm-project/vllm-ascend/issues/17616)
- **作者**: amchen2310  **时间**: 2026-09-28 12:34 CST
- **摘要**: ### Environment  - Hardware: 8 × Atlas 800T A2 (Ascend 910B3, 8 NPUs × 64 GB per node) - Software: vLLM 0.27.1 + vLLM-Ascend v0.27.1rc1 (container image built from the v0.27.1rc1 sources) - Model: `DeepSeek-V4-Flash-0731-w8a8` (ModelScope, ModelSlim W8A8 recipe; recipe metadata: `config_id: deepseek…

## 🔀 Pull Requests

### #17620 — [[Doc] Translated Doc files 2026-09-28](https://github.com/vllm-project/vllm-ascend/pull/17620)
- **作者**: vllm-ascend-ci  **时间**: 2026-09-28 13:19 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **13** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepSeek-V4-Flash.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/DeepS…

### #17619 — [[CI][Nightly] Run two EPLB nightly cases on Model Runner V1](https://github.com/vllm-project/vllm-ascend/pull/17619)
- **作者**: yjyang62  **时间**: 2026-09-28 12:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Keeps EPLB enabled for two nightly cases, but runs them on Model Runner V1:  - `tests/e2e/nightly/multi_node/external_dp/config/QWEN3_235B_PD_3_5K_1_5k.yaml` - `tests/e2e/nightly/single_node/models/configs/Kimi-K2.6-w4a8-A3.yaml`  `VLLM_USE_V2_MODEL_RUNNER=0`…

### #17618 — [[CI][Nightly] Drop EPLB settings from two nightly cases](https://github.com/vllm-project/vllm-ascend/pull/17618)
- **作者**: yjyang62  **时间**: 2026-09-28 12:56 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Removes EPLB launch settings from two nightly cases so they run without expert load balancing:  - `tests/e2e/nightly/multi_node/external_dp/config/QWEN3_235B_PD_3_5K_1_5k.yaml` - `tests/e2e/nightly/single_node/models/configs/Kimi-K2.6-w4a8-A3.yaml`  Both drop…

### #17617 — [[Feat][Kimi-K3] Migrate attention residual operators to AscendC on A3](https://github.com/vllm-project/vllm-ascend/pull/17617)
- **作者**: Zoezxb  **时间**: 2026-09-28 12:56 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Migrate Kimi K3 attention-residual operators to AscendC on the upstream `main` branch. This follows the structure of [#17564](https://github.com/vllm-project/vllm-ascend/pull/17564), while carrying the A3 `arch22` implementation and the required operator regi…

### #17615 — [[Feature][Attention] Unify MLA prolog and connect DCP current KV](https://github.com/vllm-project/vllm-ascend/pull/17615)
- **作者**: Dawn952  **时间**: 2026-09-28 12:26 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  This PR combines the changes in #17610 and #17190 as two functional changes (plus a small CI typing fix):  1. Use `cann_ops_transformer.mla_prolog` for A3 and A5 MLA decode, with or without RoPE, and remove the superseded AscendC K3 prolog implementation and …

### #17614 — [[Performance][KDA] Prepare Triton prefill state-copy plans before serving](https://github.com/vllm-project/vllm-ascend/pull/17614)
- **作者**: dontyougetthere  **时间**: 2026-09-28 12:13 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Provide a Triton alternative to the Kimi prefill state gather/clear/scatter work in #17301 (comparison head `5bcbad36fdc30aef539abf0ccc99ef959d913104`). This does not change chunk KDA math, recurrent decode, or GLM KDA.  - Prepare worker-owned plans after cac…

### #17613 — [[BugFix] Fix GLM MRV2 Issue](https://github.com/vllm-project/vllm-ascend/pull/17613)
- **作者**: lcfenglinwan  **时间**: 2026-09-28 11:56 CST
- **标签**: ready-precise
- **摘要**: ### What this PR does / why we need it?  This modification primarily addresses the validation issue of the GLM model when Model Runner V2 is enabled. The `enable_fa_quant` function is only applicable to the MLA structure, and its entry conditions need to be restricted.  ### Does this PR introduce _a…

### #17612 — [[CI][Doc][Misc] debug CI](https://github.com/vllm-project/vllm-ascend/pull/17612)
- **作者**: hw-qianlingfeng  **时间**: 2026-09-28 11:53 CST
- **标签**: documentation, module:tests, module:tools
- **摘要**: ### What this PR does / why we need it? debug CI ### Does this PR introduce _any_ user-facing change? debug CI ### How was this patch tested? debug CI  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17611 — [[Doc] omit Node0 DP start rank in DeepSeek V4 Pro guide](https://github.com/vllm-project/vllm-ascend/pull/17611)
- **作者**: ZhangwenTaoHW  **时间**: 2026-09-28 11:45 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17610 — [[Refactor] Unify A3 and A5 MLA prolog on CANN operator](https://github.com/vllm-project/vllm-ascend/pull/17610)
- **作者**: Dawn952  **时间**: 2026-09-28 11:39 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Use `cann_ops_transformer.mla_prolog` for A3 and A5 MLA decode preprocessing, with and without RoPE. Non-RoPE calls pass `rope_sin=None` and `rope_cos=None`; A3 can now enter the fused prolog path without the old hardware gate.  Remove the AscendC `mla_prolog…
