# verl-project/verl — 动态追踪

> 生成时间: 2026-07-16 09:02 CST

## AI 总结

## verl-project/verl 近期动态摘要

---

### 📋 Issue

- **#7054 — 项目被收录至 StackMap**
  有人告知 verl 已被人工审核并收录到 StackMap（一个开源 AI/Agent 工具的知识图谱地图），属于社区推广类通知，无功能性影响。

---

### 🔀 Pull Request

**🚀 新特性 / 重大变更**

- **#7050 — [ROCm] 在 AMD GPU 上运行 DeepSeek V4 GRPO**（RFC）
  提出 AMD GPU（ROCm）对 DeepSeek V4 GRPO 训练的支持方案，是扩展硬件兼容性的重要功能提案。

**🔧 Bug 修复**

- **#7051 — trainer：`gen_batch_size` 正确回退至 `train_batch_size`**
  V1 trainer 引入了 `gen_batch_size`，导致 legacy trainer 的回退逻辑失效；本 PR 修复了该回退行为。

- **#7046 — tool：移除 `BaseTool.__init__` 中的循环回退逻辑**
  `get_openai_tool_schema()` 与 `self.tool_schema` 形成循环引用，本 PR 删除了该回退以消除循环依赖。

- **#7045 — checkpoint_engine：NCCL group 初始化挂起时快速失败**
  修复了首次 `ray.util.collective` rendezvous 时可能出现的**无声无限挂起**问题，改为 fail-fast 模式，提升可靠性。

- **#7044 — rollout：处理 Qwen3 格式错误的 XML tool calls**
  让 Qwen3 XML tool parser 能容忍截断的函数头和畸形参数，仅跳过畸形部分、保留有效参数，增强鲁棒性。

**⚡ 性能优化**

- **#7049 — trainer/perf：在 separate-async 模式的吞吐量计算中纳入独立 rollout GPU**
  修正吞吐量指标的分母计算，使单独异步 rollout 的 GPU 也被计入，数据更准确。

**📦 依赖更新**

- **#7053 — pyarrow 版本约束更新：允许 ≥25.0.0（原 ≤24.0.0）**
- **#7052 — transferqueue 从 0.1.8 升至 0.1.9**

**📝 其他**

- **#7048 — 文档：新增 rl-insight 新闻条目**
- **#7047 — 测试 PR**（内容无实质变更）

---

### 🏷️ Release

本期无新版本发布。

---

> **总结**：本轮动态以**稳定性修复**为主（trainer 回退逻辑、工具循环引用、NCCL 挂起、Qwen3 畸形解析），同时有一个值得关注的**AMD GPU (ROCm) 支持 RFC**，以及常规依赖升级和文档更新。

---

## 🐛 Issues

### #7054 — [Your project is on StackMap — a curated map of the AI stack](https://github.com/verl-project/verl/issues/7054)
- **作者**: hoghweed  **时间**: 2026-07-16 06:25 CST
- **摘要**: Hi — I curate **[StackMap](https://stackmap.shipwithai.xyz?utm_source=maintainer-outreach)**, a hand-curated knowledge graph of open-source AI/agent tools. Every entry is human-reviewed: a summary, an opinionated note on when to use it (and when not), and typed edges to what it pairs with or compete…

## 🔀 Pull Requests

### #7053 — [build(deps): update pyarrow requirement from <=24.0.0,>=15.0.0 to <=25.0.0,>=25.0.0](https://github.com/verl-project/verl/pull/7053)
- **作者**: dependabot[bot]  **时间**: 2026-07-16 01:33 CST
- **标签**: dependencies, python
- **摘要**: Updates the requirements on [pyarrow](https://github.com/apache/arrow) to permit the latest version. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/apache/arrow/releases">pyarrow's releases</a>.</em></p> <blockquote> <h2>Apache Arrow 25.0.0</h2> <p>Release…

### #7052 — [build(deps): bump transferqueue from 0.1.8 to 0.1.9](https://github.com/verl-project/verl/pull/7052)
- **作者**: dependabot[bot]  **时间**: 2026-07-16 01:33 CST
- **标签**: dependencies, python
- **摘要**: Bumps [transferqueue](https://github.com/Ascend/TransferQueue) from 0.1.8 to 0.1.9. <details> <summary>Release notes</summary> <p><em>Sourced from <a href="https://github.com/Ascend/TransferQueue/releases">transferqueue's releases</a>.</em></p> <blockquote> <h2>v0.1.9</h2> <h2>Highlight</h2> <h3>🚀 N…

### #7051 — [[trainer] fix: gen_batch_size falls back to train_batch_size ](https://github.com/verl-project/verl/pull/7051)
- **作者**: Begunner  **时间**: 2026-07-15 18:07 CST
- **摘要**: ### What does this PR do?  Legacy trainer assumes `gen_batch_size` not in the config by default. V1 trainer introduces it so legacy `gen_batch_size` should fall back to `train_batch_size` correctly.

### #7050 — [[RFC] [ROCm] feat: bring-up deepseek v4 grpo on AMD GPUs](https://github.com/verl-project/verl/pull/7050)
- **作者**: PeterYang12  **时间**: 2026-07-15 17:35 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [x] Search for similar PRs. Paste at least one query link here: ... - [x] Format the PR title as `…

### #7049 — [[trainer, perf] fix: include standalone rollout GPUs in throughput denominator for separate async](https://github.com/verl-project/verl/pull/7049)
- **作者**: mikequan0425  **时间**: 2026-07-15 17:29 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  In separate-async mode, the standalone rollout runs on dedicated GPUs that are not part of the trainer resource pool. However, comp…

### #7048 — [[doc] chore: add rl-insight news](https://github.com/verl-project/verl/pull/7048)
- **作者**: tardis-key  **时间**: 2026-07-15 16:09 CST
- **摘要**: ### What does this PR do?  add rl-insight news  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `[{modules}] {type}: {description}` (This will be checked by the CI)   - `{modules}` include `fsdp`, `megatron`, `veomni`,…

### #7047 — [Test](https://github.com/verl-project/verl/pull/7047)
- **作者**: xiushen01  **时间**: 2026-07-15 15:15 CST
- **摘要**: ### What does this PR do?  > Add **concise** overview of what this PR aims to achieve or accomplish. Reference related GitHub issues and PRs that help with the review.  ### Checklist Before Starting  - [ ] Search for similar PRs. Paste at least one query link here: ... - [ ] Format the PR title as `…

### #7046 — [[tool] fix: remove circular fallback in BaseTool.__init__](https://github.com/verl-project/verl/pull/7046)
- **作者**: cyyueyang  **时间**: 2026-07-15 15:02 CST
- **摘要**: ### What does this PR do?     Remove the `or self.get_openai_tool_schema()` fallback in    `BaseTool.__init__`. It creates a circular reference:    `get_openai_tool_schema()` returns `self.tool_schema`, but when    `tool_schema=None` the `or` calls it before the attribute exists    → `AttributeError…

### #7045 — [[checkpoint_engine] fail fast on hung first NCCL group init (#6967)](https://github.com/verl-project/verl/pull/7045)
- **作者**: aryanyadav0402  **时间**: 2026-07-15 13:10 CST
- **摘要**: ## Motivation  Fixes the *silent-infinite-hang* failure mode of #6967. The first `ray.util.collective` rendezvous during checkpoint-engine weight sync (`NCCLCheckpointEngine.init_process_group` → `init_collective_group` + first `barrier`) can hang **indefinitely** on some setups — a timing race in t…

### #7044 — [[rollout] fix: handle malformed Qwen3 XML tool calls](https://github.com/verl-project/verl/pull/7044)
- **作者**: zzzzzzzxh  **时间**: 2026-07-15 11:36 CST
- **摘要**: ## Summary  - Make the Qwen3 XML tool parser tolerate truncated function headers and malformed parameters. - Skip only malformed parameters so valid parameters in the same tool call are preserved. - Add CPU regression tests for both malformed cases.  ## Root Cause  `Qwen3XMLToolParser._parse_xml_fun…
