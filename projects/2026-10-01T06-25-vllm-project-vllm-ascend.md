# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-01 14:25 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 🐛 Issue 动态
近期主要收到 2 个 Bug 反馈，均涉及特定硬件与复杂推理场景：
1. **双机 RoCE 组网报错** (#17857)：在 Atlas 800I A3 双机环境下使用 RoCE 组网运行 `DeepSeek-V4.1-Flash-w8a8` 模型时，报 `acl api failed` 错误，影响多机部署。
2. **310P MTP 批次输出中断** (#17856)：在 310P 设备上结合 MTP 投机解码与结构化输出时，如果 `accepted_prev` 大于当前查询长度，`recurrent_gated_delta_rule_v310` 内核会异常退出，导致整个 batch 的输出丢失。

---

### 🔧 PR 动态
近期合并的 PR 主要聚焦于 **性能优化、关键 Bug 修复、CI 流程清理及文档完善**：

**1. 性能优化**
* **Attention 性能提升** (#17863)：针对 A2/A3 NoPE SFA 减少了空闲核心和小型查询的 scratch 内存分配，优化资源利用率。

**2. 关键 Bug 修复**
* **MRv2 算子规格补全** (#17858)：为 kpool-indexer 模型在 MRv2 中创建 sparse SFA C8 规格，修复了之前仅在 MRv1 中生效的问题（跟进 GLM-5.3-Flash 的适配）。
* **GDN 状态保留修复** (#17854)：修复了 MTP 投机解码加结构化输出场景下，因语法验证修剪不同数量 token 导致 recurrent state 行丢失的问题。
* **构建修复** (#17853)：修复了在无 NPU 环境下（用于 UT）进行无核心构建时，因缺少默认 `SOC_VERSION` 而导致的构建失败问题。

**3. CI 与测试维护**
* 移除了 nightly 测试中不存在的 `test_custom_op` 引用 (#17860)，以及针对 `test_fused_moe.py` 的过期 `--ignore` 配置 (#17859)。
* 移动并整理了相关 CI 脚本 (#17862)。

**4. 文档更新**
* 自动翻译了 23 个文档文件 (#17865)。
* 完善了 `testing.md` 中关于容器测试环境的前置条件说明 (#17864)。
* 修复了社区治理文档中行为准则和快速入门的失效链接 (#17861)。

---

### 🚀 Release 动态
* 近期无新版本发布。

---

## 🐛 Issues

### #17857 — [[Bug]:  使用RoCE组网在Atlas 800I A3双机设备跑DeepSeek-V4.1-Flash-w8a8报错：SUSPECT REMOTE ERROR...ERR00100 PTA call acl api failed](https://github.com/vllm-project/vllm-ascend/issues/17857)
- **作者**: yaoyanxiang  **时间**: 2026-09-30 22:16 CST
- **标签**: bug, llm-model, deepseek
- **摘要**: ### Your current environment  设备：Atlas 800I A3 驱动版本：26.0.rc1 模型：DeepSeek-V4.1-Flash-w8a8 组网：A3双机roce组网   ### 🐛 Describe the bug  [Usage]: 使用RoCE组网在Atlas 800I A3双机设备跑DeepSeek-V4.1-Flash-w8a8报错：SUSPECT REMOTE ERROR...ERR00100 PTA call acl api failed

### #17856 — [[Bug][310P] recurrent_gated_delta_rule_v310 aborts the whole batch output when accepted_prev > current query length (MTP + structured output)](https://github.com/vllm-project/vllm-ascend/issues/17856)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-30 22:09 CST
- **摘要**: ### Bug description  The 310P recurrent GDN kernel (`recurrent_gated_delta_rule_v310`) bails out of the **entire kernel** — leaving every request's output unwritten — whenever one request in the batch has `accepted_tokens_prev > current_query_length`.  This is the 310P counterpart of #17837 (fixed f…

## 🔀 Pull Requests

### #17865 — [[Doc] Translated Doc files 2026-10-01](https://github.com/vllm-project/vllm-ascend/pull/17865)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-01 13:45 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **23** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/add_custom_aclnn_op.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #17864 — [[Doc][Misc] Clarify container test environment prerequisites in testing.md](https://github.com/vllm-project/vllm-ascend/pull/17864)
- **作者**: li-lizhe  **时间**: 2026-10-01 13:43 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Fixes the first gap reported in #17718: `docs/source/developer_guide/contribution/testing.md` explains how to start the container and run `pytest`, but not the environment prerequisites, so a new contributor has to rediscover them by trial and error.  This PR…

### #17863 — [[Performance][Attention] Reduce idle SFA cores and small-query scratch allocation](https://github.com/vllm-project/vllm-ascend/pull/17863)
- **作者**: LiPu-jpg  **时间**: 2026-10-01 11:39 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  A2/A3 NoPE SFA assigns one query group per task, but launches and reserves scratch for every hardware core. A one-row TND query on the tested A2 launches 20 AIC / 40 AIV despite having one query group. Bound the launch by query tensor rows and allocate scratc…

### #17862 — [[CI] mv scripts](https://github.com/vllm-project/vllm-ascend/pull/17862)
- **作者**: chen-commits  **时间**: 2026-10-01 10:59 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17861 — [[Doc] Fix code of conduct and quick start links](https://github.com/vllm-project/vllm-ascend/pull/17861)
- **作者**: pratikgx  **时间**: 2026-10-01 07:35 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Fixes two broken links.  - `docs/source/community/governance.md`: vLLM `blob/main/CODE_OF_CONDUCT.md` -> `blob/main/.github/CODE_OF_CONDUCT.md` (the file now lives under `.github/` in vllm-project/vllm) - `benchmarks/README.md`: `../docs/source/quick_start.md…

### #17860 — [[CI] Remove references to non-existent nightly test cases](https://github.com/vllm-project/vllm-ascend/pull/17860)
- **作者**: awesome-pro  **时间**: 2026-10-01 04:32 CST
- **标签**: documentation, ci/build
- **摘要**: ### What this PR does / why we need it?  Removes references to nightly test case names that do not exist in any config.  `test_custom_op` has never been an entry in `nightly_config.yaml`. The only similar name, `test_custom_op_multi_card`, was dropped in #12458 after #12119 deleted the last file und…

### #17859 — [[CI] Remove stale --ignore for test_fused_moe.py in nightly single-node workflows](https://github.com/vllm-project/vllm-ascend/pull/17859)
- **作者**: awesome-pro  **时间**: 2026-10-01 01:06 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?  Removes the stale `--ignore=tests/e2e/nightly/single_node/ops/singlecard_ops/test_fused_moe.py` from both nightly single-node workflows (generic + 560T).  The ignore was added by #5309 (2025-12-24) to skip a then-failing fused-MoE test. #16831 fixed and re-en…

### #17858 — [[Bugfix] Create sparse SFA C8 specs in MRv2 for kpool-indexer models](https://github.com/vllm-project/vllm-ascend/pull/17858)
- **作者**: macdoor  **时间**: 2026-10-01 00:33 CST
- **摘要**: ### What this PR does / why we need it?  Follow-up to #17115 (sparse SFA C8 enablement for GLM-5.3-Flash). That PR created C8 KV specs only in MRv1 (`model_runner_v1.py`); in the v2 model runner (`VLLM_USE_V2_MODEL_RUNNER=1`) the spec/allocation gates in `worker/v2/attn_utils.py` are still written a…

### #17854 — [[BugFix][GDN] Preserve recurrent state rows for ragged MTP batches](https://github.com/vllm-project/vllm-ascend/pull/17854)
- **作者**: linnea-lin-00638949  **时间**: 2026-09-30 21:30 CST
- **标签**: module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Fixes #17837 (root cause on main; release-branch fix: #17839).  With MTP speculative decoding plus structured output, grammar validation trims a different number of draft tokens per request, so the GDN batch becomes ragged. The release fix (#17839) corrected …

### #17853 — [[BugFix][Build] Fall back to a default SOC_VERSION for kernel-free builds](https://github.com/vllm-project/vllm-ascend/pull/17853)
- **作者**: lru49  **时间**: 2026-09-30 20:47 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  On `main`, `vllm_ascend/envs.py` documents `COMPILE_CUSTOM_KERNELS=0` as the setting to use when building on a machine without an NPU (for UT), but https://github.com/vllm-project/vllm-ascend/issues/17785 documents that the `SOC_VERSION` guard in `setup.py` r…
