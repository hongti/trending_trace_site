# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-22 13:16 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📋 Issue 动态
近期共收到 5 个 Issue，主要围绕 **文档改进与任务跟进**：
* 用户 TQ-66 和 jjrolddog 提交了多个文档相关反馈，记录了文档翻译/校对任务（如“任务一、任务二”）的完成进度，未涉及核心代码的 Bug 报告。

### 🔧 Pull Request (PR) 动态
PR 活动十分活跃，涵盖 Bug 修复、新特性支持、测试迁移和文档完善，具体归纳如下：

**1. 重要 Bug 修复 (针对 v0.30.0 及 KV Pool)**
* **#19443 & #19434**：针对 `releases/v0.30.0` 分支的 KV Pool (KVPP) 进行了关键修复。解决了在启用流水线并行 时 Memcache/GVA 分层路径的对象布局、地址对齐及地址表过滤问题，确保正确跳过非当前 rank 拥有的 GVA 层。

**2. 新特性支持**
* **#19430**：引入了 Qwen38 结合 Model Runner v2 与 dflash 的特性支持。
* **#19427**：提交了与 ZercMoE 相关的新功能代码。

**3. 测试与 CI 流水线优化**
* **#19442**：在 TP4 批次不变性端到端测试中，新增了 `Qwen3.5-35B-A3B` 混合 MoE 模型的测试用例，弥补了此前仅覆盖全注意力层模型的不足。
* **#19435**：将 29 个每周例行 CI 配置从 DSA-CP 迁移至 PCP，在保持数据并行 (DP) 和设备数量不变的前提下转换了并行策略。
* **#19428**：将每周的 EPLB（专家并行负载均衡）测试用例迁移至 Model Runner v2，并采用上游异步 EPLB CLI 配置。

**4. 文档与指南更新**
* **#19436**：CI 自动化翻译了 20 个中文文档文件。
* **#19432**：更正了上下文并行 的指南与图表标签，补充了 DSA-CP 启动示例及 PCP 连接器要求。
* **#19431**：统一了 MiniMax-M3 A5 在三节点环境下的 PD（Prefill/Decode）示例命令。

### 🚀 Release 动态
* **近期无新版本发布**。
* *注：尽管无新版本，但从 PR 动态来看，团队正在积极向 `v0.30.0` 分支定向合入 KV Pool 相关的 Bug 修复，表明该版本正处于重点维护与稳定性优化阶段。*

---

## 🐛 Issues

### #19441 — [[Doc]:](https://github.com/vllm-project/vllm-ascend/issues/19441)
- **作者**: TQ-66  **时间**: 2026-10-22 12:54 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  <img width="568" height="464" alt="Image" src="https://github.com/user-attachments/assets/4820b0ad-176f-4d43-b53a-414478ceef91" />  ### Suggest a potential alternative/fix  _No response_

### #19440 — [[Doc]:](https://github.com/vllm-project/vllm-ascend/issues/19440)
- **作者**: TQ-66  **时间**: 2026-10-22 12:48 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  任务二已完成  <img width="2549" height="1403" alt="Image" src="https://github.com/user-attachments/assets/e4e2ffb9-8038-4506-9bd3-e5ddb19a1fcd" />  ### Suggest a potential alternative/fix  _No response_

### #19439 — [[Doc]:](https://github.com/vllm-project/vllm-ascend/issues/19439)
- **作者**: jjrolddog  **时间**: 2026-10-22 12:38 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  <img width="2612" height="1394" alt="Image" src="https://github.com/user-attachments/assets/0cf6a282-b64f-4c9d-8393-d6681c446712" />  ### Suggest a potential alternative/fix  _No response_

### #19438 — [[Doc]:](https://github.com/vllm-project/vllm-ascend/issues/19438)
- **作者**: TQ-66  **时间**: 2026-10-22 12:38 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  任务一已完成 <img width="568" height="464" alt="Image" src="https://github.com/user-attachments/assets/84056936-454e-43a8-bffb-672bb7687ed8" />  ### Suggest a potential alternative/fix  _No response_

### #19437 — [[Doc]:](https://github.com/vllm-project/vllm-ascend/issues/19437)
- **作者**: TQ-66  **时间**: 2026-10-22 12:35 CST
- **标签**: documentation
- **摘要**: ### 📚 The doc issue  任务一已完成  [https://github.com/vllm-project/vllm-ascend/issues/11750](https://link.gitcode.com/?target=https%3A%2F%2Fgithub.com%2Fvllm-project%2Fvllm-ascend%2Fissues%2F11750&from=https%3A%2F%2Fgitcode.com%2FAscend%2FEcoDevHub%2Fdiscussions%2F5&lang=zh&theme=white)  ### Suggest a po…

## 🔀 Pull Requests

### #19443 — [[BugFix][v0.30.0][KV Pool] Skip unowned GVA layers under KVPP](https://github.com/vllm-project/vllm-ascend/pull/19443)
- **作者**: LifeOfCoffee  **时间**: 2026-10-22 13:01 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Fix the memcache/GVA layerwise path with KVPP enabled on `releases/v0.30.0`.  After KVPP filters the registered caches, the address table contains only this rank's owned layers (and replicated MTP layers), indexed densely. The GVA branch of `_groups_for_layer…

### #19442 — [[Test] Add a Qwen3.5-35B-A3B case to the TP4 batch-invariance e2e](https://github.com/vllm-project/vllm-ascend/pull/19442)
- **作者**: EliasKaslan  **时间**: 2026-10-22 12:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  The TP4 batch-invariance e2e only covered `Qwen/Qwen3-30B-A3B`, whose layers are all full attention. `Qwen/Qwen3.5-35B-A3B` is a hybrid MoE -- three gated-delta-net linear-attention layers followed by one full-attention layer, 256 routed experts with top-8 --…

### #19436 — [[Doc] Translated Doc files 2026-10-22](https://github.com/vllm-project/vllm-ascend/pull/19436)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-22 12:34 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **20** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents…

### #19435 — [[CI][PCP] Migrate weekly DSA-CP cases to PCP](https://github.com/vllm-project/vllm-ascend/pull/19435)
- **作者**: li1how  **时间**: 2026-10-22 12:09 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Migrate 29 Weekly configurations from DSA-CP to PCP. Each former DSA-CP TP group becomes TP1 with the same PCP size (4, 8, or 16), preserving DP and device counts. The 19 PD cases use MooncakeConnectorV2, including MultiConnector children, with matching P/D m…

### #19434 — [[BugFix][KVPool][v0.30.0] Fix PP GVA layout and alignment](https://github.com/vllm-project/vllm-ascend/pull/19434)
- **作者**: bowgneo  **时间**: 2026-10-22 12:05 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Fix the Memcache layerwise GVA object layout when pipeline parallelism is enabled. PP keys already identify their stage, but allocation and transfer offsets still extrapolate a global layout from average layer sizes. With uneven cache entries, PP1 adds a 3,77…

### #19432 — [[Doc] Correct context parallel guides and diagram labels](https://github.com/vllm-project/vllm-ascend/pull/19432)
- **作者**: wzx0726  **时间**: 2026-10-22 11:59 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Align the context parallel guides with the current implementation:  - Complete the DSA-CP launch example, document PCP connector requirements and DSpark SFA support, and fix DCP wording and localized anchors while preserving the compatibility matrices. - Corr…

### #19431 — [[Doc][Misc] Unify MiniMax-M3 A5 PD examples on three nodes](https://github.com/vllm-project/vllm-ascend/pull/19431)
- **作者**: yingyingzizi  **时间**: 2026-10-22 11:59 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The 950DT MiniMax-M3 guide currently mixes two-node PD examples with a three-node scale-out option, and pooling still reuses the two-node commands. Present one three-node 1P1D layout for ordinary and pooled PD: one Prefill node at DP1 × TP4 × PP2, with a 30/3…

### #19430 — [Feat/qwen38 mrv2 dflash](https://github.com/vllm-project/vllm-ascend/pull/19430)
- **作者**: sunny-rain-63  **时间**: 2026-10-22 11:59 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #19428 — [[Test][EPLB] Migrate weekly EPLB cases to model runner v2](https://github.com/vllm-project/vllm-ascend/pull/19428)
- **作者**: Spicy-Stick  **时间**: 2026-10-22 11:54 CST
- **标签**: module:tests
- **摘要**: Use the upstream async EPLB CLI configuration and explicitly select Model Runner V2 for the weekly Qwen, Kimi, and GLM cases. Preserve collection window sizes and redundant expert counts, derive step intervals from the legacy collection and algorithm intervals, and keep unrelated benchmark settings …

### #19427 — [Zercmoe new](https://github.com/vllm-project/vllm-ascend/pull/19427)
- **作者**: huoyibingli  **时间**: 2026-10-22 11:46 CST
- **标签**: documentation, ci/build, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…
