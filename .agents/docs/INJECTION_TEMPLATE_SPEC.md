# 提示词注入模板自定义管理系统 - 设计与架构规范 (INJECTION_TEMPLATE_SPEC)

## 1. 概述与核心设计目标
本系统旨在解耦 SillyTavern 插件在将角色设定与服装设定注入 **酒馆 (SillyTavern) LLM 上下文/世界书** 时的文本格式控制。

- **核心目标**：将死编码的角色/服装展开格式解耦为灵活自定义的模板方案，优化 LLM 对角色与服装数据的解析与索引效率。
- **系统职责**：插件负责维护 4 大解耦子模板的配置、占位符替换、智能消行清洗、Presets 方案管理以及 Live Preview 渲染。
- **系统预置方案保护**：系统内置 3 套标准方案（`默认方案`、`Tavern XML 格式`、`Markdown 极简卡片`），系统预置方案不可删除、不可重命名，但可恢复为初始默认内容。

---

## 2. 核心实现架构 (Implementation Architecture)

### 2.1 4 大解耦子模板 Preset 存储结构与内置方案
```json
{
  "injectionTemplates": {
    "currentPresetId": "默认方案",
    "presets": {
      "默认方案": {
        "characterListTemplate": "中文名称：{nameCN}\n英文名称：{nameEN}\n角色特征：{traits}\n五官外貌（正面）：{facial}\n五官外貌（背面）：{facialBack}\n上半身SFW（正面）：{upperSFW}\n上半身SFW（背面）：{upperSFWBack}\n下半身SFW（正面）：{lowerSFW}\n下半身SFW（背面）：{lowerSFWBack}\n上半身NSFW（正面）：{upperNSFW}\n上半身NSFW（背面）：{upperNSFWBack}\n下半身NSFW（正面）：{lowerNSFW}\n下半身NSFW（背面）：{lowerNSFWBack}\n负向提示词：{negative}\n服装列表：\n{outfits}",
        "innerOutfitTemplate": "  中文名称：{nameCN}\n  英文名称：{nameEN}\n  上半身（正面）：{upperBody}\n  上半身（背面）：{upperBodyBack}\n  下半身（正面）：{fullBody}\n  下半身（背面）：{fullBodyBack}",
        "commonCharacterListTemplate": "中文名称：{nameCN}\n英文名称：{nameEN}\n角色特征：{traits}\n五官外貌（正面）：{facial}\n五官外貌（背面）：{facialBack}\n上半身SFW（正面）：{upperSFW}\n上半身SFW（背面）：{upperSFWBack}\n下半身SFW（正面）：{lowerSFW}\n下半身SFW（背面）：{lowerSFWBack}\n上半身NSFW（正面）：{upperNSFW}\n上半身NSFW（背面）：{upperNSFWBack}\n下半身NSFW（正面）：{lowerNSFW}\n下半身NSFW（背面）：{lowerNSFWBack}\n负向提示词：{negative}",
        "enableOutfitListTemplate": "中文名称：{nameCN}\n英文名称：{nameEN}\n上半身（正面）：{upperBody}\n上半身（背面）：{upperBodyBack}\n下半身（正面）：{fullBody}\n下半身（背面）：{fullBodyBack}"
      },
      "Tavern XML 格式": {
        "characterListTemplate": "<character id=\"{nameEN}\" cn=\"{nameCN}\">\n  [Traits] {traits}\n  [Face] Front: {facial} | Back: {facialBack}\n  [Body-SFW] Front: {upperSFW}, {lowerSFW} | Back: {upperSFWBack}, {lowerSFWBack}\n  [Body-NSFW] Front: {upperNSFW}, {lowerNSFW} | Back: {upperNSFWBack}, {lowerNSFWBack}\n  [Negative] {negative}\n  [Outfits]\n{outfits}\n</character>",
        "innerOutfitTemplate": "  <outfit name=\"{nameEN}\" cn=\"{nameCN}\">\n    [Upper] Front: {upperBody} | Back: {upperBodyBack}\n    [Full] Front: {fullBody} | Back: {fullBodyBack}\n  </outfit>",
        "commonCharacterListTemplate": "<common_character id=\"{nameEN}\" cn=\"{nameCN}\">\n  [Traits] {traits}\n  [Face] Front: {facial} | Back: {facialBack}\n  [Body-SFW] Front: {upperSFW}, {lowerSFW} | Back: {upperSFWBack}, {lowerSFWBack}\n  [Negative] {negative}\n</common_character>",
        "enableOutfitListTemplate": "<common_outfit id=\"{nameEN}\" cn=\"{nameCN}\">\n  [Upper] Front: {upperBody} | Back: {upperBodyBack}\n  [Full] Front: {fullBody} | Back: {fullBodyBack}\n</common_outfit>"
      },
      "Markdown 极简卡片": {
        "characterListTemplate": "### 角色：{nameCN} ({nameEN})\n- 特征: {traits}\n- 五官: {facial} | 背面: {facialBack}\n- SFW体型: 上身: {upperSFW} ({upperSFWBack}) | 下身: {lowerSFW} ({lowerSFWBack})\n- NSFW解剖: 上身: {upperNSFW} ({upperNSFWBack}) | 下身: {lowerNSFW} ({lowerNSFWBack})\n- 负面提示词: {negative}\n- 服装列表:\n{outfits}",
        "innerOutfitTemplate": "  * 服装: {nameCN} ({nameEN})\n    - 上身: {upperBody} | 背面: {upperBodyBack}\n    - 下身: {fullBody} | 背面: {fullBodyBack}",
        "commonCharacterListTemplate": "### 通用角色：{nameCN} ({nameEN})\n- 特征: {traits}\n- 五官: {facial} | 背面: {facialBack}\n- SFW体型: 上身: {upperSFW} ({upperSFWBack}) | 下身: {lowerSFW} ({lowerSFWBack})\n- 负面提示词: {negative}",
        "enableOutfitListTemplate": "### 通用服装：{nameCN} ({nameEN})\n- 上身: {upperBody} | 背面: {upperBodyBack}\n- 下身: {fullBody} | 背面: {fullBodyBack}"
      }
    }
  }
}
```

### 2.2 占位符变量汇总对照表
| 占位符 | 对应领域含义说明 |
| :--- | :--- |
| `{nameCN}` | 中文名称 |
| `{nameEN}` | 英文标识符 |
| `{traits}` | 角色特征 (作品来源/核心概念标签/体型与年龄定位等) |
| `{facial}` | 正面面部静态特征 (发型长度/发色/瞳色/耳朵/嘴部等) |
| `{facialBack}` | 背面面部特征 (后发长度/辫子背面/颈部特征) |
| `{upperSFW}` | 裸体状态下的上半身轮廓与物理特征 |
| `{upperSFWBack}` | 背部骨骼与线条描述 |
| `{lowerSFW}` | 下半身腿部比例与轮廓描述 |
| `{lowerSFWBack}` | 臀部与双腿背面轮廓描述 |
| `{upperNSFW}` | 赤裸状态下的乳房解剖细节 |
| `{upperNSFWBack}` | 背部写实解剖细节 |
| `{lowerNSFW}` | 隐私部位标准解剖标签 |
| `{lowerNSFWBack}` | 臀部与下半身背面写实解剖细节 |
| `{negative}` | 角色负面提示词 (Negative Prompt) |
| `{outfits}` | 角色专属服装 (`innerOutfitTemplate`) 展开后的放置点 |
| `{upperBody}` | 服装上半身款式、领口、袖子、材质与剪裁细节 |
| `{upperBodyBack}` | 服装上半身背面结构与图案细节 |
| `{fullBody}` / `{lowerBody}` | 服装下半身款式（裙/裤）、鞋袜与腿部配饰 |
| `{fullBodyBack}` / `{lowerBodyBack}` | 服装下半身背面剪裁与细节描述 |

---

## 3. 智能消行清洗算法 (`applyInjectionTemplate`)

```javascript
function applyInjectionTemplate(template, data) {
  if (!template) return "";
  const lines = template.split(/\r?\n/);
  const resultLines = [];

  for (let line of lines) {
    const placeholders = line.match(/\{[a-zA-Z0-9_$]+\}/g);
    if (placeholders && placeholders.length > 0) {
      let hasValue = false;
      for (const ph of placeholders) {
        const key = ph.slice(1, -1);
        const val = data[key] !== void 0 && data[key] !== null ? String(data[key]).trim() : "";
        if (val !== "") hasValue = true;
        line = line.split(ph).join(val);
      }
      if (!hasValue) {
        // 当整行占位符变量全空且去除空白标点后无实质文字时删行
        const textOnly = line.replace(/[\s:：,，\-–—|]/g, "");
        if (textOnly === "") {
          continue;
        }
      }
    }
    resultLines.push(line);
  }
  return resultLines.join("\n");
}
```

---

## 4. 参考补充说明
- **系统预置保护**：`SYSTEM_PRESET_IDS = ["默认方案", "Tavern XML 格式", "Markdown 极简卡片"]`，禁止用户对系统方案执行删除与重命名，确保插件始终拥有可用方案。
- **变量数据来源**：角色预设数据仅表达本体物理特征，服装预设表达衣物样式。
