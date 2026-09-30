# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-09-30 13:52 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📦 Release
- **v0.27.1rc1 (2026-09-29)**
  - **版本亮点**：这是 vLLM Ascend v0.27.1 的首个候选版本，主要与上游 vLLM v0.27.1 版本保持对齐。官方表示这是一个“以模型为核心”的版本。

---

### 🔧 Pull Request (PR)

**🚀 新特性**
- **KVPP 与投机解码增强**：
  - #17820：KVPP 现支持接受 `mtp`、`dspark`、`dflash` 和 `eagle3` 参数，并允许 DFlash 和 EAGLE3 使用动态投机长度。
  - #17817：移除了 KVPP 对投机解码方法名的限制，转而基于是否分配独立的 draft KV caches 来支持缓存能力。

**🐛 重要 Bug 修复**
- **投机解码**：#17826 修复了加载 V2 EAGLE 草稿模型时仍会错误初始化 MLAPO 的问题。
- **模型算子**：#17825 修复了 `AscendApplyRotaryEmb` 未支持交错式 RoPE 的问题，从而修复了 Kimi K2.5 视觉编码器的兼容性。
- **注意力机制**：#17823 修复了当 A2/A3 DCP 行为空时（仅有 `-1` sparse indices）返回非零 softmax sum 的错误。
- **构建系统**：#17818 修复了 `build_example_for_ci` 函数中结果变量未初始化的隐患。

**♻️ 重构与测试**
- #17822：引入了 `AscendStore` v1 实现，并基于 Mooncake 服务添加了进程内的冒烟测试。
- #17821：为 `HcPre` 算子补充了测试覆盖（该算子被 GLM-5.3-Flash 和 DeepSeek mHC 层调用），包含有限迭代测试和完整算子基准测试。

**⚙️ CI 与工程化**
- #17819：引入了“维护窗口”机制，可在维护期间阻断 nightly/weekly PR 触发器，以保护第三方定时测试。
- #17824：调整并迁移了部分 CI 测试用例。

---

### ❓ Issue
- 本周期内未提供相关 Issue 动态。

---

## 🔀 Pull Requests

### #17826 — [[Bugfix][Spec Decode] Disable MLAPO when loading V2 EAGLE drafts](https://github.com/vllm-project/vllm-ascend/pull/17826)
- **作者**: Dawn952  **时间**: 2026-09-30 12:59 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  V2 EAGLE draft loading can still initialize MLAPO despite #15265. That fix propagates the draft runner identity through the V1 proposer, but V2 uses `EagleSpeculator.load_draft_model()` and `load_eagle_model(target_model, self.vllm_config)`. The loader receiv…

### #17825 — [[BugFix][Model] Honor interleaved RoPE in AscendApplyRotaryEmb](https://github.com/vllm-project/vllm-ascend/pull/17825)
- **作者**: zhao-stack  **时间**: 2026-09-30 12:37 CST
- **标签**: module:tests, module:ops, ready-precise
- **摘要**: ### What this PR does / why we need it?  Kimi K2.5's vision encoder requests `ApplyRotaryEmb(is_neox_style=False, enable_fp32_compute=True)`, but `AscendApplyRotaryEmb.forward_oot` always used the split-half `npu_rotary_mul` kernel. Pairing the wrong Q/K dimensions corrupts vision features; an image…

### #17824 — [[CI] mv cases](https://github.com/vllm-project/vllm-ascend/pull/17824)
- **作者**: chen-commits  **时间**: 2026-09-30 12:37 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  ### Does this PR introduce _any_ user-facing change?  ### How was this patch tested?  - vLLM main: https://github.com/vllm-project/vllm/commit/ced6857afa0ea7b2e3f0846a62e1394e90f15607

### #17823 — [[Bugfix][SFA] Return zero softmax sums for empty A2/A3 DCP rows](https://github.com/vllm-project/vllm-ascend/pull/17823)
- **作者**: LiPu-jpg  **时间**: 2026-09-30 12:37 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  An A2/A3 unquantized SFA row with a nonempty local KV cache but only `-1` sparse indices returns a zero attention output while reporting a nonzero softmax sum (512 in the regression). Consequently, `softmax_max + log(softmax_sum)` is a finite sentinel instead…

### #17822 — [[Refactor][KVPool] Introduce AscendStore v1 and in-process smoke coverage](https://github.com/vllm-project/vllm-ascend/pull/17822)
- **作者**: ChenZhuo888  **时间**: 2026-09-30 12:31 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Introduce the opt-in AscendStore v1 implementation and add a minimal in-process smoke test backed by a real Mooncake service. The test exercises cold Store, remote Lookup and synchronous Load after clearing only the local prefix cache.  This draft is intended…

### #17821 — [[Test][Ops] Cover HcPre finite iterations and benchmark complete operator](https://github.com/vllm-project/vllm-ascend/pull/17821)
- **作者**: LiPu-jpg  **时间**: 2026-09-30 12:29 CST
- **标签**: documentation, module:tests
- **摘要**: ### What this PR does / why we need it?  HcPre is called by the GLM-5.3-Flash and DeepSeek mHC layers, but its nightly numerical coverage fixes Sinkhorn at 20 iterations and primarily uses small, nonnegative inputs. Add 48 cases covering iteration counts 1/3/20, hidden sizes 4096/7168, 1/17/257 toke…

### #17820 — [[Feature][KVPP] Allow DFlash and EAGLE3 with dynamic speculative lengths](https://github.com/vllm-project/vllm-ascend/pull/17820)
- **作者**: recky-c  **时间**: 2026-09-30 11:57 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ## Summary - KVPP now accepts `mtp`, `dspark`, `dflash`, and `eagle3`. Draft cache layers still come from the loaded proposer and stay outside the owner partition. - Dynamic speculative lengths are no longer rejected by KVPP. Batch-size K, adaptive verification, and `dynamic_spec_config` keep using …

### #17819 — [[CI] Add maintenance window to block /nightly and /weekly PR triggers](https://github.com/vllm-project/vllm-ascend/pull/17819)
- **作者**: zhangxinyuehfad  **时间**: 2026-09-30 11:57 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Nightly/weekly PR comment triggers can be paused during a maintenance window to protect third-party scheduled tests. A single repository variable `NIGHTLY_COMMAND_BLOCK_WINDOW` controls the gate, taking an optional switch followed by an `HH:MM-HH:MM` range in…

### #17818 — [[BugFix][Build] Initialize example result variables in build_example_for_ci](https://github.com/vllm-project/vllm-ascend/pull/17818)
- **作者**: robellliu-dev  **时间**: 2026-09-30 11:54 CST
- **摘要**: ## What this PR does / why we need it?  In `csrc/build.sh`, `build_example_for_ci()` records the exit code of `build_example` with the `cmd || local var=$?` idiom, but the result variables are never initialized. On the **success** path the `local var=$?` assignment is not executed, so the subsequent…

### #17817 — [[Feature][KVPP] Support speculative decoding via cache capabilities](https://github.com/vllm-project/vllm-ascend/pull/17817)
- **作者**: recky-c  **时间**: 2026-09-30 11:53 CST
- **标签**: documentation, module:tests, module:core
- **摘要**: ### What this PR does / why we need it?  KVPP restricts speculation by method name, although its cache ownership depends on whether the proposer allocates independent draft KV caches. Remove the KVPP method whitelist and use the existing `uses_draft_kv_cache()` capability for draft ownership and sli…

## 🚀 Releases

### [v0.27.1rc1](https://github.com/vllm-project/vllm-ascend/releases/tag/v0.27.1rc1)
- **作者**: linfeng-yuan  **时间**: 2026-09-29 19:55 CST
- **摘要**: ## v0.27.1rc1 - 2026.09.24  This is the first release candidate of v0.27.1 for vLLM Ascend, aligned with upstream vLLM v0.27.1. This model-focused release adds MiniMax-M3 support on Atlas 800 A3 and 950DT Products, and new DeepSeek-V4-Flash support on 950DT Products. Other models and hardware combin…
