# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-20 10:02 CST

## AI 总结

以下是 **vllm-project/vllm** 仓库近期动态的中文摘要：

### 🐛 Issue（问题）
1. **推测解码存在 NaN 致命盲区（#53029）**：当模型前向传播产生全 NaN 的 `target_logits` 时，V2 GPU-worker 的拒绝采样器不会报错或回退，而是将 NaN 值错误洗白为超出词表的 Token ID，导致远离原点的设备断言错误，验证路径缺乏 NaN 检测。
2. **Gemma 4 + LoRA 显存非法访问（#53028）**：在使用 Gemma 4 E2B 结合 LoRA 时，若 `max_model_len` 设置大于等于 87383，引擎初始化阶段的 `lora_expand` 会出现非法内存访问。该问题在 Blackwell 及 H100 等硬件上已复现。

### 🔀 Pull Request（拉取请求）
**🚀 核心功能与性能优化**
- **融合 GLM-5.2 索引器注意力投影（#53027）**：将使用相同激活且兼容 BF16 线性方法的 `wk` 和 `weights_proj` 打包融合，提升 GLM-5.2 推理性能。
- **支持 DeepSeek V4 静态专家映射（#53022）**：为 DeepSeek V4 的 EPLB（专家并行负载均衡）添加预计算的静态专家分配 JSON 映射，优化 MoE 层专家加载。
- **代码清理（#53021）**：移除未使用的 `DeepseekV32Indexer.forward()` 实现及相关导入。

**🛠️ Bug 修复**
- **修复推测解码 Draft Logits 缓存步幅错误（#53017）**：修复了 `gumbel_sample` 中使用 `vocab_size` 替代实际 tensor stride 进行列步幅截取的危险行为，避免在词表大小与步幅不一致时出错。
- **修复 Nemotron Parse 权重绑定问题（#53020）**：当模型检查点未提供独立的 `lm_head.weight` 且配置了 `tie_word_embeddings=True` 时，正确处理权重绑定。

**🧪 CI 稳定性与基础设施建设**
- **ROCm/MI355 测试稳定性（#53025, #53024）**：修复了 AMD CI 中 MI355 FusedMoE 测试组和 FlyDSL W4A16 MoE 精度测试的失败问题。
- **修复 CI 依赖与生命周期（#53026, #53023）**：修正数据并行示例测试中不存在的依赖路径；修复 MultiConnector 精度测试中存在的生命周期竞争条件。
- **NIXL NAT 部署支持（#53018）**：为 NAT 网络环境分离了 NIXL 边带通道的本地监听地址与向对等节点广播的地址，解决地址不通问题。

### 🚀 Release（发布）
- 近期暂无新版本发布动态。

---

## 🐛 Issues

### #53029 — [[Bug][Spec Decode] All-NaN logits row is laundered into an out-of-vocab token id by the rejection sampler (device assert far from origin); NaN-metric blind spot on the verify path; uninit-scratch audit](https://github.com/vllm-project/vllm/issues/53029)
- **作者**: onesource2026  **时间**: 2026-08-20 09:49 CST
- **摘要**: ## TL;DR  If any verification row of `target_logits` is all-NaN (numerics upset in the model forward — whatever the cause), the V2 GPU-worker rejection sampler does not fail or fall back: its NaN-poisoned block-argmax chain **emits a token id equal to `vocab_size` exactly**, which propagates through…

### #53028 — [Gemma 4 E2B + LoRA: illegal memory access in lora_expand at engine init once max_model_len >= 87383](https://github.com/vllm-project/vllm/issues/53028)
- **作者**: ShuaiShao93  **时间**: 2026-08-20 09:36 CST
- **摘要**: ## Environment  vLLM 0.27.1, flashinfer-python 0.6.16.post3, triton 3.7.1, torch 2.13.0+cu130, CUDA 13.3 (driver 610.43.02). Reproduced on RTX PRO 6000 Blackwell (SM120, NVFP4 checkpoint) and H100 80GB (SM90, FP8 checkpoint). Also seen on 0.22.1.  ## Problem  With Gemma 4 E2B and `enable_lora=True`,…

## 🔀 Pull Requests

### #53027 — [[Performance][Model] Fuse GLM indexer attention projections](https://github.com/vllm-project/vllm/pull/53027)
- **作者**: WoosukKwon  **时间**: 2026-08-20 09:35 CST
- **标签**: deepseek, glm
- **摘要**: ## Summary  Fuse two pairs of GLM-5.2 sparse-indexer projections when they consume the same activation and use compatible unquantized BF16 linear methods:  - pack indexer `wk` and `weights_proj` into the attention `fused_qkv_a_proj` - pack indexer `wq_b` into the attention `q_b_proj`  The fusion is …

### #53026 — [[CI] Fix nonexistent dependency for data-parallel example test selection](https://github.com/vllm-project/vllm/pull/53026)
- **作者**: taneem-ibrahim  **时间**: 2026-08-20 09:23 CST
- **标签**: ci/build
- **摘要**: Current CI test this nonexistent dependency:  ```text tests/examples/features/data_parallel/data_parallel_offline.py ```  The real file is:  ```text examples/features/data_parallel/data_parallel_offline.py ```  This affects the single-node and two-node distributed jobs in [distributed.yaml](https://…

### #53025 — [[ROCm][CI] Stabilize MI355 FusedMoE test group](https://github.com/vllm-project/vllm/pull/53025)
- **作者**: AndreasKaratzas  **时间**: 2026-08-20 08:50 CST
- **标签**: rocm
- **摘要**: This PR addresses the three MI355 FusedMoE failures reported by [AMD CI build 12250](https://buildkite.com/vllm/amd-ci/builds/12250/list). The MI355 group was introduced by #41100 and exposed a B200-only DeepEP cleanup that #35077 had broadened to every platform, causing rocSHMEM destroy/reinitializ…

### #53024 — [[ROCm][CI] Stabilize MI355 FlyDSL MoE accuracy test](https://github.com/vllm-project/vllm/pull/53024)
- **作者**: AndreasKaratzas  **时间**: 2026-08-20 08:49 CST
- **标签**: rocm
- **摘要**: This PR stabilizes the two MI355 FlyDSL W4A16 MoE cases that failed in [AMD CI build 12250](https://buildkite.com/vllm/amd-ci/builds/12250/list). The test was introduced in #44400, and its test, kernel, and conversion code are unchanged between recent passing and failing builds; #41100 changed test …

### #53023 — [[CI] Fix MultiConnector accuracy test lifecycle](https://github.com/vllm-project/vllm/pull/53023)
- **作者**: AndreasKaratzas  **时间**: 2026-08-20 08:47 CST
- **标签**: kv-connector
- **摘要**: AMD CI build [#12250](https://buildkite.com/vllm/amd-ci/builds/12250#01a01b85-79bd-4cb8-9287-e5af327ff2ca) exposed a latent lifecycle race between the sequential normal and cross-layer test layouts. The first layout left its proxy on port 8192 while the API servers were still shutting down, so the n…

### #53022 — [[Model][EPLB] Add static expert maps for DeepSeek V4](https://github.com/vllm-project/vllm/pull/53022)
- **作者**: WoosukKwon  **时间**: 2026-08-20 08:32 CST
- **标签**: documentation, deepseek, DSv4, dflash
- **摘要**: ## Purpose  Add static expert placement for DeepSeek V4. A precomputed JSON map assigns each physical expert slot to a logical expert for every MoE layer, and the checkpoint loader places expert weights directly into those slots. The same path also remaps DSV4 router weights, correction bias, and ha…

### #53021 — [[Model] Remove unused DeepseekV32Indexer forward](https://github.com/vllm-project/vllm/pull/53021)
- **作者**: WoosukKwon  **时间**: 2026-08-20 08:29 CST
- **标签**: ready, deepseek
- **摘要**: ## Summary  Remove the unused `DeepseekV32Indexer.forward()` implementation and its now-unused `per_token_group_quant_fp8` import. Keep `self.indexer_op` construction unchanged.  ## Why  The NVIDIA non-compiled DSA path directly consumes the indexer's projections, normalization parameters, KV cache,…

### #53020 — [[Bugfix] Tie lm_head.weight for Nemotron Parse when checkpoint omits it](https://github.com/vllm-project/vllm/pull/53020)
- **作者**: aniskumar-nv  **时间**: 2026-08-20 08:15 CST
- **标签**: bug, multi-modality
- **摘要**: ## Purpose  Fixes #53019.  `nvidia/NVIDIA-Nemotron-Parse-2.0`'s checkpoint does not materialize a separate `lm_head.weight` tensor — its config sets `tie_word_embeddings=True` and the output head is tied to `decoder.embed_tokens.weight` instead (the same convention the model's Transformers reference…

### #53018 — [Add NIXL side channel advertize address for NAT'd deployments](https://github.com/vllm-project/vllm/pull/53018)
- **作者**: hcasalet  **时间**: 2026-08-20 07:34 CST
- **标签**: kv-connector
- **摘要**: ## Purpose VLLM_NIXL_SIDE_CHANNEL_HOST is currently used both to bind a NIXL connector instance's local ZMQ metadata handshake listener and as the address advertized to remote peers. There are not always the same address. A producer may need to bind to a local or internal address while being reached…

### #53017 — [[Model Runner V2][Spec Decode] Fix draft logits cache column stride in gumbel_sample](https://github.com/vllm-project/vllm/pull/53017)
- **作者**: TheEpicDolphin  **时间**: 2026-08-20 07:20 CST
- **标签**: speculative-decoding, ready, mrv2
- **摘要**: Currently, we are striding along columns in the draft logits cache using `vocab_size` instead of the actual stride value for the tensor. This is dangerous because in cases where `vocab_size != draft_logits.size(-1)`, we can end up writing to the wrong slot.
