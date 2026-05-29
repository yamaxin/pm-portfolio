# 设计规范参考

读取时机：生成HTML的CSS部分时查阅。

---

## 配色系统（蓝紫青家族）

```css
/* CSS变量定义 */
:root {
  --green: #4F6EF7;       /* 主蓝：导航/标签/按钮/section标签 */
  --green-lt: #EEF0FF;    /* 极淡蓝白：浅色背景 */
  --green-md: #C7D2FE;    /* 中蓝：边框/分割线 */
  --ink: #1A2035;         /* 深蓝墨：正文（不用纯黑）*/
  --ink-2: #374151;
  --ink-3: #6B7280;
  --paper: #FFFFFF;
  --rule: rgba(79,110,247,0.10);
}

/* 深色区块用渐变，全局禁用纯黑做背景 */
/* 引用块 */    background: linear-gradient(135deg, #2D3FBF, #4F6EF7);
/* stat暗卡 */  background: linear-gradient(135deg, #2D3FBF, #7B5CF5);
/* stat青卡 */  background: linear-gradient(135deg, #4F6EF7, #06B6D4);
/* GitHub卡1 */ background: linear-gradient(135deg, #2D3FBF, #4F6EF7);
/* GitHub卡2 */ background: linear-gradient(135deg, #0A7B90, #06B6D4);
/* Market背景*/ background: linear-gradient(160deg, #1E1B6E, #2D3FBF, #1E1B6E);
```

### 各区域专属颜色

| 区域 | 颜色 | 值 |
|------|------|-----|
| 主色（导航/按钮）| 主蓝 | `#4F6EF7` |
| 深色卡片 | 深蓝 | `#2D3FBF` |
| Market背景 | 靛蓝 | `#1E1B6E` |
| 步骤序号/AI思考 | 紫 | `#7B5CF5` |
| GitHub卡2/工具卡3 | 青 | `#06B6D4` |
| 职业轨迹（唯一暖色）| 琥珀橙 | `#F59E0B` |
| 页面背景 | 极淡蓝白 | `#F4F6FF` 或 `#f5f8ff` |

### 禁止事项
- ❌ 纯黑 `#000`/`#111` 做大面积背景
- ❌ 深绿/墨绿色系
- ❌ 通用字体（Inter/Roboto/Arial）
- ❌ 超过3种主色混用

---

## 字体（Google Fonts CDN引入）

```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,700;0,800;1,700&family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
```

| 用途 | 字体 |
|------|------|
| 大标题/姓名 | `Playfair Display`（衬线） |
| 正文/描述 | `DM Sans`（无衬线）|
| 标签/代码/数据 | `DM Mono`（等宽）|

---

## 打印样式（导出PDF用）

```css
@media print {
  #pe-toolbar, #pe-toast,
  .pe-del-btn, .pe-card-del, .pe-add-btn, .pe-card-add-btn,
  .pe-tag-add-btn, .pe-tag-del, .pe-proto-del, .gh-edit-url-btn,
  #gh-url-dlg { display: none !important; }
  [data-field]::after { display: none !important; }
  [data-field] { background: transparent !important; box-shadow: none !important; }
  nav { display: none !important; }
  * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
  .proj { page-break-inside: avoid; break-inside: avoid; }
  .hero { min-height: auto !important; padding-top: 20px !important; }
  .section { padding: 48px 0 !important; }
}
```
