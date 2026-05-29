# 编辑系统技术参考

读取时机：生成HTML时查阅，确保编辑系统完整实现。

---

## 1. 可编辑字段完整清单

所有可编辑文字必须标记：
```html
<元素 data-field="唯一id" data-field-label="显示名">内容</元素>
```

**必须标记（一个不能漏）：**

| 区域 | 必须标记的字段 |
|------|-------------|
| 封面 | 姓名、引用语、评价来源、每个标签文字、每个数字+单位+说明、联系方式文字 |
| 项目 | 标题、描述、每个指标数字+说明、每个badge文字 |
| 项目内容 | 小节标题（proj-col-label）、背景段落、步骤序号+标题+说明、功能范围标签+描述 |
| 原型图 | 图片说明文字（proto-cap）、原型区标题（proto-header）|
| AI模块 | 工具名/定位/描述/每条用法li、思考编号/标题/内容/判断标准 |
| GitHub | 项目类型标签、名称、描述、每个技术标签 |
| 市场 | 每个信号/判断/结果的标题（label）和内容、箭头符号、编号 |
| 职业 | 时间段、公司名、职位、阶段标题、描述段落 |
| 成长维度 | 维度标题、早期内容、箭头、现在内容 |

**禁止标记**：class、style属性、HTML结构标签、编辑系统按钮

---

## 2. 关键Bug防范

生成时必须遵守，否则会出现已验证的Bug：

| Bug | 原因 | 必须做的事 |
|-----|------|-----------|
| 标签显示×××文字 | 删除按钮span被contenteditable渲染 | 所有删除/添加按钮加 `contenteditable="false"` |
| PE is not defined | pe-saved-data插入到PE script内部截断 | pe-saved-data放在PE script标签之外 |
| JS语法错误 | 字符串含真实换行符 | `'<!DOCTYPE html>\n'` 用转义 `\n`，不用真实换行 |
| 图片截断 | 固定高度 | proto-img-slot用 `min-height:200px` + `height:auto` |
| 删除按钮被裁切 | 父容器overflow:hidden | 所有增删按钮的父容器不能有overflow:hidden |

---

## 3. 环境检测脚本（放在 `</head>` 之前）

```html
<script>
window.PE_LOCAL_MODE = (function() {
  var loc = window.location;
  if (loc.protocol === 'file:') return true;
  if (loc.hostname === 'localhost' || loc.hostname === '127.0.0.1') return true;
  if (loc.hostname && !loc.hostname.includes('claude.ai')) return true;
  return false;
})();
if (window.PE_LOCAL_MODE) {
  ['https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js',
   'https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js'
  ].forEach(function(src){ var s=document.createElement('script');s.src=src;document.head.appendChild(s); });
}
</script>
```

---

## 4. 工具栏位置（放在 `</body>` 之前）

**顺序严格要求**：
1. `<script id="pe-saved-data" type="application/json">{}</script>` ← 必须在PE script之外
2. `<div id="pe-toolbar">...</div>`
3. `<div id="pe-toast"></div>`
4. `<script>` PE编辑器主脚本 `</script>`

---

## 5. 保存函数核心逻辑

```javascript
function save() {
  var data = {}, imgs = {};
  // 收集字段（innerHTML保留strong等格式标签）
  document.querySelectorAll('[data-field]').forEach(function(el){
    if (!el.classList.contains('proto-img-slot')) data[el.dataset.field] = el.innerHTML;
  });
  // 收集图片
  document.querySelectorAll('[data-slot]').forEach(function(el){
    var img = el.querySelector('img');
    if (img && img.src && img.src.startsWith('data:')) imgs[el.dataset.slot] = img.src;
  });
  // 写入内嵌标签（数据跟文件走）
  var ds = document.getElementById('pe-saved-data');
  if (ds) ds.textContent = JSON.stringify({fields:data, images:imgs});
  // localStorage备份
  try { localStorage.setItem('pe_portfolio_v1', JSON.stringify(data)); localStorage.setItem('pe_images_v1', JSON.stringify(imgs)); } catch(e){}
  exitEdit();
  // 本地/Hermes：下载新文件；在线平台：localStorage已保存
  if (window.PE_LOCAL_MODE) {
    var fullHTML = '<!DOCTYPE html>\n' + document.documentElement.outerHTML; // 注意：\n是转义，不是真实换行
    var blob = new Blob([fullHTML],{type:'text/html;charset=utf-8'});
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a'); a.href=url; a.download='姓名-产品经理作品集.html';
    document.body.appendChild(a); a.click();
    setTimeout(function(){ URL.revokeObjectURL(url); document.body.removeChild(a); },1000);
  }
}
```

---

## 6. 恢复函数（三层优先级）

```javascript
function init() {
  var savedFields=null, savedImages=null;
  // 第1优先：pe-saved-data内嵌（本地文件自带）
  try {
    var ds=document.getElementById('pe-saved-data');
    if(ds){var raw=ds.textContent.trim();
      if(raw&&raw!=='{}'&&raw.length>5){var p=JSON.parse(raw);
        if(p.fields&&Object.keys(p.fields).length>0){savedFields=p.fields;savedImages=p.images||{};}}}
  } catch(e){}
  // 第2优先：localStorage（在线平台刷新恢复）
  if(!savedFields){
    try{var r=localStorage.getItem('pe_portfolio_v1');if(r){var p2=JSON.parse(r);if(p2&&Object.keys(p2).length>0)savedFields=p2;}}catch(e){}
    try{var r2=localStorage.getItem('pe_images_v1');if(r2)savedImages=JSON.parse(r2);}catch(e){}
  }
  // 第3优先：HTML原始内容（默认值）
  if(savedFields) Object.entries(savedFields).forEach(function([f,c]){
    var el=document.querySelector('[data-field="'+f+'"]');
    if(el&&!el.classList.contains('proto-img-slot')) el.innerHTML=c;
  });
  // 恢复图片省略...
}
window.addEventListener('DOMContentLoaded', init);
```

---

## 7. 增删系统支持的操作

所有删除/添加按钮必须有 `contenteditable="false"`。

支持操作：项目卡片、步骤条目（序号自动递增）、功能范围行、AI工具卡、AI思考卡、GitHub项目卡、市场洞察卡、职业轨迹、成长维度、原型图区块、标签（hero/career/badge）
