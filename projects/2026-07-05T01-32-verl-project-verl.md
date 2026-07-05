# verl-project/verl — 动态追踪

> 生成时间: 2026-07-05 01:32 UTC

## AI 总结

### 🚀 Pull Request (PR)
*   **Ascend NPU 适配 vLLM 0.18.x 兼容性修复**：新增 PR #6928 与 #6929，主要解决在 NPU 场景下使用 vLLM 0.18.0 至 0.18.x 版本时的兼容性问题。
    *   **核心变更**：在 `RotaryEmbedding` 中禁用 `flash attention`。该修复逻辑与此前针对 vLLM >0.19.0 的适配保持一致，确保在昇腾 NPU 环境下模型能够正常运行。

### 🐛 Issue
*   近期暂无新增重要 Issue 动态。

### 🎉 Release
*   近期暂无新版本发布动态。

---

## 🔀 Pull Requests

### #6929 — [[fully_async] feat: Adapt vLLM >=0.18 and vLLM < 0.19 for Ascend NPU](https://github.com/verl-project/verl/pull/6929)
- **作者**: zhouhengan1211  **时间**: 2026-07-04 02:07 UTC
- **标签**: Ascend
- **摘要**: ### What does this PR do?  This PR mainly addresses compatibility issues with vLLM 0.18.0 in NPU scenarios.  Based on the modifications originally made in vllm>0.19.0, we encountered the same issue on version 0.18 as well. Therefore, it also needs to be disabled flash attention in RotaryEmbedding si…

### #6928 — [[fully_async] feat: Adapt vLLM >=0.18 and vLLM < 0.19 for Ascend NPU](https://github.com/verl-project/verl/pull/6928)
- **作者**: zhouhengan1211  **时间**: 2026-07-04 01:44 UTC
- **标签**: Ascend
- **摘要**: ### What does this PR do? This PR mainly addresses compatibility issues with vLLM 0.18.0 in NPU scenarios.  Based on the modifications originally made in vllm>0.19.0, we encountered the same issue on version 0.18 as well. Therefore, it also needs to be disabled flash attention in RotaryEmbedding sin…
