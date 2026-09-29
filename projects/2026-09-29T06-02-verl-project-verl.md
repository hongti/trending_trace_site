# verl-project/verl — 动态追踪

> 生成时间: 2026-09-29 14:02 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 1. Issue 动态
本期未提供相关的 Issue 记录。

### 2. Release 动态
本期未发布正式的 Release 版本。但从 PR 动态来看，团队正在积极向 `release/v0.9.1` 商发分支进行代码同步、修复与文档维护，表明 **v0.9.1 版本正处于紧张的发布准备阶段**。

### 3. Pull Request (PR) 动态
近期的 PR 主要围绕架构重构、硬件适配（特别是昇腾 NPU）、核心 Bug 修复及文档更新展开。重要变更如下：

**🚨 重大架构变更与重构**
*   **#8051 [BREAKING]** **移除 Megatron 遗留 mbridge 支持**：全面转向使用 NVIDIA Megatron-Bridge 进行模型构建和 HF 权重转换。删除了旧的选择器、适配器、检查点选项及相关依赖，属于破坏性更新。
*   **#8043** **清理冗余代码**：移除了未使用的 Megatron 模型注册表（registries）。
*   **#8052** **修复 Pipeline Layout 继承问题**：解决了开启 MTP 的 actor 模型导致参考模型错误继承自定义 pipeline 布局的问题，移除了多余的 MTP 层。

**⚙️ 核心功能与逻辑修复**
*   **#8046** **vLLM 睡眠机制修复**：修复了合并后的 LoRA 权重在 vLLM 处于混合睡眠（Level 2）并被唤醒时丢失的问题，确保跨睡眠状态的权重保持。
*   **#8044** **奖励函数崩溃修复**：修复了 `prime_code` 在非连续检查（`continuous=False`）未通过测试时导致程序崩溃的 Bug，现会正常返回分数。

**🧬 硬件适配（昇腾 NPU / Ascend）**
*   **#8048 & #8049** **Qwen3-Next NPU 适配**：为 Qwen3-Next 在 NPU 上的状态分配问题添加了可选的变通方案（workaround），并同步到了 `release/v0.9.1` 分支及 FSDP 示例配置中。

**📖 文档与安装指南**
*   **#8050 & #8047** **昇腾安装文档修复**：更新了 Ascend 平台的安装指南，修复了 v0.9.1 分支克隆命令未指定分支的问题。
*   **#8045** **修复死链**：集中修复了 README、相关文档及 Ascend Docker 表格中的 6 个失效链接。

---

## 🔀 Pull Requests

### #8052 — [[megatron] fix: remove inherited MTP layers from reference pipeline parallelism layouts](https://github.com/verl-project/verl/pull/8052)
- **作者**: lxb007981  **时间**: 2026-09-29 13:54 CST
- **摘要**: ### What does this PR do?  With an MTP-enabled actor, the reference model inherits its custom pipeline layout, for example:  ``` +actor_rollout_ref.actor.megatron.override_transformer_config.pipeline_model_parallel_layout="Et*10|t*12|t*8|t*8|t*12|t*12|t*12|t*4mL" ```  The reference model only comput…

### #8051 — [[BREAKING][megatron] refactor: remove legacy mbridge support](https://github.com/verl-project/verl/pull/8051)
- **作者**: ji-huazhong  **时间**: 2026-09-29 13:35 CST
- **摘要**: ### What does this PR do?  Use NVIDIA Megatron-Bridge for model construction and HF weight conversion. Remove the vanilla_mbridge selector, legacy adapter, checkpoint options, and dependency entries; retain the use_mbridge engine setting.    ### Checklist Before Starting  - [ ] Search for similar PR…

### #8050 — [Docs update ascend install](https://github.com/verl-project/verl/pull/8050)
- **作者**: jackon-creator  **时间**: 2026-09-29 12:01 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8049 — [[model, hardware] fix: add opt-in Qwen3-Next NPU state allocation workaround](https://github.com/verl-project/verl/pull/8049)
- **作者**: lxb007981  **时间**: 2026-09-29 11:44 CST
- **摘要**: ### What does this PR do?  Port #8048 and #8033 to release/v0.9.1.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `…

### #8048 — [[recipe] fix: enable Qwen3-Next NPU workaround in FSDP example](https://github.com/verl-project/verl/pull/8048)
- **作者**: lxb007981  **时间**: 2026-09-29 11:36 CST
- **摘要**: ### What does this PR do?  #8033 added the opt-in sync-before-state workaround. Add the flag in the Qwen3-Next Ascend FSDP example to activate it.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {…

### #8047 — [verl v0.9.1商发分支中verl仓代码clone未指定分支 资料修改](https://github.com/verl-project/verl/pull/8047)
- **作者**: jackon-creator  **时间**: 2026-09-29 11:31 CST
- **摘要**: docs: pin clone command to release/v0.9.1 in Ascend install guides

### #8046 — [[vllm, rollout] fix: keep merged LoRA weights across vLLM sleep](https://github.com/verl-project/verl/pull/8046)
- **作者**: Shr3yash  **时间**: 2026-09-29 08:01 CST
- **摘要**: ### What does this PR do?  Fixes #7904 for merged LoRA (`actor_rollout_ref.model.lora.merge=true`) with `free_cache_engine=true`. Hybrid sleep was level 2, which discards vLLM parameter storage. Wake puts the buffers back empty. The merged sync then sends a PEFT export through `load_weights`, not a …

### #8045 — [[doc] fix: repair six dead links in README, docs and the ascend docker table](https://github.com/verl-project/verl/pull/8045)
- **作者**: pratikgx  **时间**: 2026-09-29 03:26 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Fixes six dead links, bundled per the contributing guidance against one-off PRs:  - README: the multi-turn RLHF release post moved under `release_log/` - `docker/ascend/supported_tags.md`: the CANN 9.1.0 row linked a 9.0.1 Dockerfile that no longer exists; now `Dockerfile.…

### #8044 — [[reward] fix: return a score from prime_code when a non-continuous check fails](https://github.com/verl-project/verl/pull/8044)
- **作者**: MohammadHijjawi97  **时间**: 2026-09-29 00:43 CST
- **摘要**: ### What does this PR do?  `prime_code.compute_score(completion, test_cases, continuous=False)` crashes for any program that does not pass every test. `metadata_list` is only assigned on the `continuous` path and `success` only inside the full-check `try`, so with the default `continuous=False` a wr…

### #8043 — [[megatron, model] refactor: remove unused Megatron model registries](https://github.com/verl-project/verl/pull/8043)
- **作者**: ji-huazhong  **时间**: 2026-09-28 23:13 CST
- **摘要**: ### What does this PR do?  as per title.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sgla…
