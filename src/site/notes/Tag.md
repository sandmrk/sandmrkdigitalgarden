---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  :root {
    --cg-green: rgb(76, 175, 80) !important;
    --cg-text: rgb(45, 55, 72) !important;
    --cg-border: rgba(0, 0, 0, 0.1) !important;
  }

  h1[data-note-icon], .header-meta, .header-tags { display: none !important; }
  #cgfan-tag-page { max-width: 850px; margin: 0 auto; padding: 40px 20px; font-family: sans-serif; }

  .tag-header { text-align: center; margin-bottom: 50px; }
  .selection-badge { 
    display: inline-block; padding: 4px 14px; background: rgb(232, 245, 233) !important; 
    color: var(--cg-green) !important; border-radius: 30px; font-size: 11px; font-weight: 800; letter-spacing: 2px;
  }
  #tag-title { font-size: 3rem !important; font-weight: 900 !important; color: var(--cg-text) !important; margin: 15px 0 !important; }
  #tag-title span { color: var(--cg-green) !important; }

  /* 增强立体感细边框卡片 */
  .tag-card { 
    display: flex !important; background: rgb(255, 255, 255) !important; 
    border: 1px solid var(--cg-border) !important; 
    border-radius: 18px !important; overflow: hidden !important; 
    margin-bottom: 25px !important; height: 180px !important; 
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 10px 20px -5px rgba(0, 0, 0, 0.08) !important;
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1) !important;
  }
  .tag-card:hover { 
    transform: translateY(-6px) !important;
    border-color: var(--cg-green) !important;
    box-shadow: 0 20px 30px -10px rgba(76, 175, 80, 0.2) !important;
  }

  .tag-card-img { width: 260px; flex-shrink: 0; height: 100%; background: rgb(247, 250, 247); border-right: 1px solid var(--cg-border); overflow: hidden; }
  .tag-card-img img { width: 100%; height: 100%; object-fit: cover; display: block; }

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
            // --- 【核心重写】反干扰提取逻辑 ---
            let imgUrl = "";

            // 1. 如果有 cover 字段，先处理 cover
            if (item.cover && item.cover.trim() !== "") {
                imgUrl = item.cover;
            } 
            // 2. 如果没有 cover，从内容中强制嗅探
            else if (item.content) {
                // 解决 Taggify 干扰：匹配所有包含 http 且以图片后缀结尾的连续文本，即使它在 <a> 标签里
                // 这里用一个更宽松的正则：抓取所有符合 URL 格式且含图片后缀的部分
                const matches = item.content.match(/https?:\/\/[^\s"'<>]+?\.(?:jpg|jpeg|png|gif|webp|large|orig)/gi);
                if (matches && matches.length > 0) {
                    imgUrl = matches[0];
                }
            }

            let finalImg = "";
            if (imgUrl) {
                // 剔除可能存在的 HTML 闭合标签干扰（如 ">）
                let cleanUrl = imgUrl.replace(/[">].*$/, '').trim();
                
                // 处理 Twitter 链接
                if (cleanUrl.includes('twimg.com')) {
                    // 彻底砍掉后缀参数，Weserv 对纯净 URL 容错率最高
                    let baseUrl = cleanUrl.split('?')[0].split(':')[0] + ':' + cleanUrl.split('?')[0].split(':')[1];
                    // 拼接高清参数
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(baseUrl)}&default=https://via.placeholder.com/600x400?text=Wait+for+Load...`;
                } else {
                    finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}`;
                }
            } else {
                finalImg = "https://via.placeholder.com/600x400?text=CGFAN+No+Image";
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
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/600x400?text=Image+404'">
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
        console.error("Tag Page Error:", e);
        container.innerHTML = "数据同步异常，请刷新重试";
    }
});
</script>