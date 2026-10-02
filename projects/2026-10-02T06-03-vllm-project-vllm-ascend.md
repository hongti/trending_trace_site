# vllm-project/vllm-ascend — 动态追踪

> 生成时间: 2026-10-02 14:03 CST

## AI 总结

以下是 GitHub 仓库 **vllm-project/vllm-ascend** 近期动态摘要：

### 📌 Pull Request (PR) 动态
近期的 PR 主要集中在文档优化与国际化建设：
1. **PR #17865：批量翻译文档**
   - 作者：vllm-ascend-ci
   - 摘要：通过自动化工具翻译了 23 个文档文件（包含开发者指南中的自定义算子设计文档等），持续推进中文本地化进程，方便中文开发者阅读与贡献。
2. **PR #17864：完善测试环境前置说明**
   - 作者：li-lizhe
   - 摘要：补充了 `testing.md` 中关于容器测试环境前置条件的说明。此前文档仅介绍了如何启动容器和运行 `pytest`，但缺少必要的环境准备步骤，该 PR 修复了这一文档缺失问题，降低了开发者的测试门槛。

### 🐛 Issue 动态
- 本期提供的动态中未包含近期的 Issue 信息。

### 🚀 Release 动态
- 本期未发布新的版本。

---

## 🔀 Pull Requests

### #17865 — [[Doc] Translated Doc files 2026-10-01](https://github.com/vllm-project/vllm-ascend/pull/17865)
- **作者**: vllm-ascend-ci  **时间**: 2026-10-01 13:45 CST
- **标签**: documentation
- **摘要**: ## Auto-Translation Summary  Translated **23** file(s):  - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/developer_guide/Design_Documents/add_custom_aclnn_op.po</code> - <code>/home/runner/_work/vllm-ascend/vllm-ascend/docs/source/locale/zh_CN/LC_MESSAGES/deve…

### #17864 — [[Doc][Misc] Clarify container test environment prerequisites in testing.md](https://github.com/vllm-project/vllm-ascend/pull/17864)
- **作者**: li-lizhe  **时间**: 2026-10-01 13:43 CST
- **标签**: documentation
- **摘要**: ### What this PR does / why we need it?  Fixes the first gap reported in #17718: `docs/source/developer_guide/contribution/testing.md` explains how to start the container and run `pytest`, but not the environment prerequisites, so a new contributor has to rediscover them by trial and error.  This PR…
