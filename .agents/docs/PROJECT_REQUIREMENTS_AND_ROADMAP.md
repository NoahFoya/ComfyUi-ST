# 智绘姬 (ST-Chatu8) 实际需求与最新规划文档 (PROJECT_REQUIREMENTS_AND_ROADMAP)

## 1. 概述与核心定位
**智绘姬 (st-chatu8 / ComfyUi-ST)** 是专为 **SillyTavern (酒馆)** 打造的高阶 AI 绘图与角色提示词 (Prompt) 扩展插件。

### 1.1 核心目标与定位
- **注入目标**：插件将角色与服装数据拼装格式化后，**注入给酒馆 (SillyTavern) 的 LLM 上下文/世界书**，供 LLM 精准解析与索引角色数据（**不直接发送给绘图 Backend**）。
- **核心功能**：**自定义提示词注入模板管理系统 (Injection Template Engine)**。
  - 将过去硬编码（死编码）的角色/服装数据展开格式解耦为**用户可自由编辑、配置与管理的 4 大子模板**。
  - 用户可自由定义注入的文本结构与格式，提高 LLM 的解析与索引效率。

---

## 2. 系统核心功能实现范围 (System Implementation Scope)

本插件需要维护和实现的注入模板功能包含以下 5 大核心模块：

### 2.1 4 大解耦子模板管理 (Four Sub-Templates)
插件提供 4 个可独立配置与编辑的子模板，决定数据注入时的渲染结构：
1. **`characterListTemplate` (角色启用列表模板)**：控制 `{{角色启用列表}}` 宏中单个特定角色的文本展开格式。
2. **`innerOutfitTemplate` (角色专属服装模板)**：控制嵌入特定角色内部 (`{outfits}`) 的专属服装文本格式。
3. **`commonCharacterListTemplate` (通用角色模板)**：控制 `{{通用角色启用列表}}` 宏中通用角色的文本格式。
4. **`enableOutfitListTemplate` (通用服装模板)**：控制 `{{通用服装启用列表}}` 宏中独立启用的通用服装文本格式。

### 2.2 占位符变量系统与 UI 快捷插入按钮
- 为 4 大子模板提供完整的占位符提取支持：
  - **角色占位符**：`{nameCN}`, `{nameEN}`, `{traits}`, `{facial}`, `{facialBack}`, `{upperSFW}`, `{upperSFWBack}`, `{lowerSFW}`, `{lowerSFWBack}`, `{upperNSFW}`, `{upperNSFWBack}`, `{lowerNSFW}`, `{lowerNSFWBack}`, `{outfits}`
  - **服装占位符**：`{nameCN}`, `{nameEN}`, `{outfitName}`, `{upperBody}`, `{upperBodyBack}`, `{fullBody}`, `{fullBodyBack}`
- 在 [character.html](file:///d:/Tools/st-chatu8/html/settings/character.html) 的模板编辑界面中，为各个子模板提供一键点击插入变量的快捷按钮 (`.stchatu8-var-btn`)，自动插入光标位置并支持剪贴板复制。

### 2.3 模板 Preset 方案管理与持久化
- **方案管理**：支持方案 Dropdown 选择、快捷预设载入、新建方案、另存为/更新、重命名、删除、恢复默认。
- **配置持久化**：安全保存与读取至原生扩展设置 `extension_settings[extensionName].injectionTemplates`。

### 2.4 智能消行清洗算法 (`applyInjectionTemplate`)
- 逐行扫描占位符：当某行内所有占位符变量均为空值且无其他有效字符时，自动整行丢弃删行，保持输出清洁，避免在 LLM 上下文中浪费 Token。

### 2.5 实时全量预览 (Live Preview)
- 在设置界面提供 Preview 窗口，分块展示 4 大子模板渲染后的实际输出样例，方便用户直观调试与验证注入格式。

---

## 3. 极简分支策略与 Dev-injectionTemplate 重构计划

### 3.1 极简双分支策略
- **`main` (生产主分支)**：稳定基线。
- **`Dev-injectionTemplate` (开发重构分支)**：当前唯一开发分支，承载自定义注入模板引擎的开发与重构。

### 3.2 Dev-injectionTemplate 待修正与重构项
1. **标准化变量按钮与映射**：核对 `character.html` 变量按钮与 `index.js` 提取函数 (`getCharacterPromptData`/`getOutfitPromptData`)，确保占位符命名（如 `lowerSFW`/`lowerNSFW`/`fullBody`）完全对应。
2. **优化消行清洗算法**：保持模板解析稳定性，避免消行时误删标签结构。
3. **完善 Live Preview**：分块清晰展现 4 大子模板的合成效果。

---

## 4. 领域认知与参考示例 (Domain Background & Reference - 非硬编码内容)

### 4.1 世界书结构参考示例 (仅作使用场景参考，非代码硬编码结构)
用户在世界书中可自由组合注入宏，例如：
```xml
<image_database>
  <character_database>
    <specific_characters>{{角色启用列表}}</specific_characters>
    <template_characters>{{通用角色启用列表}}</template_characters>
  </character_database>
  <outfit_database>
    <common_outfits>{{通用服装启用列表}}</common_outfits>
  </outfit_database>
</image_database>
```

### 4.2 预设变量实际表达含义参考 (领域背景知识)
- **角色 Preset 语义**：包含角色的裸体轮廓/物理特征/静态面部/发型/体型与标识符（人物不包含实际服装 Tags）。
- **服装 Preset 语义**：专门表达服装款式、领口、剪裁与鞋袜配饰。

---

## 5. 本地开发与测试规范 (Verification & Guidelines)
1. 提交修改前运行语法检测：`node --check index.js`
2. 重大修改同步更新 `.agents/docs/` 下的文档。
