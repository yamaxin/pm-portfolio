# 增删系统完整代码

生成HTML时，将以下7个script标签**按顺序完整复制**到PE主脚本之后。
每个script必须独立存在，禁止合并。

## 列表增删系统

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
        // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = container.parentNode
        ? Array.from(container.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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
    // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = grid.parentNode
        ? Array.from(grid.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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
    btn.type = 'button';
    btn.setAttribute('contenteditable', 'false');
    btn.setAttribute('data-pe-addbtn', label); // 用于去重
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

## 卡片级别增删系统

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
    // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = grid.parentNode
        ? Array.from(grid.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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
    // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = grid.parentNode
        ? Array.from(grid.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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
    btn.type = 'button';
    btn.setAttribute('contenteditable', 'false');
    btn.setAttribute('data-pe-addbtn', label); // 用于去重
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

## 原型区块删除

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

## GitHub卡片完整编辑系统

<script>
/* ── GitHub卡片编辑系统 ── */
(function(){
  function initGithubCards(){
    document.querySelectorAll('.github-card').forEach(function(card){
      if(card.dataset.ghInit === 'done') return;
      card.dataset.ghInit = 'done';
      // github-card 现已是 div，gh-name 字段直接编辑即为链接文字
    });
  }

  const ob = new MutationObserver(function(){
    if(document.body.classList.contains('pe-edit-active')) initGithubCards();
  });
  ob.observe(document.body, {attributes:true, attributeFilter:['class']});
  window.addEventListener('DOMContentLoaded', initGithubCards);
})();
</script>


## 开源项目增删

<script>
/* ── 开源项目增删 ── */
(function(){
  function initGithubGrid(){
    const grid = document.querySelector('.github-grid');
    if(!grid || grid.dataset.ghGridInit) return;
    grid.dataset.ghGridInit = '1';

    grid.querySelectorAll('.github-card').forEach(function(card){ addGhDel(card); });

    const addBtn = document.createElement('button');
    addBtn.className = 'pe-card-add-btn';
    addBtn.type = 'button';
    addBtn.setAttribute('contenteditable', 'false');
    addBtn.setAttribute('data-pe-addbtn', '+ 新增开源项目');
    addBtn.textContent = '+ 新增开源项目';
    addBtn.addEventListener('click', function(e){
      e.stopPropagation();
      const tpl = grid.querySelector('.github-card').cloneNode(true);
      const ts = Date.now();
      tpl.dataset.ghInit = '';
      // 清理旧按钮
      tpl.querySelectorAll('.pe-card-del').forEach(function(b){ b.remove(); });
      // 重置字段内容
      tpl.querySelectorAll('[data-field]').forEach(function(el){
        el.textContent = '点击编辑';
        el.dataset.field = el.dataset.field + '_' + ts;
        if(document.body.classList.contains('pe-edit-active')){
          el.setAttribute('contenteditable', 'true');
          el.setAttribute('spellcheck', 'false');
        }
      });
      addGhDel(tpl);
      grid.appendChild(tpl);
      PE._toast('✅ 新项目已添加，点击文字编辑');
    });
    // 去重
    document.querySelectorAll('[data-pe-addbtn="+ 新增开源项目"]').forEach(function(b){ b.remove(); });
    grid.after(addBtn);
  }

  function addGhDel(card){
    if(card.querySelector('.pe-card-del')) return;
    const btn = document.createElement('button');
    btn.className = 'pe-card-del';
    btn.type = 'button';
    btn.setAttribute('contenteditable', 'false');
    btn.innerHTML = '× 删除';
    btn.addEventListener('click', function(e){
      e.stopPropagation(); e.preventDefault();
      const grid = document.querySelector('.github-grid');
      if(grid.querySelectorAll('.github-card').length > 1){
        card.remove(); PE._toast('项目已删除');
      } else {
        PE._toast('⚠️ 至少保留一个项目');
      }
    });
    card.appendChild(btn);
  }

  const ob = new MutationObserver(function(){
    if(document.body.classList.contains('pe-edit-active')) initGithubGrid();
  });
  ob.observe(document.body, {attributes:true, attributeFilter:['class']});
})();
</script>


## 职业轨迹+成长卡片增删

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
    // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = timeline.parentNode
        ? Array.from(timeline.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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
    // 去重：先移除已有的同标签新增按钮
    if(addBtn.dataset.peAddbtn){
      const existing = grid.parentNode
        ? Array.from(grid.parentNode.querySelectorAll('[data-pe-addbtn]'))
            .filter(b => b.dataset.peAddbtn === addBtn.dataset.peAddbtn && b !== addBtn)
        : [];
      existing.forEach(b => b.remove());
    }
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

## 标签增删系统

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

