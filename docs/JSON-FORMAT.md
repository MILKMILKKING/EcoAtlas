# 生态系统演化可视化 - JSON导入格式说明

## 概述

本文档说明如何编写 JSON 文件导入生态系统演化可视化工具。

> **💡 核心功能**
> 
> - **智能坐标计算**：只需指定 `layerId`，系统会自动将元素放置到正确的层级位置
> - **可选精确定位**：如需精确控制，可手动指定 `x`、`y` 坐标
> - **增量导入**：可向现有项目追加元素，无需重新创建
> - **画布自动扩展**：导入数据超出范围时自动扩大画布

---

## 坐标系统说明

### 画布坐标结构

```
       ↑ Y轴（越小越靠上）
       │
       │    padding.top = 50
       │    ┌──────────────────────────────────────────────┐
       │    │                   标题区域                    │
       │    ├──────────────────────────────────────────────┤
       │    │  规则标准层 (layerId=4)   Y ≈ 80~150        │
       │    │  ─────────────────────────────────────────── │
       │    │  场景入口层 (layerId=3)   Y ≈ 150~220       │
       │    │  ─────────────────────────────────────────── │
       │    │  基础设施层 (layerId=2)   Y ≈ 220~300       │
       │    │  ─────────────────────────────────────────── │
       │    │  要素层     (layerId=1)   Y ≈ 300~400       │
       │    └──────────────────────────────────────────────┘
       │         │         │         │         │
       │       萌芽期    探索期    成长期    成熟期
       │       X:70~250  X:250~430 X:430~610 X:610~790
       │
       └──────────────────────────────────────────────────────→ X轴
                padding.left = 70
```

### X轴 - 发展阶段对照表

以标准 4 阶段、800px 画布为例：

| 阶段 | X 起点 | X 终点 | 推荐 X 值（中心） |
|------|--------|--------|------------------|
| 萌芽期（阶段1） | 70 | 250 | **160** |
| 探索期（阶段2） | 250 | 430 | **340** |
| 成长期（阶段3） | 430 | 610 | **520** |
| 成熟期（阶段4） | 610 | 790 | **700** |

> 计算公式：`X = padding.left + (阶段序号 - 0.5) × 阶段宽度`
> 
> 例：3阶段时 `阶段宽度 = (800 - 70 - 25) / 3 ≈ 235`

### Y轴 - 价值层级对照表

以标准 4 层级、450px 画布为例（层级从下往上排列）：

| 层级 | layerId | Y 起点 | Y 终点 | 推荐 Y 值（中心） |
|------|--------|--------|--------|------------------|
| 规则标准层 | 4 | 50 | 135 | **90** |
| 场景入口层 | 3 | 135 | 220 | **175** |
| 基础设施层 | 2 | 220 | 305 | **260** |
| 要素层 | 1 | 305 | 390 | **350** |

> 注意：Y轴**数值越小越靠上**，规则层在顶部所以 Y 值最小

### 快速定位参考表

**标准画布（800×450，4阶段4层级）的推荐坐标：**

| 位置描述 | X | Y |
|---------|---|---|
| 萌芽期-要素层 | 160 | 350 |
| 萌芽期-基础设施层 | 160 | 260 |
| 探索期-场景层 | 340 | 175 |
| 成长期-规则层 | 520 | 90 |
| 成长期-基础设施层 | 520 | 260 |
| 成熟期-场景层 | 700 | 175 |

---

## JSON结构总览

```json
{
  "projectName": "项目名称",
  "ecosystemName": "生态名称",
  "singularityName": "原爆点名称",
  "singularityDesc": "原爆点描述",
  "phases": [...],           // 发展阶段
  "valueLayers": [...],      // 价值层级
  "attributeTypes": [...],   // 属性类型
  "invariants": [...],       // 不动点（可指定x,y）
  "entities": [...],         // 公司/载体（可指定x,y）
  "connections": [...]       // 连线关系
}
```

---

## 各字段详细说明

### 1. 基本信息

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| projectName | string | ✅ | 项目名称 |
| ecosystemName | string | ✅ | 生态名称（画布顶部标题） |
| singularityName | string | - | 原爆点名称 |
| singularityDesc | string | - | 原爆点描述 |

---

### 2. phases - 发展阶段

定义横向时间轴，从左到右排列。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | number | ✅ | 唯一ID（1, 2, 3...） |
| name | string | ✅ | 阶段名称 |
| startYear | number | ✅ | 起始年份 |
| endYear | number | - | 结束年份 |

```json
"phases": [
  { "id": 1, "name": "萌芽期", "startYear": 2020, "endYear": 2022 },
  { "id": 2, "name": "探索期", "startYear": 2022, "endYear": 2024 },
  { "id": 3, "name": "成长期", "startYear": 2024, "endYear": 2026 }
]
```

**X轴对应（3阶段标准画布）：**
- 萌芽期：X ≈ 70~305，中心 **185**
- 探索期：X ≈ 305~540，中心 **420**
- 成长期：X ≈ 540~775，中心 **655**

---

### 3. valueLayers - 价值层级

定义纵向层级背景，数组顺序**从下到上**：第一个在最下方。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | number | ✅ | 唯一ID（1, 2, 3...） |
| name | string | ✅ | 层级名称 |
| color | string | ✅ | 颜色代码（如 #10B981） |
| visiblePhases | array | - | 显示在哪些阶段，填 phase id 数组 |

**推荐颜色和 Y 坐标（4层级450高画布）：**

| 层级 | id | 颜色 | Y 范围 | 推荐 Y |
|------|-----|------|---------|---------|
| 要素层 | 1 | `#10B981` 绿 | 305~390 | **350** |
| 基础设施层 | 2 | `#F59E0B` 橙 | 220~305 | **260** |
| 场景入口层 | 3 | `#EC4899` 粉 | 135~220 | **175** |
| 规则标准层 | 4 | `#3B82F6` 蓝 | 50~135 | **90** |

```json
"valueLayers": [
  { "id": 1, "name": "要素层", "color": "#10B981", "visiblePhases": [1,2,3] },
  { "id": 2, "name": "基础设施层", "color": "#F59E0B", "visiblePhases": [1,2,3] },
  { "id": 3, "name": "场景入口层", "color": "#EC4899", "visiblePhases": [2,3] },
  { "id": 4, "name": "规则标准层", "color": "#3B82F6", "visiblePhases": [3] }
]
```

---

### 4. attributeTypes - 属性类型

公司可选择的分类标签。

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | ✅ | 英文ID |
| name | string | ✅ | 显示名称 |
| color | string | ✅ | 颜色代码 |

```json
"attributeTypes": [
  { "id": "element", "name": "要素", "color": "#10B981" },
  { "id": "infra", "name": "基础设施", "color": "#F59E0B" },
  { "id": "scene", "name": "场景", "color": "#EC4899" },
  { "id": "rule", "name": "规则", "color": "#3B82F6" }
]
```

---

### 5. invariants - 不动点

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | - | 唯一ID（用于连线时需指定） |
| typeId | string | ✅ | 符号类型（见下表） |
| name | string | ✅ | 名称 |
| **layerId** | number | **推荐** | 所属层级 ID，系统会自动计算正确的 Y 坐标 |
| x | number | - | X坐标（可选，不指定则自动分配） |
| y | number | - | Y坐标（可选，不指定则根据 layerId 自动计算） |
| note | string | - | 注释 |

> **🎯 推荐做法**：只指定 `layerId`，让系统自动计算坐标。同一层级的多个元素会自动水平分散。

**符号类型：**
| typeId | 符号 | 默认颜色 |
|--------|------|---------|
| triangle | △ | 橙色 #F97316 |
| circle | ○ | 紫色 #8B5CF6 |
| square | □ | 青色 #06B6D4 |
| cross | × | 红色 #EF4444 |
| diamond | ◇ | 黄绿 #84CC16 |

```json
"invariants": [
  { 
    "id": "inv1",
    "typeId": "triangle", 
    "name": "算力", 
    "layerId": 1,
    "note": "核心生产要素，系统会自动放到要素层"
  },
  { 
    "id": "inv2",
    "typeId": "circle", 
    "name": "数据", 
    "layerId": 2,
    "note": "自动放到基础设施层"
  }
]
```

---

### 6. entities - 公司/载体

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | - | 唯一ID（用于连线时需指定） |
| name | string | ✅ | 公司名称 |
| **layerId** | number | **推荐** | 所属层级 ID，系统会自动计算正确的 Y 坐标 |
| x | number | - | X坐标（可选，不指定则自动分配） |
| y | number | - | Y坐标（可选，不指定则根据 layerId 自动计算） |
| primaryColor | string | - | 主颜色 |
| attributes | array | - | 属性ID数组 |
| note | string | - | 注释 |

> **🎯 推荐做法**：只指定 `layerId`，让系统自动计算坐标。

```json
"entities": [
  {
    "id": "nvidia",
    "name": "NVIDIA",
    "layerId": 1,
    "primaryColor": "#76B900",
    "attributes": ["element"],
    "note": "GPU算力龙头，自动放到要素层"
  },
  {
    "id": "openai",
    "name": "OpenAI",
    "layerId": 3,
    "primaryColor": "#412991",
    "attributes": ["infra", "scene"],
    "note": "自动放到场景入口层"
  }
]
```

---

### 7. connections - 连线

如需连线，必须为 invariants 和 entities 指定 id。

| 字段 | 类型 | 说明 |
|------|------|------|
| sourceType | string | "entity" 或 "invariant" |
| sourceId | string | 起点元素 ID |
| targetType | string | "entity" 或 "invariant" |
| targetId | string | 终点元素 ID |

```json
"connections": [
  { "sourceType": "invariant", "sourceId": "inv1", "targetType": "entity", "targetId": "nvidia" }
]
```

---

## 完整示例（推荐-只指定layerId）

```json
{
  "projectName": "AI生态分析",
  "ecosystemName": "AI大模型生态",
  "singularityName": "Transformer架构",
  "singularityDesc": "消除序列并行化约束",
  
  "phases": [
    { "id": 1, "name": "萌芽期", "startYear": 2017, "endYear": 2020 },
    { "id": 2, "name": "探索期", "startYear": 2020, "endYear": 2022 },
    { "id": 3, "name": "爆发期", "startYear": 2022, "endYear": 2024 }
  ],
  
  "valueLayers": [
    { "id": 1, "name": "要素层", "color": "#10B981", "visiblePhases": [1,2,3] },
    { "id": 2, "name": "基础设施层", "color": "#F59E0B", "visiblePhases": [1,2,3] },
    { "id": 3, "name": "场景入口层", "color": "#EC4899", "visiblePhases": [2,3] }
  ],
  
  "attributeTypes": [
    { "id": "compute", "name": "算力", "color": "#10B981" },
    { "id": "model", "name": "模型", "color": "#F59E0B" },
    { "id": "product", "name": "产品", "color": "#EC4899" }
  ],
  
  "invariants": [
    { "id": "inv_compute", "typeId": "triangle", "name": "算力", "layerId": 1, "note": "核心生产要素" },
    { "id": "inv_data", "typeId": "circle", "name": "数据", "layerId": 1, "note": "模型训练的燃料" },
    { "id": "inv_model", "typeId": "square", "name": "基础模型", "layerId": 2, "note": "大模型能力" }
  ],
  
  "entities": [
    { "id": "nvidia", "name": "NVIDIA", "layerId": 1, "primaryColor": "#76B900", "attributes": ["compute"], "note": "GPU算力龙头" },
    { "id": "openai", "name": "OpenAI", "layerId": 2, "primaryColor": "#412991", "attributes": ["model", "product"], "note": "GPT系列模型" },
    { "id": "msft", "name": "微软", "layerId": 3, "primaryColor": "#00A4EF", "attributes": ["compute", "product"], "note": "Azure+Copilot" },
    { "id": "google", "name": "Google", "layerId": 2, "primaryColor": "#4285F4", "attributes": ["compute", "model"], "note": "Gemini模型" }
  ],
  
  "connections": [
    { "sourceType": "invariant", "sourceId": "inv_compute", "targetType": "entity", "targetId": "nvidia" },
    { "sourceType": "invariant", "sourceId": "inv_model", "targetType": "entity", "targetId": "openai" },
    { "sourceType": "entity", "sourceId": "openai", "targetType": "entity", "targetId": "msft" }
  ]
}
```

---

# 增量导入 JSON 格式说明

## 概述

增量导入用于向**现有项目**追加新元素，而不会覆盖已有内容。

**使用场景：**
- 分批添加研究内容
- 通过 AI 生成补充数据后追加
- 渐进式完善生态图

## 增量导入 JSON 结构

```json
{
  "invariants": [...],    // 要新增的不动点
  "entities": [...],      // 要新增的公司
  "connections": [...]    // 要新增的连接（可选）
}
```

> **✨ 智能定位**
> 
> - 增量导入**不需要** `projectName`、`phases`、`valueLayers` 等配置
> - 只需要提供 `invariants`、`entities`、`connections` 中的任意一项或多项
> - 新元素会自动生成新 ID，不会与现有元素冲突
> - **只需指定 `layerId`**，系统会自动计算正确位置并分散布局
> - 如果手动指定了坐标，会使用你指定的位置

## 增量导入定位建议

**推荐做法**：只指定 `layerId`，让系统根据现有元素自动计算新元素的位置。

如需精确控制位置，可以：
1. 查看现有元素位置
2. 手动指定 `x`、`y` 坐标

## 增量导入示例

### 示例1：追加公司（推荐-只指定layerId）

```json
{
  "entities": [
    {
      "name": "Claude",
      "layerId": 2,
      "primaryColor": "#D97706",
      "attributes": ["model"],
      "note": "Anthropic的大模型，系统自动放到基础设施层"
    },
    {
      "name": "Gemini",
      "layerId": 3,
      "primaryColor": "#4285F4",
      "attributes": ["model", "product"],
      "note": "自动放到场景入口层"
    }
  ]
}
```

### 示例2：追加不动点和关联公司

```json
{
  "invariants": [
    {
      "id": "new_inv_multimodal",
      "typeId": "diamond",
      "name": "多模态",
      "layerId": 3,
      "note": "图文音视频融合，自动放到场景层"
    }
  ],
  "entities": [
    {
      "id": "new_sora",
      "name": "Sora",
      "layerId": 3,
      "primaryColor": "#00A67E",
      "note": "OpenAI视频生成，自动放到场景层"
    }
  ],
  "connections": [
    {
      "sourceType": "invariant",
      "sourceId": "new_inv_multimodal",
      "targetType": "entity",
      "targetId": "new_sora"
    }
  ]
}
```

### 示例3：仅追加连接关系

假设现有项目已有 id 为 "openai" 和 "nvidia" 的元素：

```json
{
  "connections": [
    {
      "sourceType": "entity",
      "sourceId": "openai",
      "targetType": "entity",
      "targetId": "nvidia"
    }
  ]
}
```

---

## 常见问题

**Q: 导入后元素位置不对怎么办？**
A: 用鼠标拖拽调整，或使用「空格+拖拽」平移画布查看全局。

**Q: 如何让公司显示渐变色？**
A: 给 `attributes` 数组添加2个及以上的属性ID。

**Q: 增量导入会覆盖现有内容吗？**
A: 不会。增量导入只追加新元素，现有内容保持不变。

**Q: 增量导入的连接找不到元素怎么办？**
A: 确保 `sourceId` 和 `targetId` 与现有元素或新增元素的 ID 匹配。

---

## 颜色参考

| 用途 | 颜色 | 代码 |
|------|------|------|
| 要素/资源 | 🟢 绿色 | #10B981 |
| 基础设施 | 🟠 橙色 | #F59E0B |
| 场景/产品 | 🩷 粉色 | #EC4899 |
| 规则/标准 | 🔵 蓝色 | #3B82F6 |
| 技术/数据 | 🟣 紫色 | #8B5CF6 |
| 平台 | 🩵 青色 | #06B6D4 |
