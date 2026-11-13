# vllm-project/vllm — 动态追踪

> 生成时间: 2026-11-13 15:06 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm** 近期动态的中文摘要。本次动态未包含 Release 信息，主要涵盖 2 个 Issue 和 10 个 PR。

### 🐛 Issue 动态
1. **`bad_words` 参数失效 (#67407)**：在启用 `--tokens-only` 或 `--enable-scale-out --skip-tokenizer-init` 的服务器上，调用 `/inference/v1/generate` 接口时 `sampling_params.bad_words` 会被忽略，并错误地返回 200 状态码。
2. **`rotary_embedding` 静默计算错误 (#67401)**：底层自定义算子 `vllm._custom_ops.rotary_embedding` 未校验 q/k 张量最后一维的非单位 stride（non-unit stride），可能导致静默的计算错误。

---

### 🚀 PR 动态

#### Bug 修复
- **修复无 Tokenizer 服务器上的 `bad_words` 问题 (#67408)**：针对上述 Issue #67407，在无 tokenizer 的生成服务器上明确拒绝包含 `bad_words` 的请求，而非静默忽略。
- **兼容性修复 (#67406)**：修复了 engram 与 `deepseek-v4.1-flash` 模型的兼容性问题，以支持快速启动（Fast start）。

#### 新特性与重要更新
- **KV Connector 动态权重一致性支持 (RFC #65714 系列，#67402~#67405)**：这是一组包含 5 个部分的串联 PR，旨在支持运行时模型权重更新时的 KV 缓存管理。具体包括：基于 checkpoint 内容识别 KV identity 权重、在 Rust 前端实现 `/kv_identity` 接口、在运行时权重变更时更新 KV identity，并在 identity 变更后重新检查 KV peers 以保障分布式节点间的缓存一致性。
- **投机解码优化 (#67397)**：引入独立的物理 Draft KV 缓存池和发散命中恢复机制。该特性兼容滑动窗口 Drafter（如 DFlash/DSpark），并在主模型与 Draft 前缀命中发散时保留可复用的 Main KV，大幅提升解码效率。
- **Rust 前端功能增强**：
  - **多模态输入支持 (#67400)**：`/generate` 接口现在支持同时接收 `features` 和 `content_parts`，以处理多轮多模态输入。
  - **优先级请求头解析 (#67399)**：支持读取并遵循 `X-Vllm-Priority` 请求头（覆盖 body 参数），同时会拒绝无法解析的请求头。
  - **Inkling 渲染器媒体支持 (#67398)**：Rust 版 Inkling 渲染器新增对工具调用返回结果中媒体内容（如截图）的渲染支持。 

*(注：本次输入数据中未包含 Release 相关信息)*

---

## 🐛 Issues

### #67407 — [[Bug]: `/inference/v1/generate` ignores `sampling_params.bad_words` with a 200 on `--tokens-only` and `--skip-tokenizer-init` servers](https://github.com/vllm-project/vllm/issues/67407)
- **作者**: nien-hui  **时间**: 2026-11-13 14:59 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text OS: Ubuntu 22.04.3 LTS (x86_64) Python: 3.12.14 vLLM: main @ e509ffb94361c36cbecf02801225089253fc0d49 (2026-11-12) Hardware: 1x NVIDIA RTX A6000 per server ```  </details>  ### 🐛 Desc…

### #67401 — [[Bug]: rotary_embedding silently miscomputes q/k with non-unit last-dimension stride](https://github.com/vllm-project/vllm/issues/67401)
- **作者**: LittlehamsterXu  **时间**: 2026-11-13 14:50 CST
- **标签**: bug
- **摘要**: ### Your current environment  <details> <summary>The output of <code>python collect_env.py</code></summary>  ```text Collecting environment information... ==============================         System Info ============================== OS                           : Ubuntu 26.04.1 LTS (x86_64) GCC …

## 🔀 Pull Requests

### #67408 — [[Bugfix] Reject bad_words on generate servers without a tokenizer](https://github.com/vllm-project/vllm/pull/67408)
- **作者**: nien-hui  **时间**: 2026-11-13 15:00 CST
- **标签**: bug, documentation, frontend
- **摘要**: ## Overview  On `vllm serve --tokens-only` and on `vllm serve --enable-scale-out --skip-tokenizer-init`, a `/inference/v1/generate` request with `sampling_params.bad_words` returns 200 with the same tokens as the request without it; bad words are tokenized only when the server has a tokenizer, and `…

### #67406 — [[Fast start] Fix engram compatibility with deepseek-v4.1-flash](https://github.com/vllm-project/vllm/pull/67406)
- **作者**: Isotr0py  **时间**: 2026-11-13 14:54 CST
- **标签**: documentation, needs-rebase, ci/build, deepseek
- **摘要**: ## Overview    ## Claims    ## Validation    ## Details    ---  <details> <summary> Pull Request Checklist </summary>  - [ ] I used vLLM's `/pr-checklist` skill. (Mandatory for agents, optional for humans). - [ ] AI assistance was used during the creation of this PR.  - [ ] **Design Fit:** Minimizes…

### #67405 — [[Feature][KV Connector] Check KV peers again after a KV identity change](https://github.com/vllm-project/vllm/pull/67405)
- **作者**: albertoperdomo2  **时间**: 2026-11-13 14:54 CST
- **标签**: documentation, frontend, needs-rebase, ci/build, cpu, kv-connector, mrv2, rust, nixl
- **摘要**: ## Overview  > **Merge order:** #66569 → #67402 → #67403 → #67404 → #67405. This is part 5 of 5; merge each only after the previous one.  Part 5 of 5 for RFC #65714. **Depends on #67404**; draft until it merges, so review only the last two commits. With `enforce_kv_identity`, a weight change now res…

### #67404 — [[Feature][KV Connector] Update the KV identity when weights change at runtime](https://github.com/vllm-project/vllm/pull/67404)
- **作者**: albertoperdomo2  **时间**: 2026-11-13 14:54 CST
- **标签**: documentation, frontend, ci/build, cpu, kv-connector, mrv2, rust, nixl
- **摘要**: ## Overview  > **Merge order:** #66569 → #67402 → #67403 → #67404 → #67405. This is part 4 of 5; merge each only after the previous one.  Part 4 of 5 for RFC #65714. **Depends on #67403**; draft until it merges, so review only the last three commits. The identity now follows weights changed in a run…

### #67403 — [[Feature][Frontend] Implement /kv_identity in the Rust frontend](https://github.com/vllm-project/vllm/pull/67403)
- **作者**: albertoperdomo2  **时间**: 2026-11-13 14:54 CST
- **标签**: documentation, frontend, ci/build, kv-connector, rust, nixl
- **摘要**: ## Overview  > **Merge order:** #66569 → #67402 → #67403 → #67404 → #67405. This is part 3 of 5; merge each only after the previous one.  Part 3 of 5 for RFC #65714. **Depends on #67402**; draft until it merges, so review only the last commit. Adds `GET /kv_identity` to the Rust frontend, for parity…

### #67402 — [[Feature][KV Connector] Identify the KV identity's weights by checkpoint content](https://github.com/vllm-project/vllm/pull/67402)
- **作者**: albertoperdomo2  **时间**: 2026-11-13 14:54 CST
- **标签**: documentation, frontend, ci/build, kv-connector, nixl
- **摘要**: <!-- markdownlint-disable -->  ## Overview  > **Merge order:** #66569 → #67402 → #67403 → #67404 → #67405. This is part 2 of 5; merge each only after the previous one.  Part 2 of 5 for RFC #65714. **Depends on #66569**; draft until it merges, so review only the last two commits. The identity's `weig…

### #67400 — [[Rust Frontend] Accept features together with content_parts on /generate](https://github.com/vllm-project/vllm/pull/67400)
- **作者**: BugenZhao  **时间**: 2026-11-13 14:48 CST
- **标签**: rust
- **摘要**: ## Overview  Since vllm-project/vllm#57796, Python's `/inference/v1/generate` accepts `features` and `content_parts` in one request (docs: `token_in_token_out.md`, "Multimodal inputs"), for multi-turn rollouts: the earlier images arrive already expanded as `features`, and the new turn's images arriv…

### #67399 — [[Rust Frontend] Honor X-Vllm-Priority on generate and reject unparsable request headers](https://github.com/vllm-project/vllm/pull/67399)
- **作者**: BugenZhao  **时间**: 2026-11-13 14:48 CST
- **标签**: documentation, rust
- **摘要**: ## Overview  vllm-project/vllm#67008 made Python's `/inference/v1/generate` read the `X-Vllm-Priority` header like the coupled APIs (chat, completions, responses), with the header overriding the body `priority`, and documented that the Rust frontend ignores it there. This PR honors the header on the…

### #67398 — [[Rust Frontend] Render tool-result media in the Inkling renderer](https://github.com/vllm-project/vllm/pull/67398)
- **作者**: BugenZhao  **时间**: 2026-11-13 14:48 CST
- **标签**: rust, inkling
- **摘要**: ## Overview  vllm-project/vllm#66891 taught the Python Inkling renderer to render tool results that carry media, such as a screenshot returned by a tool. The Rust Inkling renderer flattens tool-response content to text, so the same request fails with a 400 (`image_url` / `input_audio` content is not…

### #67397 — [[Core][Spec Decode] Add independent Draft KV pools and divergent-hit recovery](https://github.com/vllm-project/vllm/pull/67397)
- **作者**: SSSunzt  **时间**: 2026-11-13 14:47 CST
- **标签**: documentation, needs-rebase, mrv2, scheduler, kv-cache-manager
- **摘要**: ## Overview  Add independent physical KV-cache pools for compatible sliding-window DFlash/DSpark drafters, and preserve reusable Main KV when Main and Draft prefix hits diverge.  This is the first implementation PR for RFC #66751. It covers local cache allocation and recovery; P/D sparse page fill, …
