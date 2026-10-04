# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-04 14:14 CST

## AI 总结

以下是 **vllm-project/vllm-ascend** 仓库近期动态的中文摘要：

### 📦 Release
近期暂无版本发布动态。

### 🛠 Pull Requests (PR)
本期共有 3 个 PR，主要涉及文档更新与 GLM-5.2 模型的性能/CI 优化：

1. **文档翻译与更新 (#17873)**
   - 由 CI 机器人自动触发，成功翻译并更新了 23 个文档文件，主要涵盖开发者指南中的设计文档（如添加自定义 `aclnn` 算子等内容），提升中文用户的阅读体验。

2. **GLM-5.2 性能特性开启 (PD 分离架构) (#17872)**
   - 在 GLM-5.2 W8A8C8 128K/1K 四节点 PD（Prefill-Decode）的 weekly 测试中，针对 Decode 节点 2 和 3 启用了 `enable_npugraph_ex`、`enable_static_kernel` 和 `enable_super_kernel` 特性。这有助于进一步优化图执行效率和解码阶段的内核性能。

3. **GLM-5.2 性能特性开启 (内部 DP 架构) (#17871)**
   - 在 GLM-5.2 内部 DP（Data Parallel）的 weekly 测试中，同样在双部署命令的编译配置（`ascend_compilation_config`）中启用了上述三个内核优化特性，以提升整体部署性能。

### ❓ Issue
近期暂无公开的 Issue 动态。

---
**总结**：本期动态重点在于**针对 GLM-5.2 模型在 PD 和 DP 架构下的内核性能优化**，通过引入静态内核、超级内核及 NPUGraph 扩展来提升执行效率；同时持续维护和更新中文技术文档。

---

## 🔀 Pull Requests

### #17873 — [[Doc] Translated Doc files 2026-10-04](https://github.com/vllm-project/vllm-ascend/pull/17873)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-04 13:48 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **23** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/add_custom_aclnn_op.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #17872 — [[CI][Feature] Enable static and super kernels for GLM 5.2 PD weekly](https://github.com/vllm-project/vllm-ascend/pull/17872)
- **作者**: Wyz-134  **时间**: 2026-10-04 13:10 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Enables `enable_npugraph_ex`, `enable_static_kernel` and `enable_super_kernel` on Decode nodes 2 and 3 of the existing GLM-5.2 W8A8C8 128K/1K four-node PD weekly case, inside the same `ascend_compilation_config`, and sets `LOCAL_WORLD_SIZE="16"` in the enviro…

### #17871 — [[CI][Feature] Enable static and super kernels for GLM 5.2 internal DP weekly](https://github.com/vllm-project/vllm-ascend/pull/17871)
- **作者**: Wyz-134  **时间**: 2026-10-04 13:10 CST
- **标签**: module:tests
- **摘要**: ### What this PR does / why we need it?  Enables `enable_npugraph_ex`, `enable_static_kernel` and `enable_super_kernel` in the `ascend_compilation_config` of both deployment commands of the existing two-node GLM-5.2 W8A8C8 A3 128K internal DP weekly case, and sets `LOCAL_WORLD_SIZE="16"` in the shar…
