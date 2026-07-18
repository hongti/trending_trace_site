# verl-project/verl — 动态追踪

> 生成时间: 2026-07-18 09:03 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### .issue (问题)
* **#7088 优化 `attention_utils.py` 纯张量操作的硬件调度**
  作者指出 `index_first_axis` 和 `pad_input` 属于纯张量操作，理论上不需要针对不同硬件（如 NPU）做复杂的分支调度，建议简化或统一实现逻辑。

### 🛠 Pull Request (拉取请求)

**✨ 新特性**
* **#7085 扩展分片增量权重同步及 veomni FSDP2+EP 支持**
  将 `sharded delta weight sync` 从原本单一的 flat FSDP2 `Shard(0)` 快速路径，扩展支持至 **block placements**（包括 `Shard(k)`、多维网格及手动放置），并同时引入了 veomni FSDP2+EP 的支持。

**🐛 修复**
* **#7086 修复 Ascend NPU + vLLM sleep level2 模式下的 EP 精度问题**
  解决了在 Ascend NPU 上使用 vLLM sleep level2 及 Expert Parallelism (EP) 时 rollout 输出错误的问题，核心修复是在 L2 wake 后刷新 MoE 的通信状态。
* **#7087 修复 Ascend NPU MTP 在线权重更新问题**
  针对使用标准 checkpoint-format 加载路径时的 NPU MTP 权重更新进行修复，在 Ascend 上采用原生 reload 以满足图安全的在线重载需求。
* **#7089 改进 Megatron-Bridge ImportError 提示及 Dockerfile 测试**
  作为 #7071 的缓解措施（暂未彻底关闭该 Issue，彻底解决需维护者重建发布镜像），优化了缺少 `megatron-bridge` 时的报错指引，并增加了 Dockerfile 的冒烟测试。

**📝 文档重构**
* **#7091 重构 MLite DAPO 示例启动器指引**
  移除了过时且重复的 DeepSeek V4 Megatron-Lite DAPO 启动器，替换为指向官方维护的规范启动器的重定向，并保留了简单的失败存根提示。

### 📦 Release (发布版本)
* 近期无新版本发布信息。

---

## 🐛 Issues

### #7088 — [[util] attention_utils.py: index_first_axis and pad_input don't need hardware dispatch — they're pure tensor ops](https://github.com/verl-project/verl/issues/7088)
- **作者**: kahlun  **时间**: 2026-07-17 18:17 CST
- **摘要**: Looking at [verl/utils/attention_utils.py](https://github.com/verl-project/verl/blob/main/verl/utils/attention_utils.py#L20), it introduces different support method for attention utils,   the NPU implementations of index_first_axis and pad_input are pure Python using only torch and einops — no NPU-s…

## 🔀 Pull Requests

### #7091 — [[doc] refactor: point MLite DAPO example to canonical launcher](https://github.com/verl-project/verl/pull/7091)
- **作者**: ISEEKYAN  **时间**: 2026-07-17 23:56 CST
- **摘要**: ## Summary  - replace the stale, duplicated DeepSeek V4 Megatron-Lite DAPO launcher with a redirect to the canonical launcher maintained alongside Megatron-Lite - keep a small executable failure stub so existing callers receive an explicit migration URL instead of silently running an obsolete recipe…

### #7089 — [[megatron, model, doc] fix: improve Megatron-Bridge ImportError guidance and Dockerfile smoke](https://github.com/verl-project/verl/pull/7089)
- **作者**: chethanuk  **时间**: 2026-07-17 20:57 CST
- **摘要**: ## Summary  Mitigation-only PR for #7071. **Does not close #7071** — published GPU tags still lack `megatron-bridge`; the cure is a maintainer image rebuild/republish.  ### Problem Users of published tags such as `verlai/verl:sgl0512.dev2` hit `ModuleNotFoundError: megatron.bridge`. The Dockerfiles …

### #7087 — [[rollout, vllm] fix: use native reload for NPU MTP weight updates](https://github.com/verl-project/verl/pull/7087)
- **作者**: Mengyuyang  **时间**: 2026-07-17 17:07 CST
- **摘要**: ### What does this PR do?  This PR fixes NPU MTP online weight updates when the rollout model uses the standard checkpoint-format weight-loading path.  On Ascend, graph-safe online reload requires two conditions:  1. The target model and the MTP drafter must be updated within the same native layerwi…

### #7086 — [[vllm, rollout, worker] fix: refresh Ascend MoE comm state after L2 wake to fix ep+sleep level2 precision issue](https://github.com/verl-project/verl/pull/7086)
- **作者**: zgh-99  **时间**: 2026-07-17 16:44 CST
- **标签**: Ascend
- **摘要**: [vllm, rollout, worker] fix: refresh Ascend MoE comm state after L2 wake to fix ep+sleep level2 precision issue  ### What does this PR do?  Fixes incorrect rollout outputs on Ascend NPU + vLLM sleep level 2 (L2) + expert parallel (EP > 1) in hybrid (colocated) training.  After engine.sleep(level=2) …

### #7085 — [[ckpt,veomni] feat: block-placement sharded delta sync + veomni FSDP2+EP support](https://github.com/verl-project/verl/pull/7085)
- **作者**: ChangyiYang  **时间**: 2026-07-17 10:46 CST
- **摘要**: ## What does this PR do?  Extends the sharded delta weight sync (`delta_sharded`, #6974) beyond the flat FSDP2 `Shard(0)` fast path to **block placements** (`Shard(k)`, multi-Shard-dim meshes, and manual splits), and wires **veomni FSDP2+EP** into it end to end. Supersedes #7080 (same backend-agnost…
