# vllm-project/vllm — 动态追踪

> 生成时间: 2026-09-29 14:03 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要：

### 📋 Issue 动态
- **NIXL 支持共享进行中的 KV Prefix 加载 (#59141)**
  提出了一个 RFC，建议通过 NIXL 让多个并发的 rollout 请求共享正在进行中的 KV 前缀加载。该方案计划由父请求提供共享内存分配、引用计数以及接收失败处理机制，以减少重复加载开销并提升整体吞吐。

---

### 🔧 Pull Request 动态

**【新特性与架构优化】**
- **支持 PP 与 PCP 组合运行 (#59139)**：在 GPU Model Runner V2 中，为 MLA 模型启用了流水线并行 (PP) 与 Prefill 上下文并行 (PCP) 的联合支持，最后一个 PP 阶段会将局部的 PCP 隐藏状态恢复为全局请求状态。
- **迁移 Bounded Replay 检查点至 Prefill (#59142)**：针对 DeepSeek-V4.1 和 Mooncake P/D 部署场景，将 bounded-replay 状态的生产过程转移到 Prefill 实例中，使 Decode 实例只需重算最后的 prompt 部分，显著降低计算开销。
- **Rust 前端引入真实 XGrammar 验证 (#59143)**：此前 Rust 构建的输出语法仅作为结构化标签 JSON 进行检查，现通过真实的 XGrammar 进行编译并回放验证，确保匹配器能正确接受来自 tokenizer 的实际生成结果。

**【Bug 修复】**
- **修复全无效 Token 时的采样异常 (#59144)**：当 `allowed_token_ids` 与 `bad_words` 发生冲突导致所有 token 均不合法（logits 全为 `-inf`）时，原逻辑会静默返回 token 0，现已修复此违规行为。
- **修复 KV Offload 按比例丢弃不准的问题 (#59140)**：修复了 `ThrottledDropPolicy` 在处理压力驱散时，其实际丢弃比例与计算出的目标比例（如 75%）不匹配的逻辑错误。
- **修复 MoE FlashInfer 单侧分发大小问题 (#59134)**：在使用 block-FP8 MoE 且专家自行量化输入的场景下，修复了 FlashInfer 单侧分发准备阶段的空间尺寸计算错误。
- **修复 ROCm DeepSeek-V4.1 SWA 窗口限制 (#59136)**：在 SWA bounded replay 启动时，修复了 ROCm prefill combiner 仍输出完整窗口导致与缩短后的 `gather_lens` 不匹配的问题。
- **修复 Derender 路径解码错误 (#59138)**：修复了在 `/v1/chat/completions/derender` 等扩展路径中，非流式响应错误解码全序列 token ID 的问题。

**【CI 与文档更新】**
- **扩展 ROCm MI355 CI 覆盖 (#59137)**：添加了 MI355 镜像及对应的独立 AMD 测试任务（涵盖序列并行、AsyncTP、分布式 AgRs 等），并将 MIG-sized 任务路由到 DPX。
- **修正 ModelOpt 文档 (#59135)**：修复了文档中关于 `W4A16_NVFP4` 检查点默认选择 Marlin 内核的过时描述，使其与实际代码逻辑保持一致。

---

### 🚀 Release 动态
近期无新版本发布。

---

## 🐛 Issues

### #59141 — [[RFC][KV Connector] NIXL support for shared in-flight prefix loads](https://github.com/vllm-project/vllm/issues/59141)
- **作者**: NolenLiang  **时间**: 2026-09-29 13:12 CST
- **标签**: kv-connector
- **摘要**: Propose a NIXL follow-up to #57418 so concurrent rollout requests can share an in-flight KV prefix load. The parent provides shared allocation, reference counting and receive-failure handling. This follow-up adds NIXL compatibility checks and producer-lease coordination.  ### Why this is useful  The…

## 🔀 Pull Requests

### #59144 — [fix: handle sampling when all tokens are invalid (allowed_token_ids + bad_words contradiction)](https://github.com/vllm-project/vllm/pull/59144)
- **作者**: yashlabs-trying  **时间**: 2026-09-29 13:56 CST
- **标签**: qwen, kv-connector, mrv2, scheduler
- **摘要**: ## Summary  When \llowed_token_ids\ and \ad_words\ (or other logit processors) eliminate every legal token, all logits become \-inf\. Previously \rgmax(-inf)\ silently returned token 0, violating the invariant that only allowed tokens are generated.  ## Fix  ### Detection \\\python no_valid_token…

### #59143 — [[Rust Frontend] Replay roundtrip output grammars through XGrammar](https://github.com/vllm-project/vllm/pull/59143)
- **作者**: BugenZhao  **时间**: 2026-09-29 13:30 CST
- **标签**: ready, rust, kimi, k3
- **摘要**: ## Purpose  Rust-built output grammars are currently checked only as structural-tag JSON. Nothing compiles them with the real XGrammar or verifies that the matcher accepts a real generation from token zero, which is the meaning of `reasoning_ended = Some(true)` since #57340.  Step 2.5 of the parser-…

### #59142 — [[RFC][Feature] Transfer bounded replay checkpoints from Prefill](https://github.com/vllm-project/vllm/pull/59142)
- **作者**: wangyicong52  **时间**: 2026-09-29 13:27 CST
- **标签**: documentation, kv-connector, mrv2, scheduler
- **摘要**: ## Summary  Move DeepSeek-V4.1 bounded-replay state production to the Prefill instance for Mooncake P/D deployments. After a per-request checkpoint is certified, Decode recomputes only the final prompt token instead of replaying the last 128 tokens.  The feature is opt-in on both instances:  ```json…

### #59140 — [[Bugfix][KV Offload] Fix inaccurate proportional store dropping](https://github.com/vllm-project/vllm/pull/59140)
- **作者**: thuongvu  **时间**: 2026-09-29 13:07 CST
- **标签**: bug
- **摘要**: ## Purpose  `ThrottledDropPolicy` calculates what fraction of store attempts should be dropped based on pressure. This PR fixes how it applies that fraction.  For a calculated drop rate of 75%, the expected result over 20 store attempts is 15 drops and 5 allowed stores.  - Before: 10 dropped, 10 all…

### #59139 — [[Feat] Support PP with PCP in GPU Model Runner V2](https://github.com/vllm-project/vllm/pull/59139)
- **作者**: pisceskkk  **时间**: 2026-09-29 12:44 CST
- **标签**: mrv2
- **摘要**: ## Purpose  Enable pipeline parallelism (PP) together with Prefill Context Parallelism (PCP) in GPU Model Runner V2 for MLA models.  The last PP stage restores PCP-local hidden states to global request order before sampling. Earlier PP stages use the global request batch when receiving sampled token…

### #59138 — [Issue#59089](https://github.com/vllm-project/vllm/pull/59138)
- **作者**: seansie0830  **时间**: 2026-09-29 12:38 CST
- **摘要**: ## Purpose  Fixes #59089  In the scale-out token-in/token-out path (`/v1/chat/completions/derender` and `/v1/completions/derender`), non-streaming responses decode the full sequence of token IDs using `tokenizer.decode(...)`. When generation terminates midway through a multi-byte UTF-8 character (wh…

### #59137 — [[ROCm][CI] Expand MI355 mirrors and route MIG-sized jobs to DPX](https://github.com/vllm-project/vllm/pull/59137)
- **作者**: AndreasKaratzas  **时间**: 2026-09-29 12:27 CST
- **标签**: rocm, torch.compile, ci/build
- **摘要**: - Add MI355 mirrors and matching standalone AMD jobs for Sequence Parallel Correctness, AsyncTP Correctness, Distributed AgRs All2All, and Spec Decode AL DFlash2 Nightly. - Mirror four supported FP8 cases from the B200 Fusion E2E Config Sweep on `mi355_dpx`; keep the broader NVIDIA sweep intact. - M…

### #59136 — [[ROCm][DSv4.1] Bound the prefill SWA window at the replay start](https://github.com/vllm-project/vllm/pull/59136)
- **作者**: yuzhouo7  **时间**: 2026-09-29 12:21 CST
- **标签**: rocm, deepseek, DSv4.1
- **摘要**: ## Purpose  Under SWA bounded replay, `gather_lens` is shortened so each request's gathered SWA buffer starts at its replay start. The ROCm prefill combiner (`amd/rocm.py`) still emits a full `min(pos + 1, window)` window, so the first replayed rows index below that buffer. This adds the bound the F…

### #59135 — [[Docs] Fix stale W4A16 NVFP4 default kernel in ModelOpt docs](https://github.com/vllm-project/vllm/pull/59135)
- **作者**: WarrierRajeev  **时间**: 2026-09-29 12:19 CST
- **标签**: documentation, quantization
- **摘要**: ## Purpose  The ModelOpt docs say that for `W4A16_NVFP4` checkpoints, `auto` "currently selects Marlin". That isn't what the code does. #53014 added this sentence together with the logic that makes `auto` pick the FlashInfer CuTe-DSL kernel on SM100/SM103, which the `v0.30.0` release notes list unde…

### #59134 — [[Bugfix][MoE] Size the FlashInfer one-sided dispatch for experts that quantize their own inputs](https://github.com/vllm-project/vllm/pull/59134)
- **作者**: charon14  **时间**: 2026-09-29 12:04 CST
- **标签**: bug, nvidia, quantization
- **摘要**: ## Purpose  With a block-FP8 MoE, `--moe-backend flashinfer_cutlass` and `--all2all-backend flashinfer_nvlink_one_sided`, `FlashInferExperts.expects_unquantized_inputs` is True, so the one-sided prepare step dispatches the BF16 activations and the CUTLASS kernel quantizes them itself. `make_fp8_moe_…
