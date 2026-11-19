# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-11-19 15:48 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🐛 Issue 摘要
本期共收到 2 个 Bug 报告，均涉及底层内存与缓存管理：
* **#22452 AscendStore 同步 Mooncake 前缀命中导致错误答案**：在 A2 架构（DeepSeek-V4.1-Flash，分离式 Prefill/Decode）下，AscendStore 同步 Mooncake 前缀命中时会恢复已清零的 KV 页面，导致模型输出错误结果。
* **#22448 NPUWorker 计算大小时丢弃非 KV 分配**：在非 HiSparse 路径下，`NPUWorker.determine_available_memory()` 覆盖了 vLLM Core 测得的 `non_kv_cache_memory`，导致持久化的非 KV 内存分配被意外丢弃。

### 🔧 Pull Request 摘要
本期共有 11 个 PR，主要聚焦于性能优化、重构、文档更新及 CI 维护：

**重要变更与优化：**
* **#22450 [性能] 跳过 Kimi RecoverSSM F gate 复制**：优化了 Kimi 的融合 B/F/G 投影，避免了每层 KDA 中冗余的 `AsStrided` 拷贝操作，提升执行效率。
* **#22447 [重构] 复用继承的权重传输助手**：清理了 `NPUWorker` 中冗余的覆盖方法（如 `_check_weight_transfer_engine` 等），改为直接复用继承的权重传输工具，简化代码结构。

**测试与 CI 维护：**
* **#22446 [测试] 增加 BGE-Reranker-V2-M3 离线打分覆盖**：为 `BAAI/bge-reranker-v2-m3` 新增了端到端测试，覆盖单对、长输入等场景。
* **#22443 [CI] 新增 Codeowner**：添加 @czydyy 作为 CI 构建、基础设施相关文件的代码所有者。
* **#22441 [验证] Nightly 测试**：针对 vLLM main 分支创建的例行自动化验证 PR。

**文档更新：**
* **#22449 & #22444 更新支持矩阵**：刷新了 PCP、DCP 和 KVPP 的支持矩阵，明确了 GLM-5.3-Flash、Kimi-K3.1-Pro 的 PCP/DCP 支持状态及混合缓存布局。
* **#22451 修复失效文档链接**：修复了 `benchmarks/README.md` 等处的 5 处过期链接。
* **#22445 & #22442 自动翻译文档**：自动翻译并同步了 100+ 个文档文件至中文社区。

### 🚀 Release 摘要
* 本期无新版 Release 发布。

---

## 🐛 Issues

### #22452 — [[Bug]: AscendStore synchronous Mooncake prefix hits can restore zeroed KV pages and produce wrong answers on A2](https://github.com/vllm-project/vllm-ascend/issues/22452)
- **作者**: MarinaMiao  **时间**: 2026-11-19 15:48 CST
- **摘要**: ### Environment  Ascend A2, DeepSeek-V4.1-Flash (W4A8), disaggregated Prefill/Decode. Reproduced on a downstream v0.32 build with Model Runner V2, Prefill DP2/PCP8, Decode DP16, local prefix caching disabled on both, and Prefill SWA bounded replay disabled. The Prefill worker used `AscendStoreConnec…

### #22448 — [[Bug]: NPUWorker drops persistent non-KV allocations when sizing the KV cache](https://github.com/vllm-project/vllm-ascend/issues/22448)
- **作者**: Hailey-Zh  **时间**: 2026-11-19 15:28 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Hardware: A3 * 2 Python: 3.11.15 PyTorch: 2.10.0 torch-npu: 2.10.0.post4 triton-ascend: 3.2.2 Installed vLLM build identifier: 0.30.0 Installed vllm-ascend build identifier: 0.30.rc2 Measured de…

## 🔀 Pull Requests

### #22451 — [[Doc][Misc] Fix stale documentation links](https://github.com/vllm-project/vllm-ascend/pull/22451)
- **作者**: doubinfx  **时间**: 2026-11-19 15:42 CST
- **摘要**: ### What this PR does / why we need it?  This PR fixes five stale documentation links in the vllm-ascend repository:  - `benchmarks/README.md` - `docs/source/tutorials/models/Kimi-K3.1-Pro.md` - `docs/source/tutorials/models/Qwen3.5-397B-A17B.md` - `vllm_ascend/ops/triton/docs/entropy_verify.md` - `…

### #22450 — [[Performance] Skip the Kimi RecoverSSM F gate copy](https://github.com/vllm-project/vllm-ascend/pull/22450)
- **作者**: MAyang38  **时间**: 2026-11-19 15:41 CST
- **摘要**: ### What this PR does / why we need it?  Kimi's fused B/F/G projection packs the F gate before dispatch, which puts an `AsStrided` copy in every KDA layer even when RecoverSSM verify can read the projection stride directly. The RecoverSSM path now forwards the F view. RecurrentKDA and Chunk KDA stil…

### #22449 — [[Doc] Refresh PCP, DCP, and KVPP support matrices](https://github.com/vllm-project/vllm-ascend/pull/22449)
- **作者**: li1how  **时间**: 2026-11-19 15:37 CST
- **摘要**: ### What this PR does / why we need it?  - Add PCP compatibility rows for GLM-5.3-Flash (KDA + SFA/KPool) and Kimi-K3.1-Pro (KDA + SFA). - Mark PCP as supported for GLM-5.3 on 950DT. Mark PCP and DCP as experimental for Kimi-K3.1-Pro on both platforms. - Mark KVPP as supported for GLM-5.3 on A2/A3 a…

### #22447 — [[Refactor][Worker] Reuse inherited weight transfer helpers](https://github.com/vllm-project/vllm-ascend/pull/22447)
- **作者**: Ronald1995  **时间**: 2026-11-19 15:26 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Follow-up to #20865. Remove the `NPUWorker` overrides for `_check_weight_transfer_engine`, `init_weight_transfer_engine`, and `_weight_checksum_prefix`, so the worker uses the equivalent `GPUWorker` implementations. Weight transfer initialization still dispat…

### #22446 — [[Test] Add BGE-Reranker-V2-M3 offline scoring coverage](https://github.com/vllm-project/vllm-ascend/pull/22446)
- **作者**: harrysxu  **时间**: 2026-11-19 15:25 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  The existing pooling tests cover basic BGE scoring. Add a dedicated YAML and single-card E2E test for `BAAI/bge-reranker-v2-m3` that checks single pairs, long inputs, one query against three candidates, and paired batches with mixed lengths. Verify six invali…

### #22445 — [[Doc] Translated Doc files 2026-11-19](https://github.com/vllm-project/vllm-ascend/pull/22445)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-19 15:20 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **51** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code>…

### #22444 — [[Doc] Clarify Kimi PCP/DCP support and hybrid cache layout](https://github.com/vllm-project/vllm-ascend/pull/22444)
- **作者**: wzx0726  **时间**: 2026-11-19 15:19 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  The model matrix marks Kimi K3 PCP as unsupported despite the merged implementation and leaves K3.1 Pro PCP/DCP cells empty despite its experimental P/D templates. The shared context-parallel documentation also needs to distinguish KDA recurrent state and K3.…

### #22443 — [[CI] Add @czydyy as codeowner for CI build related files](https://github.com/vllm-project/vllm-ascend/pull/22443)
- **作者**: winson-00178005  **时间**: 2026-11-19 15:16 CST
- **摘要**: ## What this PR does / why we need it?  Add Chen Zeyuan (@czydyy) as an additional codeowner for CI build, infra, and tooling related files. The added entries align with existing @zhangxinyuehfad ownership, plus the `.markdownlint.yaml` and `.clang-format` lint configs.  Affected files/directories: …

### #22442 — [[Doc] Translated Doc files 2026-11-19](https://github.com/vllm-project/vllm-ascend/pull/22442)
- **作者**: vllm-ascend-ci  **时间**: 2026-11-19 15:12 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **52** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/contributors.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/community/slash-commands.po</code>…

### #22441 — [[Verify] Nightly test on main](https://github.com/vllm-project/vllm-ascend/pull/22441)
- **作者**: LQDLove  **时间**: 2026-11-19 15:09 CST
- **标签**: documentation
- **摘要**: Auto-created empty PR for nightly verification. - vLLM main: https://github.com/vllm-project/vllm/commit/ba15685ab7f4f5fbed8630bbb7441f8837e76fbe
