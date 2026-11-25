# OpenVINO 发布管理
以下流程描述了 OpenVINO 发布版本的管理方法。

## 发布里程碑
- 规划
- 执行（开发新功能）
- 稳定化（功能冻结、代码冻结里程碑）
- 验证
- 分发

### 规划
此阶段耗时 2-4 周，包括梳理待办事项、排定优先级、分析，以及开发者按发布经理指定的时间线做出承诺。

### 执行（开发新功能）
- [OpenVINO 贡献指南](https://github.com/openvinotoolkit/openvino/blob/master/CONTRIBUTING.md)
- [代码贡献指南](https://docs.openvino.ai/2026/about-openvino/contributing/code-contribution-guide.html)
- [OpenVINO 新手友好 Issue](https://github.com/openvinotoolkit/openvino/issues/17502)

### 稳定化（功能冻结、代码冻结里程碑）
- **功能冻结**：此里程碑确保在某一时间点后不再向软件添加新功能。这让开发团队能专注于稳定和优化现有功能、修复缺陷、提升性能，而无需承担引入新问题的风险。
- **代码冻结**：此里程碑标志着除关键缺陷修复外，不再允许新的代码变更。这有助于确保最终产品稳定可靠，因为它将最后一刻变更引入新缺陷或不稳定的风险降到最低。

### 发布验证
- 这是一个持续进行的流程，按测试类型定期执行：每晚、每两周、每周。
- 代码冻结后，测试团队可执行最终回归测试，确保近期变更未引入新缺陷，且软件满足所需质量标准。

### 分发
- OpenVINO 有多种构建分发类型：常规发布版本、长期支持、预发布版本、每夜构建。详见：[OpenVINO 发布策略](https://docs.openvino.ai/2026/about-openvino/release-notes-openvino/release-policy.html)
- 支持多种分发渠道。在此探索不同选项：[OpenVINO 下载](https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/download.html)