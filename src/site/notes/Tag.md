---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 使用 RGB 避开插件的 # 标签转换逻辑，并增加强制力 */
  :root {
    --cg-green: rgb(76, 175, 80) !important;
    --cg-text: rgb(45, 55, 72) !important;
    --cg-border: rgba(0, 0, 0, 0.1) !important;
  }

  /* 隐藏所有多余的系统元素 */
  h1[data-note-icon], .header-meta, .header-tags { display: none !important; }
  
  #cgfan-tag-page { max-width: 850px; margin: 0 auto; padding: 40px 20px; font-family: sans-serif; }

  /* 头部设计 */
  .tag-header { text-align: center; margin-bottom: 50px; }
  .selection-badge { 
    display: inline-block; padding: 4px 14px; background: rgb(232, 245, 233) !important; 
    color: var(--cg-green) !important; border-radius: 30px; font-size: 11px; font-weight: 800; letter-spacing: 2px;
  }
  #tag-title { font-size: 3rem !important; font-weight: 900 !important; color: var(--cg-text) !important; margin: 15px 0 !important; }
  #tag-title span { color: var(--cg-green) !important; }

  /* 核心：立体细边框卡片 */
  .tag-card { 
    display: flex !important; 
    background: rgb(255, 255, 255) !important; 
    border: 1px solid var(--cg-border) !important; 
    border-radius: 18px !important; 
    overflow: hidden !important; 
    margin-bottom: 25px !important; 
    height: 180px !important; 
    /* 极致立体感阴影 */
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 10px 20px -5px rgba(0, 0, 0, 0.08) !important;
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1) !important;
  }

  .tag-card:hover { 
    transform: translateY(-6px) !important;
    border-color: var(--cg-green) !important;
    box-shadow: 0 20px 30px -10px rgba(76, 175, 80, 0.2) !important;
  }

  /* 左侧图片区 */
  .tag-card-img { width: 260px; flex-shrink: 0; height: 100%; background: rgb(247, 250, 247); border-right: 1px solid var(--cg-border); overflow: hidden; }
  .tag-card-img img { width: 100%; height: 100%; object-fit: cover; display: block; transition: transform 0.5s ease; }
  .tag-card:hover .tag-card-img img { transform: scale(1.08); }

  /* 右侧文字内容 */
  .tag-card-body { flex-grow: 1; padding: 22px 28px; display: flex; flex-direction: column; justify-content: space-between; min-width: 0; }
  .tag-card-title { font-size: 1.2rem !important; font-weight: 800 !important; color: var(--cg-text) !important; text-decoration: none !important; line-height: 1.4 !important; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
  
  .tag-card-meta { font-size: 12px; color: rgb(113, 128, 150); display: flex; gap: 15px; }
  .tag-pill { font-size: 11px; padding: 2px 8px; background: rgb(241, 245, 241); border-radius: 4px; color: rgb(74, 85, 104); }
  .tag-pill.active { background: rgb(232, 245, 233); color: var(--cg-green); font-weight: bold; border: 1px solid rgba(76, 175, 80, 0.2); }

  @media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; }
    .tag-card-img { width: 100% !important; height: 180px !important; border-right: none; border-bottom: 1px solid var(--cg-border); }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" style="color:var(--cg-green); text-decoration:none; font-weight:bold; font-size:14px;">← 返回全量画廊</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:80px; color:rgb(153,153,153);">正在深度扫描灵感库...</div>
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
            // --- 终极图片提取引擎 ---
            let imgUrl = item.cover || "";
            
            if (!imgUrl && item.content) {
                // 暴力嗅探：匹配 http 且以图片后缀结尾的连续字符
                // 增加对 Twitter 各种混乱格式的兼容
                const sniffRegex = /(https?:\/\/[^\s"'<>]+?\.(?:jpg|jpeg|png|gif|webp|large|orig))/gi;
                const matches = item.content.match(sniffRegex);
                if (matches) imgUrl = matches[0];
            }

            let finalImg = "";
            if (imgUrl) {
                // 清理插件可能附带的 HTML 字符
                let cleanUrl = imgUrl.split(/[">]/)[0].trim();
                
                if (cleanUrl.includes('twimg.com')) {
                    // 彻底去除 Twitter 原有的参数，强制 Weserv 处理高清大图
                    let parts = cleanUrl.split('?')[0].split(':');
                    let baseUrl = parts[0] + (parts[1] ? ':' + parts[1] : '');
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(baseUrl)}&default=https://via.placeholder.com/600x400?text=CGFAN`;
                } else {
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}`;
                }
            } else {
                finalImg = "https://via.placeholder.com/600x400?text=CGFAN+Collection";
            }

            let dateStr = '-';
            if (item.date) {
                const d = new Date(item.date);
                if (!isNaN(d)) dateStr = d.toISOString().split('T')[0];
            }

            const tagsHtml = item.tags ? item.tags.map(t => {
                if (t === 'gardenEntry' || t === 'note') return '';
                const active = t.toLowerCase() === normalizedTarget ? 'active' : '';
                return `<span class="tag-pill ${active}">#${t}</span>`;
            }).join('') : '';

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/600x400?text=Wait+for+Load...'">
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
        container.innerHTML = "数据同步异常，请刷新重试";
    }
});
</script>