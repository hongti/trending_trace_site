# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-09 14:33 CST

## AI 总结

# vllm-ascend 仓库近期动态摘要

## 📌 Issue
本次数据中未包含 Issue 活动。

---

## 🔧 Pull Request

### 一、权重传输（WeightTransfer）重构系列
源自 #17597 的三个堆叠 PR，逐步完善 NPU IPC 机制：
- **#18078**（基础层）：加固打包张量 IPC 传输，提取最小化 NPU IPC 调用点适配。
- **#18083**（特性层）：引入**有状态的 HCCL Trainer 生命周期管理**，包含初始化与生命周期提取。
- **#18085**（完善层）：修复 NPU IPC **生命周期与失败处理**，补全 finalize 逻辑。

### 二、性能优化
- **#18081**：为 **DeepSeek V4.1** 模型启用 `@support_torch_compile`，使其主干网络被 vLLM 编译路径捕获——此前这是 vllm-ascend 中唯一未装饰的 DeepSeek 模型。

### 三、Bug 修复
- **#18079 / #18080**（同一修复）：将 DSA block size 改为 Triton `tl.constexpr` 对象。原因是 Python 类型注解在 `@triton.jit` 内部不可见，直接构造 constexpr 使内核能正确读取 block size。

### 四、CI 与基础设施
- **#18076**：补充手动镜像构建引用，将 `releases/v0.30.0` 纳入工作流，覆盖 v0.18 至 v0.30 的发布分支。
- **#18077**：为 GLM-5.1 W8A8 PrefillMC2 夜间测试启用 `enable_npugraph_ex`、`enable_static_kernel` 和 `enable_super_kernel`。
- **#18084**：**回退** C8 启动配置（因部分测试仍引用旧配置）。
- **#18082**：**回退** PR #13773 的量化方法 MLP 重构，恢复 `unified_apply_mlp` 和设备适配器内核分发。

---

## 🚀 Release
本次数据中未包含 Release 活动。

---

### 总体趋势
近期活动集中在三方面：**NPU IPC 权重传输机制的系统化重构**（拆分为三个堆叠 PR）、**DeepSeek V4.1 性能提升**（torch.compile 支持），以及**多项 CI 稳定性修复**（含两次配置回退）。

---

## 🔀 Pull Requests

### #18085 — [[BugFix][WeightTransfer] Fix NPU IPC lifecycle and failure handling](https://github.com/vllm-project/vllm-ascend/pull/18085)
- **作者**: Windfeng8  **时间**: 2026-10-09 13:01 CST
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  This is the third stacked PR for splitting the original #17597 weight-transfer change. It completes the NPU IPC lifecycle and failure-handling changes:  - finalizes receiver state transitions and importer lifetime management; - completes the update handshake a…

### #18084 — [[CI] Revert the C8 configuration](https://github.com/vllm-project/vllm-ascend/pull/18084)
- **作者**: kunpengW-code  **时间**: 2026-10-09 12:54 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  In PR https://github.com/vllm-project/vllm-ascend/pull/15514, the startup configuration for C8 was updated. Some test cases still referenced the old configuration, so revert those test case changes.  ### Does this PR introduce _any_ user-facing change?  no  #…

### #18083 — [[Feat][WeightTransfer] Add stateful HCCL trainer lifecycle](https://github.com/vllm-project/vllm-ascend/pull/18083)
- **作者**: Windfeng8  **时间**: 2026-10-09 12:49 CST
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  This is the second stacked PR from #17597 and depends on #18078.  It extracts the HCCL trainer/lifecycle layer:  - add the stateful HCCL trainer initialization and send lifecycle; - add shared HCCL trainer/worker setup helpers; - make the PyHCCL communicator l…

### #18082 — [[CI]: revert PR 13773 MLP refactor](https://github.com/vllm-project/vllm-ascend/pull/18082)
- **作者**: U1stRsouland  **时间**: 2026-10-09 12:42 CST
- **标签**: module:tests, module:ops, module:quantization
- **摘要**: Revert the quantization-method MLP refactor introduced by a7997713ec9ed1f3b8f8e4e87efaf197548f2cb3, restoring unified_apply_mlp and device-adaptor kernel dispatch.  Resolve conflicts against current main while retaining later 310P dispatch, MegaMoe weight layouts and per-layer activation metadata, A…

### #18081 — [[Performance] enable torch.compile for the deepseek V4.1](https://github.com/vllm-project/vllm-ascend/pull/18081)
- **作者**: keyi-zz  **时间**: 2026-10-09 12:39 CST
- **标签**: ready-precise
- **摘要**: - Decorate DeepseekV41Model with @support_torch_compile so the V4.1 backbone is captured by vLLM's compile path. It was the only DeepSeek model in vllm_ascend left undecorated: deepseek_v4/model.py and deepseek_v41/dspark.py already opt in. Without the decorator the backbone stays outside FX-graph c…

### #18080 — [[BugFix][Ops] Make DSA block size a Triton constexpr object](https://github.com/vllm-project/vllm-ascend/pull/18080)
- **作者**: lvvgang  **时间**: 2026-10-09 12:32 CST
- **标签**: module:ops
- **摘要**: A type annotation is not visible inside @triton.jit. Construct tl.constexpr directly so the kernel can read the block size.  ### What this PR does / why we need it? BUILD_LOCAL_METADATA_BLOCK_SIZE in vllm_ascend/ops/triton/dsa_cp.py was declared as a type annotation: ```sh BUILD_LOCAL_METADATA_BLOCK…

### #18079 — [[BugFix][Ops] Make DSA block size a Triton constexpr object](https://github.com/vllm-project/vllm-ascend/pull/18079)
- **作者**: lvvgang  **时间**: 2026-10-09 12:31 CST
- **标签**: module:ops
- **摘要**: A type annotation is not visible inside @triton.jit. Construct tl.constexpr directly so the kernel can read the block size.  ### What this PR does / why we need it?  BUILD_LOCAL_METADATA_BLOCK_SIZE in vllm_ascend/ops/triton/dsa_cp.py was declared as a type annotation: ```sh BUILD_LOCAL_METADATA_BLOC…

### #18078 — [[BugFix][WeightTransfer] Harden packed tensor IPC transfer](https://github.com/vllm-project/vllm-ascend/pull/18078)
- **作者**: Windfeng8  **时间**: 2026-10-09 12:31 CST
- **标签**: module:tests
- **摘要**: ## What this PR does / why we need it?  This is the first stacked PR from #17597. It extracts the packed-tensor IPC foundation and the minimum NPU IPC call-site adaptation needed to use it safely:  - add bounded packed NPU IPC producer/consumer helpers and explicit importer ownership; - keep the imp…

### #18077 — [[CI][Feature] Enable static and super kernels for GLM 5.1 PrefillMC2](https://github.com/vllm-project/vllm-ascend/pull/18077)
- **作者**: Wyz-134  **时间**: 2026-10-09 11:58 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Enable `enable_npugraph_ex`, `enable_static_kernel`, and `enable_super_kernel` under `ascend_compilation_config` for the existing GLM-5.1 W8A8 PrefillMC2 nightly case, following #17871. Set `LOCAL_WORLD_SIZE="16"` to match the single-node A3 TP16/DP1 configur…

### #18076 — [[CI] Complete manual image build refs](https://github.com/vllm-project/vllm-ascend/pull/18076)
- **作者**: underfituu  **时间**: 2026-10-09 11:56 CST
- **标签**: ci/build
- **摘要**: ### What this PR does / why we need it?\n\n- Adds eleases/v0.30.0 to the manual image build workflow.\n- Completes the nightly image build choices with the existing release branches from v0.18 through v0.30.\n- Adds the published release tags that correspond to those supported branches.\n- Resolves…
