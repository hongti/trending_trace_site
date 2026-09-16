# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-16 13:09 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📌 Issue 摘要
近期共提出 3 个 Issue，主要集中在 CI 失败修复与生产环境长时间运行的稳定性问题：
*   **CI 类型错误 (#16658)**：`xlite` 模块存在类型错误，导致仓库级别的 pre-commit mypy 检查频繁失败，甚至影响未修改该模块的 PR。
*   **长时间运行崩溃问题 (#16654, #16652)**：在 Atlas 800 A3（2节点 P/D 分离架构）和 A2（双机混部架构）上部署 GLM-5.2（W8A8C8 量化）时，运行 3-8 小时后分别出现 `fftsplus aicore error` 和 `hccl fftsplus task timeout`，最终导致 EngineCore 异常退出。

### 🚀 PR 摘要
近期共合并/提交 9 个 PR，涵盖新特性、Bug 修复、代码回退与文档更新：
*   **重要新特性**：
    *   **MoE 算子融合 (#16655)**：将 A3 `GroupedMatmulSituQuant` 量化算子整合至 `releases/v0.27.1rc` 分支，提升 MoE 计算效率。
    *   **新量化 Kernel (#16651)**：为 A5 架构添加 C8 `MixedQuantSparseFlashMla` 算子及 Metadata 算子，并扩展了原有的 NoPE/RoPE 接口配置（如支持 `448 NoPE + 0 RoPE`）。
*   **关键 Bug 修复**：
    *   **配置清理 (#16661)**：修复上游 `EngineArgs` 注入的 `--gdn-prefill-backend` / `--kda-prefill-backend` 键的问题，由于 Ascend 平台没有对应的 kernel 消费者，此处予以剥离清理。
    *   **A5 SFA DCP 修复 (#16656)**：修复 A5 DCP decode 中 padded-index LSE 和空 shards 的问题，正确处理稀疏索引张量中的 `-1` 填充。
*   **代码回退与 CI**：
    *   **回退性能优化 (#16659)**：撤销了 PR #16343 中在 MoE prepare 阶段用持久化零块 cat 替代 `F.pad` 的性能优化代码。
    *   **回退 CI 改动 (#16653)**：撤销了 PR #16211 的相关代码变动。
*   **文档与配置**：
    *   统一更新了各模型教程中的“多节点通信验证”步骤及默认端口 (#16657, #16650)。
    *   自动翻译了 18 个文档文件 (#16660)。
    *   新增了配置文件 yaml (#16648)。

### 📦 Release 摘要
*   本次提供的数据中**未包含正式的 Release 发布记录**。
*   但从 PR 动态来看，**v0.27.1 版本正在积极筹备中**（代码已向 `releases/v0.27.1rc` 分支迁移），预计该版本将重点引入 A3 MoE 量化算子融合等新特性。

---

## 🐛 Issues

### #16658 — [[CI][xlite] Fix repository-wide mypy failure for index_full_mask](https://github.com/vllm-project/vllm-ascend/issues/16658)
- **作者**: Yuli-yx  **时间**: 2026-09-16 11:40 CST
- **摘要**: ### Description  The repository-wide pre-commit job currently fails on an existing xlite type error, including for PRs that do not change `vllm_ascend/xlite/`:  ```text vllm_ascend/xlite/xlite.py:664: error: "XModelConfig" has no attribute "index_full_mask"  [attr-defined] Found 1 error in 1 file (c…

### #16654 — [[Bug]: GLM-5.2 A3（2 节点 P/D 分离部署），长时间（3-8小时）运行后Decode 实例出现 fftsplus aicore error ， EngineCore异常退出](https://github.com/vllm-project/vllm-ascend/issues/16654)
- **作者**: tangyun0526  **时间**: 2026-09-16 11:14 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  vllm-ascend：0.23.0镜像 CANN：9.1.0 量化：W8A8C8 硬件：Atlas 800 A3 （2 节点 P/D 分离架构，1 个 Prefill 实例（TP=16，kv_producer，enforce-eager，MTP=1）+ 1 个 Decode 实例（TP=16 / DCP=16 / EP=16，kv_consumer，FULL_DECODE_ONLY cudagraph，MTP=3））  ### 🐛 Describe the bug   GLM-5.2 A3（2 节点 P/D 分离部署），长时间（3-…

### #16652 — [[Bug]: GLM-5.2 A2 双机（每节点16卡910B）混部部署，长时间运行后出现hccl fftsplus task timeout ， EngineCore异常退出](https://github.com/vllm-project/vllm-ascend/issues/16652)
- **作者**: tangyun0526  **时间**: 2026-09-16 10:35 CST
- **标签**: bug, glm5, llm-model
- **摘要**: ### Your current environment  vllm-ascend：0.23.0镜像 CANN：9.1.0 量化：W8A8C8 硬件：Atlas 800 A2 （单机16卡910 B2C，双机混部）  ### 🐛 Describe the bug   GLM-5.2 A2 双机（每节点16卡910B）混部部署，长时间运行后出现hccl fftsplus task timeout ， EngineCore异常退出 总共3个实例（每个实例双机混部），出现4次crash，均为hccl fftsplus task timeout。 The error from device(chipI…

## 🔀 Pull Requests

### #16661 — [[BugFix][Config] Strip upstream-injected GDN/KDA prefill backend keys on Ascend](https://github.com/vllm-project/vllm-ascend/pull/16661)
- **作者**: lizy124  **时间**: 2026-09-16 12:48 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  - Upstream `EngineArgs` injects `--gdn-prefill-backend` / `--kda-prefill-backend` into `VllmConfig.additional_config`, but Ascend has no kernel consumer for either option. - The strict `extra="forbid"` `AscendConfig` schema rejects these injected keys as unkn…

### #16660 — [[Doc] Translated Doc files 2026-09-16](https://github.com/vllm-project/vllm-ascend/pull/16660)
- **作者**: vllm-ascend-ci  **时间**: 2026-09-16 12:46 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **18** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/index.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/De…

### #16659 — [Revert 16343 "[Performance][Communicator] Replace per-layer F.pad with cat …](https://github.com/vllm-project/vllm-ascend/pull/16659)
- **作者**: ajariley  **时间**: 2026-09-16 11:52 CST
- **标签**: module:tests, module:ops
- **摘要**: …of a persistent zero block in MoE prepare (#16343)"  This reverts commit 616f872747bdc89de9b1c410b4b30888cd834705.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit…

### #16657 — [[Doc][Misc] Standardize multi-node communication verification and update default ports in model tutorials](https://github.com/vllm-project/vllm-ascend/pull/16657)
- **作者**: sunshine202600  **时间**: 2026-09-16 11:27 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The documents related to the model tutorial updates are as follows: 1. Updated the "Verify Multi-node Communication" section for all models. 2. Added section numbers and updated the title of the performance evaluation section. 3. Updated the port numbers in t…

### #16656 — [[BugFix][SFA] Fix padded-index LSE and empty shards for A5 DCP](https://github.com/vllm-project/vllm-ascend/pull/16656)
- **作者**: recky-c  **时间**: 2026-09-16 11:25 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  A5 SFA DCP decode can select fewer keys on a rank than its local KV length. The sparse index tensor then contains a valid prefix followed by `-1` padding. Using the KV length as the softmax extent includes invalid entries and produces incorrect per-rank norma…

### #16655 — [[v0.27.1][Feature][MoE] Integrate A3 GroupedMatmulSituQuant fusion](https://github.com/vllm-project/vllm-ascend/pull/16655)
- **作者**: MaybeChz  **时间**: 2026-09-16 11:21 CST
- **标签**: module:tests, module:ops, module:core
- **摘要**: ### What this PR does / why we need it?  Migrates the final state of #15432 to `releases/v0.27.1rc` as two fresh signed-off commits:  - Adds the Ascend A3 `GroupedMatmulSituQuant` kernel and direct `torch.vllm_ascendC` binding. - Integrates the fused GMM1 + SiTU + per-token INT8 quant path for suppo…

### #16653 — [[CI]Revert16211](https://github.com/vllm-project/vllm-ascend/pull/16653)
- **作者**: U1stRsouland  **时间**: 2026-09-16 11:11 CST
- **标签**: module:tests
- **摘要**: …gits (#16211)"  This reverts commit 0526083dd4aae02a02afeb0de680c119d8e5e732.  ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8…

### #16651 — [[Feature][Kernel] Add A5 C8 MixedQuantSparseFlashMla with RoPE0](https://github.com/vllm-project/vllm-ascend/pull/16651)
- **作者**: Foriv  **时间**: 2026-09-16 10:35 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  Adds A5 C8 MixedQuantSparseFlashMla and its Metadata operator to vLLM-Ascend, then extends the original fixed `448 NoPE + 64 RoPE` interface with a `448 NoPE + 0 RoPE` path.  - Ports the Host, ACLNN, AICore and Metadata AICPU implementations from [cann/ops-tr…

### #16650 — [[Doc][Misc] Standardize multi-node communication verification and update default ports in model tutorials](https://github.com/vllm-project/vllm-ascend/pull/16650)
- **作者**: sunshine202600  **时间**: 2026-09-16 10:23 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The documents related to the model tutorial updates are as follows: 1. Updated the "Verify Multi-node Communication" section for all models. 2. Added section numbers and updated the title of the performance evaluation section. 3. Updated the port numbers in t…

### #16648 — [add new configyaml](https://github.com/vllm-project/vllm-ascend/pull/16648)
- **作者**: 18184157617  **时间**: 2026-09-16 10:20 CST
- **标签**: module:tests, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/84030bbe3d74d99bad477a3d2e37a973ccd8865c
