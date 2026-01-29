---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 使用 RGB 格式避开插件的自动标签转换 */
  :root {
    --tag-primary: rgb(76, 175, 80); 
    --tag-bg-main: rgb(252, 253, 252);
    --tag-border-color: rgba(0, 0, 0, 0.08); 
    --tag-text-dark: rgb(26, 32, 44);
    --tag-text-gray: rgb(74, 85, 104);
  }

  h1[data-note-icon], .header-meta { display: none !important; }
  
  #cgfan-tag-page { max-width: 850px; margin: 0 auto; padding: 40px 20px; }

  .tag-header { text-align: center; margin-bottom: 60px; }
  .selection-badge { 
    display: inline-block; padding: 5px 16px; background: rgb(232, 245, 233); 
    color: var(--tag-primary); border-radius: 30px; font-size: 11px; 
    font-weight: 800; letter-spacing: 2px; margin-bottom: 15px;
  }
  #tag-title { font-size: 3rem !important; font-weight: 900 !important; color: var(--tag-text-dark) !important; margin: 0 !important; }
  #tag-title span { color: var(--tag-primary); }
  .back-link { color: var(--tag-primary); text-decoration: none; font-weight: 700; border-bottom: 2px solid rgb(232, 245, 233); transition: all 0.3s; }

  /* 增强立体感细边框卡片 */
  .tag-card { 
    display: flex !important; background: rgb(255, 255, 255); 
    border: 1px solid var(--tag-border-color); 
    border-radius: 20px; 
    overflow: hidden; margin-bottom: 30px; height: 190px; 
    box-shadow: 0 10px 25px -5px rgba(0,0,0,0.05), 0 5px 10px rgba(0,0,0,0.03);
    transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    position: relative;
  }

  .tag-card:hover { 
    transform: translateY(-8px) scale(1.01);
    border-color: rgba(76, 175, 80, 0.5);
    box-shadow: 0 20px 40px -10px rgba(76, 175, 80, 0.15);
  }

  .tag-card-img { 
    width: 260px; flex-shrink: 0; height: 100%; background: rgb(247, 250, 247); 
    border-right: 1px solid var(--tag-border-color); 
    overflow: hidden; 
  }
  .tag-card-img img { width: 100%; height: 100%; object-fit: cover; display: block; transition: transform 0.8s; }
  .tag-card:hover .tag-card-img img { transform: scale(1.1); }

  .tag-card-body { flex-grow: 1; padding: 25px 32px; display: flex; flex-direction: column; justify-content: space-between; min-width: 0; }
  .tag-card-title { font-size: 1.25rem; font-weight: 800; color: var(--tag-text-dark); text-decoration: none; line-height: 1.5; margin-bottom: 8px; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
  
  .tag-pill { font-size: 11px; padding: 3px 10px; background: rgb(241, 245, 241); color: rgb(102, 112, 133); border-radius: 6px; }
  .tag-pill.active { background: rgb(232, 245, 233); color: var(--tag-primary); font-weight: 800; border: 1px solid rgb(200, 230, 201); }

  @media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; }
    .tag-card-img { width: 100% !important; height: 200px !important; border-right: none; border-bottom: 1px solid var(--tag-border-color); }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" class="back-link">← 返回画廊主页</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:80px; color:rgb(153, 153, 153);">正在同步灵感库...</div>
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
            if (!item.tags) return false;
            return item.tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        let html = "";
        filtered.forEach(item => {
            let imgUrl = item.cover;
            
            // 改进正则：即使内容被 Taggify 破坏了 HTML，也尝试抓取 URL
            if (!imgUrl && item.content) {
                // 1. 尝试匹配已经被插件处理过的链接文本或原始 Markdown
                const urlRegex = /(https?:\/\/[^\s"'<>]+?\.(?:jpg|jpeg|png|gif|webp))/gi;
                const matches = item.content.match(urlRegex);
                if (matches) imgUrl = matches[0];
            }

            let finalImg = "";
            if (imgUrl) {
                // 彻底清理 URL，移除末尾可能的 HTML 干扰
                let cleanUrl = imgUrl.replace(/[">].*$/, '').trim();
                if (cleanUrl.includes('twimg.com')) {
                    // 确保使用高清格式
                    let baseUrl = cleanUrl.split('?')[0];
                    finalImg = "https://images.weserv.nl/?url=" + encodeURIComponent(baseUrl) + "%3Alarge";
                } else {
                    finalImg = "https://images.weserv.nl/?url=" + encodeURIComponent(cleanUrl);
                }
            } else {
                finalImg = "https://via.placeholder.com/600x400?text=CGFAN+Collection";
            }

            let dateStr = '-';
            if (item.date) {
                const d = new Date(item.date);
                if (!isNaN(d)) dateStr = d.toISOString().split('T')[0];
            }

            const tagsHtml = item.tags.map(t => {
                if (t === 'gardenEntry' || t === 'note') return '';
                const active = t.toLowerCase() === normalizedTarget ? 'active' : '';
                return `<span class="tag-pill ${active}">#${t}</span>`;
            }).join('');

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/600x400?text=Image+Load+Error'">
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

        container.innerHTML = filtered.length > 0 ? html : "未发现相关灵感";

    } catch (e) {
        container.innerHTML = "加载异常，请刷新重试";
    }
});
</script>