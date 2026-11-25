# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-25 17:00 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issues (问题与讨论)
*   **[Bug] 高并发部署崩溃 (#23237)**：使用 `vllm-ascend:0.32.0rc1` 部署 GLM-5.3-Flash-w8a8 (1P1D) 时，48并发测试通过，但在64并发下发生崩溃。
*   **[Bug] 混合前缀缓存命中率失效 (#23238)**：在特定配置 (`hash_block_size=2`, DSv4 hybrid) 下，SWA 的 `find_longest_cache_hit` 在第二次对齐时返回 0，导致混合前缀缓存命中率静默坍塌至 0。
*   **[RFC] 废弃平衡调度 (#23229)**：提议废弃并停止维护 `balance_schedule` 功能，因其已不再作为主线功能进行积极维护。

### 🔧 Pull Requests (代码变更)
*   **[新特性] PCP 与内核增强**：
    *   **Owner-sharded KV 传输升级 (#23232)**：引入分阶段 P2P 传输、Engram 接收容量及有界重放修复，增强 KV 传输能力。
    *   **Ascend A3 验证 Demo (#23231)**：为 Ascend A3 添加了 TileLang KDA 验证 demo。
*   **[性能优化]**：
    *   **MegaMoE 权重视图缓存 (#23236)**：缓存静态 MXFP4 expert 权重视图，避免前向传播中反复构建数据类型视图，提升性能。
    *   **Qwen3.5 MRoPE 复用 (#23227)**：在同一次前向传播中跨层重用 MRoPE 查找，降低计算开销。
*   **[Bug 修复]**：
    *   **Mooncake 超时修复 (#23233)**：在 KV 传输时，若 ZeroMQ `DONE` 控制消息交换超时，将自动重试 Mooncake 层级 Push 完成操作。
    *   **MoE Router 数据类型修复 (#23230)**：修复 `AscendGateLinear` 中量化路由门控激活数据类型被错误转换的问题，保留原有激活数据类型。
    *   **KV 池初始化修复 (#23228)**：提前初始化 Memcache KV 池后端，修复构造 worker store 时的配置传递问题。
*   **[CI/文档/杂项]**：
    *   **CI 增强 (#23226)**：替换原有测试矩阵，新增 `GLM-5.3-Flash-W8A8-A3-ALL-COMBINED` 覆盖预填充、PD E2E 等组合工作负载。
    *   **文档翻译 (#23234)**：通过 deepseek-v4.1-flash 自动翻译了 116 个文档文件。
    *   **代码清理 (#23235)**：移除 v2 相关的无用死代码。

### 🚀 Release (版本发布)
*   *近期无新版本发布动态。*

---

## 🐛 Issues

### #23238 — [[Bug]: SWA `find_longest_cache_hit` returns 0 on the second reconciliation pass when `hash_block_size=2` (DSv4 hybrid), collapsing hybrid prefix-cache hits to 0](https://github.com/vllm-project/vllm-ascend/issues/23238)
- **作者**: fanshuxue-hw  **时间**: 2026-11-25 16:56 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text /usr/local/python3.12.13/lib/python3.12/site-packages/triton/language/extra/cann/extension/math_ops.py:13: FutureWarning: triton.language.extra.cann.extension.atan2 is deprecated and will be rem…

### #23237 — [[Bug]: GLM-5.3-Flash-w8a8基于vllm-ascend:0.32.0rc1版本1P1D部署启动成功，4K/1K性能测试，48并发OK，64并发崩溃](https://github.com/vllm-project/vllm-ascend/issues/23237)
- **作者**: rsy-hub4121  **时间**: 2026-11-25 16:54 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text [root:issue]$ python collect_env.py  /usr/local/python3.12.16/lib/python3.12/site-packages/triton/language/extra/cann/extension/math_ops.py:13: FutureWarning: triton.language.extra.cann.extensio…

### #23229 — [[RFC]: Deprecate and Sunset Balance Scheduling](https://github.com/vllm-project/vllm-ascend/issues/23229)
- **作者**: GDzhu01  **时间**: 2026-11-25 16:33 CST
- **标签**: RFC
- **摘要**: ### Motivation.  Balance scheduling (`balance_schedule`, enabled through `additional_config.scheduler_config.enable_balance_scheduling`) is no longer actively maintained as a mainline feature. We propose to deprecate it and remove it from vLLM Ascend main.  The feature implements a global admission …

## 🔀 Pull Requests

### #23236 — [[Performance] Cache static MXFP4 expert weight views for MegaMoE](https://github.com/vllm-project/vllm-ascend/pull/23236)
- **作者**: Hyx098130  **时间**: 2026-11-25 16:44 CST
- **标签**: module:tests, module:quantization
- **摘要**: ### What this PR does / why we need it?  The MXFP4 MegaMoE path constructs three dtype views per local expert on every forward: a packed `uint8` view of W13 and E8M0 views of the W13 and W2 scales. A layer with 128 local experts therefore constructs 384 tensor views per forward. These operations sha…

### #23235 — [[Misc] Remove v2 dead code](https://github.com/vllm-project/vllm-ascend/pull/23235)
- **作者**: zouzy5137  **时间**: 2026-11-25 16:44 CST
- **标签**: documentation, module:tests, module:core, ready-precise
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #23234 — [[Doc] Translated Doc files 2026-11-25](https://github.com/vllm-project/vllm-ascend/pull/23234)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-25 16:43 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **116** file(s) using `deepseek-v4.1-flash`:  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/ai-review.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/communit…

### #23233 — [[BugFix][KV Transfer] Retry Mooncake layerwise Push completion on timeout](https://github.com/vllm-project/vllm-ascend/pull/23233)
- **作者**: tomleeai  **时间**: 2026-11-25 16:41 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  With Mooncake V2 layerwise Push, Prefill sends a ZeroMQ `DONE` control message to Decode after the KV WRITE futures finish. If the exchange times out, the current implementation continues receiving on the same REQ socket and eventually gives up. Decode can ke…

### #23232 — [[Feature][PCP] Owner-sharded KV: staged P2P transfer, Engram receive capacity, bounded-replay fix (layer 3, stacked on #22958)](https://github.com/vllm-project/vllm-ascend/pull/23232)
- **作者**: foraxe  **时间**: 2026-11-25 16:39 CST
- **标签**: documentation, module:tests, module:core, merge-conflicts
- **摘要**: # [Feature][PCP] Owner-sharded KV: staged P2P transfer, Engram receive capacity, bounded-replay fix (layer 3, stacked on #22958)  > Draft. This PR packages the combined owner-sharded KV follow-up stack on top of > the published layers #22502 and #22958. Review only the 5 code commits plus the > rele…

### #23231 — [[Kernel] Add a TileLang KDA verification demo for Ascend A3](https://github.com/vllm-project/vllm-ascend/pull/23231)
- **作者**: dogeByte  **时间**: 2026-11-25 16:39 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  This draft brings the checkpoint projection and short correction recurrence from the GLM5Next KDA verification experiment in #22370 to an isolated Ascend A3 TileLang demo. It gives reviewers a small case where moving the cross-core wait can be tested on ident…

### #23230 — [[BugFix][MoE] Keep activation dtype for quantized router gates](https://github.com/vllm-project/vllm-ascend/pull/23230)
- **作者**: U1stRsouland  **时间**: 2026-11-25 16:36 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #23224.  `AscendGateLinear`'s tier-5 fallback cast the activation to the gate weight dtype unconditionally:  ```python if x.dtype != self.weight.dtype:     x = x.to(self.weight.dtype) ```  That is only valid for unquantized gates. A quantized gate store…

### #23228 — [[BugFix] Initialize the Memcache KV pool backend eagerly](https://github.com/vllm-project/vllm-ascend/pull/23228)
- **作者**: zouyida2052  **时间**: 2026-11-25 16:33 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Use the backend's default initialization when constructing a KV pool worker store. The worker passes the parallel configuration and connector extra configuration directly to the backend. This removes the model-specific lazy initialization request introduced i…

### #23227 — [[Performance] Reuse Qwen3.5 MRoPE lookups across layers within each forward](https://github.com/vllm-project/vllm-ascend/pull/23227)
- **作者**: Jingluo-nan  **时间**: 2026-11-25 16:33 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Extracts only forward-local MRoPE lookup reuse from #22489, rebased onto vLLM-Ascend main `e758d0ea2fdcc50581d7204f33837a15ac100102`.  Qwen3.5-family full-attention layers can share an identical rotary module and positions within one model invocation. Previou…

### #23226 — [[CI] Add GLM-5.3-Flash A3 combined RP3 workload coverage](https://github.com/vllm-project/vllm-ascend/pull/23226)
- **作者**: Wyz-134  **时间**: 2026-11-25 16:32 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  Replace the existing Flash A3 nightly matrix entry with `GLM-5.3-Flash-W8A8-A3-ALL-COMBINED`, covering prefill-dominated, PD E2E, warmed-prefix decode (two load levels), and the existing AIME2025 accuracy workload in one two-host service lifecycle. Weekly cas…
