---
name: pm-portfolio
description: "产品经理作品集生成器。当用户提到想做作品集、portfolio、找工作、换工作、准备面试材料、简历优化，或上传了简历时立即触发。通过多轮追问补全关键信息，生成一份让HR看到就说'哇'的HTML格式产品经理作品集，内嵌完整可视化编辑系统，支持文字修改、图片上传、模块增删、保存和导出PDF。适用于任何行业和方向的PM，可在 Hermes、OpenClaw、Claude.ai 等任意AI平台使用。关键词：作品集、portfolio、产品经理简历、找工作、换工作、帮我做简历、面试准备、PM作品集、想找新机会。"
---

# 产品经理作品集生成器

帮助产品经理从简历快速生成一份**专业、美观、可直接在浏览器里编辑**的 HTML 作品集。

输出是**单 HTML 文件**，内嵌完整编辑系统——用户打开后可直接点击修改所有文字、上传图片、增删模块，无需任何工具。

---

## 工作流程

遵循以下5步，不要跳步：

```
Step 1  读取简历 → 缺口分析
Step 2  多轮追问 → 补全信息
Step 3  确认内容 → 获得用户OK
Step 4  生成 HTML 作品集
Step 5  说明编辑和导出方法
```

---

## Step 1 · 读取简历，分析缺口

读取用户上传的简历（无简历时先做基础追问）。

分析以下5个维度，判断每个维度的信息是否充分：

| 维度 | 充分标准 | 常见缺口 |
|------|---------|---------|
| **A 项目深度** | 有背景+决策逻辑+成果 | 只有结果数字，没有"为什么这样做" |
| **B AI能力** | 有具体工具+具体场景 | 只说"会用AI"，没有细节 |
| **C 商业价值** | 有市场判断+验证结果 | 只有功能描述，没有商业视角 |
| **D 个人标签** | 有他人评价+差异化定位 | 只有自我描述 |
| **E 可视化证明** | 有原型截图或链接 | 纯文字，无视觉证明 |

分析完成后，**简短告诉用户你看到了什么**，然后开始追问缺口。

---

## Step 2 · 多轮追问

**原则：**
- 每轮最多3个问题，按缺口优先级排序
- B（AI能力）和 D（他人评价）优先——HR最看重
- 用户说"没有"就跳过，不要强求
- **如果简历信息已经充分（5个维度都有），可以直接进入 Step 3**

**优先追问的5个方向：**

1. **AI能力**（必问）：你用过哪些AI工具？具体用在产品工作哪个环节？有没有设计过AI产品？有PRD或原型吗？有GitHub项目？
2. **关键决策**（必问）：你最有代表性的项目，当时最关键的一个产品决策是什么？为什么这样而不是另一种方案？
3. **他人背书**（必问）：同事或研发有没有对你说过让你印象深刻的评价？
4. **差异化**：你和其他PM最不一样的地方是什么？
5. **投递方向**：你现在主要投哪个方向？（影响项目排序）

---

## Step 3 · 确认内容

追问完成后，整理展示将要生成的内容，等用户确认后再生成。用户说"直接生成"时也可直接开始。

---

## Step 4 · 生成 HTML 作品集

### ⚠️ 核心规则：直接使用模板填数据，禁止重写代码

**`assets/portfolio-template.html` 是完整可用的成品模板，包含所有编辑系统、PDF导出修复、样式代码。**

**AI的唯一任务：读取模板，把用户数据填入对应的 `data-field` 字段，输出完整HTML文件。**

**严禁：**
- ❌ 自行重写或拼接 HTML 结构
- ❌ 自行编写任何 `<script>` 标签内容
- ❌ 自行编写 `@media print` CSS
- ❌ 用 Python 分段构建 HTML（会导致 script 标签丢失、JS代码裸露在body里）

**正确做法（3步）：**
```
1. 用 python 读取完整模板：open('assets/portfolio-template.html').read()
2. 用字符串替换把占位内容换成用户真实数据
3. 输出完整 HTML 文件，文件名：{姓名}-产品经理作品集.html
```

---

### 字段替换映射表

替换规则：找到模板里对应 `data-field` 元素的文本内容，替换成用户数据。
用 `>旧内容<` 格式匹配（含尖括号），避免误替换属性值。

#### 封面
| data-field | 模板占位内容 | 替换为 |
|-----------|------------|-------|
| `eyebrow` | `toB产品经理 · AI产品设计者 · Build in Public 践行者` | 用户职位定位 |
| `name` | `姓名` | 真实姓名 |
| `quote` | `评语1！` | 他人评价原话 |
| `quote-author` | `— 研发工程师 A` | 评价来源 |
| `tag1`~`tag4` | `5年+ toB 产品`等 | 个人标签 |
| `phone` | `186-0000-0000` | 手机号 |
| `email` | `18600000000@163.com` | 邮箱 |
| `github` | `github.com/0000` | GitHub链接或空 |
| `stat1-n`~`stat4-n` | `10`、`5`、`千万`、`99.95` | 核心数字 |
| `stat1-u`~`stat4-u` | `年+`、`+`、`+`、`%` | 单位 |
| `stat1-l`~`stat4-l` | `toB 行业产品经验`等 | 数字说明 |
| `stat5` | `xxxxxxx等` | KA客户名称 |
| `stat5-l` | `xxx客户深度服务` | KA说明 |

同时替换 `{{姓名}}` 为真实姓名（出现在`<title>`和`.nav-brand`中）。

#### 项目1（p1-*）
| data-field | 替换为 |
|-----------|-------|
| `p1-b1` | 项目标签 |
| `p1-title` | 项目标题 |
| `p1-sub` | 项目一句话描述 |
| `p1-m1n`/`p1-m1l` ~ `p1-m3n`/`p1-m3l` | 3个核心指标数字+说明 |
| `p1-bg` | 背景+市场判断（2-3句） |
| `p1-s1t`/`p1-s1d` ~ `p1-s3t`/`p1-s3d` | 3个核心决策标题+说明 |
| `p1-p0`/`p1-p1`/`p1-p2` | P0/P1/P2功能范围 |
| `p1-diff` | 竞品核心差异 |
| `p1-cap1`/`p1-cap2` | 原型图说明文字 |
| `p1-outcome` | 项目成果 |
| `outcome_label_1` | 成果标题（保持"项目成果"即可） |

#### 项目2（p4-*）
| data-field | 替换为 |
|-----------|-------|
| `p4-b1`/`p4-b2` | 项目标签（可2个） |
| `p4-title`/`p4-sub` | 标题和描述 |
| `p4-m1n`/`p4-m1l` ~ `p4-m3n`/`p4-m3l` | 3个指标 |
| `p4-bg` | 背景 |
| `p4-k1t`/`p4-k1d` ~ `p4-k3t`/`p4-k3d` | 3个行业知识点标题+说明 |
| `p4-s1t`/`p4-s1d` ~ `p4-s3t`/`p4-s3d` | 3个策略标题+说明 |
| `p4-cap1`/`p4-cap2` | 原型图说明 |
| `p4-outcome` | 项目成果 |

#### AI模块
| data-field | 替换为 |
|-----------|-------|
| `ai-intro` | AI模块介绍 |
| `t1-name`/`t1-role`/`t1-desc` | 工具1名称/定位/描述 |
| `t1-u1`~`t1-u4` | 工具1的4个用法 |
| `t2-name`/`t2-role`/`t2-desc`/`t2-u1`~`t2-u4` | 工具2（同上） |
| `t3-name`/`t3-role`/`t3-desc`/`t3-u1`~`t3-u4` | 工具3（同上） |
| `th1-num`/`th1-title`/`th1-text`/`th1-verdict` | 思考1编号/标题/内容/判断 |
| `th2-*`/`th3-*` | 思考2、3（同上） |
| `gh1-label`/`gh1-name`/`gh1-desc`/`gh_tag_1`~`gh_tag_3` | GitHub项目1 |
| `gh2-label`/`gh2-name`/`gh2-desc`/`gh_tag_4`~`gh_tag_6` | GitHub项目2 |

#### 市场洞察
| data-field | 替换为 |
|-----------|-------|
| `market-intro` | 模块介绍 |
| `m1-signal`/`m1-judge`/`m1-result` | 市场信号/判断/结果 |
| `m2-*`/`m3-*` | 第2、3组（同上） |

#### 职业轨迹
| data-field | 替换为 |
|-----------|-------|
| `c1-period`/`c1-co`/`c1-role`/`c1-title`/`c1-desc` | 第1段职业信息 |
| `ttag_1`/`ttag_4` | 第1段标签 |
| `c2-period`/`c2-co`/`c2-role`/`c2-title`/`c2-desc` | 第2段职业信息 |
| `ttag_6`/`ttag_7` | 第2段标签 |
| `gl_1`~`gl_3` | 3个成长维度标题 |
| `g1-from`/`g1-to` ~ `g3-from`/`g3-to` | 早期→现在状态 |

---

### 生成代码模板（必须使用此函数，禁止直接字符串替换）

```python
import re

with open('/path/to/assets/portfolio-template.html', 'r', encoding='utf-8') as f:
    html = f.read()

def replace_field(html, field, new_content):
    """
    安全替换 data-field 字段内容。
    先隔离 script 标签，防止 JS 代码被误替换导致报错。
    ⚠️ 禁止用 html.replace('>旧内容<', ...) 直接替换——JS里有同名字符串会被破坏。
    """
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

name = '用户姓名'
html = html.replace('{{姓名}}', name)
html = replace_field(html, 'name', name)
html = replace_field(html, 'quote', '他人评价原话')
html = replace_field(html, 'phone', '138xxxx0000')
# ... 其余字段同理（参考上方字段映射表）

with open(name + '-产品经理作品集.html', 'w', encoding='utf-8') as f:
    f.write(html)
```
---

## 质量自检（生成完成后逐项确认）

**生成方式验证（最关键）：**
- [ ] HTML 是通过读取模板文件 + 字符串替换生成的，未手动拼接任何代码
- [ ] 文件中没有裸露在 `<body>` 里的 JS 代码（所有 JS 都在 `<script>` 标签内）
- [ ] `{{姓名}}` 占位符已全部替换（搜索"{{" 应无结果）

**内容完整性：**
- [ ] 所有 `data-field` 占位内容已替换为用户真实信息（无"根据简历生成"、"xxxxxxx"等字样）
- [ ] 封面有他人评价或强有力定位语
- [ ] 每个项目有关键决策内容（非空）
- [ ] GitHub链接已填入（无链接则填"暂无"）

**PDF导出完整性：**
- [ ] `@media print` 块以 `/* ① 页面设置 */` 开头（模板原样，未被修改）
- [ ] `PDF PRINT HANDLER` script 以 `/* ====== PDF PRINT HANDLER` 开头（模板原样）
- [ ] `.hero` 基础CSS中无 `min-height:100vh`
- [ ] `@media print` 隐藏列表包含 `.pe-proto-del`

---

## Step 5 · 告知 PDF 导出方法

生成后告知用户以下导出步骤（**必须按此操作，否则样式异常**）：

```
导出 PDF：
1. 用 Chrome 打开 HTML 文件
2. Ctrl+P（Mac: Cmd+P）打开打印
3. 目标选"另存为PDF"
4. 展开"更多设置"：
   ✅ 勾选"背景图形"——否则渐变色全部消失变白底
   ❌ 取消勾选"页眉和页脚"——否则会出现文件路径和日期
5. 纸张：A4
6. 边距：默认（CSS已设置好15mm，选"默认"或"无"均可）
7. 缩放：100%
```
