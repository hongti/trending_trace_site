# verl-project/verl-recipe — 动态追踪

> 生成时间: 2026-11-22 15:19 CST

## AI 总结

以下是 GitHub 仓库 **verl-project/verl-recipe** 近期动态的中文摘要：

### 📋 Issue 动态
*   **#181** [deepeyes] 修复工具调用失败仍获得奖励的问题
    *   **摘要**: 报告了一个奖励计算漏洞。`compute_score()` 仅通过标签判断是否使用了工具，导致即使 `crop` (裁剪) 工具调用失败返回错误，正确答案依然会错误地获得 1.2 倍的工具奖励。
*   **#180** [deepeyes] 支持图像缩放工具中的归一化边界框
    *   **摘要**: 提出功能请求，希望在 DeepEyes 缩放工具中支持 0–1000 的归一化边界框网格，与现有的像素坐标模式共存，以便更好地适应模型或提示词输出的归一化坐标。

### 🔀 PR 动态
*   **#179** [deepeyes] fix: 要求工具响应成功才能获得工具奖励
    *   **重要修复**: 对应 Issue #181。修改了奖励机制，只有当工具调用成功返回有效响应时，答案才能获得 1.2 倍的工具奖励，堵住了仅凭标签存在即发放奖励的漏洞。
*   **#178** [deepeyes] feat: 支持图像缩放的归一化边界框
    *   **新特性**: 对应 Issue #180。在应用图像裁剪和调整大小策略前，将 0-1000 的归一化边界框转换为像素坐标。同时添加了工具 schema 和可选的 `TOOL_CONFIG_PATH` 覆盖配置，且保留了像素坐标的默认模式。
*   **#177** [recipe] feat: 新增 Qwen3.5-397B 模型 DeepSWE 训练配方
    *   **新特性**: 为 `deepswe_tpu` 目录添加了两项内容：
        1. 在 TPU v7x 上支持 64k 上下文的 `Qwen3.5-397B-A17B` 模型训练配方；
        2. 在训练数据中过滤掉了 54 个 R2E-Gym 样本。

### 🚀 Release 动态
*   近期无新的 Release 发布。

---

## 🐛 Issues

### #181 — [[deepeyes] Failed crop calls can still receive the tool reward](https://github.com/verl-project/verl-recipe/issues/181)
- **作者**: archted  **时间**: 2026-11-22 15:10 CST
- **摘要**: ## Problem  `compute_score()` detects tool use from `<tool_call>` or `<tool_response>` tags. A failed crop still has these tags, so a correct answer receives the 1.2 tool bonus even when the tool returns an error.  ## Reproduction  Pass a trajectory containing a crop call, an error tool response, an…

### #180 — [[deepeyes] Support normalized bounding boxes in the image zoom tool](https://github.com/verl-project/verl-recipe/issues/180)
- **作者**: archted  **时间**: 2026-11-22 14:06 CST
- **摘要**: ## Requested capability  Support an explicit 0–1000 bounding-box grid in the DeepEyes zoom tool, alongside the existing pixel-coordinate mode.  When a model or prompt uses normalized coordinates, the current pixel interpretation can return a successful crop of the wrong region. For a loaded 2000×100…

## 🔀 Pull Requests

### #179 — [[deepeyes] fix: require a successful tool response for the tool bonus](https://github.com/verl-project/verl-recipe/pull/179)
- **作者**: archted  **时间**: 2026-11-22 13:30 CST
- **摘要**: An answer can currently receive the 1.2 tool bonus even when the crop tool fails, because tool_call/tool_response tags are sufficient evidence of tool use. A correct answer with a failed crop therefore scores 2.0 instead of 0.8.  Require the existing image tool success-response prefix inside a tool_…

### #178 — [[deepeyes] feat: support normalized bounding boxes for image zoom](https://github.com/verl-project/verl-recipe/pull/178)
- **作者**: archted  **时间**: 2026-11-22 11:28 CST
- **摘要**: Convert 0-1000 bounding boxes to pixel coordinates before applying the existing image crop and resize policy. Add a tool schema and an optional TOOL_CONFIG_PATH override, preserving the pixel default and public tool name.  Cover coordinate conversion, real image loading, invalid boxes, and config se…

### #177 — [[recipe] feat: add Qwen3.5-397B model DeepSWE recipe](https://github.com/verl-project/verl-recipe/pull/177)
- **作者**: askhat-g  **时间**: 2026-11-22 09:25 CST
- **摘要**: ## Summary  Two additions to the `deepswe_tpu` recipe:  1. A training recipe for `Qwen3.5-397B-A17B` at 64k context on TPU v7x, built on top of    `run_qwen3_5_35b_deepswe.sh`. 2. Filter out out 54 R2E-Gym tasks in `prepare_data.py` whose own gold patch does not reproduce the    expected test result…
