# vllm-project/vllm — 动态追踪

> 生成时间: 2026-08-25 10:05 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 最近动态的中文摘要：

### 📝 Issue 摘要
* **SageMaker 集成导致日志级别异常 (#53655)**：导入 `vllm.entrypoints.openai.api_server` 会连带导入 SageMaker 集成，这将共享的根 vllm handler 日志级别修改为 ERROR，导致 vLLM 启动日志被意外抑制。

### 🔧 Pull Request 摘要

**🐛 Bug 修复**
* **Gemma4 工具调用解析修复 (#53657)**：修复解析器无法处理 `call:name(...)` 括号格式的问题（此前仅支持 `{...}`），防止工具名称损坏及后续调用被错误吸收。
* **LoRA 路由错误修复 (#53654)**：修复开启 `enable_tower_connector_lora` 时，TOWER/CONNECTOR 类型的 LoRA 映射被错误路由至语言模型 punica wrapper 的问题。
* **推测解码提议方法修复 (#53652)**：确保选择推测解码提议方法时，正确遵循 speculators 模型自带的 `default_proposal_method` 必填字段。
* **lm_head 类型转换修复 (#53651)**：支持在 `head_dtype` 转换路径中处理携带 `UnquantizedLinearMethod` 的未量化 lm_head（此前仅支持 Embedding 方法）。
* **KV Cache Offload 关键竞态修复 (#53648)**：修复高并发下 V1 KV cache offloading 的两个严重问题：调度器块边界竞态条件，以及请求完成时的 tiering KeyError 崩溃。

**🚀 性能与新特性**
* **Blackwell 架构性能大幅优化 (#53649)**：为 Blackwell 架构自动调优（Autotune）批量不变性 Triton 内核，端到端（E2E）延迟降低 **33.6%**。
* **推测解码支持 PCP-sharded MTP (#53653)**：在 MRV2 PCP 路径中增加对单模块 MTP（多 Token 预测）预填充的支持，同时保留了 PCP 的长上下文扩展能力。
* **Encoder Config 标准化 (#53656)**：为仅生产者（producer-only）的 EC 实例自动启用 `mm_encoder_only`，统一使用 `is_mm_encoder_only` 作为访问器，并拒绝不兼容的配置。

**📚 文档与测试**
* **扩展验证模型列表 (#53650)**：将 `ibm-granite/granite-3.1-3b-a800m-instruct` (GraniteMoeForCausalLM) 添加至批量不变性验证通过的模型列表中。
* **推测解码测试改进 (#53647)**：将测试的 GPU 显存门限从 32GiB 降至 16GiB 以兼容 CI 环境，并改用 token ID 对比 logprobs 以正确处理概率平局情况。

### 📦 Release 摘要
* 近期无新版本发布信息。

---

## 🐛 Issues

### #53655 — [[Bug]: model_hosting_container_standards changes a shared root/vllm handler to ERROR and suppresses vLLM startup logs](https://github.com/vllm-project/vllm/issues/53655)
- **作者**: wenjinhust  **时间**: 2026-08-25 09:26 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text vLLM: 0.18.1.dev44+gcfae1bf2d Python: 3.11.14 model-hosting-container-standards: 0.1.16 Platform: reproduced with vllm-ascend/NPU, but the root cause is platform-independent ```  </de…

## 🔀 Pull Requests

### #53657 — [[Bugfix] Handle parenthesized Gemma4 tool calls](https://github.com/vllm-project/vllm/pull/53657)
- **作者**: taneem-ibrahim  **时间**: 2026-08-25 09:51 CST
- **标签**: bug, tool-calling
- **摘要**: ## Purpose  Fixes #53642. Gemma4 may emit tool calls as `call:name(...)`, but the parser only accepted `{...}`. It remained in `TOOL_NAME`, corrupting the name and absorbing later calls. This adds parenthesis handling and a safe tool-end transition.  ## Reproducer  ```python output = '<|tool_call>ca…

### #53656 — [[Config][EC] Normalize producer-only encoder config](https://github.com/vllm-project/vllm/pull/53656)
- **作者**: gty111  **时间**: 2026-08-25 09:46 CST
- **摘要**: ## Summary  - Automatically enable `mm_encoder_only` for EC producer-only instances. - Keep `is_mm_encoder_only` as the normalized `VllmConfig` accessor. - Reject producer-only EC configuration for non-multimodal models.  ## Tests  - `.venv/bin/python -m pytest tests/config/test_multimodal_config.py…

### #53654 — [[Bugfix] Don't route TOWER/CONNECTOR LoRA mappings to the language wrapper](https://github.com/vllm-project/vllm/pull/53654)
- **作者**: sikso1892  **时间**: 2026-08-25 09:23 CST
- **标签**: bug
- **摘要**: ## Summary  With `enable_tower_connector_lora` on, `_set_adapter_mapping` routes a TOWER or CONNECTOR `LoRAMapping` to the language-model punica wrapper whenever the model's `get_mm_mapping()` has no matching module prefixes — the dispatch falls through to the final `else`. Since `_execute_mm_encode…

### #53653 — [[Spec Decode] Support PCP-sharded MTP prefill](https://github.com/vllm-project/vllm/pull/53653)
- **作者**: GirasoleY  **时间**: 2026-08-25 08:32 CST
- **标签**: speculative-decoding, mrv2
- **摘要**: <!-- markdownlint-disable -->  ## Purpose  Related RFC: #25749  Add single-module MTP support to the MRV2 PCP path while preserving PCP's long-prefill scaling:  - keep both the target prefill and the first MTP draft pass PCP-sharded - restore target and draft outputs to global sequence order only at…

### #53652 — [[Bugfix] Honor default_proposal_method when selecting the speculators proposal method](https://github.com/vllm-project/vllm/pull/53652)
- **作者**: MaCoredroid  **时间**: 2026-08-25 07:59 CST
- **标签**: bug
- **摘要**: ## Purpose  The speculators schema names the active proposal method via `default_proposal_method` — a required field with a pydantic membership validator, carried by published speculators checkpoints — but vLLM never reads it (`grep -rn default_proposal_method vllm/` → zero hits outside one test fix…

### #53651 — [[Bugfix] Accept an unquantized-linear lm_head in the head_dtype cast path](https://github.com/vllm-project/vllm/pull/53651)
- **作者**: MaCoredroid  **时间**: 2026-08-25 07:59 CST
- **标签**: bug, quantization
- **摘要**: ## Purpose  An lm_head *excluded* from quantization (e.g. a ModelOpt `exclude_modules` config) carries `UnquantizedLinearMethod` rather than `UnquantizedEmbeddingMethod`. Both hold plain, unpacked weights, but two sites accept only the embedding method:  - the `head_dtype` cast path in `LogitsProces…

### #53650 — [[Doc] Add Granite 3.1 MoE to batch invariance tested models](https://github.com/vllm-project/vllm/pull/53650)
- **作者**: ShengleiFu  **时间**: 2026-08-25 07:37 CST
- **标签**: documentation
- **摘要**: ## Purpose  Add `ibm-granite/granite-3.1-3b-a800m-instruct` to the list of models validated for batch invariance.  This expands model coverage to `GraniteMoeForCausalLM`, which uses 40 experts with top-8 routing.  Related to #27433.  ## Duplicate check  I searched the open issues and pull requests a…

### #53649 — [[Perf] Autotune batch invariance triton kernel in blackwell, 33.6% E2E latency reduction](https://github.com/vllm-project/vllm/pull/53649)
- **作者**: yewentao256  **时间**: 2026-08-25 07:17 CST
- **标签**: ready
- **摘要**: ## Purpose  Autotune batch invariance triton kernel in blackwell and get perf  ## Test  `VLLM_BATCH_INVARIANT=1 vllm bench latency   --model=Qwen/Qwen3-1.7B`  ```bash # now  Avg latency: 0.5762738614963988 seconds 10% percentile latency: 0.5759288525208831 seconds 25% percentile latency: 0.576097038…

### #53648 — [[Bugfix][KV Offload] Fix offloading scheduler chunk bounds race and tiering KeyError on request completion](https://github.com/vllm-project/vllm/pull/53648)
- **作者**: yangspirit  **时间**: 2026-08-25 07:15 CST
- **标签**: bug, kv-connector
- **摘要**: ## Purpose This PR fixes two critical race conditions / crashes in vLLM V1 KV cache offloading under high concurrency: 1. **Offloading Scheduler Block Bounds Race Condition (`scheduler.py`)**:    - **Root Cause**: In `OffloadingConnectorScheduler._build_store_jobs()`, `req_status.storable_chunks()` …

### #53647 — [[Bugfix] Make speculative-decode logprob test tie-aware](https://github.com/vllm-project/vllm/pull/53647)
- **作者**: sudhirpol522  **时间**: 2026-08-25 07:14 CST
- **标签**: bug
- **摘要**: ## Purpose  Addresses #53620.  This test-only change:  * Lowers the GPU gate from 32 GiB to 16 GiB so the test can run on the `h200_18gb` CI slice. * Compares logprobs per position by token ID instead of dictionary order. * Requires identical sampled tokens while allowing valid rank and top-k member…
