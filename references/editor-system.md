# 编辑系统完整代码参考

生成HTML时，将以下代码块**按顺序、按位置**插入。所有代码直接复制使用，不需要重写。

---

## 1. 可编辑字段标记规则

所有可编辑文字节点必须加双属性：
```html
<元素 data-field="唯一id" data-field-label="显示名">内容</元素>
```

**必须标记的范围（一个不能漏）：**

| 区域 | 必须标记的字段 |
|------|-------------|
| 封面 | 姓名、引用语、评价来源、每个标签、每个数字+单位+说明、联系方式文字 |
| 项目 | 标题、描述、每个指标+说明、badge文字、小节标题、背景段落、步骤序号+标题+说明 |
| 功能范围 | P0/P1/P2标签文字、功能描述 |
| 原型图 | 图片说明（proto-cap）、原型区标题（proto-header）|
| AI模块 | 工具名/定位/描述/每条用法li、思考编号/标题/内容/判断标准 |
| GitHub | 项目类型标签、名称、描述、每个技术标签 |
| 市场 | 信号/判断/结果的标题label和内容、箭头符号、编号 |
| 职业 | 时间段、公司名、职位、阶段标题、描述段落 |
| 成长维度 | 维度标题、早期内容、箭头、现在内容 |

**禁止**：class属性、style属性、HTML结构标签、编辑系统按钮（按钮必须加 `contenteditable="false"`）

---

## 2. 关键Bug防范（已验证）

| Bug | 必须做的事 |
|-----|-----------|
| 标签显示×××文字 | 所有删除/添加按钮加 `contenteditable="false"` + 绝对定位圆形按钮 |
| PE is not defined | pe-saved-data放在PE主script标签之外（body内、工具栏div之前）|
| JS语法错误 | `'<!DOCTYPE html>\n'` 用转义 `\n`，不用真实换行 |
| 图片截断 | proto-img-slot用 `min-height:200px` + `height:auto`，禁止固定高度 |
| 增删按钮被裁切 | 所有增删按钮的父容器不能有 `overflow:hidden` |
| 下载后增删失效 | toggle()开启编辑时必须清除所有dataset init标志（见PE主脚本）|
| Market无增删 | initMarketCards内联实现删除逻辑，不依赖外部addDelBtn函数 |
| GitHub点击跳转 | 编辑模式下click事件统一preventDefault，非编辑模式放行 |

---

## 3. HTML文件结构（插入位置说明）

```
<!DOCTYPE html>
<html>
<head>
  ... 字体CDN ...
  <style>
    ... 页面CSS ...
    /* 编辑器CSS插在这里（见第4节）*/
  </style>
  /* 环境检测脚本插在</head>之前（见第5节）*/
</head>
<body>
  ... 页面内容（所有文字加data-field）...

  /* 工具栏区域插在</body>之前（见第6节，顺序严格）*/
  1. pe-saved-data标签
  2. pe-toolbar div
  3. pe-toast div
  4. PE主脚本
  5. 图片上传脚本
  6. 增删系统脚本（7个）
</body>
</html>
```

---

## 4. 编辑器CSS（插入到 `</style>` 之前）

```css
/* ════════════════════════════════════
   EDITOR SYSTEM — 完全隔离，不影响模板样式
════════════════════════════════════ */

/* 悬浮工具栏 */
#pe-toolbar {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  background: rgba(15,23,42,0.95);
  backdrop-filter: blur(16px);
  border-radius: 100px;
  padding: 8px 8px 8px 20px;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 8px 40px rgba(15,23,42,0.35), 0 0 0 1px rgba(147,197,253,0.15);
  transition: all 0.3s ease;
  font-family: 'DM Sans', sans-serif;
}
#pe-toolbar.is-editing {
  background: rgba(20,50,36,0.97);
  box-shadow: 0 8px 40px rgba(37,99,235,0.3), 0 0 0 1px rgba(110,231,183,0.2);
}
.pe-status {
  font-size: 11px;
  font-weight: 600;
  color: #555;
  letter-spacing: .08em;
  text-transform: uppercase;
  font-family: 'DM Mono', monospace;
  padding-right: 8px;
  transition: color .3s;
}
#pe-toolbar.is-editing .pe-status { color: #93c5fd; }
.pe-divider { width:1px; height:18px; background:rgba(255,255,255,.1); flex-shrink:0; }
.pe-btn {
  height: 36px;
  padding: 0 18px;
  border-radius: 100px;
  border: none;
  font-family: 'DM Sans', sans-serif;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: all .2s;
  white-space: nowrap;
}
.pe-btn-edit  { background:#2563eb; color:#fff; }
.pe-btn-edit:hover  { background:#1d4ed8; }
.pe-btn-edit.active { background:#93c5fd; color:#111; }
.pe-btn-save  { background:rgba(255,255,255,.08); color:#aaa; font-size:12px; }
.pe-btn-save:hover  { background:rgba(255,255,255,.15); color:#fff; }
.pe-btn-copy  { background:rgba(147,197,253,.15); color:#93c5fd; font-size:12px; border:1px solid rgba(147,197,253,.35); }
.pe-btn-copy:hover  { background:rgba(147,197,253,.25); }
.pe-btn-pdf   { background:#fff; color:#111; }
.pe-btn-pdf:hover   { background:#eff6ff; color:#2563eb; }

/* 编辑状态下 data-field 元素的样式 */
.pe-edit-active [data-field] {
  outline: none;
  cursor: text;
  border-radius: 3px;
  transition: background .15s, box-shadow .15s;
}
.pe-edit-active [data-field]:hover {
  background: rgba(37,99,235,.07);
  box-shadow: 0 0 0 1.5px rgba(37,99,235,.3);
}
.pe-edit-active [data-field]:focus {
  background: rgba(37,99,235,.09);
  box-shadow: 0 0 0 2px rgba(26,107,74,.55);
}
/* 悬浮提示 — 用伪元素，不修改DOM */
.pe-edit-active [data-field]:not(:focus)::after {
  content: attr(data-field-label);
  position: absolute;
  top: -20px;
  left: 0;
  font-size: 10px;
  font-weight: 600;
  color: #fff;
  background: #2563eb;
  padding: 1px 7px;
  border-radius: 4px;
  pointer-events: none;
  opacity: 0;
  transition: opacity .15s;
  white-space: nowrap;
  font-style: normal;
  font-family: 'DM Mono', monospace;
  z-index: 100;
  letter-spacing: .06em;
  text-transform: uppercase;
}
.pe-edit-active [data-field]:not(:focus):hover::after { opacity: 1; }
/* 需要relative定位才能显示伪元素 */
.pe-edit-active [data-field] { position: relative; }
/* PE系统内部按钮不参与编辑 */
.pe-del-btn, .pe-card-del, .pe-card-add-btn,
.pe-tag-del, .pe-tag-add-btn, .pe-proto-del,
.pe-add-btn, .gh-edit-url-btn {
  -webkit-user-modify: read-only !important;
  user-modify: read-only !important;
}

/* Toast */
#pe-toast {
  position: fixed;
  bottom: 82px;
  left: 50%;
  transform: translateX(-50%) translateY(8px);
  background: rgba(17,17,17,0.95);
  color: #93c5fd;
  padding: 9px 22px;
  border-radius: 100px;
  font-size: 12px;
  font-weight: 600;
  font-family: 'DM Mono', monospace;
  z-index: 9998;
  opacity: 0;
  transition: opacity .25s, transform .25s;
  pointer-events: none;
  white-space: nowrap;
  max-width: 90vw;
}
#pe-toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

/* 打印时隐藏所有编辑相关元素 */
@media print {
  /* 隐藏编辑相关元素 */
  #pe-toolbar, #pe-toast,
  .pe-del-btn, .pe-card-del, .pe-add-btn,
  .pe-card-add-btn, .pe-tag-add-btn, .pe-tag-del,
  .pe-proto-del, .gh-edit-url-btn, #gh-url-dlg { display: none !important; }

  /* 清除编辑状态样式 */
  [data-field]::after { display: none !important; }
  [data-field] {
    background: transparent !important;
    box-shadow: none !important;
    cursor: default !important;
  }

  /* 隐藏导航 */
  nav.nav { display: none !important; }

  /* 保留背景色（需要浏览器开启"背景图形"） */
  * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }

  /* 分页控制 */
  .proj { page-break-inside: avoid; break-inside: avoid; }
  .section { page-break-before: auto; }
  .hero { min-height: auto !important; padding-top: 20px !important; }
  .section { padding: 48px 0 !important; }

  /* body 背景 */
  body { background: #F4F6FF !important; }
}


/* ── 原型图槽 ── */
.proto-img-slot {
  width: 100%;
  min-height: 200px;
  background: #f8f9fc;
  position: relative;
  cursor: default;
  transition: background .2s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.proto-img-slot img {
  width: 100%;
  height: auto;
  max-width: 100%;
  display: block;
  object-fit: contain;
}
.proto-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  pointer-events: none;
  padding: 48px 24px;
}
.proto-ph-icon { font-size: 32px; opacity: .3; }
.proto-ph-text { font-size: 13px; font-weight: 600; color: #94a3b8; }
.proto-ph-hint { font-size: 11px; color: #cbd5e1; font-family: 'DM Mono', monospace; }

/* 编辑模式下的上传提示 */
.pe-edit-active .proto-img-slot {
  cursor: pointer;
  border: 2px dashed #7B5CF5;
}
.pe-edit-active .proto-img-slot::after {
  content: '📷  点击上传图片';
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
  font-weight: 700;
  color: #7B5CF5;
  background: rgba(240,237,255,0.88);
  font-family: 'DM Sans', sans-serif;
  letter-spacing: .02em;
  opacity: 0;
  transition: opacity .2s;
  z-index: 5;
}
.pe-edit-active .proto-img-slot:hover::after { opacity: 1; }
/* 已有图片时，蒙层更透明 */
.proto-img-slot.has-image {
  min-height: unset;
  background: transparent;
}
.pe-edit-active .proto-img-slot.has-image::after {
  background: rgba(239,246,255,0.65);
}


/* ── 列表增删系统 ── */
.pe-edit-active [data-list] {
  position: relative;
}
/* 每个可删除的item，编辑模式下显示删除按钮 */
.pe-del-btn {
  display: none;
  position: absolute;
  top: -8px;
  right: -8px;
  width: 22px;
  height: 22px;
  background: #ef4444;
  color: #fff;
  border: none;
  border-radius: 50%;
  font-size: 13px;
  line-height: 1;
  cursor: pointer;
  align-items: center;
  justify-content: center;
  z-index: 200;
  padding: 0;
  font-weight: 700;
  box-shadow: 0 2px 10px rgba(239,68,68,.5);
  transition: transform .15s;
  flex-shrink: 0;
}
.pe-del-btn:hover { transform: scale(1.15); }
.pe-edit-active [data-deletable] { position: relative; }
.pe-edit-active [data-deletable]:hover > .pe-del-btn,
.pe-edit-active [data-deletable]:hover .pe-del-btn { display: flex; }

/* 添加按钮 */
.pe-add-btn {
  display: none;
  width: 100%;
  padding: 8px;
  margin-top: 10px;
  background: rgba(79,110,247,.06);
  border: 1.5px dashed rgba(79,110,247,.3);
  border-radius: 8px;
  color: #4F6EF7;
  font-size: 12px;
  font-weight: 600;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .2s;
  text-align: center;
}
.pe-add-btn:hover {
  background: rgba(79,110,247,.1);
  border-color: #4F6EF7;
}
.pe-edit-active .pe-add-btn { display: block; }


/* ── 卡片级别增删 ── */
.pe-card-del {
  display: none;
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 4px 12px;
  background: rgba(239,68,68,0.9);
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
  z-index: 100;
  font-family: 'DM Sans', sans-serif;
  transition: background .15s;
  backdrop-filter: blur(4px);
}
.pe-card-del:hover { background: #ef4444; }
.pe-edit-active .proj,
.pe-edit-active .ai-tool-card,
.pe-edit-active .thinking-card { position: relative; }
.pe-edit-active .pe-card-del { display: block; }

.pe-card-add-btn {
  display: none;
  width: 100%;
  padding: 12px;
  margin-top: 16px;
  background: rgba(79,110,247,.05);
  border: 2px dashed rgba(79,110,247,.25);
  border-radius: 12px;
  color: #4F6EF7;
  font-size: 13px;
  font-weight: 600;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .2s;
  text-align: center;
}
.pe-card-add-btn:hover {
  background: rgba(79,110,247,.1);
  border-color: #4F6EF7;
}
.pe-edit-active .pe-card-add-btn { display: block; }


.pe-proto-del {
  display: none;
  position: absolute;
  top: 16px;
  right: 16px;
  padding: 5px 14px;
  background: rgba(239,68,68,0.85);
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
  z-index: 100;
  font-family: 'DM Sans', sans-serif;
  transition: background .15s;
}
.pe-proto-del:hover { background: #ef4444; }


/* ── GitHub卡片编辑链接按钮 ── */
.gh-edit-url-btn {
  display: none;
  position: absolute;
  bottom: 12px;
  right: 12px;
  padding: 4px 12px;
  background: rgba(255,255,255,0.15);
  border: 1px solid rgba(255,255,255,0.3);
  border-radius: 6px;
  color: #fff;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  font-family: 'DM Mono', monospace;
  letter-spacing: .04em;
  backdrop-filter: blur(4px);
  transition: background .15s;
  z-index: 10;
}
.gh-edit-url-btn:hover { background: rgba(255,255,255,0.25); }
.pe-edit-active .gh-edit-url-btn { display: block; }

/* 编辑模式下 github-card 阻止跳转视觉提示 */
.pe-edit-active .github-card { cursor: default; }
.pe-edit-active .github-card::after { opacity: 0 !important; }


/* ── 标签增删 ── */
.pe-tag-add-btn {
  display: none;
  padding: 4px 12px;
  border: 1.5px dashed rgba(79,110,247,.4);
  border-radius: 100px;
  background: transparent;
  color: #4F6EF7;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
  font-family: 'DM Sans', sans-serif;
  transition: all .2s;
  height: 28px;
  align-items: center;
}
.pe-tag-add-btn:hover { background: rgba(79,110,247,.08); border-color: #4F6EF7; }
.pe-edit-active .pe-tag-add-btn { display: inline-flex; }

.pe-tag-del {
  display: none;
  position: absolute;
  top: -7px;
  right: -7px;
  width: 16px;
  height: 16px;
  background: #ef4444;
  color: #fff;
  border-radius: 50%;
  font-size: 11px;
  font-weight: 700;
  line-height: 16px;
  text-align: center;
  cursor: pointer;
  z-index: 50;
  pointer-events: all;
  box-shadow: 0 1px 4px rgba(239,68,68,.4);
  transition: transform .15s;
  /* 关键：不参与 contenteditable */
  user-select: none;
  -webkit-user-select: none;
}
.pe-tag-del:hover { transform: scale(1.2); background: #dc2626; }
.pe-edit-active .tag .pe-tag-del,
.pe-edit-active .badge .pe-tag-del { display: block; }
/* 标签需要 relative 定位才能显示绝对定位的删除按钮 */
.pe-edit-active .tag,
.pe-edit-active .badge { position: relative; overflow: visible !important; }
```

---

## 5. 环境检测脚本（插入到 `</head>` 之前）

<script>
// 环境检测：判断是否在本地/Hermes环境（可以下载文件）
window.PE_LOCAL_MODE = (function() {
  const loc = window.location;
  // file:// 协议 = 本地文件直接打开
  if (loc.protocol === 'file:') return true;
  // localhost 或 127.0.0.1 = Hermes/本地服务
  if (loc.hostname === 'localhost' || loc.hostname === '127.0.0.1') return true;
  // 非 claude.ai 的其他域名也视为本地模式
  if (loc.hostname && !loc.hostname.includes('claude.ai')) return true;
  return false;
})();

// 本地模式下动态加载 jsPDF
if (window.PE_LOCAL_MODE) {
  var s = document.createElement('script');
  s.src = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js';
  document.head.appendChild(s);
  var s2 = document.createElement('script');
  s2.src = 'https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js';
  document.head.appendChild(s2);
}
</script>

---

## 6. 工具栏区域（插入到 `</body>` 之前，顺序严格）

### 6.1 pe-saved-data标签（必须在PE主script之外）

```html
<script id="pe-saved-data" type="application/json">{}</script>
```

### 6.2 工具栏HTML

```html
<div id="pe-toolbar">
  <span class="pe-status" id="pe-status">预览模式</span>
  <div class="pe-divider"></div>
  <button class="pe-btn pe-btn-edit" id="pe-btn-edit" onclick="PE.toggle()">✏️ 编辑</button>
  <button class="pe-btn pe-btn-save" id="pe-btn-save" style="display:none" onclick="PE.save()" title="保存并下载新HTML文件">💾 保存文件</button>
  <button class="pe-btn pe-btn-copy" id="pe-btn-copy" style="display:none" onclick="PE.copyCode()" title="复制完整HTML代码，粘贴到文本编辑器另存">📋 复制代码</button>
  <div class="pe-divider"></div>
  <button class="pe-btn pe-btn-pdf" onclick="PE.pdf()">📄 导出 PDF</button>
</div>
<div id="pe-toast"></div>
```

---

## 7. PE编辑器主脚本（工具栏之后）

<script>
/* ═══════════════════════════════════════
   PE — Portfolio Editor
   设计原则：
   1. 只有 data-field 属性的元素才可编辑
   2. 编辑操作全部通过 contenteditable 实现
   3. 不修改任何 class / style / 结构属性
   4. 保存用 localStorage，刷新后自动恢复
   5. 导出用复制HTML代码，不触发文件下载
═══════════════════════════════════════ */
const PE = (() => {
  let active = false;
  const STORE_KEY = 'pe_portfolio_v1';

  // 初始化：多层次恢复策略
  // 优先级：pe-saved-data(内嵌) > localStorage > 原始HTML内容
  function init() {
    let savedFields = null;
    let savedImages = null;

    // 1. 先从 pe-saved-data 内嵌标签读（适合本地文件/已保存的文件）
    try {
      const dataScript = document.getElementById('pe-saved-data');
      if (dataScript) {
        const raw = dataScript.textContent.trim();
        if (raw && raw !== '{}' && raw.length > 5) {
          const parsed = JSON.parse(raw);
          if (parsed.fields && Object.keys(parsed.fields).length > 0) {
            savedFields = parsed.fields;
            savedImages = parsed.images || {};
            console.log('PE: 从内嵌数据恢复', Object.keys(savedFields).length, '个字段');
          }
        }
      }
    } catch(e) {}

    // 2. 降级：从 localStorage 读（适合 Claude.ai 环境刷新后恢复）
    if (!savedFields) {
      try {
        const lsRaw = localStorage.getItem(STORE_KEY);
        if (lsRaw) {
          const lsParsed = JSON.parse(lsRaw);
          if (lsParsed && Object.keys(lsParsed).length > 0) {
            savedFields = lsParsed;
            console.log('PE: 从localStorage恢复', Object.keys(savedFields).length, '个字段');
          }
        }
      } catch(e) {}
    }
    // 图片单独从 localStorage 读
    if (!savedImages) {
      try {
        const imgRaw = localStorage.getItem('pe_images_v1');
        if (imgRaw) savedImages = JSON.parse(imgRaw);
      } catch(e) {}
    }

    // 3. 应用恢复的字段内容
    if (savedFields) {
      Object.entries(savedFields).forEach(([field, content]) => {
        const el = document.querySelector('[data-field="' + field + '"]');
        if (el && !el.classList.contains('proto-img-slot')) {
          el.innerHTML = content;
        }
      });
    }

    // 4. 应用恢复的图片
    if (savedImages) {
      Object.entries(savedImages).forEach(([slot, src]) => {
        if (!src) return;
        const el = document.querySelector('[data-slot="' + slot + '"]');
        if (!el) return;
        const ph = el.querySelector('.proto-placeholder');
        let img = el.querySelector('img');
        if (!img) { img = document.createElement('img'); el.appendChild(img); }
        img.src = src;
        img.style.cssText = 'width:100%;height:auto;max-width:100%;display:block;object-fit:contain;';
        if (ph) ph.style.display = 'none';
        el.classList.add('has-image');
      });
    }
  }

  // 开启 / 关闭编辑模式
  function toggle() {
    active = !active;
    // 每次进入编辑模式时，清除所有初始化标志，确保增删系统重新注入按钮
    if (active) {
      document.querySelectorAll('[data-list-init],[data-proj-init],[data-tool-init],[data-think-init],[data-career-init],[data-growth-init],[data-gh-grid-init],[data-tag-init],[data-proto-init]').forEach(el => {
        delete el.dataset.listInit;
        delete el.dataset.projInit;
        delete el.dataset.toolInit;
        delete el.dataset.thinkInit;
        delete el.dataset.careerInit;
        delete el.dataset.growthInit;
        delete el.dataset.ghGridInit;
        delete el.dataset.tagInit;
        delete el.dataset.protoInit;
      });
      // 同时清除 dataset 里的各种 init 标志
      // 清除所有增删初始化标志（camelCase dataset属性）
      const initKeys = ['listInit','projInit','toolInit','thinkInit','careerInit',
        'growthInit','ghGridInit','ghInit','tagInit','tagDelInit','protoInit'];
      document.querySelectorAll('*').forEach(el => {
        initKeys.forEach(k => { if (el.dataset[k]) delete el.dataset[k]; });
      });
    }
    document.body.classList.toggle('pe-edit-active', active);
    document.getElementById('pe-toolbar').classList.toggle('is-editing', active);
    document.getElementById('pe-status').textContent = active ? '编辑模式' : '预览模式';
    document.getElementById('pe-btn-edit').textContent = active ? '✓ 退出' : '✏️ 编辑';
    document.getElementById('pe-btn-edit').classList.toggle('active', active);
    document.getElementById('pe-btn-save').style.display = active ? '' : 'none';
    document.getElementById('pe-btn-copy').style.display = active ? '' : 'none';

    // 只对 data-field 元素开关 contenteditable
    document.querySelectorAll('[data-field]').forEach(el => {
      if (active) {
        el.setAttribute('contenteditable', 'true');
        el.setAttribute('spellcheck', 'false');
      } else {
        el.removeAttribute('contenteditable');
      }
    });

    toast(active ? '编辑模式已开启 — 点击任意文字即可修改' : '已退出编辑模式');
  }

  // 保存：双模式
  // 本地/Hermes → 下载包含修改内容的新HTML文件
  // Claude.ai  → 写入localStorage + 退出编辑
  function save() {
    // 收集所有 data-field 内容
    const data = {};
    document.querySelectorAll('[data-field]').forEach(el => {
      if (!el.classList.contains('proto-img-slot')) {
        data[el.dataset.field] = el.innerHTML;
      }
    });

    // 收集图片
    const imgs = {};
    document.querySelectorAll('[data-slot]').forEach(el => {
      const img = el.querySelector('img');
      if (img && img.src && img.src.startsWith('data:')) {
        imgs[el.dataset.slot] = img.src;
      }
    });

    // 写入隐藏 script 标签（内嵌持久化，两种环境都执行）
    const dataScript = document.getElementById('pe-saved-data');
    if (dataScript) {
      dataScript.textContent = JSON.stringify({ fields: data, images: imgs });
    }

    // 同时写入 localStorage
    try {
      localStorage.setItem(STORE_KEY, JSON.stringify(data));
      localStorage.setItem('pe_images_v1', JSON.stringify(imgs));
    } catch(e) {}

    // 退出编辑模式
    function exitEdit() {
      if (!active) return;
      active = false;
      document.body.classList.remove('pe-edit-active');
      document.getElementById('pe-toolbar').classList.remove('is-editing');
      document.getElementById('pe-status').textContent = '预览模式';
      document.getElementById('pe-status').classList.remove('active');
      document.getElementById('pe-btn-edit').textContent = '✏️ 编辑';
      document.getElementById('pe-btn-edit').classList.remove('active');
      document.getElementById('pe-btn-save').style.display = 'none';
      document.getElementById('pe-btn-copy').style.display = 'none';
      document.querySelectorAll('[data-field]').forEach(el => el.removeAttribute('contenteditable'));
    }

    if (window.PE_LOCAL_MODE) {
      // ── 本地/Hermes模式：生成新HTML文件直接下载 ──
      exitEdit();
      const fullHTML = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;
      try {
        const blob = new Blob([fullHTML], { type: 'text/html;charset=utf-8' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        // 从页面的姓名字段动态读取，避免硬编码
        const nameEl = document.querySelector('[data-field="name"]');
        const personName = nameEl ? nameEl.textContent.trim() : '产品经理';
        a.download = personName + '-产品经理作品集.html';
        document.body.appendChild(a);
        a.click();
        setTimeout(() => { URL.revokeObjectURL(url); document.body.removeChild(a); }, 1000);
        toast('✅ 已下载保存 — 用新文件替换旧文件即可');
      } catch(e) {
        fallbackCopy(fullHTML);
        toast('📋 已复制代码 → 粘贴到文本编辑器另存为 .html');
      }
    } else {
      // ── Claude.ai模式：localStorage保存，提示刷新恢复 ──
      exitEdit();
      toast('✅ 内容已保存 — 刷新页面后自动恢复');
    }
  }

  // 复制完整 HTML 代码
  function copyCode() {
    // 临时移除 contenteditable，获取干净代码
    const fields = document.querySelectorAll('[data-field]');
    fields.forEach(el => el.removeAttribute('contenteditable'));
    document.body.classList.remove('pe-edit-active');
    document.getElementById('pe-toolbar').classList.remove('is-editing');

    const code = '<!DOCTYPE html>\n' + document.documentElement.outerHTML;

    // 恢复编辑状态
    document.body.classList.add('pe-edit-active');
    document.getElementById('pe-toolbar').classList.add('is-editing');
    fields.forEach(el => {
      el.setAttribute('contenteditable', 'true');
      el.setAttribute('spellcheck', 'false');
    });

    // 写入剪贴板
    if (navigator.clipboard && navigator.clipboard.writeText) {
      navigator.clipboard.writeText(code)
        .then(() => toast('📋 HTML代码已复制 → 粘贴到文本编辑器另存为 .html 文件'))
        .catch(() => fallbackCopy(code));
    } else {
      fallbackCopy(code);
    }
  }

  function fallbackCopy(text) {
    const ta = document.createElement('textarea');
    ta.value = text;
    ta.style.cssText = 'position:fixed;opacity:0;top:0;left:0;';
    document.body.appendChild(ta);
    ta.select();
    try {
      document.execCommand('copy');
      toast('📋 HTML代码已复制 → 粘贴到文本编辑器另存为 .html 文件');
    } catch(e) {
      toast('请按 Ctrl+U 查看源码后手动另存');
    }
    document.body.removeChild(ta);
  }

  // 导出 PDF — 双模式
  // 本地/Hermes → html2canvas + jsPDF 直接下载PDF
  // Claude.ai  → window.print() 打印另存为PDF
  async function pdf() {
    const wasActive = active;
    if (wasActive) {
      document.querySelectorAll('[data-field]').forEach(el => el.removeAttribute('contenteditable'));
      document.body.classList.remove('pe-edit-active');
      document.getElementById('pe-toolbar').classList.remove('is-editing');
    }
    const toolbar = document.getElementById('pe-toolbar');
    const toastEl = document.getElementById('pe-toast');
    toolbar.style.display = 'none';
    toastEl.style.display = 'none';

    function restore() {
      toolbar.style.display = '';
      toastEl.style.display = '';
      if (wasActive) {
        document.body.classList.add('pe-edit-active');
        document.getElementById('pe-toolbar').classList.add('is-editing');
        document.querySelectorAll('[data-field]').forEach(el => {
          el.setAttribute('contenteditable', 'true');
          el.setAttribute('spellcheck', 'false');
        });
      }
    }

    if (window.PE_LOCAL_MODE && window.jspdf && window.html2canvas) {
      // ── 本地/Hermes：html2canvas + jsPDF ──
      toastEl.style.display = '';
      toast('⏳ 正在生成PDF，请稍候...');
      try {
        const { jsPDF } = window.jspdf;
        const body = document.body;
        const canvas = await html2canvas(body, {
          scale: 2, useCORS: true, allowTaint: true,
          logging: false, backgroundColor: '#F4F6FF',
          width: body.scrollWidth, height: body.scrollHeight,
          scrollX: 0, scrollY: 0
        });
        const doc = new jsPDF({ orientation: 'portrait', unit: 'mm', format: 'a4', compress: true });
        const pageW = 210, pageH = 297;
        const pxPerMm = canvas.width / pageW;
        const pageHpx = pageH * pxPerMm;
        const pages = Math.ceil(canvas.height / pageHpx);
        for (let i = 0; i < pages; i++) {
          if (i > 0) doc.addPage();
          const srcY = i * pageHpx;
          const srcH = Math.min(pageHpx, canvas.height - srcY);
          const pc = document.createElement('canvas');
          pc.width = canvas.width; pc.height = srcH;
          pc.getContext('2d').drawImage(canvas, 0, -srcY);
          doc.addImage(pc.toDataURL('image/jpeg', 0.92), 'JPEG', 0, 0, pageW, srcH / pxPerMm);
        }
        const d = new Date();
        const ds = `${d.getFullYear()}${String(d.getMonth()+1).padStart(2,'0')}${String(d.getDate()).padStart(2,'0')}`;
        doc.save('\u6768\u7855-\u4ea7\u54c1\u7ecf\u7406\u4f5c\u54c1\u96c6-' + ds + '.pdf');
        toast('✅ PDF已生成并下载！');
      } catch(err) {
        console.error('PDF生成失败:', err);
        toast('⚠️ PDF生成失败，切换为打印方式');
        setTimeout(() => { window.print(); }, 500);
      } finally {
        restore();
      }
    } else {
      // ── Claude.ai 或库未加载：window.print() ──
      if (!window.PE_LOCAL_MODE) {
        toast('正在打开打印对话框 — 选择"另存为PDF"并勾选"背景图形"');
      }
      setTimeout(() => {
        window.print();
        restore();
      }, window.PE_LOCAL_MODE ? 2000 : 400);
    }
  }

  // Toast 通知
  let toastTimer;
  function toast(msg) {
    const el = document.getElementById('pe-toast');
    el.textContent = msg;
    el.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => el.classList.remove('show'), 3000);
  }

  // Ctrl+S 快捷键
  document.addEventListener('keydown', e => {
    if ((e.ctrlKey || e.metaKey) && e.key === 's' && active) {
      e.preventDefault();
      save();
    }
  });

  // 页面加载恢复
  window.addEventListener('DOMContentLoaded', init);



  return { toggle, save, copyCode, pdf, _toast: toast };
})();

/* ─── REVEAL ANIMATION ─── */
const ro = new IntersectionObserver(entries => {
  entries.forEach(e => { if (e.isIntersecting) { e.target.classList.add('in'); ro.unobserve(e.target); } });
}, { threshold: 0.08 });
document.querySelectorAll('.reveal').forEach(el => ro.observe(el));

/* ─── NAV ACTIVE ─── */
window.addEventListener('scroll', () => {
  let cur = '';
  document.querySelectorAll('section[id]').forEach(s => {
    if (window.scrollY >= s.offsetTop - 80) cur = s.id;
  });
  document.querySelectorAll('.nav-links a').forEach(a => {
    const on = a.getAttribute('href') === '#' + cur;
    a.style.color = on ? 'var(--green)' : '';
    a.style.fontWeight = on ? '600' : '';
  });
}, { passive: true });
</script>

---

## 8. 图片上传脚本

<script>
/* ── 原型图上传 ── */
(function(){
  const STORE_IMG = 'pe_images_v1';

  // 恢复已保存图片
  function restoreImages(){
    try{
      const imgs = JSON.parse(localStorage.getItem(STORE_IMG)||'{}');
      Object.entries(imgs).forEach(([slot, src])=>{ applyImg(slot, src); });
    }catch(e){}
  }

  function applyImg(slot, src){
    const el = document.querySelector('[data-slot="'+slot+'"]');
    if(!el) return;
    // 清空placeholder，插入img
    const ph = el.querySelector('.proto-placeholder');
    let img = el.querySelector('img');
    if(!img){ img = document.createElement('img'); el.appendChild(img); }
    img.src = src;
    img.style.cssText='width:100%;height:auto;max-width:100%;display:block;object-fit:contain;';
    if(ph) ph.style.display='none';
    el.classList.add('has-image');
  }

  function saveImg(slot, src){
    try{
      const imgs = JSON.parse(localStorage.getItem(STORE_IMG)||'{}');
      imgs[slot] = src;
      localStorage.setItem(STORE_IMG, JSON.stringify(imgs));
    }catch(e){}
  }

  // 创建隐藏file input
  const inp = document.createElement('input');
  inp.type='file'; inp.accept='image/*';
  inp.style.cssText='position:fixed;opacity:0;pointer-events:none;top:0;left:0;';
  document.body.appendChild(inp);
  let pendingSlot = null;

  inp.addEventListener('change', function(){
    const file = this.files[0];
    if(!file || !pendingSlot) return;
    const reader = new FileReader();
    reader.onload = ev => {
      applyImg(pendingSlot, ev.target.result);
      saveImg(pendingSlot, ev.target.result);
      PE._toast('✅ 图片已上传并保存');
    };
    reader.readAsDataURL(file);
    this.value='';
  });

  // 点击图片槽触发上传（仅编辑模式）
  document.addEventListener('click', function(e){
    if(!document.body.classList.contains('pe-edit-active')) return;
    const slot = e.target.closest('[data-slot]');
    if(!slot) return;
    e.preventDefault();
    e.stopPropagation();
    pendingSlot = slot.dataset.slot;
    inp.click();
  });

  // 把 copyCode 时图片也带进去（localStorage图片已存在data-slot元素里）
  // restoreImages已整合到init()中，无需单独注册

  // 暴露给PE
  PE._restoreImages = restoreImages;
})();

</script>

---

## 9. 增删系统脚本（按顺序全部插入）

### 列表增删系统

<script>
/* ── 列表增删系统 ── */
(function(){

  // 给所有 proj-step 加删除按钮 + 给 proj-steps 加添加按钮
  function initStepLists() {
    document.querySelectorAll('.proj-steps').forEach(list => {
      if (list.dataset.listInit) return;
      list.dataset.listInit = '1';
      list.setAttribute('data-list', 'steps');

      // 给每个step加删除btn
      list.querySelectorAll('.proj-step').forEach(step => addDelBtn(step, () => {
        if (list.querySelectorAll('.proj-step').length > 1) step.remove();
        else PE._toast('⚠️ 至少保留一条');
      }));

      // 添加按钮
      const addBtn = makeAddBtn('+ 添加步骤', () => {
        const tpl = list.querySelector('.proj-step').cloneNode(true);
        // 清空文字，自动编号序号
        const stepCount = list.querySelectorAll('.proj-step').length + 1;
        tpl.querySelectorAll('[data-field]').forEach(el => {
          const ts = Date.now();
          if (el.classList.contains('step-num')) {
            el.textContent = String(stepCount);
          } else {
            el.textContent = '点击编辑';
          }
          el.dataset.field = el.dataset.field + '_' + ts;
          if (document.body.classList.contains('pe-edit-active')) {
            el.setAttribute('contenteditable', 'true');
            el.setAttribute('spellcheck', 'false');
          }
        });
        // 重新绑定删除按钮
        const del = tpl.querySelector('.pe-del-btn');
        if (del) del.remove();
        addDelBtn(tpl, () => {
          if (list.querySelectorAll('.proj-step').length > 1) tpl.remove();
        });
        list.appendChild(tpl);
        PE._toast('✅ 已添加步骤，点击文字编辑内容');
      });
      list.after(addBtn);
    });
  }

  // 给 tool-uses 加增删
  function initToolLists() {
    document.querySelectorAll('.tool-uses').forEach(list => {
      if (list.dataset.listInit) return;
      list.dataset.listInit = '1';

      list.querySelectorAll('li').forEach(li => addDelBtn(li, () => {
        if (list.querySelectorAll('li').length > 1) li.remove();
        else PE._toast('⚠️ 至少保留一条');
      }));

      const addBtn = makeAddBtn('+ 添加用法', () => {
        const li = document.createElement('li');
        li.textContent = '点击编辑';
        li.dataset.field = 'tool_use_' + Date.now();
        li.dataset.fieldLabel = '用法';
        if (document.body.classList.contains('pe-edit-active')) {
          li.setAttribute('contenteditable', 'true');
          li.setAttribute('spellcheck', 'false');
        }
        addDelBtn(li, () => {
          if (list.querySelectorAll('li').length > 1) li.remove();
        });
        list.appendChild(li);
        PE._toast('✅ 已添加，点击文字编辑');
      });
      list.after(addBtn);
    });
  }

  // 给功能范围区块的每一行加删除，并加添加按钮
  function initFeatureBlocks() {
    document.querySelectorAll('.proj-col').forEach(col => {
      // 找包含 P0/P1/P2 的flex div容器
      const featureBlocks = col.querySelectorAll('div[style*="flex-direction:column"][style*="gap:8px"]');
      featureBlocks.forEach(container => {
        if (container.dataset.listInit) return;
        container.dataset.listInit = '1';
        container.setAttribute('data-list', 'features');

        container.querySelectorAll(':scope > div').forEach(row => {
          addDelBtn(row, () => {
            if (container.querySelectorAll(':scope > div').length > 1) row.remove();
            else PE._toast('⚠️ 至少保留一条');
          });
        });

        const addBtn = makeAddBtn('+ 添加功能项', () => {
          const tpl = container.querySelector(':scope > div').cloneNode(true);
          // 修改标签
          const label = tpl.querySelector('span:first-child');
          if (label) {
            label.textContent = 'NEW';
            label.dataset.field = 'feat_label_' + Date.now();
            if(document.body.classList.contains('pe-edit-active')){
              label.setAttribute('contenteditable','true');
              label.setAttribute('spellcheck','false');
            }
          }
          const text = tpl.querySelector('[data-field]');
          if (text) {
            text.textContent = '点击编辑功能描述';
            text.dataset.field = 'feature_' + Date.now();
            if (document.body.classList.contains('pe-edit-active')) {
              text.setAttribute('contenteditable', 'true');
            }
          }
          const del = tpl.querySelector('.pe-del-btn');
          if (del) del.remove();
          addDelBtn(tpl, () => tpl.remove());
          container.appendChild(tpl);
          PE._toast('✅ 已添加，点击文字编辑');
        });
        container.after(addBtn);
      });
    });
  }

  // 给 market-card 加删除，给 market-grid 加添加
  function initMarketCards() {
    const grid = document.querySelector('.market-grid');
    if (!grid || grid.dataset.listInit) return;
    grid.dataset.listInit = '1';

    // 内联删除按钮（addDelBtn在另一个作用域）
    function addMktDel(card) {
      if (card.querySelector('.pe-del-btn')) return;
      const btn = document.createElement('button');
      btn.className = 'pe-del-btn';
      btn.textContent = '×';
      btn.setAttribute('contenteditable', 'false');
      btn.title = '删除此项';
      btn.addEventListener('click', (e) => {
        e.stopPropagation(); e.preventDefault();
        if (grid.querySelectorAll('.market-card').length > 1) card.remove();
        else PE._toast('⚠️ 至少保留一条');
      });
      card.setAttribute('data-deletable', '1');
      card.appendChild(btn);
    }

    grid.querySelectorAll('.market-card').forEach(card => {
      addMktDel(card);
    });

    const addBtn = makeAddBtn('+ 添加市场洞察', () => {
      const tpl = grid.querySelector('.market-card').cloneNode(true);
      tpl.querySelectorAll('[data-field]').forEach(el => {
        el.textContent = '点击编辑';
        el.dataset.field = el.dataset.field + '_' + Date.now();
        if (document.body.classList.contains('pe-edit-active')) {
          el.setAttribute('contenteditable', 'true');
        }
      });
      const del = tpl.querySelector('.pe-del-btn');
      if (del) del.remove();
      // 重新添加删除按钮（内联方式）
      const newDelBtn = document.createElement('button');
      newDelBtn.className = 'pe-del-btn';
      newDelBtn.textContent = '×';
      newDelBtn.setAttribute('contenteditable', 'false');
      newDelBtn.addEventListener('click', (e) => {
        e.stopPropagation(); e.preventDefault();
        if (grid.querySelectorAll('.market-card').length > 1) tpl.remove();
      });
      tpl.setAttribute('data-deletable', '1');
      tpl.appendChild(newDelBtn);
      grid.appendChild(tpl);
      PE._toast('✅ 已添加，点击文字编辑');
    });
    addBtn.style.margin = '16px 0 0';
    grid.after(addBtn);
  }

  // 辅助：创建删除按钮
  function addDelBtn(el, onDel) {
    el.setAttribute('data-deletable', '1');
    const btn = document.createElement('button');
    btn.className = 'pe-del-btn';
    btn.textContent = '×';
    btn.title = '删除此项';
    btn.addEventListener('click', (e) => {
      e.stopPropagation();
      e.preventDefault();
      onDel();
    });
    el.appendChild(btn);
  }

  // 辅助：创建添加按钮
  function makeAddBtn(label, onClick) {
    const btn = document.createElement('button');
    btn.className = 'pe-add-btn';
    btn.textContent = label;
    btn.addEventListener('click', (e) => {
      e.stopPropagation();
      onClick();
    });
    return btn;
  }

  // 在编辑模式开启时初始化
  const origToggle = PE.toggle;
  // 监听编辑模式变化
  const ob = new MutationObserver(() => {
    if (document.body.classList.contains('pe-edit-active')) {
      initStepLists();
      initToolLists();
      initFeatureBlocks();
      initMarketCards();
    }
  });
  ob.observe(document.body, { attributes: true, attributeFilter: ['class'] });

})();

</script>

### 卡片级别增删系统

<script>
/* ── 卡片级别增删系统 ── */
(function(){

  // ① 项目卡片增删
  function initProjCards(){
    const container = document.querySelector('.projects-section .wrap');
    if(!container || container.dataset.projInit) return;
    container.dataset.projInit = '1';

    // 给每个 proj 卡片加删除按钮
    container.querySelectorAll('.proj').forEach(card => {
      addCardDel(card, () => {
        if(container.querySelectorAll('.proj').length > 1){ card.remove(); PE._toast('项目已删除'); }
        else PE._toast('⚠️ 至少保留一个项目');
      });
    });

    // 添加新项目按钮（放在最后一个 proj 后面）
    const addBtn = makeCardAddBtn('+ 新增项目卡片', () => {
      const tpl = container.querySelector('.proj').cloneNode(true);
      // 清空所有 data-field 内容
      tpl.querySelectorAll('[data-field]').forEach(el => {
        if(!el.classList.contains('proto-img-slot')){
          el.textContent = '点击编辑';
          el.dataset.field = el.dataset.field + '_new_' + Date.now();
          if(document.body.classList.contains('pe-edit-active')){
            el.setAttribute('contenteditable','true');
            el.setAttribute('spellcheck','false');
          }
        }
      });
      // 移除旧删除按钮，加新的
      tpl.querySelectorAll('.pe-card-del').forEach(b=>b.remove());
      addCardDel(tpl, ()=>{ tpl.remove(); PE._toast('项目已删除'); });
      // 清空原型槽
      tpl.querySelectorAll('.proto-img-slot').forEach(slot=>{
        const img=slot.querySelector('img'); if(img) img.remove();
        const ph=slot.querySelector('.proto-placeholder'); if(ph) ph.style.display='';
        slot.classList.remove('has-image');
        slot.dataset.slot = slot.dataset.slot + '_new_' + Date.now();
      });
      container.querySelector('.projects-intro').after(tpl);
      tpl.scrollIntoView({behavior:'smooth',block:'center'});
      PE._toast('✅ 新项目卡片已添加，点击文字编辑');
    });
    // 插在 intro 和第一个 proj 之间之后
    const lastProj = [...container.querySelectorAll('.proj')].at(-1);
    if(lastProj) lastProj.after(addBtn);
  }

  // ② 工具卡片增删
  function initToolCards(){
    const grid = document.querySelector('.ai-grid');
    if(!grid || grid.dataset.toolInit) return;
    grid.dataset.toolInit = '1';

    grid.querySelectorAll('.ai-tool-card').forEach(card => {
      addCardDel(card, () => {
        if(grid.querySelectorAll('.ai-tool-card').length > 1){ card.remove(); PE._toast('工具卡片已删除'); }
        else PE._toast('⚠️ 至少保留一个');
      });
    });

    const addBtn = makeCardAddBtn('+ 新增工具', () => {
      const tpl = grid.querySelector('.ai-tool-card').cloneNode(true);
      tpl.querySelectorAll('[data-field]').forEach(el=>{
        el.textContent='点击编辑';
        el.dataset.field = el.dataset.field+'_'+Date.now();
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable','true'); el.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('li').forEach(li=>{
        li.textContent='点击编辑';
        li.dataset.field='tool_use_'+Date.now();
        if(document.body.classList.contains('pe-edit-active')){
          li.setAttribute('contenteditable','true'); li.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('.pe-card-del,.pe-del-btn,.pe-add-btn').forEach(b=>b.remove());
      addCardDel(tpl,()=>{ tpl.remove(); });
      grid.appendChild(tpl);
      PE._toast('✅ 新工具卡片已添加');
    });
    grid.after(addBtn);
  }

  // ③ AI思考卡片增删
  function initThinkingCards(){
    const grid = document.querySelector('.ai-thinking-grid');
    if(!grid || grid.dataset.thinkInit) return;
    grid.dataset.thinkInit = '1';

    grid.querySelectorAll('.thinking-card').forEach(card => {
      addCardDel(card, () => {
        if(grid.querySelectorAll('.thinking-card').length > 1){ card.remove(); PE._toast('思考卡片已删除'); }
        else PE._toast('⚠️ 至少保留一个');
      });
    });

    const addBtn = makeCardAddBtn('+ 新增思考', () => {
      const tpl = grid.querySelector('.thinking-card').cloneNode(true);
      tpl.querySelectorAll('[data-field]').forEach(el=>{
        el.textContent='点击编辑';
        el.dataset.field = el.dataset.field+'_'+Date.now();
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable','true'); el.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('.pe-card-del,.pe-del-btn,.pe-add-btn').forEach(b=>b.remove());
      addCardDel(tpl,()=>{ tpl.remove(); });
      grid.appendChild(tpl);
      PE._toast('✅ 新思考卡片已添加');
    });
    grid.after(addBtn);
  }

  // 辅助：卡片删除按钮（右上角，醒目）
  function addCardDel(card, onDel){
    if(card.querySelector('.pe-card-del')) return;
    const btn = document.createElement('button');
    btn.className = 'pe-card-del';
    btn.innerHTML = '× 删除';
    btn.title = '删除此卡片';
    btn.setAttribute('contenteditable', 'false');
    btn.addEventListener('click', e=>{ e.stopPropagation(); e.preventDefault(); onDel(); });
    card.appendChild(btn);
  }

  // 辅助：卡片添加按钮
  function makeCardAddBtn(label, onClick){
    const btn = document.createElement('button');
    btn.className = 'pe-card-add-btn';
    btn.textContent = label;
    btn.addEventListener('click', e=>{ e.stopPropagation(); onClick(); });
    return btn;
  }

  // 监听编辑模式开启
  const ob2 = new MutationObserver(()=>{
    if(document.body.classList.contains('pe-edit-active')){
      initProjCards();
      initToolCards();
      initThinkingCards();
      // 显示所有卡片删除按钮
      document.querySelectorAll('.pe-card-del,.pe-card-add-btn').forEach(b=>{ b.style.display=''; });
    } else {
      // 隐藏
      document.querySelectorAll('.pe-card-del,.pe-card-add-btn').forEach(b=>{ b.style.display='none'; });
    }
  });
  ob2.observe(document.body,{attributes:true,attributeFilter:['class']});

})();

</script>

### 原型区块删除

<script>
/* ── 原型区块删除 ── */
(function(){
  function initProtoDel(){
    document.querySelectorAll('.proj-proto').forEach(proto => {
      if(proto.dataset.protoInit) return;
      proto.dataset.protoInit = '1';
      proto.style.position = 'relative';

      const btn = document.createElement('button');
      btn.className = 'pe-proto-del';
      btn.innerHTML = '× 删除原型区块';
      btn.addEventListener('click', e => {
        e.stopPropagation();
        proto.remove();
        PE._toast('原型区块已删除');
      });
      proto.appendChild(btn);
    });
  }

  const ob3 = new MutationObserver(()=>{
    if(document.body.classList.contains('pe-edit-active')){
      initProtoDel();
      document.querySelectorAll('.pe-proto-del').forEach(b=>b.style.display='block');
    } else {
      document.querySelectorAll('.pe-proto-del').forEach(b=>b.style.display='none');
    }
  });
  ob3.observe(document.body,{attributes:true,attributeFilter:['class']});
})();

</script>

### GitHub卡片完整编辑系统

<script>
/* ── GitHub卡片完整编辑系统 ── */
(function(){

  // 链接编辑弹窗
  const dlg = document.createElement('div');
  dlg.id = 'gh-url-dlg';
  dlg.style.cssText = 'display:none;position:fixed;inset:0;background:rgba(0,0,0,0.5);z-index:9999;align-items:center;justify-content:center;';
  dlg.innerHTML = `
    <div style="background:#fff;border-radius:16px;padding:28px 32px;width:480px;max-width:90vw;box-shadow:0 24px 60px rgba(0,0,0,0.25);">
      <div style="font-size:16px;font-weight:700;margin-bottom:6px;color:#1A2035;">编辑项目链接</div>
      <div style="font-size:13px;color:#64748b;margin-bottom:20px;">修改 GitHub 仓库地址</div>
      <input id="gh-url-input" type="text" placeholder="https://github.com/..."
        style="width:100%;padding:11px 14px;border:1.5px solid #e2e8f0;border-radius:8px;
               font-size:14px;font-family:'DM Mono',monospace;outline:none;color:#1A2035;
               transition:border-color .2s;" />
      <div style="display:flex;gap:10px;justify-content:flex-end;margin-top:16px;">
        <button id="gh-url-cancel" style="padding:9px 20px;border:1.5px solid #e2e8f0;border-radius:8px;
          background:#fff;color:#64748b;font-size:13px;font-weight:600;cursor:pointer;font-family:'DM Sans',sans-serif;">取消</button>
        <button id="gh-url-confirm" style="padding:9px 20px;border:none;border-radius:8px;
          background:#4F6EF7;color:#fff;font-size:13px;font-weight:600;cursor:pointer;font-family:'DM Sans',sans-serif;">确认</button>
      </div>
    </div>`;
  document.body.appendChild(dlg);

  let currentCard = null;

  document.getElementById('gh-url-cancel').onclick = () => { dlg.style.display = 'none'; };
  document.getElementById('gh-url-confirm').onclick = () => {
    const url = document.getElementById('gh-url-input').value.trim();
    if(url && currentCard){
      currentCard.setAttribute('href', url);
      currentCard.setAttribute('data-gh-href', url);
      PE._toast('✅ 链接已更新');
    }
    dlg.style.display = 'none';
  };
  document.getElementById('gh-url-input').addEventListener('focus', function(){
    this.style.borderColor='#4F6EF7';
  });
  document.getElementById('gh-url-input').addEventListener('blur', function(){
    this.style.borderColor='#e2e8f0';
  });

  // 初始化每张 github-card
  function initGithubCards(){
    document.querySelectorAll('.github-card').forEach(card => {
      if(card.dataset.ghInit) return;
      card.dataset.ghInit = '1';

      // 编辑模式下阻止卡片跳转（文字仍可点击编辑）
      card.addEventListener('click', function(e){
        if(document.body.classList.contains('pe-edit-active')){
          e.preventDefault();
          e.stopPropagation();
        }
      });

      // 编辑链接按钮
      const editBtn = document.createElement('button');
      editBtn.className = 'gh-edit-url-btn';
      editBtn.textContent = '🔗 编辑链接';
      editBtn.addEventListener('click', e => {
        e.stopPropagation();
        e.preventDefault();
        currentCard = card;
        document.getElementById('gh-url-input').value = card.getAttribute('data-gh-href') || card.getAttribute('href') || '';
        dlg.style.display = 'flex';
        setTimeout(()=>document.getElementById('gh-url-input').focus(), 50);
      });
      card.appendChild(editBtn);
    });
  }

  // 监听编辑模式
  const ob = new MutationObserver(()=>{
    initGithubCards();
  });
  ob.observe(document.body, {attributes:true, attributeFilter:['class']});
  window.addEventListener('DOMContentLoaded', initGithubCards);

})();

</script>

### 开源项目增删

<script>
/* ── 开源项目增删 ── */
(function(){
  function initGithubGrid(){
    const grid = document.querySelector('.github-grid');
    if(!grid || grid.dataset.ghGridInit) return;
    grid.dataset.ghGridInit = '1';

    grid.querySelectorAll('.github-card').forEach(card => {
      addGhDel(card);
    });

    const addBtn = document.createElement('button');
    addBtn.className = 'pe-card-add-btn';
    addBtn.textContent = '+ 新增开源项目';
    addBtn.addEventListener('click', e => {
      e.stopPropagation();
      const tpl = grid.querySelector('.github-card').cloneNode(true);
      const ts = Date.now();
      tpl.setAttribute('href', '#');
      tpl.setAttribute('data-gh-href', '#');
      tpl.dataset.ghInit = '';
      tpl.querySelectorAll('[data-field]').forEach(el => {
        el.textContent = '点击编辑';
        el.dataset.field = el.dataset.field + '_' + ts;
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable','true');
          el.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('.pe-card-del,.gh-edit-url-btn').forEach(b=>b.remove());
      addGhDel(tpl);
      // 重新初始化编辑链接按钮
      const editBtn = document.createElement('button');
      editBtn.className = 'gh-edit-url-btn';
      editBtn.textContent = '🔗 编辑链接';
      editBtn.addEventListener('click', ev => {
        ev.stopPropagation(); ev.preventDefault();
        // 触发链接编辑
        tpl.click();
      });
      tpl.appendChild(editBtn);
      grid.appendChild(tpl);
      PE._toast('✅ 新项目已添加，点击文字编辑');
    });
    grid.after(addBtn);
  }

  function addGhDel(card){
    if(card.querySelector('.pe-card-del')) return;
    const btn = document.createElement('button');
    btn.className = 'pe-card-del';
    btn.innerHTML = '× 删除';
    btn.addEventListener('click', e => {
      e.stopPropagation(); e.preventDefault();
      const grid = document.querySelector('.github-grid');
      if(grid.querySelectorAll('.github-card').length > 1){ card.remove(); PE._toast('项目已删除'); }
      else PE._toast('⚠️ 至少保留一个项目');
    });
    card.appendChild(btn);
  }

  const ob = new MutationObserver(()=>{
    if(document.body.classList.contains('pe-edit-active')) initGithubGrid();
  });
  ob.observe(document.body,{attributes:true,attributeFilter:['class']});
})();

</script>

### 职业轨迹 + 成长卡片增删

<script>
/* ── 职业轨迹 + 成长卡片增删 ── */
(function(){

  // 职业轨迹增删
  function initCareerTimeline(){
    const timeline = document.querySelector('.career-timeline');
    if(!timeline || timeline.dataset.careerInit) return;
    timeline.dataset.careerInit = '1';

    timeline.querySelectorAll('.career-item').forEach(item => addCareerDel(item, timeline));

    const addBtn = document.createElement('button');
    addBtn.className = 'pe-card-add-btn';
    addBtn.textContent = '+ 新增职业经历';
    addBtn.addEventListener('click', e => {
      e.stopPropagation();
      const tpl = timeline.querySelector('.career-item').cloneNode(true);
      const ts = Date.now();
      tpl.querySelectorAll('[data-field]').forEach(el => {
        el.textContent = '点击编辑';
        el.dataset.field = el.dataset.field + '_' + ts;
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable','true');
          el.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('.pe-card-del').forEach(b=>b.remove());
      addCareerDel(tpl, timeline);
      timeline.appendChild(tpl);
      PE._toast('✅ 新职业经历已添加');
    });
    timeline.after(addBtn);
  }

  function addCareerDel(item, timeline){
    if(item.querySelector('.pe-card-del')) return;
    const btn = document.createElement('button');
    btn.className = 'pe-card-del';
    btn.innerHTML = '× 删除';
    btn.style.top = '0px';
    btn.addEventListener('click', e => {
      e.stopPropagation();
      if(timeline.querySelectorAll('.career-item').length > 1){ item.remove(); PE._toast('经历已删除'); }
      else PE._toast('⚠️ 至少保留一条');
    });
    item.style.position = 'relative';
    item.appendChild(btn);
  }

  // 成长卡片增删
  function initGrowthGrid(){
    const grid = document.querySelector('.growth-grid');
    if(!grid || grid.dataset.growthInit) return;
    grid.dataset.growthInit = '1';

    grid.querySelectorAll('.growth-card').forEach(card => addGrowthDel(card, grid));

    const addBtn = document.createElement('button');
    addBtn.className = 'pe-card-add-btn';
    addBtn.textContent = '+ 新增成长维度';
    addBtn.addEventListener('click', e => {
      e.stopPropagation();
      const tpl = grid.querySelector('.growth-card').cloneNode(true);
      const ts = Date.now();
      tpl.querySelectorAll('[data-field]').forEach(el => {
        el.textContent = '点击编辑';
        el.dataset.field = el.dataset.field + '_' + ts;
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable','true');
          el.setAttribute('spellcheck','false');
        }
      });
      tpl.querySelectorAll('.pe-card-del').forEach(b=>b.remove());
      addGrowthDel(tpl, grid);
      grid.appendChild(tpl);
      PE._toast('✅ 新成长维度已添加');
    });
    grid.after(addBtn);
  }

  function addGrowthDel(card, grid){
    if(card.querySelector('.pe-card-del')) return;
    const btn = document.createElement('button');
    btn.className = 'pe-card-del';
    btn.innerHTML = '× 删除';
    btn.addEventListener('click', e => {
      e.stopPropagation();
      if(grid.querySelectorAll('.growth-card').length > 1){ card.remove(); PE._toast('维度已删除'); }
      else PE._toast('⚠️ 至少保留一个');
    });
    card.appendChild(btn);
  }

  const ob = new MutationObserver(()=>{
    if(document.body.classList.contains('pe-edit-active')){
      initCareerTimeline();
      initGrowthGrid();
    }
  });
  ob.observe(document.body,{attributes:true,attributeFilter:['class']});
})();

</script>

### 标签增删系统

<script>
/* ── 标签增删系统 ── */
(function(){

  function initTagGroups(){
    // hero-tags, career-tags, proj-badges
    document.querySelectorAll('.hero-tags, .career-tags, .proj-badges').forEach(group => {
      if(group.dataset.tagInit) return;
      group.dataset.tagInit = '1';

      // 每个tag加删除按钮
      group.querySelectorAll('.tag, .badge').forEach(tag => addTagDel(tag, group));

      // 添加按钮
      const addBtn = document.createElement('button');
      addBtn.className = 'pe-tag-add-btn';
      addBtn.textContent = '+ 标签';
      addBtn.addEventListener('click', e => {
        e.stopPropagation();
        const isHero = group.classList.contains('hero-tags');
        const isBadge = group.classList.contains('proj-badges');
        const newTag = document.createElement('span');
        if(isBadge){
          newTag.className = 'badge badge-prd';
          newTag.textContent = '新标签';
        } else {
          newTag.className = 'tag tag-outline';
          newTag.textContent = '新标签';
        }
        newTag.dataset.field = 'new_tag_' + Date.now();
        newTag.dataset.fieldLabel = '标签';
        if(document.body.classList.contains('pe-edit-active')){
          newTag.setAttribute('contenteditable','true');
          newTag.setAttribute('spellcheck','false');
        }
        addTagDel(newTag, group);
        // 插在 addBtn 前
        group.insertBefore(newTag, addBtn);
        // 自动聚焦编辑
        setTimeout(()=>newTag.focus(), 50);
        PE._toast('✅ 标签已添加，点击编辑内容');
      });
      group.appendChild(addBtn);
    });
  }

  function addTagDel(tag, group){
    if(tag.dataset.tagDelInit) return;
    tag.dataset.tagDelInit = '1';
    tag.style.position = 'relative';
    const btn = document.createElement('span');
    btn.className = 'pe-tag-del';
    btn.textContent = '×';
    btn.title = '删除标签';
    btn.setAttribute('contenteditable', 'false'); // 阻止被编辑
    btn.addEventListener('click', e => {
      e.stopPropagation();
      e.preventDefault();
      tag.remove();
      PE._toast('标签已删除');
    });
    tag.appendChild(btn);
  }

  const ob = new MutationObserver(()=>{
    if(document.body.classList.contains('pe-edit-active')) initTagGroups();
  });
  ob.observe(document.body,{attributes:true,attributeFilter:['class']});
})();

</script>

