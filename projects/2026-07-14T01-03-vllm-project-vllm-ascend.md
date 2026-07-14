# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-07-14 09:03 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🚀 Release (版本发布)
近期无新版本发布。

---

### 🚨 Issue (问题动态)
1. **【高危安全漏洞】脚本注入风险 (#11964)**：报告了 `/cherry-pick` 工作流存在严重安全隐患。攻击者可通过构造恶意的 PR 标题/正文，将其直接注入到使用 `PAT_TOKEN` 的 `run: shell` 块中，导致脚本注入。
2. **【CI 异常】自动化流程中断 (#11966)**：`main2main` 自动同步工作流未完成所有计划步骤即停止，目前需要人工介入审查对应的 Draft PR (#11965)。

---

### 🔧 Pull Request (代码变更)

**✨ 新特性与模型适配**
- **#11963 [Feature] 适配 Gemma4 模型**：在 Ascend 上适配 Gemma4-E2B 和 Gemma4-E4B。针对 TND FIA 仅支持 64/128/192 head dim 的限制，将 `global_head_dim=512` 的全注意力层通过 BSH FIA 回退机制进行路由，并修复了 K 相关问题。
- **#11965 [Misc] 适配 vLLM 主分支**：跟进 vLLM 上游最新代码（24/40 步），修补了上游删除 `deepseekv4_tool_parser` 引起的兼容性问题。

**🛠️ 关键修复与回退**
- **#11961 [BugFix] 回退 A3 SFA C8**：回退了导致 H2D 同步错误和 A5 MLAPO 故障的提交 (52feb2be)。
- **#11960 [Ops] 回退 torch_reserved_memory 获取**：回退了在 model forward 中获取 `torch_reserved_memory` 的操作，因为该操作会增加 0.5ms 的 TPOT 延迟开销。
- **#11955 [BugFix] 优化 FIA graph replay**：修复了 FIA 图重放时的配置查询开销问题，改为在捕获注意力参数时直接存储 `sliding_window` 值，避免重放时读取 `hf_text_config`。
- **#11959 / #11958 [BugFix] 修复 310P 内存计算**：修复了 310P RC 环境下的内存计算错误（分别针对 main 分支和 v0.23.0 版本）。

**♻️ 重构与优化**
- **#11956 [Refactor] 移除多流重叠门控**：移除了 multistream overlap gate 机制。

**📝 文档与 CI**
- **#11962 [Docs] 明确 CPU-only 构建验证**：补充文档，阐明仅 CPU 环境下的构建验证流程。
- **#11957 [CI] 自动更新测试时间预估**：由 CI 工作流自动生成，更新 `test_config.yaml` 中的测试预计耗时。

---

## 🐛 Issues

### #11966 — [main2main manual review required (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/issues/11966)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-14 07:51 CST
- **摘要**: ## Summary  main2main automation stopped before completing all planned steps.  ## Context  - Draft PR: https://github.com/vllm-project/vllm-ascend/pull/11965 - Commit range: `1f486d96a17303ce8db8e02be39545b2be338446`...`a5d19cbb95872c4b426c06735733568542fa33db` - Status: `failed`  ## Final Summary  …

### #11964 — [Security: /cherry-pick workflow interpolates attacker-controlled PR title/body into a run: shell with PAT_TOKEN (script injection)](https://github.com/vllm-project/vllm-ascend/issues/11964)
- **作者**: kobihikri  **时间**: 2026-07-14 03:08 CST
- **摘要**: ### Summary  `.github/workflows/pr_cherry_pick_command.yml` interpolates attacker-controlled `client_payload` fields **directly into `run:` shell blocks**, in a job that exports a `PAT_TOKEN`. Because the workflow is reachable by **any** GitHub user via the `/cherry-pick` slash command, this looks l…

## 🔀 Pull Requests

### #11965 — [[Misc]feat: adapt to vLLM main (a5d19cbb)](https://github.com/vllm-project/vllm-ascend/pull/11965)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-14 07:50 CST
- **摘要**: ### What this PR does / why we need it?  vllm upstream `1f486d96...a5d19cbb` (24/40 steps).  #### vllm_ascend/patch/platform/__init__.py  - Cause: upstream deleted `vllm/tool_parsers/deepseekv4_tool_parser.py` (DeepSeekV4ToolParser), replaced with ParserEngine-based DeepSeekV4EngineToolParser - Chan…

### #11963 — [[Feature] Adapt Gemma4-E2B and Gemma4-E4B on Ascend](https://github.com/vllm-project/vllm-ascend/pull/11963)
- **作者**: sunyuecheng01  **时间**: 2026-07-14 01:19 CST
- **标签**: module:tests, module:ops
- **摘要**: ## Summary - Adapt Gemma4-E2B and Gemma4-E4B on Ascend by routing full-attention   layers with `global_head_dim=512` through a BSH FIA fallback, since TND   FIA only supports head dims 64/128/192. - Fix KV-sharing consumer layers: skip cache writes, and during PrefillNoCache   read the producer's pa…

### #11962 — [docs: clarify CPU-only build verification](https://github.com/vllm-project/vllm-ascend/pull/11962)
- **作者**: ChenDAO1  **时间**: 2026-07-14 00:21 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11961 — [[BugFix] Revert A3 SFA C8](https://github.com/vllm-project/vllm-ascend/pull/11961)
- **作者**: lijiahang226  **时间**: 2026-07-13 23:01 CST
- **标签**: module:tests, ready
- **摘要**: This reverts commit 52feb2be59f6f7af2b36fc674e16c25f98f63438.  ### What this PR does / why we need it?  After introducing this commit, several problems (including H2D synchronization error and A5 MLAPO being disabled unintentionally) have been encountered in the A5 PD disaggregation scenario. Thus t…

### #11960 — [[Ops][BugFix]revert "get torch_reserved_memory" in model forward](https://github.com/vllm-project/vllm-ascend/pull/11960)
- **作者**: Biuapha  **时间**: 2026-07-13 22:40 CST
- **标签**: ready
- **摘要**: … TPOT  ### What this PR does / why we need it? revert "get torch_reserved_memory" in model forward, it cost 0.5ms in TPOT  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab…

### #11959 — [[BugFix][310P] Fix memory calculation for 310P RC](https://github.com/vllm-project/vllm-ascend/pull/11959)
- **作者**: YangShuai52  **时间**: 2026-07-13 22:31 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11958 — [[BugFix][310P][v0.23.0] Fix memory calculation for 310P RC](https://github.com/vllm-project/vllm-ascend/pull/11958)
- **作者**: YangShuai52  **时间**: 2026-07-13 22:30 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #11957 — [[CI] Auto-update estimated test times in test_config.yaml](https://github.com/vllm-project/vllm-ascend/pull/11957)
- **作者**: vllm-ascend-ci  **时间**: 2026-07-13 22:20 CST
- **摘要**: ## Summary  This PR was auto-generated by the **Update estimated test times** [workflow](https://github.com/vllm-project/vllm-ascend/actions/runs/29244280485).  It updates the `estimated_times` values in `.github/workflows/scripts/test_config.yaml` based on actual elapsed times collected from CI wor…

### #11956 — [[Refactor] Remove multistream overlap gate](https://github.com/vllm-project/vllm-ascend/pull/11956)
- **作者**: ningjingbengxiaohai  **时间**: 2026-07-13 22:07 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM version: v0.23.0 - vLLM main: https://github.com/vllm-project/vllm/commit/1f486d96a17303ce8db8e02be39545b2be338446

### #11955 — [[bugfix] Avoid config lookup in FIA graph replay](https://github.com/vllm-project/vllm-ascend/pull/11955)
- **作者**: CXY-Katrina  **时间**: 2026-07-13 21:50 CST
- **标签**: ready
- **摘要**: ### What - Store sliding_window in full_graph_fia captured attention params. - Use the captured value during graph replay instead of reading hf_text_config with hasattr.  ### Why Avoids config lookup overhead in the FIA full-graph replay path while preserving the SWA block table handling.  ### Tests…
