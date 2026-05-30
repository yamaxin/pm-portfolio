---
name: pm-portfolio
description: "产品经理作品集生成器。当用户提到想做作品集、portfolio、找工作、换工作、准备面试材料、简历优化，或上传了简历时立即触发。通过多轮追问补全关键信息，生成一份让HR看到就说'哇'的HTML格式产品经理作品集，内嵌完整可视化编辑系统，支持文字修改、图片上传、模块增删、保存和导出PDF。适用于任何行业和方向的PM，可在 Hermes、OpenClaw、Claude.ai 等任意AI平台使用。关键词：作品集、portfolio、产品经理简历、找工作、换工作、帮我做简历、面试准备、PM作品集、想找新机会。"
---

# 产品经理作品集生成器

## ⚠️ 工作方式（最高优先级，先读此节）

**本 Skill 的工作方式：生成 JSON 数据 + Python 脚本注入模板 = 输出 HTML 文件。**

禁止：
- ❌ 生成、重写或复制任何 HTML/CSS/JS 代码
- ❌ 读取 editor-system.md 或 addel-system.md（不需要）
- ❌ 读取或引用任何已有的作品集 HTML 文件
- ❌ 执行"部署"、"安装"等额外操作
- ❌ 生成 backup 文件

正确流程（严格按此执行）：
1. 收集用户信息（Step 1-3）
2. 生成包含所有字段内容的 JSON（Step 4）
3. 用 Python 脚本把 JSON 注入模板（Step 5）
4. 输出最终 HTML 文件给用户（Step 6）

---

## Step 1 · 读取简历，分析缺口

读取用户上传的简历（无简历时先做基础追问）。

分析以下5个维度：

| 维度 | 充分标准 | 常见缺口 |
|------|---------|---------|
| **A 项目深度** | 有背景+决策逻辑+成果 | 只有结果数字，没有"为什么这样做" |
| **B AI能力** | 有具体工具+具体场景 | 只说"会用AI"，没有细节 |
| **C 商业价值** | 有市场判断+验证结果 | 只有功能描述，没有商业视角 |
| **D 个人标签** | 有他人评价+差异化定位 | 只有自我描述 |
| **E 可视化证明** | 有原型截图或链接 | 纯文字，无视觉证明 |

分析完成后简短告知用户，然后追问缺口。

---

## Step 2 · 多轮追问

- 每轮最多3个问题，B（AI能力）和 D（他人评价）优先
- 用户说"没有"就跳过；信息充分时直接进入 Step 3

追问方向：
1. AI工具：具体用在哪些环节？有没有AI产品设计经验？GitHub项目？
2. 关键决策：最有代表性的项目，最关键的决策是什么？为什么这样而不是另一种方案？
3. 他人背书：同事或研发有没有印象深刻的评价？
4. 差异化：和其他PM最不一样的地方？
5. 投递方向：主要投哪个方向？

---

## Step 3 · 确认内容

整理展示将要生成的内容，等用户确认。用户说"直接生成"时直接开始。

---

## Step 4 · 生成 JSON 数据

读取 `references/fields.md`，按照其中的字段列表，逐一用用户的真实信息填写所有字段。

**JSON 格式：**
```json
{
  "fields": {
    "name": "用户真实姓名",
    "eyebrow": "用户定位标语",
    "quote": "他人评价语句",
    ...（所有字段）
  }
}
```

**填写规则：**
- 所有值是纯文字，不含 HTML 标签
- 数字字段只写数字（如 `"stat1-n": "6"`）
- 没有对应信息的字段写 `""`
- 内容来自当前用户，不得抄任何已有文件的内容

**内容质量要求：**
- 项目描述：有背景 + 关键决策（为什么这样做）+ 策略步骤 + 成果
- 数字有含义（如 `"stat3-l": "累计营收增量贡献"` 配合 `"stat3-n": "千万"` 和 `"stat3-u": "+"`)
- 他人评价作为 `quote` 的主标语
- `eyebrow` 是3-5个关键词，用 ` · ` 分隔

---

## Step 5 · Python 脚本注入模板

JSON 生成完成后，立即执行以下 Python 脚本，把 JSON 注入模板文件：

```python
import re, json

# 读取模板
skill_dir = '/Users/你的用户名/.hermes/skills/pm-portfolio'
template_path = skill_dir + '/assets/portfolio-template.html'
with open(template_path, 'r', encoding='utf-8') as f:
    html = f.read()

# 构造 pe-saved-data 内容（将上一步生成的 JSON 填入此处）
data = {
    "fields": {
        # 将 Step 4 生成的所有字段粘贴到这里
    }
}

# 注入模板（只替换 pe-saved-data 这一个标签）
json_str = json.dumps(data, ensure_ascii=False)
html = re.sub(
    r'<script id="pe-saved-data" type="application/json">.*?</script>',
    '<script id="pe-saved-data" type="application/json">' + json_str + '</script>',
    html,
    flags=re.DOTALL
)

# 同时更新 title 和 nav-brand
name = data['fields'].get('name', '')
if name:
    html = re.sub(r'<title>[^<]*</title>', f'<title>{name} · 产品经理作品集</title>', html)
    html = re.sub(r'(<div class="nav-brand">)[^<]*(</div>)', rf'\g<1>{name} Portfolio\g<2>', html)

# 输出文件
output_path = f'/Users/你的用户名/Desktop/{name}-产品经理作品集.html'
with open(output_path, 'w', encoding='utf-8') as f:
    f.write(html)

print(f'✅ 已生成：{output_path}')
```

**执行要点：**
- `skill_dir` 路径根据实际 Hermes skills 目录调整
- 脚本只替换 `pe-saved-data` 一行，不碰其他任何代码
- 执行成功后输出文件路径

---

## Step 6 · 告知用户

文件生成后告知用户：
1. 在桌面找到 `{姓名}-产品经理作品集.html`，用 Chrome 打开
2. 点击右下角 ✏️ 编辑，点击文字直接修改
3. 💾 保存文件 → 下载新版本；Ctrl+S 快捷键
4. 导出PDF：本地直接生成；在线打印→另存为PDF→勾选"背景图形"

---

## 质量自检（Step 5 执行后确认）

- [ ] 文件已生成，路径正确
- [ ] 用 Chrome 打开，封面显示的是当前用户姓名（不是"姓名"占位符）
- [ ] 内容是当前用户的真实信息，不是任何其他人的数据
- [ ] 编辑按钮可以点击
