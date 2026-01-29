---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 1. 样式隔离：全 RGB 格式防止插件干扰 */
  :root {
    --cg-primary: rgb(76, 175, 80) !important;
    --cg-card-bg: rgb(255, 255, 255) !important;
    --cg-text-title: rgb(33, 37, 41) !important;
    --cg-text-meta: rgb(108, 117, 125) !important;
    --cg-border-subtle: rgba(0, 0, 0, 0.1) !important;
  }

  /* 隐藏系统默认组件 */
  h1[data-note-icon], .header-meta, .header-tags, .footer { display: none !important; }
  
  #cgfan-tag-page { max-width: 800px; margin: 0 auto; padding: 40px 15px; }

  /* 头部装饰 */
  .tag-header { text-align: center; margin-bottom: 50px; }
  .selection-badge { 
    display: inline-block; padding: 4px 14px; background: rgb(232, 245, 233) !important; 
    color: var(--cg-primary) !important; border-radius: 30px; font-size: 11px; font-weight: 800; letter-spacing: 2px;
  }
  #tag-title { font-size: 2.5rem !important; font-weight: 900 !important; color: var(--cg-text-title) !important; margin: 15px 0 !important; }
  #tag-title span { color: var(--cg-primary) !important; }
  .back-btn { color: var(--cg-primary); text-decoration: none; font-weight: bold; border-bottom: 1px solid transparent; }
  .back-btn:hover { border-bottom-color: var(--cg-primary); }

  /* 2. 画廊级立体卡片 (仿首页小图风格) */
  .tag-card { 
    display: flex !important; 
    background: var(--cg-card-bg) !important; 
    border: 1px solid var(--cg-border-subtle) !important; 
    border-radius: 16px !important; 
    overflow: hidden !important; 
    margin-bottom: 25px !important; 
    height: 160px !important; /* 稍微调低高度，更像首页质感 */
    /* 核心：立体阴影 */
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 10px 20px -5px rgba(0, 0, 0, 0.08) !important;
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1) !important;
  }

  .tag-card:hover { 
    transform: translateY(-5px) !important;
    border-color: var(--cg-primary) !important;
    box-shadow: 0 15px 30px -10px rgba(76, 175, 80, 0.2) !important;
  }

  /* 左侧缩略图 */
  .tag-card-img { 
    width: 160px; /* 正方形展示，和首页小图逻辑一致 */
    flex-shrink: 0; 
    height: 100%; 
    background: rgb(248, 249, 250); 
    border-right: 1px solid var(--cg-border-subtle); 
    overflow: hidden; 
  }
  .tag-card-img img { 
    width: 100%; height: 100%; object-fit: cover; display: block; 
    transition: transform 0.6s ease;
  }
  .tag-card:hover .tag-card-img img { transform: scale(1.1); }

  /* 右侧详情 */
  .tag-card-body { 
    flex-grow: 1; padding: 18px 24px; display: flex; 
    flex-direction: column; justify-content: space-between; min-width: 0; 
  }
  .tag-card-title { 
    font-size: 1.15rem !important; font-weight: 700 !important; color: var(--cg-text-title) !important; 
    text-decoration: none !important; line-height: 1.4 !important; 
    display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; 
  }
  .tag-card-meta { font-size: 12px; color: var(--cg-text-meta); display: flex; gap: 15px; }
  
  /* 标签 */
  .tag-card-tags { display: flex; gap: 6px; overflow: hidden; height: 22px; }
  .tag-pill { 
    font-size: 10px; padding: 2px 8px; background: rgb(241, 243, 245); 
    border-radius: 4px; color: var(--cg-text-meta); border: 1px solid transparent;
  }
  .tag-pill.active { 
    background: rgb(232, 245, 233); color: var(--cg-primary); 
    font-weight: bold; border-color: rgba(76, 175, 80, 0.2); 
  }

  @media (max-width: 600px) {
    .tag-card { height: 140px !important; }
    .tag-card-img { width: 120px !important; }
    .tag-card-body { padding: 12px 16px !important; }
    .tag-card-title { font-size: 1rem !important; }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" class="back-btn">← 返回全量画廊</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:50px; color:#999;">正在请求小图资源...</div>
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
        container.innerHTML = "请返回首页选择标签";
        return;
    }

    titleSpan.innerText = targetTag;
    const normalizedTarget = targetTag.replace('#', '').toLowerCase();

    try {
        const response = await fetch('/searchIndex.json?v=' + Date.now());
        const allData = await response.json();
        
        const filtered = allData.filter(item => {
            if (item.url === '/' || item.url.includes('/tag/')) return false;
            return item.tags && item.tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        let html = "";
        filtered.forEach(item => {
            let imgUrl = "";

            // 1. 尝试抓取 cover
            if (item.cover && item.cover.length > 5) {
                imgUrl = item.cover;
            } 
            // 2. 暴力嗅探内容中的 URL
            else if (item.content) {
                const matches = item.content.match(/https?:\/\/[^\s"'<>]+\.(?:jpg|jpeg|png|gif|webp|large|orig)/gi);
                if (matches) imgUrl = matches[0];
            }

            // --- 关键：统一抓取小图的格式控制 ---
            let finalImg = "";
            if (imgUrl) {
                let cleanUrl = decodeURIComponent(imgUrl).split(/[">)]/)[0].trim();
                
                if (cleanUrl.includes('twimg.com')) {
                    // 砍掉参数，强制请求 400px 宽度的小图 (w=400)
                    cleanUrl = cleanUrl.split('?')[0].split(':')[0] + ':' + cleanUrl.split('?')[0].split(':')[1];
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=400&h=400&fit=cover&il`;
                } else {
                    // 非 Twitter 图片也进行等比小图处理
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=400&h=400&fit=cover`;
                }
            } else {
                finalImg = "https://via.placeholder.com/400x400?text=CGFAN";
            }

            let dateStr = '-';
            if (item.date) {
                const d = new Date(item.date);
                if (!isNaN(d)) dateStr = d.toISOString().split('T')[0];
            }

            const tagsHtml = item.tags ? item.tags.map(t => {
                if (t === 'gardenEntry' || t === 'note') return '';
                return `<span class="tag-pill ${t.toLowerCase() === normalizedTarget ? 'active' : ''}">#${t}</span>`;
            }).join('') : '';

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/400x400?text=Wait+Load';">
                </a>
                <div class="tag-card-body">
                    <div>
                        <a href="${item.url}" class="tag-card-title">${item.title}</a>
                        <div class="tag-card-meta">
                            <span>👤 ${item.author || 'CGFan'}</span>
                            <span>📅 ${dateStr}</span>
                        </div>
                    </div>
                    <div class="tag-card-tags">${tagsHtml}</div>
                </div>
            </div>`;
        });

        container.innerHTML = filtered.length > 0 ? html : "未发现相关灵感";

    } catch (e) {
        container.innerHTML = "加载异常，请重试";
    }
});
</script>