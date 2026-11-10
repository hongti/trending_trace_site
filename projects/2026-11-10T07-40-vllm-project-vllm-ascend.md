# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-10 15:40 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue 动态
*   **Kimi-K3 并发输出不一致 Bug (#21424)**
    *   **问题现象：** 在 Ascend A5 设备（32台/4节点，DP4 TP8 PP1）上运行 Kimi-K3 W4A8 MXFP 及 DSpark 投机解码时，发现即使在同一 DP rank 下使用固定种子，并发执行 prefill/decode 会导致输出结果发生变化。
    *   **影响：** 可能影响 Ascend 平台上 Kimi-K3 模型并发推理的确定性和稳定性。

### 🔧 PR 动态

**✨ 新特性与性能优化**
*   **MooncakeConnectorV2 引入心跳租约 (#21425, #21427)**：针对 KV Transfer 组件进行优化。此前生产者 KV 会在固定超时后被释放，导致仍在等待的解码器无法获取数据。新特性引入了可续约的心跳租约机制，确保 KV 数据在解码器消费完毕前不被提前清理。
*   **DeepSeek-V4.1 性能优化 (#21417)**：为适配图重放和序列并行引入的 token padding，在 AllGather MoE 路径中默认跳过 padded tokens 的专家路由，避免无效计算，提升模型运行性能。

**📖 文档与配置更新**
*   **Kimi-K3 池化 PD 模板支持 1M 上下文 (#21421, #21422)**：将 950DT/A3 Prefill 和 Decode 模板的 `--max-model-len` 统一设置为 1048576，对齐了文档中关于 1M 上下文配置的说明。
*   **多语言文档翻译 (#21426)**：自动翻译了 34 个文档文件（包含 KV Cache Pool Guide 等开发者指南）。

**🤖 CI 与测试验证**
*   **硬件环境配置优化 (#21423)**：CI 流程中允许在 a2/a3 环境使用 'ascend-1980'，在 a5 环境使用 'npu'。
*   **夜间验证与冒烟测试 (#21419, #21420, #21428)**：针对 `main` 分支和 `releases/v0.32.0rc` 版本创建了自动夜间测试验证 PR；同时在 karc 集群上发起了离线端到端冒烟测试（标记为 DO NOT MERGE）。

### 🚀 Release 动态
*   近期无正式版发布，但 PR 活动显示团队正在进行 **v0.32.0rc (候选发布版)** 的夜间自动化验证工作，为下一个正式版本做准备。

---

## 🐛 Issues

### #21424 — [[Bug]: Kimi-K3 with DSpark on Ascend: seeded output changes with concurrency on the same DP rank](https://github.com/vllm-project/vllm-ascend/issues/21424)
- **作者**: moonseeker1  **时间**: 2026-11-10 15:00 CST
- **标签**: bug
- **摘要**: ### Your current environment  32 Ascend A5 devices / 4 nodes, DP4 TP8 PP1 DCP8 EP32; Kimi-K3 W4A8 MXFP + Kimi-K3-DSpark-Block5 (3 speculative tokens). Observed November 5, 2026. Co-located prefill/decode. EvalScope 1.14.0.  Image recorded in experiment family: swr.cn-north-12.myhuaweicloud.com/atlas…

## 🔀 Pull Requests

### #21428 — [[DO NOT MERGE] ci(karc-smoke): rerun karc offline E2E validation on a fresh branch](https://github.com/vllm-project/vllm-ascend/pull/21428)
- **作者**: Longwt123  **时间**: 2026-11-10 15:40 CST
- **摘要**: Temporary end-to-end smoke validation for the offline karc cluster. **Do not merge.**  This repeats the validated smoke configuration on a fresh branch so the previous branch/PR is left untouched. Changes are confined to two workflow files:  - `.github/workflows/_selected_tests.yaml`   - set `UV_DEF…

### #21427 — [[Feature][KV Transfer] Add heartbeat leases to MooncakeConnectorV2](https://github.com/vllm-project/vllm-ascend/pull/21427)
- **作者**: yFeng-dev  **时间**: 2026-11-10 15:36 CST
- **标签**: documentation, module:tests
- **摘要**: Development continues in the original PR #21425, which includes the duplicate-DONE fix. The `mooncake-v2-heartbeat-leases` branch remains available for fetching.  ### What this PR does / why we need it?  MooncakeConnectorV2 currently releases producer KV after a fixed timeout even when a decoder is …

### #21426 — [[Doc] Translated Doc files 2026-11-10](https://github.com/vllm-project/vllm-ascend/pull/21426)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-10 15:13 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **34** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/KV_Cache_Pool_Guide.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #21425 — [[Feature][KV Transfer] Add heartbeat leases to MooncakeConnectorV2](https://github.com/vllm-project/vllm-ascend/pull/21425)
- **作者**: yFeng-dev  **时间**: 2026-11-10 15:01 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  MooncakeConnectorV2 currently releases producer KV after a fixed timeout even when a decoder is still waiting to consume it. Add renewable heartbeat leases so scheduler-admitted decode requests can retain producer KV while waiting for scheduling or completing…

### #21423 — [[CI] enable using 'ascend-1980' in a2, a3 and "npu" in a5](https://github.com/vllm-project/vllm-ascend/pull/21423)
- **作者**: xqchen7  **时间**: 2026-11-10 14:43 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it? enable using 'ascend-1980' in a2, a3 and "npu" in a5  ### Does this PR introduce _any_ user-facing change? NA  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   …

### #21422 — [[Doc][Misc] Set Kimi-K3 pooled PD templates to 1M context (v0.30.0)](https://github.com/vllm-project/vllm-ascend/pull/21422)
- **作者**: MaybeChz  **时间**: 2026-11-10 14:36 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Align Section 5.4 pooled PD serving with the 1M configuration in Section 5.3: set `--max-model-len 1048576` in all four 950DT/A3 Prefill and Decode templates, and enable Decode DCP with size 8 on 950DT products and 16 on A3. Update the English note and Chines…

### #21421 — [[Doc][Misc] Set Kimi-K3 pooled PD templates to 1M context](https://github.com/vllm-project/vllm-ascend/pull/21421)
- **作者**: MaybeChz  **时间**: 2026-11-10 14:36 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Align Section 5.4 pooled PD serving with the 1M configuration in Section 5.3: set `--max-model-len 1048576` in all four 950DT/A3 Prefill and Decode templates, and enable Decode DCP with size 8 on 950DT products and 16 on A3. Update the English note and Chines…

### #21420 — [[Verify] Nightly test on releases/v0.32.0rc](https://github.com/vllm-project/vllm-ascend/pull/21420)
- **作者**: LQDLove  **时间**: 2026-11-10 14:28 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/544f182d9f0c9bfe8bd7645525951d5e3692a542

### #21419 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/21419)
- **作者**: LQDLove  **时间**: 2026-11-10 14:27 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/24420b7a341af37a9bc3186e6fc6062512e0aa58

### #21417 — [[Performance][Model] Skip padded DSV4.1 expert routes by default](https://github.com/vllm-project/vllm-ascend/pull/21417)
- **作者**: liuyao0  **时间**: 2026-11-10 14:27 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  DeepSeek-V4.1 introduces token padding for graph replay and sequence parallelism (SP). In the AllGather MoE path, these padded tokens may still be assigned to local experts, resulting in unnecessary computations in the grouped GEMMs.  This PR eliminates these…
