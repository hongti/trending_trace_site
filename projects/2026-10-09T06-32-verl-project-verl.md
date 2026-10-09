# verl-project/verl — 动态追踪

> 生成时间: 2026-10-09 14:32 CST

## AI 总结

以下是 **verl-project/verl** 仓库最近动态的中文摘要：

### 🐛 Issue 动态
1. **全异步 Megatron 性能优化请求 (#8160)**：指出在 Router Replay (R3) 场景下，`routed_experts` 是数据传输中的最大列。建议在全异步 TransferQueue 路径下，按流水线并行 (PP) 阶段仅传输该阶段自身层所需的数据，以降低开销。
2. **FSDP 模型加载 Bug (#8154)**：发现在 FSDP 加载 BF16 模型时，会将 FP32 的 RoPE `inv_freq` 量化，导致最终计算的对数概率发生改变。

### 🔀 PR 动态

**新特性与功能增强**
- **Rollout Trace 支持 (#8158)**：新增了对 rollout 追踪功能的支持。
- **TPU GRPO 支持 (#8155)**：通过 `verl-hardware-plugin` 启用 TPU 硬件支持，结合 TorchTitan 训练与 vLLM 部署，可在 TPU 上运行 Qwen3-4B 的 GRPO 任务。
- **Torchtitan 填充支持 (#8150)**：为 TorchTitan 后端新增了 `pad_to_length` 功能支持。

**Bug 修复与优化**
- **工具状态保持修复 (#8156)**：优化了 `BaseTool` 实例的生命周期，使其在单次轨迹期间只创建一次并结束时才释放。这使得工具能在多轮对话中保持状态。
- **LoRA 空集合报错修复 (#8152)**：修复了 FSDP 下 LoRA 参数集合为空时，导致 vLLM 适配器注册抛出 `StopIteration` 异常并使接收方无响应退出的问题，现在会快速失败。
- **PyTorch Profiler 扩展 (#8151)**：改进了现有的 PyTorch Profiler，使其能够支持 CUDA 以外的其他 PyTorch 兼容硬件设备。

**测试与 CI/CD**
- **集群与 CI 维护 (#8161, #8157, #8159)**：因 A2 集群暂时不可用，将 nightly ppo 测试回退至 A3 集群；修复了 Ascend A2 CI 的模型路径问题；并新增从主分支同步 Ascend Quay 仓库描述的功能。
- **聊天模板测试 (#8153)**：补充了针对包含推理内容 (`reasoning_content`) 的聊天模板轨迹的测试覆盖。

### 🚀 Release 动态
- 本次周期内无新的 Release 版本发布。

---

## 🐛 Issues

### #8160 — [[fully_async, megatron, perf] Narrow routed_experts transfer to each PP stage's own layers under router replay (R3)](https://github.com/verl-project/verl/issues/8160)
- **作者**: weiziyoung  **时间**: 2026-10-09 11:24 CST
- **摘要**: ### Feature request  Under router replay (R3), routed_experts is the single largest column in the batch ([tokens, num_layers, topk], uint8). On the TransferQueue (main_ppo_sync.py / fully-async) path, every rank fetches the full tensor across all layers, even though each pipeline-parallel (PP) stage…

### #8154 — [[Bug] FSDP BF16 model loading quantizes FP32 RoPE inv_freq and changes log probabilities](https://github.com/verl-project/verl/issues/8154)
- **作者**: duerwuyi  **时间**: 2026-10-08 23:54 CST
- **标签**: bug
- **摘要**: ### System Info  - verl: 0.9.1 - Python: 3.12.13 - PyTorch: 2.13.0+cu129 - Transformers: 5.9.0 - CUDA runtime: 12.9 - GPU: NVIDIA RTX PRO 6000 Blackwell Workstation Edition - FSDP: single GPU, world_size=1 (NO_SHARD) - Configuration: `forward_only=true`, explicit `model_dtype=bfloat16` - Mixed preci…

## 🔀 Pull Requests

### #8161 — [[ci] chore: use a3-8 runner for nightly ppo qwen3-8b fsdp vllm test](https://github.com/verl-project/verl/pull/8161)
- **作者**: lxb007981  **时间**: 2026-10-09 11:48 CST
- **摘要**: ### What does this PR do?  A2 cluster is temporarily unavailable. Rolling back to A3 cluster for nightly ppo qwen3-8b fsdp vllm test.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {description}`…

### #8159 — [[ci] feat: sync Ascend Quay repository description from main](https://github.com/verl-project/verl/pull/8159)
- **作者**: lxb007981  **时间**: 2026-10-09 11:06 CST
- **摘要**: ### What does this PR do?  Add a version-controlled description for quay.io/ascend/verl.  Sync the description on relevant main branch changes.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `[{modules}] {type}: {des…

### #8158 — [verl supprot rollout trace](https://github.com/verl-project/verl/pull/8158)
- **作者**: mengchengTang  **时间**: 2026-10-09 10:41 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #8157 — [Fix model path on ascend a2 ci](https://github.com/verl-project/verl/pull/8157)
- **作者**: lxb007981  **时间**: 2026-10-09 09:59 CST
- **标签**: Ascend
- **摘要**: ### What does this PR do?  Test ascend ci.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`, `sg…

### #8156 — [[tool, rollout] fix: create BaseTool instances once per trajectory and release them when it ends](https://github.com/verl-project/verl/pull/8156)
- **作者**: nataliekung  **时间**: 2026-10-09 07:37 CST
- **摘要**: ### What does this PR do?  `ToolAgentLoop._call_tool` runs `create`, `execute` and `release` for every tool call. A `BaseTool` therefore cannot keep state from one turn to the next, although `BaseTool.create` is documented as creating "a tool instance for a trajectory" and `docs/sglang_multiturn/mul…

### #8155 — [[hardware] feat: enable TPU GRPO with TorchTitan and vLLM via verl-hardware-plugin](https://github.com/verl-project/verl/pull/8155)
- **作者**: lixali  **时间**: 2026-10-09 01:16 CST
- **摘要**: ### What does this PR do?  Adds `examples/tpu/grpo/run_qwen3_4b_torchtitan.sh` to run Qwen3-4B-Base GRPO on GSM8K using TorchTitan training, vLLM rollout, and `verl-hardware-plugin`.  Builds on [PR #50](https://github.com/jialei777/verl-upstream/pull/50).    ### API and Usage Example  On an existing…

### #8153 — [[tool] test: cover reasoning-bearing chat-template trajectories](https://github.com/verl-project/verl/pull/8153)
- **作者**: JiahaoTanXX  **时间**: 2026-10-08 21:33 CST
- **摘要**: ### What does this PR do?  Address the `reasoning_content` coverage TODO in `scripts/chat_template_checker.py`. The current text fixtures contain no separate assistant reasoning, and the single-turn adapter can only construct `role` and `content`.  Add three registered text trajectories: a single-tu…

### #8152 — [[fsdp] fix: fail fast when LoRA parameter collection is empty](https://github.com/verl-project/verl/pull/8152)
- **作者**: lxb007981  **时间**: 2026-10-08 21:10 CST
- **摘要**: ### What does this PR do?  An empty LoRA state dict reaches vLLM's adapter registration, where v0.23.0 raises StopIteration while accessing the first adapter weight. The receiver then exits without acknowledging the final weight bucket, leaving the sender blocked waiting for a reply.  Assert that co…

### #8151 — [[hardware, profile] fix: improve existing pytorch profiling to support other pytorch supported device](https://github.com/verl-project/verl/pull/8151)
- **作者**: kahlun  **时间**: 2026-10-08 20:39 CST
- **摘要**: ### What does this PR do?  `get_torch_profiler` knew exactly one device activity, `ProfilerActivity.CUDA`, and `TorchProfilerToolConfig` accepted exactly one device keyword, `"cuda"`. On an accelerator that is not CUDA there was then no way to record device kernels: the default `contents=[]` asks fo…

### #8150 — [[trainer] feat: add pad_to_length support for torchtitan backend](https://github.com/verl-project/verl/pull/8150)
- **作者**: wuxibin89  **时间**: 2026-10-08 20:21 CST
- **摘要**: ### What does this PR do?  As comment on https://github.com/verl-project/verl/pull/8098#discussion_r4216692582, add `pad_to_length` support for torchtitan backend.  ```yaml actor_rollout_ref:   actor:     torchtitan:       pad_to_length: true       pad_to_length_bucket: 512 ```
