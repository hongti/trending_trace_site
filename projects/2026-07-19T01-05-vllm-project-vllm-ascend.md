# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-19 09:05 CST

## AI 总结

## vllm-project/vllm-ascend 近期动态摘要

---

### 🐛 Issue

1. **#12349 — DeepSeek-V3.12 2p1d 性能劣化**
   - A3 四机 2P1D 高吞吐配置下，vllm-ascend `26.1.0-B050` 输出吞吐 12372 tps，较 `26.0.0-B150`（14540 tps）劣化约 **15%**；大 EP 同样存在同等程度劣化。

2. **#12345 — DeepSeek V4 Pro 在 A3/A5 上内存泄漏**
   - A3/A5 部署 DeepSeek V4 Pro 时出现内存泄漏，PD 分离部署与常规部署均受影响。

---

### 🔀 Pull Request

**🔧 Bug 修复（关键）**

| PR | 修复内容 |
|---|---|
| **#12344** | Python 3.12 下 `torch-npu` 的 `get_memory_info` / `memory_allocated` 无法正确委托到 NPU，导致 NPU 内存 profiling 不一致，现已修复 |
| **#12343** | Qwen2 在 Ascend 编译路径使用 NPU rotary operator 时输出错误，修复为回退 eager 模式规避受影响算子 |
| **#12342** | `enable_npugraph_ex = False` 覆盖仅作用于主进程 `AscendConfig`，spawned worker 重新构造配置时丢失该设置，现已持久化到 worker 进程 |
| **#12352** | CPU UT 采集阶段因 polib 依赖导致 CI 失败，移除该依赖 |

**✨ 新特性 / 功能变更**

| PR | 内容 |
|---|---|
| **#12351** | 将原 `enable_sparse_c8` 拆分为两个独立配置：`enable_sparse_sfa_c8`（packed C8 KV cache，用于 Sparse Flash Attention）和 `enable_sparse_li_c8`（用于 Latent Inference），粒度更细 |
| **#12347** | Memcache 现要求逐层 GVA 保存路径显式完成新分配对象的数据拷贝，新增 `batched_layerwise_finalize` API |

**📦 CI / 文档 / 杂项**

| PR | 内容 |
|---|---|
| **#12353** | 文档头部通知栏更新，标注当前为稳定版 **v0.24.0rc** 文档 |
| **#12350 / #12348** | 从华为云镜像安装 Node.js ARM64 及 OpenCode（npm registry），解决 Ubuntu 22.04 仅提供 Node 12、setup-node 下载路径不可用等问题 |
| **#12346** | 重构 main2main workflow，使用 unchanged branch |

---

### 🚀 Release

本期未收录 Release 信息。文档 PR (#12353) 提及当前稳定版本为 **v0.24.0rc**，暗示新版本即将或已发布，但具体 Release 内容未在此次动态中出现。

---

## 🐛 Issues

### #12349 — [[Bug]: DeepSeek-V3.12 2p1d 性能相比release/v0.18.0性能劣化](https://github.com/vllm-project/vllm-ascend/issues/12349)
- **作者**: SparrowMu  **时间**: 2026-07-18 21:52 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>DeepSeek-V3.1-Terminus，PD分离，高吞吐配置，性能劣化15%</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  A3 四机2P1D，高吞吐配置（卡50ms），3.5K输入1.5K输出，并发1152，vllm-ascend 26.1.0-B050输出吞吐12372tps，vllm-ascend 26.0.0-B150输出吞吐14540tp…

### #12345 — [[Bug]: Memory leak when deploying DeepSeek V4 Pro on A3/A5](https://github.com/vllm-project/vllm-ascend/issues/12345)
- **作者**: yiz-liu  **时间**: 2026-07-18 20:44 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Your output of above commands here ```  </details>   ### 🐛 Describe the bug  We observed a memory leak when deploying DeepSeek V4 Pro on Ascend A3 and A5.  The issue occurs in both deployment mo…

## 🔀 Pull Requests

### #12353 — [[Doc][Misc] Update documentation header notification bar](https://github.com/vllm-project/vllm-ascend/pull/12353)
- **作者**: herizhen  **时间**: 2026-07-19 08:50 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it? This pull request updates the notification bar in the documentation header template to show that the user is viewing the stable release (v0.24.0rc) documentation and links to the latest developer preview.   ### Does this PR introduce _any_ user-facing change? …

### #12352 — [[BugFix][Doc] Avoid polib dependency during CPU UT collection](https://github.com/vllm-project/vllm-ascend/pull/12352)
- **作者**: zhao-stack  **时间**: 2026-07-19 07:55 CST
- **标签**: module:tools, ready
- **摘要**: ### What this PR does / why we need it?  #### Root cause and provenance  The CPU unit-test lane in [run 29653393420](https://github.com/vllm-project/vllm-ascend/actions/runs/29653393420) fails while collecting `tests/ut/_tools/test_generate_zh_docs.py`:  ```text tests/ut/_tools/test_generate_zh_docs…

### #12351 — [[Feature][v0.23.0] Split c8 to li c8 and sfa c8](https://github.com/vllm-project/vllm-ascend/pull/12351)
- **作者**: ZYang6263  **时间**: 2026-07-19 02:13 CST
- **标签**: documentation, module:tests, module:core, ready
- **摘要**: ### What this PR does / why we need it? This PR splits the original `enable_sparse_c8` option into two independent configurations:  - `enable_sparse_sfa_c8`: enables the packed C8 KV cache used by Sparse Flash Attention. - `enable_sparse_li_c8`: enables the C8 key and scale caches used by LightningI…

### #12350 — [[CI]fix(ci): install Node from Huawei Cloud mirror](https://github.com/vllm-project/vllm-ascend/pull/12350)
- **作者**: pkking  **时间**: 2026-07-18 22:10 CST
- **标签**: ci/build
- **摘要**: Download and verify a pinned Node.js ARM64 archive directly from Huawei Cloud, avoiding setup-node and GitHub download paths. Install OpenCode through Huawei Cloud's npm registry.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch t…

### #12348 — [[CI][Misc] Install Node and OpenCode from Huawei Cloud mirrors](https://github.com/vllm-project/vllm-ascend/pull/12348)
- **作者**: pkking  **时间**: 2026-07-18 21:48 CST
- **标签**: ci/build
- **摘要**: ## What this PR does / why we need it?  The Main2Main job failed in `Install opencode + main2main-flow` with:  ```text npm: command not found ```  Ubuntu 22.04's apt repository only provides Node.js 12, which is too old for the current `opencode-ai` postinstall script. This change therefore:  - down…

### #12347 — [[v0.23.0][Feature][KVCache] Finalize layerwise Memcache puts](https://github.com/vllm-project/vllm-ascend/pull/12347)
- **作者**: Pz1116  **时间**: 2026-07-18 21:31 CST
- **标签**: ready
- **摘要**: ### What this PR does / why we need it?  Memcache now requires the layerwise GVA save path to explicitly finish newly allocated objects after their data has been copied.  This PR:  - exposes the new `batch_write_finish(keys, results)` API through `MemcacheBackend`; - carries keys created by `batch_a…

### #12346 — [[CI] Refactor main2main workflow to use unchanged branch](https://github.com/vllm-project/vllm-ascend/pull/12346)
- **作者**: wjunLu  **时间**: 2026-07-18 21:21 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it? Refactor main2main workflow to use unchanged branch  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.24.0 - vLLM main: https://github.com/vllm-project/vllm/commit/85c09e9885e346ea1612da30ebff5a75f67d2350

### #12344 — [[Ops][BugFix] Fix NPU memory profiling on Python 3.12](https://github.com/vllm-project/vllm-ascend/pull/12344)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-18 20:44 CST
- **标签**: module:tests
- **摘要**: ## Problem  Generic vLLM memory profiling calls `torch.accelerator.get_memory_info` and `memory_allocated`. Under Python 3.12 with torch-npu, those generic APIs do not consistently delegate to the NPU allocator.  ## Fix  Keep the compatibility mapping in the centralized accelerator patch module:  - …

### #12343 — [[Ops][BugFix] Fix Qwen2 compiled-path outputs on Ascend](https://github.com/vllm-project/vllm-ascend/pull/12343)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-18 20:41 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Qwen2 can produce incorrect outputs when the Ascend compiled path uses the NPU rotary operator. Falling back the whole model to eager mode avoids the affected operator but also disables graph execution.  This patch routes `Qwen2ForCausalLM` rotary embedding t…

### #12342 — [[BugFix] Persist enable_npugraph_ex override for worker processes](https://github.com/vllm-project/vllm-ascend/pull/12342)
- **作者**: ShuhaoZhangTony  **时间**: 2026-07-18 20:36 CST
- **标签**: module:tests, module:core
- **摘要**: ## Problem  When `platform.py` overrides `enable_npugraph_ex = False` for PIECEWISE or NONE cudagraph modes, the change only affects the main-process `AscendConfig`. Spawned workers reconstruct their configuration from `vllm_config.additional_config`, which can still contain the default value. Worke…
