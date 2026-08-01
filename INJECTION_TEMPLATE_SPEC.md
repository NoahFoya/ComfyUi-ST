# 注入模板自定义功能 - 需求与技术架构设计文档

## 1. 概述与需求背景
在 SillyTavern (酒馆) 的 `st-chatu8` 插件中，原本在【角色管理】页面下，角色启用列表、通用角色启用列表、通用服装启用列表的 Prompt 文本格式是固定的硬编码模式（硬编码在 JavaScript 函数中）。

为了使用户能够根据不同的 AI 模型（例如 Standard Chinese、Markdown 结构化格式、Danbooru Tags 纯英格式、紧凑微调格式等）自由定制和扩展注入 Prompt 的外观与结构，需提供**全局注入模板管理系统**。

---

## 2. 核心功能与规划 (Roadmap)

### 2.1 注入模板方案预设管理 (Preset Management)
- **多预设方案切换**：支持用户新建、修改、另存为、重命名、恢复默认、导出 `.json`、导入 `.json` 以及删除自定义模板方案。
- **快捷模板填充 (Quick Presets)**：
  - **标准中文格式**：清晰包含角色名、特征、前后视角、服装信息。
  - **Markdown 列表格式**：以 `- **角色/服装**` 结构化 Markdown 形式输出。
  - **Danbooru Tag 纯英格式**：去除冗余中文，纯英文逗号分隔 Tag。
  - **紧凑微调格式**：一句话高密度 Prompt 格式。

### 2.2 四大展开模板自定义
1. **角色启用列表展开模板 (`{{角色启用列表}}`)**：
   - 对应配置键：`characterListTemplate`
   - 支持动态占位符替换，如 `{nameCN}`, `{nameEN}`, `{traits}`, `{facial}`, `{facialBack}`, `{upperSFW}`, `{lowerSFW}`, `{upperNSFW}`, `{lowerNSFW}`, `{outfits}`。
2. **角色内部服装缩进模板 (`{outfits}`)**：
   - 对应配置键：`innerOutfitTemplate`
   - 当角色分配有特定服装时，替代全局通用服装格式注入到 `{outfits}` 占位符处。
3. **通用角色列表展开模板 (`{{通用角色启用列表}}`)**：
   - 对应配置键：`commonCharacterListTemplate`
4. **通用服装列表展开模板 (`{{通用服装启用列表}}`)**：
   - 对应配置键：`enableOutfitListTemplate`

### 2.3 实时预览与数据安全
- **实时预览**：在UI界面点击“实时预览展开效果”，提供样例数据（如爱丽丝/白衬衫）的无破坏渲染展示。
- **数据结构兼容性**：支持根节点与子页面层级的数据向上/向下平滑迁移。

---

## 3. 代码架构与改动逻辑 (Technical Implementation)

### 3.1 属性适配器 (`getCharacterSettingsRoot`)
确保获取到的插件全局设置路径在所有视角模式（例如通用属性与主设置分离或统一）下都能一致读取。

### 3.2 默认数据初始化 (`ensureInjectionTemplatesInit`)
包含标准中文模板默认值及多预设存储模型。

### 3.3 模板插值引擎 (`applyInjectionTemplate`)
正则全局匹配并优雅替换 `{placeholder}` 占位符。

### 3.4 动态生成函数改造
重构 `generateCharacterListText()`, `generateOutfitEnableListText()`, `generateCommonCharacterListText()`。

### 3.5 UI 结构改动 (`html/settings/character.html`)
增加第 7 个子导航按钮 `<a ... data-sub-tab="ch-sub-tab-injection-templates">注入模板管理</a>` 与控制表单组件。

### 3.6 防缓存拉取优化 (`index.js`)
为 `fetch` 追加 `?_v=${Date.now()}` 保证修改即时刷新。
