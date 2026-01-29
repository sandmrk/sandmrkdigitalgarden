---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
  /* 1. 样式隔离层：全部使用 RGB 和 !important 防止插件干扰 */
  :root {
    --cg-green: rgb(76, 175, 80) !important;
    --cg-text: rgb(45, 55, 72) !important;
    --cg-text-light: rgb(113, 128, 150) !important;
    --cg-border: rgba(0, 0, 0, 0.08) !important;
    --cg-bg: rgb(252, 253, 252) !important;
  }

  /* 隐藏系统默认元素 */
  h1[data-note-icon], .header-meta, .header-tags, .footer { display: none !important; }
  
  #cgfan-tag-page { 
    max-width: 850px; margin: 0 auto; padding: 40px 20px; 
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; 
  }

  /* 头部区域 */
  .tag-header { text-align: center; margin-bottom: 50px; }
  .selection-badge { 
    display: inline-block; padding: 5px 16px; background: rgb(232, 245, 233) !important; 
    color: var(--cg-green) !important; border-radius: 30px; font-size: 11px; font-weight: 800; letter-spacing: 1px;
  }
  #tag-title { font-size: 3rem !important; font-weight: 900 !important; color: var(--cg-text) !important; margin: 15px 0 !important; }
  #tag-title span { color: var(--cg-green) !important; }
  
  /* 返回链接按钮化 */
  .back-btn {
    display: inline-block; margin-top: 15px; padding: 8px 20px;
    color: var(--cg-green) !important; background: rgba(76, 175, 80, 0.05);
    border-radius: 20px; text-decoration: none !important; font-weight: bold; font-size: 13px;
    transition: all 0.2s;
  }
  .back-btn:hover { background: rgba(76, 175, 80, 0.15); transform: translateY(-1px); }

  /* 2. 核心卡片样式：细边框 + 立体投影 */
  .tag-card { 
    display: flex !important; 
    background: rgb(255, 255, 255) !important; 
    /* 关键：1px 细边框 */
    border: 1px solid var(--cg-border) !important; 
    border-radius: 20px !important; 
    overflow: hidden !important; 
    margin-bottom: 25px !important; 
    height: 190px !important; 
    /* 关键：双层投影制造悬浮感 */
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.02), 0 10px 15px -3px rgba(0, 0, 0, 0.05) !important;
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1) !important;
    position: relative;
    box-sizing: border-box; /* 防止边框撑大布局 */
  }

  /* 悬停动效 */
  .tag-card:hover { 
    transform: translateY(-6px) !important;
    border-color: rgba(76, 175, 80, 0.5) !important;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04) !important;
  }

  /* 左侧图片容器 */
  .tag-card-img { 
    width: 280px; flex-shrink: 0; height: 100%; 
    background: rgb(247, 250, 247); 
    border-right: 1px solid var(--cg-border); 
    overflow: hidden; 
  }
  .tag-card-img img { 
    width: 100%; height: 100%; object-fit: cover; display: block; 
    transition: transform 0.6s ease;
  }
  .tag-card:hover .tag-card-img img { transform: scale(1.05); }

  /* 右侧内容容器 */
  .tag-card-body { 
    flex-grow: 1; padding: 25px 30px; display: flex; 
    flex-direction: column; justify-content: space-between; min-width: 0; 
  }
  
  .tag-card-title { 
    font-size: 1.25rem !important; font-weight: 800 !important; color: var(--cg-text) !important; 
    text-decoration: none !important; line-height: 1.4 !important; 
    display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; 
  }
  
  .tag-card-meta { 
    font-size: 13px; color: var(--cg-text-light); display: flex; gap: 15px; align-items: center; 
  }
  
  /* 标签 Pill 样式 */
  .tag-pill { 
    font-size: 11px; padding: 3px 10px; background: rgb(241, 245, 241); 
    border-radius: 6px; color: rgb(100, 116, 139); margin-right: 5px;
  }
  .tag-pill.active { 
    background: rgb(232, 245, 233); color: var(--cg-green); 
    font-weight: 800; border: 1px solid rgba(76, 175, 80, 0.2); 
  }

  /* 移动端适配 */
  @media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; }
    .tag-card-img { width: 100% !important; height: 200px !important; border-right: none; border-bottom: 1px solid var(--cg-border); }
    #tag-title { font-size: 2.2rem !important; }
  }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <a href="/" class="back-btn">← 返回画廊主页</a>
    </div>
    <div id="tag-results">
        <div style="text-align:center; padding:80px; color:rgb(153,153,153);">
            正在同步灵感数据库...
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
            // --- 提取逻辑 (修正版) ---
            let imgUrl = "";

            // 1. 优先读取 cover
            if (item.cover && item.cover.trim().length > 5) {
                imgUrl = item.cover;
            } 
            // 2. 暴力提取内容中的 URL
            else if (item.content) {
                // 正则：忽略大小写，匹配以图片后缀结尾的 http 链接
                const matches = item.content.match(/https?:\/\/[^\s"'<>]+\.(?:jpg|jpeg|png|gif|webp|large|orig)/gi);
                if (matches && matches.length > 0) {
                    imgUrl = matches[0];
                }
            }

            // --- 链接清洗与反代构建 (修正版) ---
            let finalImg = "";
            if (imgUrl) {
                try {
                    // 解码 + 清理尾部杂质 (移除 > 或 " 或 ) 等闭合符)
                    let cleanUrl = decodeURIComponent(imgUrl).split(/[">)]/)[0].trim();

                    // Twitter 链接特异性处理
                    if (cleanUrl.includes('twimg.com')) {
                        // 1. 移除 :large, :orig 等后缀
                        cleanUrl = cleanUrl.replace(/:(large|orig|medium|small|thumb)$/, '');
                        // 2. 移除 ?format=xxx 等参数
                        cleanUrl = cleanUrl.split('?')[0];
                        
                        // 3. 强制指定 Weserv 的参数：&w=600 (宽) &output=webp (快)
                        finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=600&output=webp&default=https://via.placeholder.com/600x400?text=Wait`;
                    } else {
                        // 普通图片
                        finalImg = `https://images.weserv.nl/?url=${encodeURIComponent(cleanUrl)}&w=600&output=webp`;
                    }
                } catch(err) {
                    finalImg = "https://via.placeholder.com/600x400?text=Url+Error";
                }
            } else {
                finalImg = "https://via.placeholder.com/600x400?text=CGFAN";
            }

            // 日期处理
            let dateStr = '-';
            if (item.date) {
                const d = new Date(item.date);
                if (!isNaN(d)) dateStr = d.toISOString().split('T')[0];
            }

            // 标签生成
            const tagsHtml = item.tags ? item.tags.map(t => {
                if (t === 'gardenEntry' || t === 'note') return '';
                const active = t.toLowerCase() === normalizedTarget ? 'active' : '';
                return `<span class="tag-pill ${active}">#${t}</span>`;
            }).join('') : '';

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${finalImg}" loading="lazy" onerror="this.src='https://via.placeholder.com/600x400?text=Load+Fail'">
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

        container.innerHTML = filtered.length > 0 ? html : "<div style='text-align:center;padding:50px;color:#999'>未发现相关内容</div>";

    } catch (e) {
        console.error(e);
        container.innerHTML = "加载异常，请刷新重试";
    }
});
</script>