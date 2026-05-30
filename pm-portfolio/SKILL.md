---
name: pm-portfolio
description: "产品经理作品集生成器。当用户提到想做作品集、portfolio、找工作、换工作、准备面试材料、简历优化，或上传了简历时立即触发。通过多轮追问补全关键信息，生成一份让HR看到就说'哇'的HTML格式产品经理作品集，内嵌完整可视化编辑系统，支持文字修改、图片上传、模块增删、保存和导出PDF。适用于任何行业和方向的PM，可在 Hermes、OpenClaw、Claude.ai 等任意AI平台使用。关键词：作品集、portfolio、产品经理简历、找工作、换工作、帮我做简历、面试准备、PM作品集、想找新机会。"
---

# 产品经理作品集生成器

## ⚠️ 输出方式（最高优先级，先读此节）

**本 Skill 的输出：生成一个 HTML 文件，直接提供给用户下载。**

禁止：
- ❌ 读取、引用或复制任何已有文件（含环境中其他 HTML 文件）
- ❌ 把已有文件当作本次输出结果
- ❌ 执行"部署"、"安装"等额外操作
- ❌ 生成 backup 文件或任何附属文件
- ❌ 用 `<script src="...">` 引入外部 CDN（PDF库必须用动态加载方式）

正确做法：
- ✅ 根据当前对话中用户信息，从零生成全新 HTML
- ✅ HTML = **用户真实内容** + **从参考文件完整复制的编辑系统代码**
- ✅ 将 HTML 作为文件直接输出，文件名：`{姓名}-产品经理作品集.html`

---

## 工作流程

```
Step 1  读取简历 → 缺口分析
Step 2  多轮追问 → 补全信息
Step 3  确认内容 → 获得用户OK
Step 4  读取参考文件 → 生成 HTML → 自检 → 输出
Step 5  告知用户下载和使用方法
```

---

## Step 1 · 读取简历，分析缺口

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

## Step 4 · 生成 HTML 文件

### 第一步：读取参考文件（必须全部读取，不可跳过）

按顺序读取以下文件：

1. **`references/design-system.md`** — 包含：配色、字体规范、**完整页面CSS**（最后一章节）
2. **`references/editor-system.md`** — 包含：字段标记规则、Bug防范、**编辑器CSS**、环境检测脚本、工具栏HTML、PE主脚本、图片上传脚本
3. **`references/addel-system.md`** — 包含：7个增删系统脚本（必须全部复制）

**如果任一文件无法读取，停止生成，告知用户。**

### 第二步：内容正确性检查

生成前自问（任一为否则停止）：
- 封面姓名是当前用户的姓名吗？
- 所有项目来自当前用户的简历吗？
- 有没有从已有文件复制任何内容？

### 第三步：生成规则

**规则1：CSS和HTML结构来源如下（禁止自行设计，必须按此来源）：**

- **页面CSS**：完整复制 `references/design-system.md` 末尾的「完整页面CSS」章节
- **编辑器CSS**：完整复制 `references/editor-system.md` 第4节，紧接在页面CSS之后
- **HTML结构**：5个模块的class名必须与增删系统匹配，关键class名如下：

  ```
  封面标签容器:     class="hero-tags"
  项目卡片:        class="proj reveal"（外层div）
  项目步骤列表:     class="proj-steps"
  功能范围容器:     style含"flex-direction:column"和"gap:8px"的div
  原型图容器:       class="proj-proto"
  AI工具卡网格:     class="ai-grid"
  AI思考卡网格:     class="ai-thinking-grid"
  GitHub卡网格:    class="github-grid"
  市场洞察网格:     class="market-grid"
  职业时间轴:       class="career-timeline"
  成长维度网格:     class="growth-grid"
  ```

**规则2：编辑系统代码必须从参考文件完整复制，禁止自行重写。** 生成的HTML包含11个script标签：

```
script 1:  环境检测         来自 editor-system.md 第5节
           ⚠️ 禁止用<script src="">静态引入CDN，必须用动态加载
script 2:  pe-saved-data    固定：<script id="pe-saved-data" type="application/json">{}</script>
script 3:  PE主脚本         来自 editor-system.md 第7节
           ⚠️ 必须完整复制，禁止重写；特别注意第7节save函数里
              const fullHTML = '<!DOCTYPE html>\n' 中的\n
              必须是反斜杠+n两个字符，不能是真实换行符
script 4:  图片上传         来自 editor-system.md 第8节
script 5:  列表增删系统     来自 addel-system.md
script 6:  卡片级别增删     来自 addel-system.md
script 7:  原型区块删除     来自 addel-system.md
script 8:  GitHub卡片编辑   来自 addel-system.md
script 9:  开源项目增删     来自 addel-system.md
script 10: 职业轨迹增删     来自 addel-system.md
script 11: 标签增删系统     来自 addel-system.md
```

**规则3：pe-saved-data 必须是空 `{}`，禁止携带任何数据。**

**规则4：工具栏内元素（pe-status、pe-divider、pe-btn-*）不能加 data-field。**

### 第四步：作品集内容结构（5个模块）

**MODULE 1 · 封面**
- 姓名（当前用户真实姓名）+ 他人评价主标语
- 3-4个定位标签 + 4-5个核心数字 + 联系方式+GitHub

**MODULE 2 · 代表项目（3-4个）**
每个项目：背景、**关键决策**、策略步骤（3-5步）、成果、原型图占位槽
维度覆盖：①AI/新技术 ②系统设计 ③商业化 ④行业深度

**MODULE 3 · AI工具实践**
- 3个工具卡片（名称+定位+场景+4条用法）
- 3个AI思考（结论+场景+判断标准）
- 2个GitHub项目（链接直接写在href里）

**MODULE 4 · 市场洞察**：3组 市场信号→我的判断→验证结果

**MODULE 5 · 职业轨迹**：时间轴（2-3条）+ 成长维度对比（3个）

### 第五步：输出前质量自检（必须逐项确认，不通过则修正）

**内容正确性：**
- [ ] 封面姓名是当前用户姓名（非杨硕、张三等任何其他人）
- [ ] 所有项目内容来自当前用户简历
- [ ] 无从任何已有文件复制的内容

**编辑系统完整性：**
- [ ] 共11个独立script标签，无 `<script src="">` 静态CDN引入
- [ ] pe-saved-data 内容是 `{}`
- [ ] 包含函数：`toggle` `exitEdit` `save` `init` `pdf` `copyCode`
- [ ] 包含函数：`initProjCards` `initStepLists` `initFeatureBlocks` `initToolCards` `initThinkingCards`
- [ ] 包含函数：`initMarketCards` `initCareerTimeline` `initGrowthGrid`
- [ ] 包含函数：`initGithubCards` `initGithubGrid` `initTagGroups` `initProtoDel`
- [ ] 工具栏内无 data-field 属性
- [ ] save函数里 `'<!DOCTYPE html>\n'` 的 `\n` 是反斜杠+n，不是真实换行

---

## Step 5 · 告知使用方法

文件生成后告知用户：
1. 下载 HTML 文件，用 Chrome 打开
2. 点击右下角 ✏️ 编辑，点击文字直接修改
3. 💾 保存文件 → 下载包含修改的新 HTML；Ctrl+S 快捷键
4. 导出PDF：本地直接生成；在线平台打印→另存为PDF→勾选"背景图形"
