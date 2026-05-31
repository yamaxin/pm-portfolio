# 产品经理作品集生成器 Skill

> 帮助产品经理从简历快速生成一份**专业、美观、可在浏览器直接编辑**的 HTML 作品集。

---

## 目录结构

```
pm-portfolio/
├── README.md                        # 本文件
├── SKILL.md                         # Skill 主文件，AI 执行入口
├── assets/
│   └── portfolio-template.html      # 完整作品集模板（含编辑系统）
└── references/
    ├── fields.md                    # 所有 data-field 字段映射表
    ├── editor-system.md             # 编辑系统技术文档
    ├── addel-system.md              # 增删系统技术文档
    └── design-system.md             # 设计规范文档
```

---

## 快速上手

**第一步：安装 Skill**
将整个 `pm-portfolio/` 目录放入你的 Skill 目录（如 `/mnt/skills/user/pm-portfolio/`）。

**第二步：触发 Skill**
对 AI 说任何与作品集相关的词即可触发，例如：
- "帮我做一份产品经理作品集"
- "我想找工作，帮我做个 portfolio"
- "简历优化"、"面试材料"

**第三步：上传简历**
上传你的 PDF 或 Word 简历，AI 会自动分析并多轮追问补全信息。

**第四步：获取 HTML 文件**
AI 生成 `{姓名}-产品经理作品集.html`，用浏览器打开即可使用。

**第五步：编辑并导出最终文档**

编辑完成后，在工具栏选择导出方式：

```
方式 A · 导出纯展示 HTML（推荐）
─────────────────────────────────
点击工具栏 🌐 导出 HTML
→ 下载 {姓名}-产品经理作品集-final.html
→ 无编辑代码，视觉效果与编辑版完全一致
→ 可直接发给 HR，或用浏览器打印成 PDF

方式 B · 导出 PDF
─────────────────────────────────
点击工具栏 📄 导出 PDF，或 Chrome Cmd+P（Mac）/ Ctrl+P（Win）
→ 目标：另存为 PDF
→ ✅ 勾选"背景图形"
→ ❌ 取消"页眉和页脚"
→ 纸张：A4，边距：默认，缩放：100%
注意：PDF 因浏览器打印引擎限制可能有少量空白，属正常现象
```

---

## 作品集功能

### 编辑功能
点击工具栏 **✏️ 编辑** 按钮进入编辑模式：
- **文字编辑**：直接点击任意文字修改
- **图片上传**：点击原型图区域上传截图
- **增删模块**：增删项目卡片、步骤、标签、职业轨迹等
- **删除原型区块**：不需要截图时可整块删除

### 保存功能
- **💾 保存文件**（Hermes/本地环境）：下载包含所有编辑内容的新 HTML 文件
- **📋 复制代码**（Claude.ai 环境）：复制完整 HTML 代码，粘贴到文本编辑器另存为 `.html`

### 导出功能
| 按钮 | 输出 | 适用场景 |
|------|------|---------|
| 🌐 导出 HTML | `{姓名}-产品经理作品集-final.html` | 发给 HR、长期存档、在线分享 |
| 📄 导出 PDF | 通过浏览器打印对话框 | 投递简历附件 |

**导出 HTML** 会剥离所有编辑代码，输出纯展示版，视觉效果与编辑版完全一致。

**导出 PDF** 推荐设置（Chrome）：
```
Ctrl+P → 另存为PDF
✅ 勾选"背景图形"
❌ 取消"页眉和页脚"
纸张：A4，边距：默认，缩放：100%
```

---

## 作品集内容结构

生成的作品集包含以下五个模块：

| 模块 | 内容 |
|------|------|
| 01 封面 | 姓名、定位、他人评价、核心数字、联系方式 |
| 02 代表项目 | 2 个以上项目，含背景、决策、步骤、成果、截图 |
| 03 AI产品思考与工具实践 | 工具卡片、AI思考、GitHub项目 |
| 04 市场洞察与商业判断 | 3组市场信号→判断→验证结果 |
| 05 职业轨迹 | 时间轴、成长维度对比 |

---

## 技术说明

### 核心设计原则
**AI 只填数据，不写代码。** 模板是唯一真相来源，AI 通过字符串替换将用户数据填入 `data-field` 字段，不得自行拼接 HTML 结构或编写 script 内容。

### 替换函数（必须使用）
```python
import re

def replace_field(html, field, new_content):
    """安全替换，先隔离 script 标签防止 JS 代码被误替换"""
    scripts = []
    def save_script(m):
        scripts.append(m.group(0))
        return f'SCRIPT_PLACEHOLDER_{len(scripts)-1}_END'
    html_no_scripts = re.sub(r'<script[^>]*>.*?</script>', save_script, html, flags=re.DOTALL)
    pattern = r'(data-field="' + re.escape(field) + r'"[^>]*>)[^<]*'
    new_html, _ = re.subn(pattern, lambda m: m.group(1) + new_content, html_no_scripts)
    for i, script in enumerate(scripts):
        new_html = new_html.replace(f'SCRIPT_PLACEHOLDER_{i}_END', script)
    return new_html
```

### localStorage 隔离
每个用户的数据存储在独立的 key 下（`pe_portfolio_v1_{姓名}`），不同人的作品集数据不会互相污染。

---

## 已知限制

- PDF 导出因浏览器打印引擎限制，部分页面可能存在少量空白，属于正常现象。建议使用"导出 HTML"获得完整视觉效果。
- 图片以 base64 内联存储，文件较大时建议使用较小尺寸的截图。
- 编辑模式下的修改需点击"保存文件"或"复制代码"才能持久化，仅 localStorage 的自动保存在清除浏览器缓存后会丢失。

---

## 版本历史

| 版本 | 主要变更 |
|------|---------|
| v19 | 导出 HTML 功能修复（PE主脚本误保留问题），新增项目删除原型区块修复 |
| v18 | 新增项目原型区块删除按钮直接注入 |
| v17 | 新增项目插入位置修复（末尾而非首位） |
| v16 | STORE_IMG 命名空间化，旧 key 清除 |
| v15 | 导出 HTML 功能上线（🌐 导出 HTML 按钮） |
| v14 | PDF 空白问题修复（hero 单列、career-section 背景透明） |
| v13 | proto-cap 彻底删除，图片说明不再显示 |
| v12 | localStorage 命名空间化，防止跨用户数据污染 |
| v11 | 新方案：AI填数据而非拼代码，replace_field 安全替换函数 |
