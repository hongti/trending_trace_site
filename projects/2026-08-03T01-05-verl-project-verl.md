# verl-project/verl — 动态追踪

> 生成时间: 2026-08-03 09:05 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl** 近期动态的中文摘要：

### 🐛 Issue (问题)
*   **#7226 Megatron 模型合并器导致各 rank 重复上传相同 checkpoint**
    *   **重点**：这是一个分布式控制流 Bug。在使用 Megatron 模型合并代码路径时，每个 rank 都会上传相同的 checkpoint 文件，导致冗余上传及潜在的资源浪费/冲突。该问题与具体的 GPU 或模型配置无关。

### 🔧 Pull Request (拉取请求)
*   **#7228 修复 `pyarrow` 兼容性问题：锁定 `datasets` 最低版本**
    *   **重点**：修复依赖冲突。此前 `datasets` 库未设定最低版本限制，导致 pip 可能会将旧版的 `datasets` 与新版 `pyarrow` 组合安装，从而引发兼容性报错。此 PR 在 `setup.py` 等配置文件中固定了 `datasets` 的最低版本。
*   **#7227 新增 vLLM consumer 支持 `delta_sharded` 权重同步**
    *   **重点**：重要新特性。为 vLLM rollout 模块新增了 consumer，以支持此前 PR #6974 引入的 `delta_sharded` 检查点引擎格式。该功能可实现增量分片权重的同步，有助于提升大模型训练/推理时的权重更新效率。

### 🚀 Release (发布)
*   近期无新版本发布动态。

---

## 🐛 Issues

### #7226 — [[Bug] Megatron model merger uploads the same checkpoint from every rank](https://github.com/verl-project/verl/issues/7226)
- **作者**: KANIKIG  **时间**: 2026-08-02 11:19 CST
- **摘要**: ### System Info  Current `verl-project/verl` Megatron model-merger code path. This is a distributed control-flow issue and is not specific to a particular GPU, Python, or model configuration.  ### Information  - The problem arises when using the built-in Megatron model merger with `hf_upload` enable…

## 🔀 Pull Requests

### #7228 — [[misc] fix: pin datasets minimum version for pyarrow compatibility](https://github.com/verl-project/verl/pull/7228)
- **作者**: PoojanTa  **时间**: 2026-08-03 05:26 CST
- **摘要**: ### What does this PR do?  `datasets` is currently unpinned in `setup.py`, `requirements.txt`, and `requirements-npu.txt`. That lets pip resolve an old `datasets` alongside a modern `pyarrow`, which produces an environment that fails as soon as verl touches training data: `verl.utils.dataset.rl_data…

### #7227 — [[ckpt, rollout, vllm] feat: add vLLM consumer for delta-sharded weight sync](https://github.com/verl-project/verl/pull/7227)
- **作者**: ShuoleiWang  **时间**: 2026-08-02 13:21 CST
- **摘要**: ### What does this PR do?  This PR adds a vLLM rollout consumer for the `delta_sharded` checkpoint-engine wire format introduced by [verl #6974](https://github.com/verl-project/verl/pull/6974).  `delta_sharded` already reduces trainer-side materialization, gather traffic, and cross-node payloads by …
