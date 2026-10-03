# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-03 13:38 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 🛠️ Pull Requests (PR)
本周期内共有 5 个 PR，主要涉及 CI 同步、Bug 修复与新功能引入：

*   **CI 与代码同步**
    *   **#17870**: 推进 vLLM 的 main2main 同步轨道，将其从 v0.30.0 更新至最新的 vLLM main 分支（0928版本）。
    *   **#17869**: 提交空 PR，用于触发针对最新上游 main 分支的 CI 测试。
*   **Bug 修复**
    *   **#17868**: 修复转置操作符，使其支持 block-strided（块跨度）KV 缓存。此举解决了 QWEN3_235B_PD 服务器在 Nightly-A3 测试中出现的报错问题。
    *   **#17867**: 修复 `MooncakeConnectorV1` 中的 KV 传输问题，正确归一化组合注意力缓存，修正了维度 0 索引块的原有逻辑缺陷。
*   **新特性**
    *   **#17866**: 优化 KV Pool 机制。当请求在 KV 池中部分命中并在最终预填充阶段计算出更多 token 时，允许保存新计算的后缀，解决了原先元数据守卫抑制整个请求 SAVE 操作的问题。

### 🐛 Issues
*   本周期内暂无公开的 Issue 动态。

### 🚀 Release
*   本周期内暂无新版本发布。

---

## 🔀 Pull Requests

### #17870 — [[CI][Misc] mian2main vllm 0928](https://github.com/vllm-project/vllm-ascend/pull/17870)
- **作者**: zhangxinyuehfad  **时间**: 2026-10-03 11:36 CST
- **标签**: module:tests, module:ops, module:core, main2main
- **摘要**: ### What this PR does / why we need it?  The PR advances the main2main lane from vLLM v0.30.0 (tag commit `ced6857afa0ea7b2e3f0846a62e1394e90f15607`) to vLLM main `8cc9aa5ad350f6f80cf3bebb298d410803ff75a9` (installed as `0.30.0.dev`), adapting vllm-ascend to every upstream change landed after the v0…

### #17869 — [[CI] Empty PR on latest main](https://github.com/vllm-project/vllm-ascend/pull/17869)
- **作者**: LQDLove  **时间**: 2026-10-02 23:38 CST
- **摘要**: ### What this PR does / why we need it? Empty PR to trigger CI against the latest upstream main (`4ebb3090fc3c13c26557dcaae353882e70246649`). No files are changed: the commit tree is identical to base.  ### Does this PR introduce _any_ user-facing change? No.  ### How was this patch tested? CI on th…

### #17868 — [[BugFix][Ops] Support block-strided KV caches in transpose operator](https://github.com/vllm-project/vllm-ascend/pull/17868)
- **作者**: zhaochuang001  **时间**: 2026-10-02 23:09 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  Follow-up to #17867 and the block-major cache layout introduced in #14340.  The QWEN3_235B_PD server logs from [Nightly-A3 run 37008046214](https://github.com/vllm-project/vllm-ascend/actions/runs/37008046214/job/110857916893) report:  > Parameter vCache of t…

### #17867 — [[BugFix][KV Transfer] Normalize combined attention caches in MooncakeConnectorV1](https://github.com/vllm-project/vllm-ascend/pull/17867)
- **作者**: zhaochuang001  **时间**: 2026-10-02 19:47 CST
- **标签**: module:tests, ready-precise
- **摘要**: ### What this PR does / why we need it?  MooncakeConnectorV1 expects dimension 0 of each normalized cache tensor to index blocks. MRv1 can instead provide one attention tensor shaped `[2, num_kernel_blocks, block_size, num_heads, head_dim]`, including the block-major, non-contiguous layout relevant …

### #17866 — [[Feature[KV Pool] Save computed suffixes after partial pool hits](https://github.com/vllm-project/vllm-ascend/pull/17866)
- **作者**: bowgneo  **时间**: 2026-10-02 15:46 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  When a request hits 32 tokens in the KV pool and computes to 64 in its final prefill step, the current metadata guard suppresses SAVE for the whole request. The newly computed `[32,64)` suffix is never stored, and the default decode policy may leave it withou…
