# 注入模板自定义功能 - 需求与技术架构设计文档

## 1. 概述与需求背景
在 SillyTavern (酒馆) 的 `st-chatu8` 插件中，原本在【角色管理】页面下，角色启用列表、通用角色启用列表、通用服装启用列表的 Prompt 文本格式是固定的硬编码模式。

为了使用户能够根据不同的 AI 模型（例如 Standard Chinese、XML 结构化、Markdown 分割、Danbooru Tags 纯英格式等）自由定制和扩展注入 Prompt 的外观与结构，并**彻底解决多角色同时注入时容易造成属性与角色名称混淆、难以解析**的问题，提供**全局注入模板管理系统 v2.0**。

---

## 2. 核心功能与规划 (Roadmap)

### 2.1 快捷添加占位符变量 (Quick Variable Insertion)
- **输入框焦点记忆**：系统自动记忆用户最近一次点击或聚焦的模板文本框（`characterListTemplate`, `innerOutfitTemplate`, `commonCharacterListTemplate`, `enableOutfitListTemplate`）。
- **下拉选单 + 插入按钮**：UI 提供列出所有可用角色与服装占位符（带中文含义）的下拉菜单，点击“插入到当前光标”后自动将选定变量（如 `{nameCN}`, `{facialBack}`, `{outfits}`）插入在对应输入框的光标焦点处。

### 2.2 注入模板方案预设管理 (Preset Management & Multi-Character Isolation)
- **多预设方案切换**：支持用户新建、修改保存、另存为、重命名、恢复默认、导出 `.json`、导入 `.json` 以及删除自定义模板方案。
- **重构多角色隔离快捷预设 (Multi-Character Boundary Presets)**：
  - **默认原生带分割线格式 (`default_chinese`)**：还原原版字段结构，并增加多角色上下隔离边框 `========================================` 明确多角色块界限。
  - **XML 节点角色隔离格式 (`xml_isolated`)**：采用 `<character_card name="{nameCN}">` 显式 XML 结构完全隔离各角色属性，对大模型解析最为友好。
  - **Markdown 独立角色块格式 (`markdown_isolated`)**：以三级标题 `### 👤 角色卡：{nameCN}` 与块分割线 `---` 形成独立角色的展示域。
  - **Tag 定界标签格式 (`tags_scoped`)**：使用 `[CHARACTER_START: {nameCN}]` ... `[CHARACTER_END]` 定界符进行纯英文/Tag 注入。

### 2.3 四大展开模板自定义
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

### 2.4 手动保存与实时预览
- **手动保存原则**：快捷预设填充与输入框打字均只更新 DOM 界面，不自动保存。用户需明确点击【保存】按钮生效。
- **实时预览**：点击“实时预览展开效果”，直接读取当前 DOM 文本域的实时模板字符串，即时渲染样例数据展示。

### 2.5 本地离线缓存与 Git 隔离策略
- 在根目录使用 `.cache_local/` 保存代码副本与 `CHANGELOG_LOCAL.md`。
- 将 `.cache_local/` 加入 `.gitignore` 避免受到 Git 变动或分支切换的污染与影响。

---

## 3. 代码架构与改动逻辑 (Technical Implementation)

### 3.1 属性适配器 (`getCharacterSettingsRoot`)
确保获取到的插件全局设置路径在所有视角模式下都能一致读取。

### 3.2 默认数据初始化 (`ensureInjectionTemplatesInit`)
包含标准中文带隔离模板默认值及多预设存储模型。

### 3.3 变量光标插入器 (`insertVariableToActiveTemplateInput`)
监听 4 个文本框的 `focus` 事件，点击“插入变量”按钮时解析光标起点/终点并拼接字符串。

### 3.4 动态生成函数改造
重构 `generateCharacterListText()`, `generateOutfitEnableListText()`, `generateCommonCharacterListText()`。

### 3.5 UI 结构改动 (`html/settings/character.html`)
增加快捷变量插入下拉框及按钮，更新快捷预设下拉菜单选项。
