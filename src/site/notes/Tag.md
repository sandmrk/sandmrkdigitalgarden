---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 1. 基础视觉增强 */
  :root {
    --tag-primary: #4CAF50;
    --tag-bg-main: #fcfdfc;
    --tag-border-color: rgba(0, 0, 0, 0.08); /* 细边框颜色 */
    --tag-text-dark: #1a202c;
    --tag-text-gray: #4a5568;
  }

  h1[data-note-icon], .header-meta { display: none !important; }
  #cgfan-tag-page { max-width: 850px; margin: 0 auto; padding: 40px 20px; }

  /* 2. 头部美化 */
  .tag-header { text-align: center; margin-bottom: 60px; }
  .selection-badge { 
    display: inline-block; padding: 5px 16px; background: #e8f5e9; 
    color: var(--tag-primary); border-radius: 30px; font-size: 11px; 
    font-weight: 800; letter-spacing: 2px; margin-bottom: 15px;
  }
  #tag-title { font-size: 3rem !important; font-weight: 900 !important; color: var(--tag-text-dark) !important; margin: 0 !important; }
  #tag-title span { color: var(--tag-primary); }
  .back-link { color: var(--tag-primary); text-decoration: none; font-weight: 700; border-bottom: 2px solid #e8f5e9; transition: all 0.3s; }
  .back-link:hover { border-bottom-color: var(--tag-primary); }

  /* 3. 精致细边框卡片 */
  .tag-card { 
    display: flex !important; background: #ffffff; 
    /* 核心修改：增加 1px 细边框 */
    border: 1.2px solid var(--tag-border-color); 
    border-radius: 20px; 
    overflow: hidden; margin-bottom: 30px; height: 190px; 
    box-shadow: 0 10px 25px -5px rgba(0,0,0,0.05);
    transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    position: relative;
  }

  .tag-card:hover { 
    transform: translateY(-6px) scale(1.01);
    /* 悬停时边框变亮变绿 */
    border-color: rgba(76, 175, 80, 0.4);
    box-shadow: 0 20px 35px -10px rgba(76, 175, 80, 0.12);
  }

  /* 图片展示区 */
  .tag-card-img { 
    width: 260px; flex-shrink: 0; height: 100%; background: #f7faf7; 
    border-right: 1.2px solid var(--tag-border-color); /* 图片右侧细分界线 */
    overflow: hidden; 
  }
  .tag-card-img img { width: 100%; height: 100%; object-fit: cover; display: block; transition: transform 0.8s; }
  .tag-card:hover .tag-card-img img { transform: scale(1.1); }

  /* 文字内容区 */
  .tag-card-body { flex-grow: 1; padding: 25px 32px; display: flex; flex-direction: column; justify-content: space-between; min-width: 0; }
  .tag-card-title { font-size: 1.25rem; font-weight: 800; color: var(--tag-text-dark); text-decoration: none; line-height: 1.5; margin-bottom: 8px; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
  
  .tag-card-meta { font-size: 13px; color: var(--tag-text-gray); display: flex; gap: 15px; align-items: center; }
  .tag-pill { font-size: 11px; padding: 3px 10px; background: #f1f5f1; color: #667085; border-radius: 6px; border: 1px solid transparent; }
  .tag-pill.active { background: #e8f5e9; color: var(--tag-primary); border-color: #c8e6c9; font-weight: 800; }

  @media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; }
    .tag-card-img { width: 100% !important; height: 200px !important; border-right: none; border-bottom: 1.2px solid var(--tag-border-color); }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" class="back-link">← 返回画廊主页</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:80px; color:#999;">正在同步数据库...</div>
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
            // --- 核心修复：图片链接提取逻辑 ---
            let imgUrl = item.cover;
            
            // 如果没有 cover，从内容中提取
            if (!imgUrl && item.content) {
                // 扫描 Markdown 图片语法
                const mdMatch = item.content.match(/!\[.*?\]\((.*?)\)/);
                if (mdMatch) {
                    imgUrl = mdMatch[1];
                } else {
                    // 扫描普通的 http 图片链接
                    const urlMatch = item.content.match(/https?:\/\/[^\s]+?\.(jpg|jpeg|png|gif|webp)/i);
                    if (urlMatch) imgUrl = urlMatch[0];
                }
            }

            // --- 关键：强制转为指定的 Weserv 编码格式 ---
            let finalImg = "";
            if (imgUrl) {
                // 清理链接中的特殊字符，确保只有纯净的 URL
                let cleanUrl = imgUrl.split('?')[0].split(')')[0].trim();
                // 针对 twimg 特殊处理，拼接 :large 或高质量参数
                if (cleanUrl.includes('twimg.com')) {
                    finalImg = "https://images.weserv.nl/?url=" + encodeURIComponent(cleanUrl) + "%3Alarge";
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