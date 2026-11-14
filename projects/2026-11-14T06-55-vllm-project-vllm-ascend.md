# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-14 14:55 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🚀 Pull Requests (PR)
近期 PR 活动主要集中在文档修复、Bug 修复以及引入新特性方面：

**1. 新特性**
*   **MoE A3 MSD SiTU 融合** (#21868)：为 MoE（混合专家）模型引入了可选的 A3 MSD SiTU 融合接口，支持 INT8 输入使用专用的 A3-only GE/ACLNN 后端。

**2. Bug 修复**
*   **构建修复** (#21870)：修复了 RTY/UT 构建过程中错误丢弃 AICPU 源码和目标文件的问题。
*   **上游测试修复** (#21869)：修复了 Ascend TP（张量并行）分片批次中路由专家捕获失败的问题，解决了 E2E 上游测试报错。
*   **算子状态修复** (#21864)：修复了在 310P 硬件上处理 ragged MTP（多Token预测）时，recurrent GDN 状态行未正确保留的问题。

**3. 文档与 CI 更新**
*   **文档配置纠正** (#21873, #21872, #21871)：集中修复了多份文档中的配置错误，包括更新过时的 P/D 配置键为 `weight_nz_mode`、修复量化文档中无效的离线端口参数、以及将基准测试示例中的参数更正为 `--backend`。
*   **GLM-5.3-Flash 适配** (#21866, #21865)：为 GLM-5.3-Flash 模型添加了基于 Ascend 910B 的 W8A8 P/D（预填充/解码分离）部署文档；同时对齐了该模型 A3 PD 的 CI 每日夜间测试配置。

**4. 其他优化**
*   **KV 传输日志增强** (#21867)：为 MooncakeConnectorV2 添加了简洁的心跳日志，提升了 KV Transfer 机制的日常运维可观测性。

### 🐛 Issues
*   本次提供的动态数据中未包含具体的 Issue 列表（部分 Issue 如 #17787 和 #21619 已在上述 PR 中被成功修复）。

### 📦 Release
*   近期无新的 Release 版本发布。

---

## 🔀 Pull Requests

### #21873 — [[Doc] Replace obsolete P/D config keys with weight_nz_mode](https://github.com/vllm-project/vllm-ascend/pull/21873)
- **作者**: wzi690421-droid  **时间**: 2026-11-14 14:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The P/D examples in `large_scale_ep.md` and `ucm_deployment.md` pass `enable_weight_nz_layout` and/or `enable_prefill_optimizations` in `--additional-config`. Neither key is declared or filtered by the current Ascend configuration factory. In a standard vLLM …

### #21872 — [[Doc] Fix offline port and API entrypoint in quantization docs](https://github.com/vllm-project/vllm-ascend/pull/21872)
- **作者**: wzi690421-droid  **时间**: 2026-11-14 14:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The offline example in `quantization.md` passes `port=8000` to `LLM`. That keyword is forwarded to `EngineArgs`, which has no `port` field, so construction fails. Remove the argument and its port-selection comment.  The online example invokes `vllm.entrypoint…

### #21871 — [[Doc] Use --backend in the serving benchmark example](https://github.com/vllm-project/vllm-ascend/pull/21871)
- **作者**: wzi690421-droid  **时间**: 2026-11-14 14:47 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The `vllm bench serve` example in `benchmarks/README.md` uses `--endpoint-type "openai-chat"`, which is rejected as an unrecognized argument. Replace it with `--backend openai-chat` to select the chat-completions request format.  The [pinned upstream parser](…

### #21870 — [[BugFix][Build] Stop dropping AICPU sources and objects from RTY/UT builds](https://github.com/vllm-project/vllm-ascend/pull/21870)
- **作者**: Sunfanxi0828  **时间**: 2026-11-14 13:49 CST
- **摘要**: ### What this PR does / why we need it?  Fixes #17787.  `csrc/cmake/rty_obj_func.cmake` globs AICPU sources with `${MODULE_DIR}/*_aicpu*.cpp` in all three macros (`add_modules_sources`, `add_modules_sources_with_soc`, `add_mc2_modules_sources`), but the RTY macros never define `MODULE_DIR`. They par…

### #21869 — [[BugFix][Upstream] Fix routed experts capture on Ascend TP-sharded batches (#21619)](https://github.com/vllm-project/vllm-ascend/pull/21869)
- **作者**: Sunfanxi0828  **时间**: 2026-11-14 13:47 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fixes #21619. The weekly E2E-upstream run (`e2e-upstream_singlecard`, shard 2) fails two cases in `tests/model_executor/test_routed_experts_capture.py`:  1. `test_routed_experts_capturer_dp_ep_gathered_shards` 2. `test_routed_experts_capturer_sp_modular_gathe…

### #21868 — [[Feature][MoE] Add opt-in A3 MSD SiTU fusion through shared interfaces](https://github.com/vllm-project/vllm-ascend/pull/21868)
- **作者**: jcy1112  **时间**: 2026-11-14 13:45 CST
- **标签**: documentation, module:tests, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  Add the A3 A8W4 MSD backend from hust17yixuan/vllm-ascend-kimi#59 through the existing SiTU Torch interfaces. INT8 inputs use an A3-only, versioned GE/ACLNN backend; FP8 inputs keep the current A5 implementation. This ports only the arch32 backend from donor …

### #21867 — [[Misc][KV Transfer] Add concise heartbeat logs for MooncakeConnectorV2](https://github.com/vllm-project/vllm-ascend/pull/21867)
- **作者**: yFeng-dev  **时间**: 2026-11-14 12:10 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The renewable leases introduced in #21425 report failures but provide little visibility into normal heartbeat delivery and lease handling. Add three concise log events for MooncakeConnectorV2:  - D emits one DEBUG `event=send` after each successful batch send…

### #21866 — [[Doc][Recipe] Add Ascend 910B W8A8 PD support to GLM-5.3-Flash overlay](https://github.com/vllm-project/vllm-ascend/pull/21866)
- **作者**: JosephYouNew  **时间**: 2026-11-14 11:47 CST
- **标签**: ready-precise, recipe
- **摘要**: ### What this PR does / why we need it?  Extends the existing `zai-org/GLM-5.3-Flash` overlay with **Ascend 910B** (Atlas 800I A2) support for the W8A8 checkpoint in prefill/decode disaggregation. The overlay previously only carried the `ascend_950pr_8x` block; nothing in it is changed.  Why: 1M-con…

### #21865 — [[CI] Align GLM-5.3-Flash A3 PD nightly with current tutorial](https://github.com/vllm-project/vllm-ascend/pull/21865)
- **作者**: Wyz-134  **时间**: 2026-11-14 11:04 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Align the ordinary A3 GLM-5.3-Flash PD nightly serving configuration with the current [A3 PD tutorial](https://github.com/vllm-project/vllm-ascend/blob/d0ec914c8ec7c321a981bd6b89f8cb66ca3bd587/docs/source/tutorials/models/GLM5.3-Flash.md#glm53-flash-5-3-a3). …

### #21864 — [[Ops][BugFix] Preserve recurrent GDN state rows for ragged MTP on 310P](https://github.com/vllm-project/vllm-ascend/pull/21864)
- **作者**: ychy7001  **时间**: 2026-11-14 10:26 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Resubmission of #18730. The original pull request was automatically closed when its head fork was accidentally deleted. This pull request restores the original head `50fbb5a95f56` and carries over the original validation below.  Fixes #17856 for the standalon…
