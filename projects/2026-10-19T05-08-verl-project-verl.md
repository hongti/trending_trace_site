# verl-project/verl — 动态追踪

> 生成时间: 2026-10-19 13:08 CST

## AI 总结

以下是 **verl-project/verl** 仓库近期动态的中文摘要：

### 📝 Issue (问题)
*   **#8281 Rollout 共享提示词上限忽略截断策略**
    *   作者: Jackmeson1
    *   摘要: 报告了在处理长文本提示词时，系统未遵循文档中记录的 `data.truncation` 策略的问题。当前的共享提示词上限逻辑会导致不正确的截断。

### 🚀 Pull Request (拉取请求)

**新特性 / 功能增强：**
*   **#8279 新增 Ascend 硬件上的 FAPO Megatron 训练支持** (作者: xldeng-chn)
    *   在华为 Ascend 环境下引入核心修改，支持使用 Megatron actor、vLLM-Ascend rollout 以及共置的 GenRM 进行 Qwen2.5-Math-7B + FAPO-GenRM-4B 模型的训练。
*   **#8274 将 VeOmni 提升为基于 uv 的一等训练后端** (作者: ETOgaosion)
    *   在通用的 uv-managed CUDA 13 技术栈上，将 VeOmni 正式接入为一等训练后端，修复了代理网络过滤后的开放镜像构建问题，并增加了 GPU CI 覆盖。
*   **#8275 添加 Awesome-Perception-Aware-RLVR 到项目列表** (作者: zsxm1998)
    *   在 README 的 "Awesome Projects Built with verl" 列表中新增了一个推荐项目。

**Bug 修复：**
*   **#8280 修复 TPU TorchTitan 脚本中的并行尺寸计算问题** (作者: jialei777)
    *   修复了在张量并行大小（TP > 1）时，TPU 示例脚本无法自动计算数据并行分片大小（DP shard size）的问题。
*   **#8278 修复 CI 中缺少 policy_loss 字段导致的报错** (作者: ETOgaosion)
    *   解决了在分数居中和 Liger TP 融合交叉熵代码合入后，由于轻量级配置缺少 `policy_loss` 字段导致的 `cpu_unit_tests` 测试失败问题。
*   **#8277 修复激活卸载中元组输出绑定的 Bug** (作者: Eliaaazzz)
    *   将所有可微分的顶层 tuple 输出通过相同的 activation-offload group-commit autograd 节点进行路由，修复了之前的处理漏洞。
*   **#8276 修复 search_r1 subEM 奖励函数匹配逻辑** (作者: flyingdoubleG)
    *   修正了 `subem_check` 中的子字符串匹配问题，改为全词匹配并忽略空的 golden answers，避免对不包含正确答案的预测给出奖励。

### 📦 Release (发布版本)
*   本次提供的数据中未包含版本发布（Release）信息。

---

## 🐛 Issues

### #8281 — [[rollout] Shared prompt cap ignores documented data.truncation policies](https://github.com/verl-project/verl/issues/8281)
- **作者**: Jackmeson1  **时间**: 2026-10-19 12:42 CST
- **摘要**: ### System Info  verl 0.10.0.dev, commit 4b26c788d41ce932ce7706c5ee8e249ab52f4c51. Python 3.12.15, Linux/WSL2. CPU reproduction; no model or rollout server.  ### Information  Overlong text prompts are left-truncated with a warning regardless of `data.truncation`, including `error`, `right`, and `mid…

## 🔀 Pull Requests

### #8280 — [[hardware, doc] fix: auto-compute DATA_PARALLEL_SHARD_SIZE when TENSOR_PARALLEL_SIZE > 1 in TPU TorchTitan scripts](https://github.com/verl-project/verl/pull/8280)
- **作者**: jialei777  **时间**: 2026-10-19 12:13 CST
- **摘要**: ## Summary  Update the TPU TorchTitan example scripts (`examples/tpu/grpo/run_qwen3_4b_torchtitan.sh`, `examples/tpu/grpo/run_qwen3_0_6b_torchtitan.sh`, and `examples/tpu/sft/run_qwen3_0_6b_torchtitan.sh`) so `DATA_PARALLEL_SHARD_SIZE` defaults to `$((TOTAL_TRAINER_CHIPS / TENSOR_PARALLEL_SIZE))` in…

### #8279 — [[hardware, megatron, reward, doc] feat: add FAPO Megatron training on Ascend](https://github.com/verl-project/verl/pull/8279)
- **作者**: xldeng-chn  **时间**: 2026-10-19 11:14 CST
- **摘要**: ### What does this PR do?  Adds the core `verl` changes for issue #16: Qwen2.5-Math-7B + FAPO-GenRM-4B training on Ascend with a Megatron actor, vLLM-Ascend rollout, and a colocated GenRM.  The delivery is paired with recipe PR https://github.com/verl-project/verl-ascend-recipe/pull/152. The recipe …

### #8278 — [[CI] fix: guard actor policy_loss lookup when config lacks the field](https://github.com/verl-project/verl/pull/8278)
- **作者**: ETOgaosion  **时间**: 2026-10-19 10:27 CST
- **摘要**: ## Summary `main` is red on `cpu_unit_tests`: after #8010 (score centering) and #7945 (Liger TP fused CE) both landed, the megatron liger CPU test builds a lightweight config via a monkeypatched `omega_conf_to_dataclass` that returns a `SimpleNamespace` without `policy_loss`, while `ActorRolloutRefW…

### #8277 — [[training_utils] fix: bind tuple outputs to the activation offload commit node](https://github.com/verl-project/verl/pull/8277)
- **作者**: Eliaaazzz  **时间**: 2026-10-19 09:12 CST
- **摘要**: ### What does this PR do?  Fixes #7472 by routing every differentiable top-level tuple output through the same activation-offload group-commit autograd node.  Previously, `ActivationHandler.forward()` bound only the first output. Backward through an auxiliary recurrent-state tensor could therefore r…

### #8276 — [[reward] fix: match whole words in search_r1 subEM and ignore empty golden answers](https://github.com/verl-project/verl/pull/8276)
- **作者**: flyingdoubleG  **时间**: 2026-10-18 22:36 CST
- **摘要**: ### What does this PR do?  `search_r1_like_qa_em.subem_check` matches a golden answer as a *character* substring of the prediction, so the subEM reward is given for answers that do not contain the golden answer:  | answer | golden | subEM before | after | |---|---|---|---| | comparison | Paris | 1 |…

### #8275 — [[doc] feat: add Awesome-Perception-Aware-RLVR to awesome work list](https://github.com/verl-project/verl/pull/8275)
- **作者**: zsxm1998  **时间**: 2026-10-18 18:59 CST
- **摘要**: ### What does this PR do?  Adds [Awesome-Perception-Aware-RLVR](https://github.com/zsxm1998/awesome-perception-aware-rlvr) to "Awesome Projects Built with `verl`" in README.md (one line, after the last entry).  The project is built on EasyR1 (a verl fork). It implements 19 perception-aware RLVR and …

### #8274 — [[veomni,ci] feat: add uv-based veomni training backend and cover it in GPU CI](https://github.com/verl-project/verl/pull/8274)
- **作者**: ETOgaosion  **时间**: 2026-10-18 16:00 CST
- **摘要**: ## Summary  Promotes **VeOmni to a first-class uv training backend** on the universal uv-managed CUDA 13 stack, fixes the open image build behind a filtering egress proxy, and adds CI coverage.  ### Training backend - VeOmni installable as a standalone backend: `uv run --frozen --all-packages --extr…
