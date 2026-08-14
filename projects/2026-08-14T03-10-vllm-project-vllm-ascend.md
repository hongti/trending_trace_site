# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-08-14 11:10 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue (问题反馈)
- **DeepSeek-V4-Flash 单机部署报错** (#14260)：
  用户反馈按照 DSpark 部署文档，在单机环境部署 `DeepSeek-V4-Flash-0731-w8a8` 模型时出现运行时报错（Runtime Error），亟待排查修复。

---

### 🔧 PR (代码合并)

**1. Bug 修复**
- **MTP 隐藏缓冲区分配逻辑修复** (#14250)：修复了 DeepSeek-V4 在未开启投机解码时仍错误分配 `_mtp_hidden_buffer` 的问题，现调整为仅在投机解码启用时分配，避免资源浪费。
- **KV Offload Mamba 状态块传输修复** (#14254)：修复了 Mooncake 接收端在处理对齐的 Mamba 状态组时，索引单位混用导致的错误，规范了块传输逻辑。

**2. 新特性与功能增强**
- **KV Transfer 指标完善** (#14252, #14253)：为 `MooncakeHybridConnector` 和 `SfaRemoteD2HConnector` 补齐了 `KVConnectorStats` 支持。至此，`kv_p2p` 系列的四个连接器全部支持通过 vLLM 标准管道上报 KV 传输指标，便于 P/D 分离等场景的监控。
- **MRV2 PP 传输调试日志** (#14257)：为 Model Runner V2 流水线并行（PP）的 IntermediateTensors 传输增加了 DEBUG 级别的 DFX 探针（通过计算 SHA256 指纹辅助排查数据传输问题）。
- **Prefill 适配** (#14255)：进行了 prefill 阶段的适配工作。
- **引擎调试统计** (#14259)：WIP 状态，新增 engine stat 调试功能。

**3. 文档更新**
- 更新了 310P 文档的镜像配置说明及图捕获大小示例 (#14261)。
- 新增了 `Qwen3.8-2.4T-A95B` 模型的部署指南文档 (#14258)。

**4. CI/测试**
- 新增 a5 环境的 nightly 多卡测试用例 (#14256)。

---

### 🚀 Release (版本发布)
- 近期暂无新版 Release 发布。

---

## 🐛 Issues

### #14260 — [[Bug]: 根据DeepSeek-V4-Flash DSpark部署文档单机部署DeepSeek-V4-Flash-0731-w8a8报错](https://github.com/vllm-project/vllm-ascend/issues/14260)
- **作者**: lxr88  **时间**: 2026-08-14 11:01 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> 驱动版本：26.0.rc1 vllm ascend版本 vllm                                     0.25.1+empty                 /vllm-workspace/vllm vllm_ascend                              0.19.1rc2.dev1265+g50e0ce608 /vllm-workspace/vllm-ascend  VLLM Ascend镜像版本：DeepSeekV4-flash-0731-a3 C…

## 🔀 Pull Requests

### #14261 — [[Doc][v0.23.0] Updated 310P document mirroring instructions and graph capture size examples](https://github.com/vllm-project/vllm-ascend/pull/14261)
- **作者**: YangShuai52  **时间**: 2026-08-14 11:06 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #14259 — [WIP: zhongxin: add debug stat](https://github.com/vllm-project/vllm-ascend/pull/14259)
- **作者**: wenjinhust  **时间**: 2026-08-14 10:50 CST
- **标签**: module:core
- **摘要**: add engine stat  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #14258 — [[Doc]The guide doc of Qwen3.8-2.4T-A95B model.](https://github.com/vllm-project/vllm-ascend/pull/14258)
- **作者**: Karryking3  **时间**: 2026-08-14 10:48 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? The guide doc of Qwen3.8-2.4T-A95B model.  ### Does this PR introduce _any_ user-facing change? No  ### How was this patch tested? Use the doc's command deployed model in A3 - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d39…

### #14257 — [[Feat][Worker] Add MRV2 PP transfer DFX logs](https://github.com/vllm-project/vllm-ascend/pull/14257)
- **作者**: LostFox11  **时间**: 2026-08-14 10:47 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Adds DEBUG-only DFX probes for Model Runner V2 pipeline-parallel `IntermediateTensors` transfers.  - Computes per-tensor SHA256 fingerprints before sending and carries them through the existing tensor-dict metadata path. - Verifies the reconstructed receive p…

### #14256 — [[CI] add a5 nightly multi test](https://github.com/vllm-project/vllm-ascend/pull/14256)
- **作者**: xqchen7  **时间**: 2026-08-14 10:46 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it? enable a5 nightly multi test  ### Does this PR introduce _any_ user-facing change? enable a5 nightly multi test  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9…

### #14255 — [prefill适配](https://github.com/vllm-project/vllm-ascend/pull/14255)
- **作者**: fazhenyao  **时间**: 2026-08-14 10:37 CST
- **标签**: module:ops
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.27.1 - vLLM main: https://github.com/vllm-project/vllm/commit/58d3918e3ea0a544ffedadad2ba84559e9c51d8f

### #14254 — [[BugFix][KV Offload] Normalize aligned Mamba state block transfer](https://github.com/vllm-project/vllm-ascend/pull/14254)
- **作者**: Dawn952  **时间**: 2026-08-14 10:28 CST
- **标签**: module:tests
- **摘要**: ## Problem  For aligned Mamba state groups, the Mooncake receiver selected a remote block with:  ```python len(remote_group_block_ids) - num_speculative_tokens - 1 ```  This mixes two different units: a block count and a speculative-token count. With DSpark enabled, short metadata lists can either r…

### #14253 — [[Feature][KV Transfer] Support KVConnectorStats for SfaRemoteD2HConnector](https://github.com/vllm-project/vllm-ascend/pull/14253)
- **作者**: MrlixiangWE  **时间**: 2026-08-14 10:09 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Last of the four `kv_p2p` connectors, after #14237, #14243 and #14252. With this one the whole family reports KV transfer metrics through the standard vLLM pipeline instead of nothing.  Two things differ from the Mooncake connectors. The transport is memfabri…

### #14252 — [[Feature][KV Transfer] Support KVConnectorStats for MooncakeHybridConnector](https://github.com/vllm-project/vllm-ascend/pull/14252)
- **作者**: MrlixiangWE  **时间**: 2026-08-14 10:02 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Third connector in the series after #14237 (plain Mooncake) and #14243 (layerwise): `MooncakeHybridConnector` had no KV transfer metrics either, so hybrid P/D deployments run blind — no transfer latency, no bytes, no failure counters.  Same approach as the ot…

### #14250 — [[BugFix] allocate MTP hidden buffer only for speculative decoding](https://github.com/vllm-project/vllm-ascend/pull/14250)
- **作者**: 845473182  **时间**: 2026-08-14 09:33 CST
- **摘要**: ### What this PR does / why we need it?  DeepSeek-V4 currently allocates `_mtp_hidden_buffer` for every deployment, even when speculative decoding is disabled. The buffer is only consumed by the MTP draft path, so in regular inference it permanently occupies NPU memory without being used.  This PR: …
