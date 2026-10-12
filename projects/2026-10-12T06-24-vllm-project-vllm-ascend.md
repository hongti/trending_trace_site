# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-12 14:24 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### Issue (议题)
*   **PD分离架构预热崩溃 (#18374)**：报告在 Atlas 800 A2 (910B) 硬件环境下，使用 v0.27.1rc1 版本进行 PD分离架构的 prefill 预热时发生崩溃，报错 `aclrtAllocatorGetByStream 'stream is invalid'`（环境配置为 4 节点 prefill + 4 节点 decode）。

### Pull Request (拉取请求)
**新特性与优化**
*   **Mamba PMU 边界交接支持 (#18366, #18367)**：集成上游 vLLM 的 Mamba PMU（进程内存单元）边界交接机制，分别支持与 Ascend Mooncake 和 MTP 的结合使用，优化 P/D 架构的数据传输。
*   **PCP 图支持回移 (#18365)**：在 `v0.30.0` 版本中支持 MRV2 PCP 配合 PIECEWISE 和 FULL_AND_PIECEWISE 图，并实现跨流水线阶段的状态传播。

**Bug 修复**
*   **修复 MiniMax-M3 Eagle3 补丁冲突 (#18371, #18370)**：针对主分支和 `v0.30.0` 分支，跳过对 MiniMax-M3 的通用 Eagle3 PP 辅助补丁，防止其覆盖模型原生实现导致功能破坏。
*   **拦截非法并行配置 (#18373)**：在配置校验阶段直接拒绝“细粒度张量并行(TP)与流水线并行(PP)”组合使用的情况，避免引发进程组初始化错误。
*   **修复内存探测启动失败 (#18369)**：放宽内存探测断言条件（从严格大于 `>` 改为大于等于 `>=`），解决在内存波动场景下因可用内存未增加而导致的启动中断。

**文档与 CI**
*   **新增 GLM-5.3 部署指南 (#18375)**：添加并完善 GLM-5.3-Flash Prefill-Decode 在 Atlas A2/A3 平台基于 MooncakeConnector 的非池化部署文档。
*   **澄清 KVPP 镜像版本 (#18372)**：明确说明 `v0.23.0` 镜像不包含 KVPP，指出 `v0.30.0rc1` 版本开始包含基础实现，并说明了 A5 专用的 `-a5` 镜像标签。
*   **CI 流程增强 (#18368)**：在由 PR 触发或手动触发的夜间测试中，增加指定文件或目录的下载功能（作为 GitHub Actions artifact）。

### Release (发布)
*   *近期无 Release 版本发布。*

---

## 🐛 Issues

### #18374 — [[Bug]: PD-disaggregation prefill warmup crash with EH0012 aclrtAllocatorGetByStream 'stream is invalid' on A2 (DSA-CP + DCP8), v0.27.1rc1](https://github.com/vllm-project/vllm-ascend/issues/18374)
- **作者**: amchen2310  **时间**: 2026-10-12 14:05 CST
- **摘要**: ### Environment  - **Hardware**: 8× nodes Atlas 800 A2 (910B, 64GB × 8 per node) — 4 prefill + 4 decode - **Software**: vllm-ascend v0.27.1rc1, CANN 9.1.0, vLLM (in image, official v0.27.1rc1 base) - **Model**: GLM-5.3-w8a8c8 (ascend quantization, MoE, MTP) - **Topology**: PD disaggregation; Prefill…

## 🔀 Pull Requests

### #18375 — [[Doc][Feature] Add GLM-5.3 A2 PD deployment guide](https://github.com/vllm-project/vllm-ascend/pull/18375)
- **作者**: calyoung80  **时间**: 2026-10-12 14:14 CST
- **标签**: documentation, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  - Adds and refines the GLM-5.3-Flash Prefill-Decode deployment documentation for Atlas A2/A3 platforms.  - Documents non-pooled PD deployment using MooncakeConnectorV2.  - Documents pooled PD deployment using MultiConnector, combining MooncakeConnectorV2 and …

### #18373 — [[BugFix][Config] Reject fine-grained TP with pipeline parallelism](https://github.com/vllm-project/vllm-ascend/pull/18373)
- **作者**: do420  **时间**: 2026-10-12 13:52 CST
- **标签**: module:tests, module:core
- **摘要**: ## Summary - Fail fast during fine-grained TP config validation when `pipeline_parallel_size > 1` and any fine-grained TP size is configured. - Avoid initializing process groups whose current rank-grid construction can mix pipeline stages and hang on collectives. - Add regression coverage for all fo…

### #18372 — [[Docs] Clarify KVPP image version availability](https://github.com/vllm-project/vllm-ascend/pull/18372)
- **作者**: do420  **时间**: 2026-10-12 13:46 CST
- **标签**: documentation
- **摘要**: ## Summary - Explain that the `v0.23.0` image does not include KVPP and identify the `v0.30.0rc1` release line as containing the base implementation. - Point out the A5-specific `-a5` image tag and warn that some combinations documented on `main` may require a newer image. - Add the matching Chinese…

### #18371 — [[BugFix][Worker] Skip Eagle3 PP aux patch for MiniMax-M3](https://github.com/vllm-project/vllm-ascend/pull/18371)
- **作者**: CXY-Katrina  **时间**: 2026-10-12 13:09 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  MiniMax-M3 already propagates Eagle3 auxiliary hidden states across pipeline stages. The generic Eagle3 PP patch overwrites its native implementation.  Return `False` specifically for `MiniMaxM3Model`, preserving its native `forward` and intermediate-tensor m…

### #18370 — [[BugFix][Worker][v0.30.0] Skip Eagle3 PP aux patch for MiniMax-M3](https://github.com/vllm-project/vllm-ascend/pull/18370)
- **作者**: CXY-Katrina  **时间**: 2026-10-12 13:06 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  MiniMax-M3 already propagates Eagle3 auxiliary hidden states across pipeline stages. The generic Eagle3 PP patch replaces this native implementation and can break the model's PP path.  Return `False` from `patch_eagle3_pp_aux_propagation` specifically for `Mi…

### #18369 — [[BugFix][Worker] Accept unchanged free-memory snapshots in memory profiling](https://github.com/vllm-project/vllm-ascend/pull/18369)
- **作者**: bjptw  **时间**: 2026-10-12 12:41 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Relax the memory-profiling assertion from `>` to `>=`: an unchanged free-memory snapshot is legitimate, and the strict comparison aborts startup under memory-virtualization layers.  `NPUWorker.determine_available_memory()` asserts `init_snapshot.free_memory >…

### #18368 — [[CI] Add requested file and directory downloads to nightly runs](https://github.com/vllm-project/vllm-ascend/pull/18368)
- **作者**: czydyy  **时间**: 2026-10-12 12:38 CST
- **标签**: documentation, ci/build, module:tests
- **摘要**: ### What this PR does / why we need it?  Add `--download` to PR-triggered and manually dispatched nightly runs so a requested file or directory can be downloaded as a GitHub Actions artifact. Accept absolute paths, paths relative to the test repository, and per-node mappings such as `[{"node0":"/pat…

### #18367 — [[Feature][KV Transfer] Reuse upstream Mamba PMU boundary handoff with MTP](https://github.com/vllm-project/vllm-ascend/pull/18367)
- **作者**: iKeybot-code  **时间**: 2026-10-12 12:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Reduced follow-up to #16528 for the P/D and MTP integration parts of RFC #16451. Reuse upstream vLLM PMU matching, Mamba checkpoints, CoW and allocation; do not patch vLLM source or duplicate those mechanisms.  - ST03: temporarily disable the dedicated EAGLE …

### #18366 — [[Feat][PD] Integrate upstream Mamba PMU boundary handoff with Mooncake](https://github.com/vllm-project/vllm-ascend/pull/18366)
- **作者**: iKeybot-code  **时间**: 2026-10-12 12:11 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Integrate the remaining Mamba PMU P/D handoff requirements with Ascend Mooncake, following RFC #16451. This is an independent, reduced adaptation informed by #16528; that PR is not modified or superseded automatically.  Only two production files change:  - Pr…

### #18365 — [[Feature][Worker][v0.30.0] Support PCP with PIECEWISE and FULL_AND_PIECEWISE graphs](https://github.com/vllm-project/vllm-ascend/pull/18365)
- **作者**: tangdafu  **时间**: 2026-10-12 12:01 CST
- **标签**: documentation, module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Backport #17995 at commit aa6e1fb90947646ef7626640bdbae68550a838cb to `releases/v0.30.0`.  Enable MRV2 PCP with PIECEWISE and FULL_AND_PIECEWISE graphs, propagate the FULL graph flag to DSA metadata builders, use the appropriate actual or padded token extent …
