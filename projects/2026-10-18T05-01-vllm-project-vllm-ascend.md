# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-18 13:01 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### 📝 Pull Request (PR) 动态
近期 PR 活动主要集中在上游代码适配、CI/CD 构建修复、核心 Bug 修复以及性能优化，且包含多个针对 `v0.30.0` 版本的改进：

1. **上游适配与 CI 构建**
   - **适配 vLLM 主分支** (#19103)：将 vllm-ascend 代码适配至 vLLM 主分支截至 10 月 14 日的最新提交。
   - **Nightly 构建与依赖修复** (#19106, #19104, #19102, #19100)：修复了 nightly 镜像和 e2e 运行时安装中的依赖问题，确保在 v0.30.0 nightly 构建中保留基础镜像的依赖版本（如 `torch-npu`、`triton-ascend` 等的精确版本锁定）。同时 CI 适配了 memcache memfabriq 源。

2. **核心 Bug 修复**
   - **隔离 Memcache 传输失败** (#19101)：修复了 KVPool 中 Memcache 传输失败导致异常抛出的问题，避免其影响 layer fences 或队列阻塞。
   - **EPLB 缓冲区同步保护** (#19098)：在 v0.30.0 版本中，为 HIXL READ 操作前的 staging-buffer 复用添加了同步保护（fence），并回带了相关单元测试。

3. **性能优化**
   - **推测解码性能提升** (#19099)：在 v0.30.0 版本中，跳过了 K3 FlashMLA 目标 CPU 长度同步操作，减少了 MRV2 runner 在推测解码时的等待时间。
   - **HIXL 专家存储合并注册** (#19097)：将 EPLB 中原本分散分配的数千个 NZ 专家权重和接收缓冲区打包为单次注册，大幅降低了 peer connection 的资源开销。

### 💡 Issue 动态
- 本次提供的动态数据中暂无相关 Issue 更新。

### 🚀 Release 动态
- 近期暂无新版本发布。

---

## 🔀 Pull Requests

### #19106 — [[BugFix][CI] Keep the image's daily deps in nightly e2e runtime installs](https://github.com/vllm-project/vllm-ascend/pull/19106)
- **作者**: zhangxinyuehfad  **时间**: 2026-10-18 12:03 CST
- **标签**: ci/build, module:tests
- **摘要**: From main branch https://github.com/vllm-project/vllm-ascend/pull/19104  ### What this PR does / why we need it?  This is a temporary solution; a final method will be provided later.  requirements-nightly.txt, now used by the nightly image layer and the e2e runtime install, has no -r requirements.tx…

### #19105 — [add exit](https://github.com/vllm-project/vllm-ascend/pull/19105)
- **作者**: yangyao827  **时间**: 2026-10-18 11:58 CST
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #19104 — [[BugFix][CI] Keep the image's daily deps in nightly e2e runtime installs](https://github.com/vllm-project/vllm-ascend/pull/19104)
- **作者**: zhangxinyuehfad  **时间**: 2026-10-18 11:15 CST
- **标签**: ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  This is a temporary solution; a final method will be provided later.  requirements-nightly.txt, now used by the nightly image layer and the e2e runtime install, has no `-r requirements.txt`: that stopped the daily torch-npu from being replaced, but it also st…

### #19103 — [[Misc]feat: adapt to vLLM main (273a0496)](https://github.com/vllm-project/vllm-ascend/pull/19103)
- **作者**: submitter-dev  **时间**: 2026-10-18 09:58 CST
- **标签**: module:tests, module:ops, module:core, module:quantization, main2main
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to October 14.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | `vllm_ascend/__init__.py` | [c85616ef](https://github.com/vllm-project/…

### #19102 — [[CI] Preserve base image dependency versions in v0.30.0 nightly builds](https://github.com/vllm-project/vllm-ascend/pull/19102)
- **作者**: czydyy  **时间**: 2026-10-18 09:20 CST
- **摘要**: ### What this PR does / why we need it?  Nightly images are built on top of daily base images, but `requirements-dev.txt` includes `requirements.txt`. Exact pins for `torch-npu`, `triton-ascend`, `memfabric_hybrid`, and `memcache_hybrid` cause pip to replace the versions already installed in the bas…

### #19101 — [[BugFix][KVPool] Isolate Memcache transfer failures](https://github.com/vllm-project/vllm-ascend/pull/19101)
- **作者**: bowgneo  **时间**: 2026-10-18 02:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Main-branch counterpart of #19090. Memcache returns failed operation statuses, but Python transfer paths can raise on these results and leave layer fences or queue tasks incomplete. Handle returned statuses inside AscendStoreConnector.  **Synchronous hybrid r…

### #19100 — [[CI] adapt memcache memfabriq source 0.30.0](https://github.com/vllm-project/vllm-ascend/pull/19100)
- **作者**: xqchen7  **时间**: 2026-10-18 02:31 CST
- **摘要**: ### What this PR does / why we need it? [CI] adapt memcache memfabriq source ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested? NA  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditio…

### #19099 — [[v0.30.0][Performance][SpecDecode] Skip audited K3 FlashMLA target CPU length sync](https://github.com/vllm-project/vllm-ascend/pull/19099)
- **作者**: SparrowMu  **时间**: 2026-10-18 02:17 CST
- **摘要**: ### What this PR does / why we need it?  The MRV2 runner unconditionally stages rejection-corrected computed-token counts to CPU for speculative decoding, then waits for the copy in `_update_seq_lens_cpu()` before submitting the next forward. This creates a host wait even when the selected attention…

### #19098 — [[BugFix][v0.30.0][EPLB] Fence staging-buffer reuse before HIXL READs](https://github.com/vllm-project/vllm-ascend/pull/19098)
- **作者**: SparrowMu  **时间**: 2026-10-18 01:59 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Backport the HIXL staging-buffer synchronization and its unit test from [`f978348e9045fb045a4281d189339ef2ca1f1a45`](https://github.com/vllm-project/vllm-ascend/commit/f978348e9045fb045a4281d189339ef2ca1f1a45) (#17681) to `releases/v0.30.0`. The release branc…

### #19097 — [[Performance][EPLB] Compact HIXL expert storage into one registration](https://github.com/vllm-project/vllm-ascend/pull/19097)
- **作者**: freyfwt  **时间**: 2026-10-18 01:52 CST
- **标签**: documentation, module:tests, ready-a5, ready-precise
- **摘要**: ### What this PR does / why we need it?  HIXL EPLB can register thousands of separately allocated NZ expert weights and receive buffers, making peer connection expensive. Pack participating storages into one page-aligned device allocation per rank during HIXL session initialization, before address e…
