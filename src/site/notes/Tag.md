---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 1. 基础配置：隐藏多余元素，对齐首页色调 */
  :root {
    --tag-primary: #4CAF50;
    --tag-bg-sub: #f9fbf9;
    --tag-text-main: #2d3748;
    --tag-text-muted: #718096;
    --tag-border: rgba(0,0,0,0.06);
  }

  h1[data-note-icon], .header-meta { display: none !important; }
  #cgfan-tag-page { max-width: 850px; margin: 0 auto; padding: 20px; font-family: inherit; }

  /* 2. 头部美化 */
  .tag-header { text-align: center; padding: 60px 0 40px; }
  .selection-badge { 
    display: inline-block; padding: 4px 14px; background: #e8f5e9; 
    color: var(--tag-primary); border-radius: 30px; font-size: 11px; 
    font-weight: 800; letter-spacing: 2px; margin-bottom: 12px;
  }
  #tag-title { font-size: 2.8rem !important; font-weight: 800 !important; color: var(--tag-text-main) !important; margin: 0 !important; }
  #tag-title span { color: var(--tag-primary); }
  .tag-subtitle { color: var(--tag-text-muted); margin-top: 10px; font-size: 0.95rem; }
  .back-link { 
    display: inline-block; margin-top: 25px; color: var(--tag-primary); 
    text-decoration: none; font-weight: 700; font-size: 0.9rem;
    transition: all 0.2s; border-bottom: 2px solid transparent;
  }
  .back-link:hover { border-bottom-color: var(--tag-primary); opacity: 0.8; }

  /* 3. 卡片列表布局 (仿首页美化) */
  .tag-card { 
    display: flex !important; background: #fff; border: 1px solid var(--tag-border); 
    border-radius: 20px; overflow: hidden; margin-bottom: 28px; height: 190px; 
    box-shadow: 0 4px 20px rgba(0,0,0,0.03); transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
  }
  .tag-card:hover { 
    transform: translateY(-5px) translateX(8px); 
    border-color: var(--tag-primary); 
    box-shadow: 0 15px 35px rgba(76, 175, 80, 0.12); 
  }

  /* 左侧图片区 */
  .tag-card-img { 
    width: 280px; flex-shrink: 0; height: 100%; background: #f0f2f0; 
    display: block; overflow: hidden;
  }
  .tag-card-img img { 
    width: 100%; height: 100%; object-fit: cover; display: block; 
    transition: transform 0.6s ease;
  }
  .tag-card:hover .tag-card-img img { transform: scale(1.08); }

  /* 右侧文字内容 */
  .tag-card-body { 
    flex-grow: 1; padding: 28px 32px; display: flex; 
    flex-direction: column; justify-content: space-between; min-width: 0; 
  }
  .tag-card-title { 
    font-size: 1.3rem; font-weight: 700; color: var(--tag-text-main); 
    text-decoration: none; line-height: 1.45; display: -webkit-box; 
    -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; 
    margin-bottom: 10px;
  }
  .tag-card-title:hover { color: var(--tag-primary); }

  /* 作者与日期信息 */
  .tag-card-meta { 
    font-size: 13px; color: var(--tag-text-muted); display: flex; gap: 20px; 
    align-items: center; margin-bottom: 15px;
  }
  .tag-card-meta span { display: flex; align-items: center; gap: 6px; }

  /* 标签云与高亮逻辑 */
  .tag-card-tags { display: flex; flex-wrap: wrap; gap: 8px; height: 26px; overflow: hidden; }
  .tag-pill { 
    font-size: 11px; padding: 3px 10px; background: #f5f7f5; 
    color: var(--tag-text-muted); border-radius: 6px; border: 1px solid #edf2f0;
    transition: all 0.2s;
  }
  .tag-pill.active { 
    background: #e8f5e9; color: var(--tag-primary); 
    border-color: #c8e6c9; font-weight: 800; 
  }

  /* 4. 移动端适配：垂直堆叠 */
  @media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; margin: 0 10px 25px; }
    .tag-card-img { width: 100% !important; height: 200px !important; }
    .tag-card-body { padding: 20px !important; }
    .tag-card-title { font-size: 1.15rem; }
    .tag-card-tags { height: auto; } /* 移动端展开所有标签 */
  }

  /* 加载动画 */
  .loading-state { text-align: center; padding: 80px 0; color: var(--tag-text-muted); }
  .spinner {
    width: 35px; height: 35px; border: 3px solid #eee; border-top: 3px solid var(--tag-primary);
    border-radius: 50%; margin: 0 auto 20px; animation: tag-spin 1s linear infinite;
  }
  @keyframes tag-spin { to { transform: rotate(360deg); } }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <p class="tag-subtitle">正在为您聚合灵感索引...</p>
        <a href="/" class="back-link">← 返回全量画廊</a>
    </div>
    <div id="tag-results">
        <div class="loading-state">
            <div class="spinner"></div>
            <p>正在从画廊提取灵感...</p>
        </div>
    </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', async () => {
    const params = new URLSearchParams(window.location.search);
    const targetTag = params.get('tag');
    const container = document.getElementById('tag-results');
    const titleSpan = document.querySelector('#tag-title span');

    if (!targetTag) {
        titleSpan.innerText = "全部";
        container.innerHTML = `<div class="loading-state">请从首页选择感兴趣的标签。</div>`;
        return;
    }

    titleSpan.innerText = targetTag;
    const normalizedTarget = targetTag.replace('#', '').toLowerCase();

    try {
        const response = await fetch('/searchIndex.json?v=' + Date.now());
        const allData = await response.json();
        
        // 过滤系统页面并匹配标签
        const filtered = allData.filter(item => {
            if (item.url === '/' || item.url.includes('/tag/')) return false;
            if (!item.tags) return false;
            return item.tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        if (filtered.length === 0) {
            container.innerHTML = `<div class="loading-state">尚未收录关于 #${targetTag} 的内容。</div>`;
            return;
        }

        let html = "";
        filtered.forEach(item => {
            // 封面图逻辑
            let imgUrl = item.cover;
            if (!imgUrl && item.content) {
                const mdMatch = item.content.match(/!\[.*?\]\((.*?)\)/);
                if (mdMatch) { imgUrl = mdMatch[1]; } 
                else {
                    const urlMatch = item.content.match(/https?:\/\/[^\s]+?\.(jpg|jpeg|png|gif|webp)/i);
                    if (urlMatch) imgUrl = urlMatch[0];
                }
            }
            if (!imgUrl) imgUrl = "https://via.placeholder.com/600x400?text=CGFAN+Collection";

            // 反代处理
            if (imgUrl.includes('twimg.com')) {
                imgUrl = `https://images.weserv.nl/?url=${encodeURIComponent(imgUrl)}&w=600`;
            }

            // 日期美化
            let dateStr = '-';
            if (item.date) {
                const d = new Date(item.date);
                if (!isNaN(d)) dateStr = d.toISOString().split('T')[0];
            }

            // 标签高亮：标记当前活跃标签
            const tagsHtml = item.tags.map(t => {
                if (t === 'gardenEntry' || t === 'note') return '';
                const isActive = t.toLowerCase() === normalizedTarget ? 'active' : '';
                return `<span class="tag-pill ${isActive}">#${t}</span>`;
            }).join('');

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${imgUrl}" loading="lazy">
                </a>
                <div class="tag-card-body">
                    <div>
                        <a href="${item.url}" class="tag-card-title">${item.title}</a>
                        <div class="tag-card-meta">
                            <span>👤 ${item.author}</span>
                            <span>📅 ${dateStr}</span>
                        </div>
                    </div>
                    <div class="tag-card-tags">${tagsHtml}</div>
                </div>
            </div>`;
        });

        container.innerHTML = html;
        document.querySelector('.tag-subtitle').innerText = `已为您发现 ${filtered.length} 个相关灵感`;

    } catch (e) {
        console.error(e);
        container.innerHTML = `<div class="loading-state">同步失败，请刷新首页重试。</div>`;
    }
});
</script>