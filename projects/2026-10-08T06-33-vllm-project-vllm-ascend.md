# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-08 14:33 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### Issue 动态
- **FA3 与投机解码混合批次崩溃 (#17991)**：当启用 FA3 和 ngram 投机解码，且同一个批次中同时包含 prefill 和 decode 请求时，推理过程会触发 aicore 异常（错误码 507015）并崩溃。

### PR 动态
**新特性与功能增强**
- **DeepSeek V4.1 PCP 支持**：新增在 Ascend A5 上的预填充上下文并行（PCP）支持，包含 O projection 权重切分，并支持 MRV2 结合 Shared Engram 的单节点部署 (#17992, #17989)。
- **GQA Decode 上下文并行**：标准 GQA 注意力后端现支持 decode 上下文并行，历史 KV 分片使用 DCP 通信，当前投机 token 段使用新机制 (#17990)。
- **Ascend MRV2 支持 NGram 投机解码**：适配上游 vLLM 的 MRV2 NGram 投机解码功能，填补了 Ascend 平台对该方法的支持空白 (#17985)。
- **GLM-5.3 Flash DSA CP 支持**：针对 GLM-5.3 Flash 使用 NoPE sparse MLA 和 KPool indexer 的情况，启用 SFA-only DSA CP，解决了原有 DSA CP 路径假设 RoPE 元数据导致的不兼容问题 (#17987)。

**Bug 修复**
- **MiniMax-M3 启动失败**：修复了在 MRV2、TP4、DP4 和 EAGLE3 配置下首次全图捕获预热时，`AscendMiniMaxM3IndexerImpl.forward()` 查找 `_is_m` 导致的启动失败问题 (#17994)。
- **ACLNN 构建错误处理**：修复 `csrc/build_aclnn.sh` 在接收到未识别的 `SOC_VERSION`（如 Ascend910B1）时静默跳过编译并返回成功的问题，现在会直接报错 (#17993)。
- **CI 缓存依赖问题**：移除 csrc 构建缓存对第三方 `regex` 库的依赖，修复了 CI 在安装项目依赖前调用该命令导致的环境异常 (#17986)。

**CI 与测试**
- 新增 GLM-5.1 W8A8C8 128k 上下文的 CI 测试配置 (#17988)。
- 更新 Pipeline Parallel size 相关的 CI 测试 (#17984)。

### Release 动态
- 本时段内无新版本发布。

---

## 🐛 Issues

### #17991 — [[Bug]: FA3 crashes with mixed prefill and speculative-decoding batches](https://github.com/vllm-project/vllm-ascend/issues/17991)
- **作者**: heiyan-2020  **时间**: 2026-10-08 13:56 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of `python collect_env.py`</summary>  ```text Collecting environment information... PyTorch version: 2.10.0+cpu Is debug build: False  OS: Ubuntu 22.04.5 LTS (aarch64) GCC version: (Ubuntu 11.4.0-1ubuntu1~22.04.3) 11.4.0 Clang version: 15.0…

## 🔀 Pull Requests

### #17994 — [[BugFix][Model] Avoid current-config lookup in MiniMax-M3 MRV2 forward](https://github.com/vllm-project/vllm-ascend/pull/17994)
- **作者**: yingyingzizi  **时间**: 2026-10-08 14:25 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  MiniMax-M3 A3 decode startup can fail at the first FULL graph-capture warmup with MRV2, TP4, DP4 and EAGLE3. `AscendMiniMaxM3IndexerImpl.forward()` calls `_is_mrv2_idle_dp_dummy()`, which reads `get_current_vllm_config().use_v2_model_runner`. The capture path…

### #17993 — [[BugFix][Build] Fail on unsupported SOC_VERSION in ACLNN build](https://github.com/vllm-project/vllm-ascend/pull/17993)
- **作者**: hanyu-ux  **时间**: 2026-10-08 14:24 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  When `csrc/build_aclnn.sh` receives an unrecognized SOC_VERSION, such as `Ascend910B1` or `ascend999`, it currently skips all ACLNN compilation and returns success. The `subprocess.check_call` caller in setup.py therefore cannot detect that this build step pr…

### #17992 — [[Feature][PCP] Add DeepSeek V4.1 PCP support on Ascend A5](https://github.com/vllm-project/vllm-ascend/pull/17992)
- **作者**: li1how  **时间**: 2026-10-08 14:23 CST
- **标签**: documentation, module:tests, module:ops, module:core, module:quantization, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Depends on #17927; this branch is based on its `00b2b9b67` revision.  - Add MRV2 PCP support for DeepSeek V4.1 on Ascend A5, including PCP O projection weight sharding and Q/KV multistream preprocessing. - Support PCP token routing for Engram. - Preserve the …

### #17990 — [[Feature][Attention] Support GQA decode context parallelism](https://github.com/vllm-project/vllm-ascend/pull/17990)
- **作者**: weiguihua2  **时间**: 2026-10-08 13:48 CST
- **标签**: module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  Enable the standard GQA attention backend to use decode context parallelism. Historical KV shards use DCP collectives, while the current speculative-token segment contributes once through the shared history/current merge policy. Reuse the v1 backend metadata …

### #17989 — [[Feat][Model] Add DeepSeek V4.1 MRV2 PCP with shared Engram](https://github.com/vllm-project/vllm-ascend/pull/17989)
- **作者**: vrooml  **时间**: 2026-10-08 13:12 CST
- **标签**: documentation, module:tests, module:core, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Enable DeepSeek V4.1 prefill context parallelism with Model Runner V2, including Engram-enabled single-node deployments. The submission is based on PR #16899's official squash-merge commit (`51e47bbb77a2a8561ab2a4a1e236b01befda1cab`). Only the PCP feature com…

### #17988 — [[CI] test GLM-5.1-W8A8C8-128k-1k-90-50-PD.yaml](https://github.com/vllm-project/vllm-ascend/pull/17988)
- **作者**: chen-commits  **时间**: 2026-10-08 12:39 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ee0da84ab9e04ac7610e28580af62c365e898389

### #17987 — [[Feature] Enable SFA-only DSA CP for GLM-5.3 Flash](https://github.com/vllm-project/vllm-ascend/pull/17987)
- **作者**: lijiahang226  **时间**: 2026-10-08 12:07 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  GLM-5.3 Flash uses NoPE sparse MLA and a KPool indexer, while the existing DSA CP path assumes RoPE metadata and does not provide the KPool query/cache layout needed for token sharding.  Enable SFA-only context parallelism within the TP group: - Build local q…

### #17986 — [[BugFix][CI] Remove regex dependency from csrc build cache](https://github.com/vllm-project/vllm-ascend/pull/17986)
- **作者**: AuroraEmiya  **时间**: 2026-10-08 11:29 CST
- **标签**: module:tests, module:tools
- **摘要**: ### What this PR does / why we need it?  The csrc cache engine imports third-party `regex` before handling `snapshot-key`. CI invokes that command before installing project dependencies, so an environment without `regex` raises `ModuleNotFoundError`. The restore helper fails open with `supported=fal…

### #17985 — [[Feature][MRV2] Support NGram speculative decoding on Ascend](https://github.com/vllm-project/vllm-ascend/pull/17985)
- **作者**: Silas-Zeng  **时间**: 2026-10-08 11:24 CST
- **标签**: module:tests, module:ops, module:core, merge-conflicts
- **摘要**: ### Motivation  Upstream vLLM added MRV2 NGram speculative decoding in [#40704](https://github.com/vllm-project/vllm/pull/40704). Ascend MRV2 does not yet support the method, as reported in [#16235](https://github.com/vllm-project/vllm-ascend/issues/16235).  This draft PR brings NGram support to Asc…

### #17984 — [[CI] PP-size](https://github.com/vllm-project/vllm-ascend/pull/17984)
- **作者**: guxin108  **时间**: 2026-10-08 11:23 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607
