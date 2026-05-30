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

---

## 完整页面CSS（插入到 `<style>` 标签内，编辑器CSS之前）

```css
/* ─── TOKENS ─── */
:root {
  --cream:#f0f5ff; --paper:#ffffff; --ink:#1A2035;
  --ink-2:#374151; --ink-3:#6B7280; --ink-4:#94a3b8;
  --blue:#4F6EF7; --blue-lt:#EEF0FF; --blue-md:#C7D2FE;
  --blue-dark:#2D3FBF; --blue-deep:#1E1B6E;
  --rule:rgba(15,23,42,0.08); --shadow:0 2px 24px rgba(15,23,42,0.08);
  --r:12px; --r-lg:20px;
  --green:#4F6EF7; --green-lt:#EEF0FF; --green-md:#C7D2FE;
}
*,*::before,*::after{margin:0;padding:0;box-sizing:border-box;}
html{scroll-behavior:smooth;}
body{background:#F4F6FF;color:var(--ink);font-family:'DM Sans',sans-serif;font-weight:400;line-height:1.6;overflow-x:hidden;}
a{color:inherit;text-decoration:none;}
::-webkit-scrollbar{width:5px;}
::-webkit-scrollbar-track{background:var(--cream);}
::-webkit-scrollbar-thumb{background:#93c5fd;border-radius:3px;}

/* ─── NAV ─── */
.nav{position:fixed;top:0;left:0;right:0;z-index:200;height:60px;display:flex;align-items:center;justify-content:space-between;padding:0 48px;background:rgba(244,246,255,0.94);backdrop-filter:blur(16px);border-bottom:1px solid var(--rule);}
.nav-brand{font-family:'Playfair Display',serif;font-size:18px;font-weight:700;}
.nav-links{display:flex;gap:32px;list-style:none;}
.nav-links a{font-size:13px;font-weight:500;color:var(--ink-3);letter-spacing:.04em;transition:color .2s;}
.nav-links a:hover{color:#4F6EF7;}

/* ─── LAYOUT ─── */
.wrap{max-width:1160px;margin:0 auto;padding:0 48px;}
.section{padding:100px 0;}
.section+.section{border-top:1px solid rgba(37,99,235,0.08);}
.label{font-size:11px;font-weight:600;letter-spacing:.14em;text-transform:uppercase;color:#4F6EF7;font-family:'DM Mono',monospace;margin-bottom:12px;}
.section-title{font-family:'Playfair Display',serif;font-size:clamp(34px,4vw,52px);font-weight:800;line-height:1.1;letter-spacing:-.02em;}
.section-title em{font-style:italic;color:#4F6EF7;}

/* ─── REVEAL ─── */
.reveal{opacity:0;transform:translateY(28px);transition:opacity .65s ease,transform .65s ease;}
.reveal.in{opacity:1;transform:translateY(0);}
.reveal-d1{transition-delay:.08s;}.reveal-d2{transition-delay:.16s;}
.reveal-d3{transition-delay:.24s;}.reveal-d4{transition-delay:.32s;}

/* ─── HERO ─── */
.hero{min-height:100vh;padding-top:60px;display:grid;grid-template-columns:1fr 1fr;align-items:center;background:linear-gradient(160deg,#EEF0FF 0%,#E5E9FF 40%,#EBF4FF 100%);position:relative;overflow:hidden;}
.hero::before{content:'PM';position:absolute;right:-20px;bottom:-40px;font-family:'Playfair Display',serif;font-size:380px;font-weight:800;color:rgba(37,99,235,.06);line-height:1;pointer-events:none;user-select:none;}
.hero-left{padding:80px 56px 80px 48px;display:flex;flex-direction:column;gap:0;}
.hero-eyebrow{display:inline-flex;align-items:center;gap:10px;margin-bottom:28px;}
.hero-dot{width:8px;height:8px;background:#4F6EF7;border-radius:50%;animation:blink 2.2s infinite;}
@keyframes blink{0%,100%{opacity:1}50%{opacity:.3}}
.hero-eyebrow-text{font-size:12px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:#4f46e5;font-family:'DM Mono',monospace;}
.hero-name{font-family:'Playfair Display',serif;font-size:clamp(64px,7vw,96px);font-weight:800;line-height:.95;letter-spacing:-.03em;margin-bottom:24px;}
.hero-quote-block{background:linear-gradient(135deg,#2D3FBF 0%,#4F6EF7 100%);color:#fff;border-radius:var(--r-lg);padding:28px 32px;margin-bottom:36px;position:relative;overflow:hidden;}
.hero-quote-block::before{content:'"';position:absolute;top:-12px;left:20px;font-family:'Playfair Display',serif;font-size:120px;font-weight:800;color:rgba(255,255,255,.06);line-height:1;pointer-events:none;}
.hero-quote-text{font-size:15px;line-height:1.75;font-style:italic;color:rgba(255,255,255,.88);margin-bottom:12px;position:relative;}
.hero-quote-author{font-size:12px;color:rgba(255,255,255,.7);font-weight:600;letter-spacing:.06em;font-family:'DM Mono',monospace;}
.hero-tags{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:36px;}
.tag{padding:5px 14px;border-radius:100px;font-size:12px;font-weight:500;letter-spacing:.03em;}
.tag-dark{background:#4F6EF7;color:#fff;}
.tag-green{background:var(--green-lt);color:var(--green);}
.tag-outline{border:1.5px solid var(--rule);color:var(--ink-2);}
.hero-contacts{display:flex;gap:20px;flex-wrap:wrap;}
.hero-contact-item{display:flex;align-items:center;gap:7px;font-size:13px;color:var(--ink-3);transition:color .2s;}
.hero-contact-item:hover{color:var(--green);}
.hero-contact-item svg{width:14px;height:14px;fill:currentColor;flex-shrink:0;}
.hero-right{padding:80px 48px 80px 0;display:grid;grid-template-columns:1fr 1fr;grid-template-rows:auto auto auto;gap:16px;align-content:center;}
.stat-card{background:var(--paper);border-radius:var(--r-lg);padding:28px 24px 24px;border:1px solid rgba(79,110,247,0.1);box-shadow:0 2px 16px rgba(79,110,247,0.06);transition:transform .25s,box-shadow .25s;}
.stat-card:hover{transform:translateY(-4px);box-shadow:0 12px 36px rgba(0,0,0,.1);}
.stat-card.dark{background:linear-gradient(135deg,#2D3FBF 0%,#7B5CF5 100%);border-color:transparent;}
.stat-card.green{background:linear-gradient(135deg,#4F6EF7 0%,#06B6D4 100%);border-color:transparent;}
.stat-num{font-family:'Playfair Display',serif;font-size:48px;font-weight:800;line-height:1;letter-spacing:-.03em;margin-bottom:6px;color:var(--ink);}
.stat-card.dark .stat-num,.stat-card.green .stat-num{color:#fff;}
.stat-num .unit{font-size:24px;color:var(--green);}
.stat-card.dark .unit,.stat-card.green .unit{color:rgba(255,255,255,.6);}
.stat-desc{font-size:12px;font-weight:500;color:var(--ink-3);letter-spacing:.03em;text-transform:uppercase;font-family:'DM Mono',monospace;}
.stat-card.dark .stat-desc,.stat-card.green .stat-desc{color:rgba(255,255,255,.6);}
.stat-card.wide{grid-column:span 2;}

/* ─── PROJECTS ─── */
.projects-section{background:#F7F8FF;}
.projects-intro{display:grid;grid-template-columns:1fr 1fr;gap:48px;align-items:end;margin-bottom:64px;}
.projects-intro-right{font-size:15px;line-height:1.8;color:var(--ink-2);}
.proj{background:var(--paper);border-radius:var(--r-lg);border:1px solid rgba(79,110,247,0.1);box-shadow:0 4px 24px rgba(79,110,247,0.07);margin-bottom:40px;}
.proj:last-child{margin-bottom:0;}
.proj-head{padding:44px 48px 36px;border-bottom:1px solid var(--rule);}
.proj-badges{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:16px;}
.badge{padding:3px 12px;border-radius:100px;font-size:11px;font-weight:600;letter-spacing:.06em;text-transform:uppercase;font-family:'DM Mono',monospace;}
.badge-ai{background:#EEF0FF;color:#4F6EF7;}
.badge-sys{background:#eff6ff;color:#1d4ed8;}
.badge-biz{background:#fff7ed;color:#c2410c;}
.badge-infra{background:#f5f3ff;color:#6d28d9;}
.badge-live{background:#f0fdf4;color:#15803d;border:1px solid #bbf7d0;}
.badge-prd{background:#fafafa;color:#555;border:1px solid #ddd;}
.proj-title{font-family:'Playfair Display',serif;font-size:28px;font-weight:800;letter-spacing:-.02em;line-height:1.15;margin-bottom:10px;}
.proj-sub{font-size:14px;line-height:1.75;color:var(--ink-2);max-width:700px;}
.proj-metrics{display:flex;gap:28px;flex-wrap:wrap;margin-top:20px;}
.metric{display:flex;flex-direction:column;}
.metric-n{font-family:'Playfair Display',serif;font-size:28px;font-weight:800;color:var(--green);letter-spacing:-.02em;line-height:1;}
.metric-l{font-size:11px;font-weight:500;color:var(--ink-3);text-transform:uppercase;letter-spacing:.06em;margin-top:3px;font-family:'DM Mono',monospace;}
.proj-body{display:grid;grid-template-columns:1fr 1fr;}
.proj-col{padding:36px 48px;}
.proj-col+.proj-col{border-left:1px solid var(--rule);}
.proj-col-label{font-size:11px;font-weight:600;letter-spacing:.12em;text-transform:uppercase;color:#4f46e5;font-family:'DM Mono',monospace;margin-bottom:14px;}
.proj-col-text{font-size:14px;line-height:1.8;color:var(--ink-2);margin-bottom:20px;}
.proj-steps{list-style:none;display:flex;flex-direction:column;gap:14px;}
.proj-step{display:flex;gap:12px;}
.step-num{width:22px;height:22px;border-radius:6px;background:#7B5CF5;color:#fff;font-size:11px;font-weight:700;display:flex;align-items:center;justify-content:center;flex-shrink:0;margin-top:2px;font-family:'DM Mono',monospace;}
.step-body strong{display:block;font-size:13px;font-weight:600;margin-bottom:3px;}
.step-body span{font-size:12px;color:var(--ink-3);line-height:1.6;}
.proj-outcome{padding:24px 48px;background:linear-gradient(135deg,#EEF0FF,#E8E4FF);border-top:1px solid rgba(37,99,235,0.08);display:flex;gap:12px;align-items:flex-start;}
.outcome-icon{width:32px;height:32px;background:linear-gradient(135deg,#4F6EF7,#7B5CF5);border-radius:8px;display:flex;align-items:center;justify-content:center;flex-shrink:0;margin-top:2px;font-size:16px;}
.outcome-label{font-size:11px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:var(--green);font-family:'DM Mono',monospace;margin-bottom:5px;}
.outcome-text{font-size:14px;line-height:1.75;color:var(--ink-2);}
.outcome-text strong{color:var(--ink);font-weight:600;}
.proj-proto{padding:0 48px 36px;border-top:1px solid var(--rule);}
.proto-header{padding-top:28px;font-size:11px;font-weight:600;letter-spacing:.12em;text-transform:uppercase;color:var(--ink-4);font-family:'DM Mono',monospace;margin-bottom:16px;}
.proto-grid{display:grid;grid-template-columns:1fr 1fr;gap:16px;}
.proto-frame{border-radius:10px;border:1px solid var(--rule);}
.proto-cap{padding:10px 14px;font-size:12px;font-weight:500;color:var(--ink-3);border-top:1px solid var(--rule);background:var(--paper);}

/* 诊断弹窗 */
.mock-diag{background:#fff;}
.mock-diag-header{padding:12px 16px;border-bottom:1px solid #f0f0f0;display:flex;align-items:center;justify-content:space-between;}
.mock-diag-title{font-size:13px;font-weight:700;}
.mock-diag-badge{background:#fef2f2;color:#dc2626;font-size:10px;font-weight:700;padding:2px 8px;border-radius:4px;}
.mock-diag-body{padding:12px 16px;}
.mock-diag-alert{background:#fff7ed;border:1px solid #fed7aa;border-radius:6px;padding:8px 10px;margin-bottom:10px;}
.mock-diag-alabel{font-size:10px;font-weight:700;color:#ea580c;margin-bottom:3px;}
.mock-diag-acontent{font-size:10px;color:#7c3aed;font-family:'DM Mono',monospace;}
.mock-causes-title{font-size:11px;font-weight:600;color:#374151;margin-bottom:6px;}
.mock-cause{display:flex;gap:6px;align-items:flex-start;margin-bottom:4px;font-size:10px;color:#6b7280;}
.cause-dot{width:5px;height:5px;background:#f59e0b;border-radius:50%;flex-shrink:0;margin-top:3px;}
.mock-actions{margin-top:10px;display:flex;flex-direction:column;gap:5px;}
.mock-action-row{display:flex;align-items:center;justify-content:space-between;background:#f9fafb;border-radius:5px;padding:6px 8px;}
.mock-action-text{font-size:10px;color:#374151;}
.mock-exec{background:#2563eb;color:#fff;border:none;border-radius:3px;padding:2px 8px;font-size:9px;font-weight:700;}
.mock-sensitive{background:#f1f5f9;color:#64748b;border-radius:3px;padding:2px 8px;font-size:9px;font-weight:600;}

/* ─── AI MODULE ─── */
.ai-section{background:#F8F7FF;}
.ai-grid{display:grid;grid-template-columns:1fr 1fr 1fr;gap:24px;margin-bottom:56px;}
.ai-tool-card{border:1px solid rgba(37,99,235,0.12);border-radius:var(--r-lg);padding:28px 24px;position:relative;overflow:hidden;transition:transform .25s,box-shadow .25s;background:var(--paper);}
.ai-tool-card:hover{transform:translateY(-4px);box-shadow:0 12px 40px rgba(0,0,0,.09);}
.ai-tool-card::before{content:'';position:absolute;top:0;left:0;right:0;height:3px;background:#4F6EF7;}
.ai-tool-card:nth-child(2)::before{background:#7B5CF5;}
.ai-tool-card:nth-child(3)::before{background:#06B6D4;}
.tool-emoji{font-size:26px;margin-bottom:14px;}
.tool-name{font-size:19px;font-weight:700;letter-spacing:-.01em;margin-bottom:4px;}
.tool-role{font-size:11px;font-weight:600;color:var(--green);letter-spacing:.08em;text-transform:uppercase;font-family:'DM Mono',monospace;margin-bottom:12px;}
.tool-desc{font-size:13px;line-height:1.75;color:var(--ink-2);margin-bottom:16px;}
.tool-uses{list-style:none;display:flex;flex-direction:column;gap:7px;}
.tool-uses li{font-size:12px;color:var(--ink-3);display:flex;gap:7px;align-items:flex-start;line-height:1.5;}
.tool-uses li::before{content:'→';color:var(--green);font-weight:700;flex-shrink:0;}
.ai-thinking-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:20px;margin-top:48px;}
.thinking-card{background:linear-gradient(160deg,#F0EDFF,#EAE4FF);border-radius:var(--r);padding:24px 22px;border-top:3px solid #7B5CF5;}
.thinking-num{font-size:11px;font-weight:600;color:#7B5CF5;letter-spacing:.1em;font-family:'DM Mono',monospace;margin-bottom:12px;}
.thinking-title{font-size:16px;font-weight:700;letter-spacing:-.01em;margin-bottom:10px;line-height:1.3;color:#1A2035;}
.thinking-text{font-size:13px;line-height:1.75;color:var(--ink-2);margin-bottom:14px;}
.thinking-verdict{background:var(--paper);border-radius:8px;padding:12px 14px;border-left:3px solid #7B5CF5;}
.thinking-verdict-label{font-size:10px;font-weight:600;color:#7B5CF5;letter-spacing:.1em;font-family:'DM Mono',monospace;margin-bottom:4px;}
.thinking-verdict-text{font-size:12px;color:var(--ink);line-height:1.6;}
.github-grid{display:grid;grid-template-columns:1fr 1fr;gap:16px;margin-top:48px;}
.github-card{background:linear-gradient(135deg,#2D3FBF 0%,#4F6EF7 100%);border-radius:var(--r-lg);padding:24px 26px;position:relative;transition:transform .25s,box-shadow .25s;display:block;}
.github-card:hover{transform:translateY(-4px);box-shadow:0 12px 36px rgba(79,110,247,.3);}
.github-card:nth-child(2){background:linear-gradient(135deg,#0A7B90 0%,#06B6D4 100%);}
.github-card:nth-child(2):hover{box-shadow:0 12px 36px rgba(6,182,212,.3);}
.github-card::after{content:'↗';position:absolute;top:20px;right:22px;font-size:18px;color:rgba(255,255,255,.4);transition:color .2s;}
.github-card:hover::after{color:#fff;}
.gh-label{font-size:10px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:rgba(255,255,255,.55);font-family:'DM Mono',monospace;margin-bottom:10px;}
.gh-name{font-size:20px;font-weight:700;color:#fff;letter-spacing:-.01em;margin-bottom:8px;font-family:'DM Mono',monospace;}
.gh-desc{font-size:13px;line-height:1.7;color:rgba(255,255,255,.75);margin-bottom:16px;}
.gh-tags{display:flex;gap:8px;flex-wrap:wrap;}
.gh-tag{padding:2px 10px;border:1px solid rgba(255,255,255,.25);border-radius:100px;font-size:11px;color:rgba(255,255,255,.7);}

/* ─── MARKET ─── */
.market-section{background:linear-gradient(160deg,#1E1B6E 0%,#2D3FBF 50%,#1E1B6E 100%);color:#fff;}
.market-section .label{color:#A5B4FC;}
.market-section .section-title em{color:#A5B4FC;}
.market-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:16px;background:transparent;border-radius:var(--r-lg);margin-top:56px;}
.market-card{background:rgba(255,255,255,0.06);padding:36px 32px;transition:background .25s;backdrop-filter:blur(2px);border-radius:16px;position:relative;}
.market-card:hover{background:rgba(255,255,255,0.12);}
.market-card:nth-child(1){border-top:2px solid #7B5CF5;}
.market-card:nth-child(2){border-top:2px solid #06B6D4;}
.market-card:nth-child(3){border-top:2px solid #4F6EF7;}
.market-card-num{font-family:'Playfair Display',serif;font-size:13px;font-weight:700;color:rgba(255,255,255,.25);margin-bottom:20px;font-style:italic;}
.market-signal-label{font-size:10px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:#a5b4fc;font-family:'DM Mono',monospace;margin-bottom:8px;}
.market-signal{font-size:14px;color:#aaa;line-height:1.7;margin-bottom:20px;}
.market-arrow{font-size:20px;color:#333;margin-bottom:12px;}
.market-judge-label{font-size:10px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:rgba(255,255,255,.5);font-family:'DM Mono',monospace;margin-bottom:8px;}
.market-judge{font-size:15px;font-weight:600;color:#fff;line-height:1.65;margin-bottom:20px;}
.market-result-label{font-size:10px;font-weight:600;letter-spacing:.1em;text-transform:uppercase;color:#a5b4fc;font-family:'DM Mono',monospace;margin-bottom:6px;}
.market-result{font-size:13px;color:#aaa;line-height:1.7;}
.market-result strong{color:#A5B4FC;}
.market-strip{margin-top:56px;display:grid;grid-template-columns:repeat(4,1fr);gap:1px;background:rgba(147,197,253,0.08);border-radius:var(--r-lg);overflow:hidden;}
.market-strip-item{background:rgba(255,255,255,0.07);padding:28px 24px;display:flex;flex-direction:column;align-items:center;text-align:center;gap:6px;}
.market-strip-n{font-family:'Playfair Display',serif;font-size:36px;font-weight:800;color:#fff;}
.market-strip-n span{color:#93c5fd;}
.market-strip-l{font-size:11px;color:#555;font-family:'DM Mono',monospace;letter-spacing:.06em;text-transform:uppercase;}

/* ─── CAREER ─── */
.career-section{background:#FFFCF5;}
.career-timeline{margin-top:56px;}
.career-item{display:grid;grid-template-columns:180px 1fr;gap:40px;padding-bottom:48px;position:relative;}
.career-item:not(:last-child)::after{content:'';position:absolute;left:176px;top:10px;bottom:0;width:1px;background:var(--rule);}
.career-dot{position:absolute;left:172px;top:8px;width:9px;height:9px;background:#F59E0B;border-radius:50%;border:2px solid #FFFCF5;box-shadow:0 0 0 2px #F59E0B;}
.career-meta{text-align:right;padding-right:20px;}
.career-period{font-size:12px;color:var(--ink-3);font-family:'DM Mono',monospace;margin-bottom:4px;}
.career-co{font-size:14px;font-weight:700;}
.career-role{font-size:12px;color:#F59E0B;font-weight:600;margin-top:2px;}
.career-title{font-size:20px;font-weight:700;letter-spacing:-.01em;margin-bottom:10px;}
.career-desc{font-size:14px;line-height:1.8;color:var(--ink-2);margin-bottom:14px;}
.career-tags{display:flex;gap:8px;flex-wrap:wrap;}
.growth-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:20px;margin-top:56px;}
.growth-card{background:var(--paper);border-radius:var(--r);padding:24px;border:1px solid rgba(79,110,247,0.1);}
.growth-label{font-size:11px;font-weight:600;color:#F59E0B;letter-spacing:.1em;text-transform:uppercase;font-family:'DM Mono',monospace;margin-bottom:14px;}
.growth-from{font-size:13px;color:var(--ink-3);margin-bottom:6px;padding-left:12px;border-left:2px solid var(--ink-4);}
.growth-arrow{font-size:18px;color:var(--green);margin:8px 0;padding-left:4px;}
.growth-to{font-size:14px;font-weight:600;color:var(--ink);padding-left:12px;border-left:2px solid #F59E0B;line-height:1.5;}
```
