# verl-project/verl — 动态追踪

> 生成时间: 2026-07-28 09:01 CST

## AI 总结

# verl-project/verl 近期动态摘要

---

## 📋 Issue

| # | 标题 | 要点 |
|---|------|------|
| #7167 | NCCL checkpoint engine broadcast 优化 | 建议将同节点的 trainer GPU 加入广播组，以优化权重广播性能 |
| #7166 | FSDP engine 内存泄漏 | `forward_step` 返回的 `model_output` 未 detach，导致跨 micro-batch 的 autograd 计算图累积，造成 **GB 级内存泄漏** |
| #7163 | 过长 prompt 过滤时 SIGTERM 处理异常 | 多进程数据过滤场景下 SIGTERM 信号处理不当，导致 Ray worker 异常 |

---

## 🔀 Pull Request

### 新特性 / 功能增强
- **#7171** — 复现 TMEM LoCoMo 实验（DFlash），实现论文中 rank-6 LoRA online-SFT 路径，基于 `arXiv:2606.04536`
- **#7162** — 性能优化：避免每次 `tqbridge` 调用都创建新线程/事件循环，并升级 TransferQueue 依赖至 0.1.9

### 重要修复
- **#7169** — 修复 `transformers >= 5.4.0` 上 `is_flash_attn_greater_or_equal_2_10` 被移除导致的 **ImportError**，影响 Qwen2-VL / GLM4V 等模型导入
- **#7168** — 修复 vLLM < 0.16 不支持 `weights_iterator` 参数导致权重同步失败，**降级为桶式权重同步**
- **#7165** — 修复 FSDP2 路径未遵守 Hugging Face 的 `_keep_in_fp32_modules` 声明，导致本应保持 fp32 的模块被错误降精度
- **#7164** — 修复 Ray worker 中过长 prompt 过滤导致的 **挂起(hang)** 问题（对应 Issue #7163）
- **#7170** — Rollout 启动时对无效 replica 拓扑 **快速失败**，避免在分布式启动后才报错

### 重构 / 改进
- **#7161** — 将 `unfuse_moe_params` 从 vLLM 移至 FSDP 后端，职责更清晰
- **#7160** — 修复 placement group 按 IP **字符串排序**导致 rank 顺序与节点顺序不一致的问题，改为数值排序
- **#7159** — 更新 FSDP engine 单元测试 mock，使其与当前 `_gradient_sync_context` 接口一致

---

## 🚀 Release

> 本期无新版本发布。

---

### 📌 总结

本期动态以**修复为主**，重点关注：
1. **兼容性**：`transformers >= 5.4.0` 和 `vLLM < 0.16` 两处兼容性修复，影响面较广
2. **内存与稳定性**：FSDP 内存泄漏（#7166）和 Ray worker 挂起（#7164）是高优先级问题
3. **精度正确性**：FSDP2 未保留 fp32 模块（#7165）可能导致训练结果异常，建议尽快合入

---

## 🐛 Issues

### #7167 — [[ckpt]: NCCL checkpoint engine broadcast optimization](https://github.com/verl-project/verl/issues/7167)
- **作者**: parinayc20  **时间**: 2026-07-27 23:23 CST
- **摘要**: # Feature Request  Adding all the trainer GPUs (or possibly just the trainer GPUs located on the same node as the master GPU) to the broadcast group when broadcasting updated weights from the master trainer rank to the rollout ranks results in higher transfer bandwidth and lower end-to-end latency i…

### #7166 — [FSDP engine: forward_step stores model_output undetached, pinning autograd graphs across micro-batches (multi-GB leak)](https://github.com/verl-project/verl/issues/7166)
- **作者**: xuefei-wang  **时间**: 2026-07-27 23:17 CST
- **摘要**: ## Summary  `FSDPEngineWithLMHead.forward_step` returns `model_output` **undetached** in the dict that `forward_backward_batch` accumulates into `output_lst`. For any policy loss that does not backward through *every* tensor stored in `model_output`, the untraversed entries keep their autograd graph…

### #7163 — [filter_overlong_prompts: SIGTERM handling issue during multi-process data filtering](https://github.com/verl-project/verl/issues/7163)
- **作者**: mikequan0425  **时间**: 2026-07-27 22:00 CST
- **标签**: bug
- **摘要**: ### System Info  ----------Python Info---------- Version      : 3.11.9 Compiler     : GCC 12.2.0 Build        : ('main', 'May 11 2026 12:00:44') Arch         : ('64bit', '') ------------Pip Info----------- Version      : 26.1.2 Directory    : /usr/local/lib/python3.11/site-packages/pip vllm         …

## 🔀 Pull Requests

### #7171 — [[sglang, model, data] feat: reproduce TMEM LoCoMo with DFlash](https://github.com/verl-project/verl/pull/7171)
- **作者**: hagiss  **时间**: 2026-07-28 08:55 CST
- **摘要**: ### What does this PR do?  This PR adds a reproducible implementation of the pre-RL TMEM LoCoMo experiment from Table 1 of arXiv:2606.04536.  It implements the paper-locked online-SFT path: rank-6 LoRA on the last four FFN gate/up/down projections, SVD initialization with frozen A and trainable B, p…

### #7170 — [[rollout] fix: fail fast on invalid replica topologies](https://github.com/verl-project/verl/pull/7170)
- **作者**: MagellaX  **时间**: 2026-07-28 06:12 CST
- **摘要**: ### What does this PR do?  Rollout server setup currently floor-divides the available GPU pool by each replica's footprint. Invalid configurations can therefore reach distributed startup before failing, create zero replicas, or silently leave GPUs unused.  This PR validates the topology before const…

### #7169 — [[model] fix: import crash on transformers >= 5.4.0 (is_flash_attn_greater_or_equal_2_10 removed)](https://github.com/verl-project/verl/pull/7169)
- **作者**: thuwyh  **时间**: 2026-07-28 05:15 CST
- **摘要**: ### What does this PR do?  Fixes an `ImportError` that breaks `import verl.trainer.sft_trainer` (and anything else importing `verl/models/transformers/qwen2_vl.py` or `glm4v.py`) on **transformers >= 5.4.0**:  ``` File "verl/models/transformers/qwen2_vl.py", line 29, in <module>     from transformer…

### #7168 — [[rollout, vllm] fix: fall back to bucketed weight sync when vLLM reload_weights lacks weights_iterator](https://github.com/verl-project/verl/pull/7168)
- **作者**: layahaasini  **时间**: 2026-07-28 04:00 CST
- **摘要**: ### What does this PR do?  Fixes #7130.  The standard weight sync calls `reload_weights(weights_iterator=..., is_checkpoint_format=True)`, but that kwarg only exists in vLLM >= 0.16. On older versions — including 0.12.0, the newest the `.[vllm]` extra installs — `reload_weights()` takes no arguments…

### #7165 — [[fsdp] fix: preserve Hugging Face fp32-keep modules in FSDP2](https://github.com/verl-project/verl/pull/7165)
- **作者**: chengcuiping  **时间**: 2026-07-27 22:37 CST
- **摘要**: <html><head></head><body><h3>What does this PR do?</h3><p>Fixes #7092.  verl's FSDP2 path did not honor Hugging Face's `_keep_in_fp32_modules` and `_keep_in_fp32_modules_strict` declarations. As a result, modules that Hugging Face intentionally keeps in FP32 could be silently reduced to a lower prec…

### #7164 — [[data, ray] fix: prevent overlong prompt filtering hang in Ray workers](https://github.com/verl-project/verl/pull/7164)
- **作者**: mikequan0425  **时间**: 2026-07-27 22:02 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  fix issue https://github.com/verl-project/verl/issues/7163  The fix logic is as follows： - Enabled only when num_proc > 1, and the …

### #7162 — [[perf] Prevent creating new threads/event loop for each `tqbridge` call & update TQ version](https://github.com/verl-project/verl/pull/7162)
- **作者**: 0oshowero0  **时间**: 2026-07-27 21:11 CST
- **摘要**: ### What does this PR do?  - Prevent creating new thread & event loop for each tqbridge decorated call https://github.com/verl-project/verl/pull/7157 - Update TransferQueue dependency to 0.1.9 (release detail: https://github.com/Ascend/TransferQueue/releases/tag/v0.1.9)  ### Checklist Before Startin…

### #7161 — [[fsdp] refactor: move unfuse_moe_params to FSDP backend](https://github.com/verl-project/verl/pull/7161)
- **作者**: wuxibin89  **时间**: 2026-07-27 19:13 CST
- **摘要**: ### What does this PR do?  Move unfuse_moe_params from vllm to FSDP backend

### #7160 — [fix(ray): sort placement groups by numeric IP instead of string, so rank order follows node order](https://github.com/verl-project/verl/pull/7160)
- **作者**: weich97  **时间**: 2026-07-27 17:40 CST
- **摘要**: ### Problem  `sort_placement_group_by_node_ip` orders placement groups — and therefore global ranks — by the **string** form of each node's IP (`verl/single_controller/ray/base.py`):  ```python return sorted(pgs, key=lambda pg: pg_ip[pg.id]) ```  Because the last octet is not zero-padded, string ord…

### #7159 — [[fsdp, tests] fix: update FSDP engine test mock](https://github.com/verl-project/verl/pull/7159)
- **作者**: Tjh-UKN  **时间**: 2026-07-27 16:16 CST
- **摘要**: ### What does this PR do?  Updates the FSDP CPU unit-test fixture to match the current `FSDPEngine.forward_backward_batch` contract. The production method enters `_gradient_sync_context(...)` for each backward micro-batch; the `SimpleNamespace` test double did not provide that method, causing the CP…
