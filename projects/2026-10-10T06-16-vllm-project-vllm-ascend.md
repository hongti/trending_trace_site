# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-10 14:16 CST

## AI 总结

# vllm-ascend 仓库动态摘要（2026-10-10）

## 📋 Issue

- **#18178 [Bug] EPLB 初始化失败**
  在 Ascend A3 上使用投机解码（draft model）+ `expandable_segments:True` 时，EPLB 注册内存失败（status 103900）。环境为 vllm 0.30.0 + vllm-ascend 0.19.1rc2。

## 🔧 Pull Requests

### 🐛 Bug 修复（重点关注）

| PR | 说明 |
|---|---|
| #18182 / #18184 | **DSpark 全词表模型 draft token 映射修复**：为 Ascend 上的 DeepSeek V4/V4.1、Kimi K3 DSpark 模型类定义 `draft_id_to_target_id = None`，因其 draft token ID 已与 target 一致，避免错误映射。#18184 为回溯到 `releases/v0.30.0`。 |
| #18183 / #18185 | **DSpark 无 TurboQuant 后端修复**：当 TurboQuant transform 缺失时，将其视为禁用状态，保留普通 CP cache 写入路径，避免 `AttributeError`。#18185 为回溯到 v0.30.0。 |
| #18181 | **量化 MoE LoRA 的 SiTU 激活修复**：W8A8 MoE LoRA 配置 SiTU 时会错误落入 SwiGLU 路径，导致第二次动态量化前数值不正确；现正确 honor SiTU 激活。 |
| #18180 | **pyarrow 版本上限限制**：pyarrow 26.0.0 新增运行时检查要求更高 NumPy 版本，回溯到 v0.30.0 将 pyarrow 限制在 26 以下。 |

### ⚙️ 算子优化

- **#18186**：`scatter_nd_update_sk`（Arch22/A2）的 HP 半分区核心分割逻辑与 ops-nn hp 分支对齐——对小行（rowBytes < 8KB）限制 `hpCoreNum`，提升算子性能与一致性。

### 📚 文档

- **#18187**：自动翻译 **29** 个文档文件（含自定义 aclnn op 设计文档等），覆盖中文本地化。
- **#18179**：修复 GLM5.2 PD 分离式部署脚本中 `$2` 端口参数为空的问题（4× Atlas 800 A3 场景）。
- **#18177**：新增 **Atlas 850/850E/950 SuperPod** PD 分离式部署配置文档，补充原有 A2/A3 网络与容器设置。

## 🚀 Release

> 本次动态中无新版本发布。

---

**总结**：本期活动以 **v0.30.0 版本的稳定性回溯**为主线，集中修复了 DSpark 模型（DeepSeek V4/V4.1、Kimi K3）在投机解码场景下的 token 映射与 TurboQuant 兼容性问题，并解决了量化 MoE LoRA 的 SiTU 激活缺陷。算子层面优化了 `scatter_nd_update_sk` 的 HP 分区逻辑，文档方面扩展了 SuperPod 系列硬件的 PD 部署支持。

---

## 🐛 Issues

### #18178 — [[Bug] EPLB init fails with "HIXL EPLB register memory failed with status 103900" when speculative decoding (draft model) + expandable_segments:True on Ascend A3](https://github.com/vllm-project/vllm-ascend/issues/18178)
- **作者**: baolongsun  **时间**: 2026-10-10 11:54 CST
- **标签**: bug
- **摘要**: ### Your current environment  - Hardware: Ascend A3  - OS / Python: Python 3.11.10 - image: dev-26.2.0.day20261009-800I-A3-py311-openEuler24.03-lts-aarch64 - vllm: 0.30.0 - vllm-ascend: 0.19.1rc2.dev2635+gac1b7bceb.d20261009 - CANN / torch-npu: 9.2.0/2.10.0.post7  ### Environment variables  ### star…

## 🔀 Pull Requests

### #18187 — [[Doc] Translated Doc files 2026-10-10](https://github.com/vllm-project/vllm-ascend/pull/18187)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-10 13:45 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **29** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/add_custom_aclnn_op.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #18186 — [[Ops] scatter_nd_update_sk: align HP small-row core partition with ops-nn](https://github.com/vllm-project/vllm-ascend/pull/18186)
- **作者**: ZT-AIA  **时间**: 2026-10-10 13:06 CST
- **摘要**: ### What  Align the HP (half-partition) core-splitting logic of `scatter_nd_update_sk` (Arch22/A2) with the ops-nn hp branch: for small rows (rowBytes < 8KB, `HP_LARGE_ROW_BYTES`), limit `hpCoreNum_` so that each core gets at least 16 rows (`HP_MIN_ROWS_PER_CORE`), avoiding inter-core scheduling ove…

### #18185 — [[Attention][BugFix] handle DSpark backends without TurboQuant](https://github.com/vllm-project/vllm-ascend/pull/18185)
- **作者**: qjgggsse  **时间**: 2026-10-10 12:42 CST
- **标签**: ready-precise
- **摘要**: cherry-pick of PR https://github.com/vllm-project/vllm-ascend/pull/18183/  ### What this PR does / why we need it?  Treat a missing TurboQuant transform as disabled when storing DSpark context KV. This preserves the ordinary CP cache write path and avoids AttributeError without changing existing Tur…

### #18184 — [[BugFix][v0.30.0] Define draft token mapping for full-vocabulary DSpark models](https://github.com/vllm-project/vllm-ascend/pull/18184)
- **作者**: drslark  **时间**: 2026-10-10 12:41 CST
- **摘要**: Backport of https://github.com/vllm-project/vllm-ascend/pull/18182 to `releases/v0.30.0`.  ### What this PR does / why we need it?  Define `draft_id_to_target_id = None` on the Ascend DeepSeek V4, DeepSeek V4.1, and Kimi K3 DSpark model classes. Their draft token IDs already match target token IDs, …

### #18183 — [[BugFix]: handle DSpark backends without TurboQuant](https://github.com/vllm-project/vllm-ascend/pull/18183)
- **作者**: qjgggsse  **时间**: 2026-10-10 12:40 CST
- **标签**: ready-precise
- **摘要**: ### What this PR does / why we need it?  Treat a missing TurboQuant transform as disabled when storing DSpark context KV. This preserves the ordinary CP cache write path and avoids AttributeError without changing existing TurboQuant behavior.  ### Does this PR introduce _any_ user-facing change?  No…

### #18182 — [[BugFix] Define draft token mapping for full-vocabulary DSpark models](https://github.com/vllm-project/vllm-ascend/pull/18182)
- **作者**: drslark  **时间**: 2026-10-10 12:39 CST
- **摘要**: ### What this PR does / why we need it?  Define `draft_id_to_target_id = None` on the Ascend DeepSeek V4, DeepSeek V4.1, and Kimi K3 DSpark model classes. Their draft token IDs already match target token IDs, so they need no remapping. This exposes the attribute read by the DSpark speculator.  The l…

### #18181 — [[BugFix][LoRA] Honor SiTU activation in quantized MoE LoRA](https://github.com/vllm-project/vllm-ascend/pull/18181)
- **作者**: ArtlexYoung  **时间**: 2026-10-10 12:29 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  W8A8 MoE LoRA batches configured with SiTU fall through to SwiGLU in `_apply_moe_activation`, producing incorrect values before the second dynamic quantization. For example, gate=8 and up=-100 produce approximately -99.97 with default SiTU, versus -799.73 wit…

### #18180 — [[Cherry-pick][releases/v0.30.0][BugFix]: Cap pyarrow below 26 in the dev environment (from #18161)](https://github.com/vllm-project/vllm-ascend/pull/18180)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-10 12:15 CST
- **标签**: ready-precise
- **摘要**: Cherry-pick of PR #18161 onto `releases/v0.30.0`.  Original PR: #18161 Original author: @zhangxinyuehfad  ---  ### What this PR does / why we need it?  pyarrow 26.0.0 added a runtime check that requires NumPy >= 2, but the CI environment resolves numpy 1.26.4. requirements-dev.txt pulls sentence_tra…

### #18179 — [[Doc] Fix empty $2 port parameter in GLM5.2 PD disaggregation startup scripts](https://github.com/vllm-project/vllm-ascend/pull/18179)
- **作者**: wangbo23  **时间**: 2026-10-10 12:02 CST
- **标签**: documentation
- **摘要**: ## Description  In section 5.1.3.1 (4× Atlas 800 A3 PD disaggregation) of `docs/source/tutorials/models/GLM5.2.md`, the prefill node templates reference `--port $2`, but the prefill nodes are started directly with `bash run_dp_template.sh` **without any positional parameters** — only the decode node…

### #18177 — [[Doc][v0.30.0] Document Atlas 850/850E/950 SuperPod PD configuration](https://github.com/vllm-project/vllm-ascend/pull/18177)
- **作者**: Yuli-yx  **时间**: 2026-10-10 11:52 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The multi-node Mooncake PD guide currently covers A2/A3 networking and container setup. Add the configuration needed for Atlas 850/850E/950 SuperPod (Ascend 950PR/950DT) deployments:  - Default super plane (UB port and UBC protocol) with automatic `ASCEND_LOC…
