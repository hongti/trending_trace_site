# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-31 14:10 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态的中文摘要：

### Issue
本次更新周期内未提供相关 Issue 动态。

### Pull Request (PR)
本次共涵盖 10 个 PR，主要涉及新特性适配、代码重构、CI/测试优化及文档翻译：

- **核心特性与功能更新**
  - **适配 vLLM 主干**：将 vllm-ascend 适配至 vLLM 10月30日的最新主干代码（#20426）。
  - **BF16 PLE 主机卸载**：为 Ascend 上的 Qwen4Exp 模型添加 BF16 PLE 主机卸载功能，通过掩码 Triton kernel 直接在 NPU 聚合数据，避免了 H2D（Host-to-Device）数据拷贝开销（#20424）。
  - **多源 HCCL 权重广播**：支持 RL 训练器的多源 HCCL 权重广播，解决了流水线并行训练器无法跨 stage 发送参数的问题（#20419）。
  - **AscendC 替代压缩器**：使用 AscendC 替换 DeepSeek V4.1 的原生 Compressor（#20417）。

- **重构与清理**
  - **MoE 参数清理**：在 `fused_moe` 重构后，移除了 DeepSeek-V4 中不再使用的 `tid2eid` 参数（#20420）。
  - **旧测试用例淘汰**：停用了 kimi-2.5、minimax-m2.5、minimax-m2.7 等过时的模型测试用例（#20421）。

- **CI 与测试基础设施**
  - **测试 Runner 迁移**：将 A2 和 A3-560T 单节点测试用例从通用 runner 池迁移至专用测试 runner（#20422）。
  - **清理 Nightly 任务**：移除了已迁移至 PR E2E 测试中的单卡算子 nightly 记录，避免指向空目录（#20423）。

- **文档与国际化**
  - 自动翻译了 66 个文档文件（#20425）以及 Kimi-K3 相关教程文档（#20418）。

### Release
本次更新周期内未发布新版本。

---

## 🔀 Pull Requests

### #20426 — [[Misc]feat: adapt to vLLM main (24420b7a)](https://github.com/vllm-project/vllm-ascend/pull/20426)
- **作者**: submitter-dev  **时间**: 2026-10-31 13:48 CST
- **标签**: module:tests, module:ops, merge-conflicts
- **摘要**: ### What this PR does / why we need it?  Adapt vllm-ascend to vLLM main commits up to October 30.  ### Changes  | Files | Upstream vLLM change | vllm-ascend adaptation | |-------|---------------------|------------------------| | — | [50a184f4](https://github.com/vllm-project/vllm/commit/50a184f43583…

### #20425 — [[Doc] Translated Doc files 2026-10-31](https://github.com/vllm-project/vllm-ascend/pull/20425)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-31 13:30 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **66** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/context_parallel.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/develop…

### #20424 — [ [Feature] Add BF16 PLE host offload for Qwen4Exp on Ascend](https://github.com/vllm-project/vllm-ascend/pull/20424)
- **作者**: yeatschou0-ui  **时间**: 2026-10-31 12:14 CST
- **标签**: documentation, module:tests, module:ops, module:core
- **摘要**: ## What this PR does / why we need it?    Keep BF16 PLE/ngram embedding tables in CANN-registered host memory and gather selected rows directly on NPU via a masked Triton kernel,   avoiding H2D copy and host-side gather. Projections and convolution states remain on the NPU.    ### Key changes    - *…

### #20423 — [[CI][Test] Remove migrated single-card operator nightly entries](https://github.com/vllm-project/vllm-ascend/pull/20423)
- **作者**: Foriv  **时间**: 2026-10-31 11:49 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: ### What this PR does / why we need it?  Follow-up to #20307, which moves the single-card operator tests into PR E2E. The old nightly entries would otherwise point at a directory without test modules after that migration.  - Remove `singlecard-ops-a3` and `singlecard-ops-a5` from the nightly matrix.…

### #20422 — [[CI] Migrate A2 and A3-560T single-node os labels to test runners](https://github.com/vllm-project/vllm-ascend/pull/20422)
- **作者**: CuixyHW  **时间**: 2026-10-31 11:46 CST
- **摘要**: ### What this PR does / why we need it?  **Background.** The A2 and A3-560T single-node test cases currently run on the general runner pools (`linux-aarch64-a2b3-*` / `linux-aarch64-a3-800i-*`). These cases are being migrated to the dedicated test runner pools, so the `os` labels in the nightly conf…

### #20421 — [[Test] sunset kimi-2.5 minimax-m2.5 minimax-m2.7 test cases](https://github.com/vllm-project/vllm-ascend/pull/20421)
- **作者**: chen-commits  **时间**: 2026-10-31 11:35 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  ---  ### Checklist  #### Testing  - [ ] **Boundary / edge-case coverage**: I reviewed whether this change involves   boundary conditions (e.g. empty input, single/zero eleme…

### #20420 — [[Refactor] Remove unused tid2eid param after fused_moe refactor.](https://github.com/vllm-project/vllm-ascend/pull/20420)
- **作者**: menogrey  **时间**: 2026-10-31 11:22 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: ### What this PR does / why we need it? **Background.** DeepSeek-V4 hash-routed MoE layers do not run top-k over router logits: the experts for a token come from a checkpoint table, `topk_ids = tid2eid[input_ids]`. When V4 support landed (#9270, `7bce23cc0`), expert selection still lived inside the …

### #20419 — [[Feature] Support multi-source HCCL weight broadcasts for RL trainers](https://github.com/vllm-project/vllm-ascend/pull/20419)
- **作者**: Annarine  **时间**: 2026-10-31 11:16 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  HCCL weight transfer currently assumes one trainer broadcaster at group rank 0. A pipeline-partitioned trainer cannot send the parameters owned by another stage through that adapter, even when vLLM already supports ownership-based multi-source broadcasts.  Re…

### #20418 — [[v0.30.0][Doc] Translated Doc files 2026-10-31](https://github.com/vllm-project/vllm-ascend/pull/20418)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-31 11:10 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **1** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/tutorials/models/Kimi-K3.po</code>  ---  [Workflow run](https://github.com/vllm-project/vllm-ascend/actions/runs/41162201337) - vLLM main: https://github.co…

### #20417 — [[Feature][Model] Replace DeepSeek V4.1 compressor with AscendC](https://github.com/vllm-project/vllm-ascend/pull/20417)
- **作者**: zhangwenTao140  **时间**: 2026-10-31 11:08 CST
- **标签**: documentation, module:tests, module:ops
- **摘要**: Port vllm-project/vllm-ascend#17282 at 340d0ac9a783d034054623269e3028cec3a9617d onto main 95753074cb37f6299024937993100de9c8d7000d. Preserve all vendored CompressorV2 sources, bindings, native adapter and regression coverage.  Resolve current hardware-profile and multistream context, follow the V1 r…
