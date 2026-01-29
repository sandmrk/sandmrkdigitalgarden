---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  :root {
    --cg-green: rgb(76, 175, 80) !important;
    --cg-border: rgba(0, 0, 0, 0.08) !important;
    --cg-text: rgb(33, 37, 41) !important;
    --cg-bg-card: rgb(255, 255, 255) !important;
  }

  h1[data-note-icon], .header-meta, .header-tags, .footer { display: none !important; }
  #cgfan-tag-page { max-width: 800px; margin: 0 auto; padding: 40px 15px; font-family: sans-serif; }

  .tag-header { text-align: center; margin-bottom: 50px; }
  .selection-badge { 
    display: inline-block; padding: 4px 14px; background: rgb(232, 245, 233) !important; 
    color: var(--cg-green) !important; border-radius: 30px; font-size: 11px; font-weight: 800;
  }
  #tag-title { font-size: 2.5rem !important; font-weight: 900 !important; color: var(--cg-text) !important; margin: 15px 0 !important; }
  #tag-title span { color: var(--cg-green) !important; }
  .back-btn { color: var(--cg-green); text-decoration: none; font-weight: bold; border-bottom: 1px solid transparent; }

  /* 仿首页小图卡片 */
  .tag-card { 
    display: flex !important; 
    background: var(--cg-bg-card) !important; 
    border: 1px solid var(--cg-border) !important; 
    border-radius: 16px !important; 
    overflow: hidden !important; 
    margin-bottom: 25px !important; 
    height: 140px !important; /* 降低高度 */
    box-shadow: 0 4px 10px rgba(0,0,0,0.03) !important;
    transition: all 0.3s ease !important;
  }

  .tag-card:hover { 
    transform: translateY(-4px) !important;
    border-color: var(--cg-green) !important;
    box-shadow: 0 15px 30px rgba(76, 175, 80, 0.15) !important;
  }

  /* 正方形小图 */
  .tag-card-img { 
    width: 140px; flex-shrink: 0; height: 100%; 
    background: rgb(248, 249, 250); border-right: 1px solid var(--cg-border); 
  }
  .tag-card-img img { width: 100%; height: 100%; object-fit: cover; display: block; }

  .tag-card-body { flex-grow: 1; padding: 15px 20px; display: flex; flex-direction: column; justify-content: space-between; min-width: 0; }
  .tag-card-title { font-size: 1.1rem !important; font-weight: 700 !important; color: var(--cg-text) !important; text-decoration: none !important; line-height: 1.4 !important; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
  .tag-card-meta { font-size: 12px; color: rgb(108, 117, 125); display: flex; gap: 12px; }
  
  .tag-pill { font-size: 10px; padding: 2px 8px; background: rgb(241, 243, 245); border-radius: 4px; color: rgb(108, 117, 125); }
  .tag-pill.active { background: rgb(232, 245, 233); color: var(--cg-green); font-weight: bold; }

  @media (max-width: 600px) {
    .tag-card { height: 130px !important; }
    .tag-card-img { width: 120px !important; }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" class="back-btn">← 返回画廊主页</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:50px; color:#999;">正在重新生成画廊...</div>
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

            // 1. 尝试从 Frontmatter 获取
            if (item.cover && item.cover.length > 5) {
                imgUrl = item.cover;
            } 
            // 2. 尝试从 HTML 内容中抓取 src="..."
            else if (item.content) {
                // 现在的 content 是 HTML，所以我们要找 img 标签的 src
                const htmlMatch = item.content.match(/src=["'](.*?)["']/);
                if (htmlMatch) {
                    imgUrl = htmlMatch[1];
                } else {
                    // 如果还是没有，尝试找纯文本链接（兜底）
                    const textMatch = item.content.match(/https?:\/\/[^\s"'<>]+\.(?:jpg|jpeg|png|gif|webp)/i);
                    if (textMatch) imgUrl = textMatch[0];
                }
            }

            let finalImg = "";
            if (imgUrl) {
                // 解码并清理
                let cleanUrl = decodeURIComponent(imgUrl).split(/[">)]/)[0].trim();
                
                // 处理 Twitter 链接
                if (cleanUrl.includes('twimg.com')) {
                    // 去掉所有参数，只留纯净路径
                    let parts = cleanUrl.split('?')[0].split(':');
                    let baseUrl = parts[0] + (parts[1] && parts[1].length > 5 ? ':' + parts[1] : '');
                    
                    // Weserv 小图参数: w=300, h=300, fit=cover (正方形裁切)
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(baseUrl)}&w=300&h=300&fit=cover&il`;
                } else {
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=300&h=300&fit=cover`;
                }
            } else {
                finalImg = "https://via.placeholder.com/300x300?text=No+Img";
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
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/300x300?text=Error';">
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
        console.error(e);
        container.innerHTML = "同步异常，请刷新重试";
    }
});
</script>