---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
h1[data-note-icon], .header-meta { display: none !important; }
</style>

<div id="cgfan-tag-page">
    <div class="tag-header">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title">#<span>...</span></h1>
        <p>CGFAN 灵感索引库</p>
        <a href="/" class="back-link">← 返回画廊主页</a>
    </div>

    <div id="tag-results">
        <div class="loading-state">
            <div class="spinner"></div>
            <p>正在从全站索引提取灵感...</p>
        </div>
    </div>
</div>

<style>
/* 容器布局 */
#cgfan-tag-page { max-width: 800px; margin: 0 auto; padding: 20px; }
.tag-header { text-align: center; margin-bottom: 50px; }

/* 顶部元素 */
.selection-badge {
    display: inline-block; padding: 4px 12px; background: #e8f5e9;
    color: #4CAF50; border-radius: 20px; font-size: 12px; font-weight: 800; letter-spacing: 1px;
}
#tag-title { font-size: 2.5rem !important; margin: 10px 0 !important; color: #2d3748; }
#tag-title span { color: #4CAF50; }
.back-link { color: #4CAF50; text-decoration: none; font-weight: bold; border-bottom: 1px solid transparent; }
.back-link:hover { border-bottom-color: #4CAF50; }

/* 卡片样式核心 */
.tag-card {
    display: flex !important;
    background: #fff;
    border: 1px solid rgba(0,0,0,0.06);
    border-radius: 16px;
    overflow: hidden;
    margin-bottom: 24px;
    height: 180px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.02);
    transition: transform 0.2s, box-shadow 0.2s;
}

.tag-card:hover {
    transform: translateY(-4px);
    border-color: #4CAF50;
    box-shadow: 0 12px 25px rgba(76, 175, 80, 0.1);
}

/* 左侧图片区 */
.tag-card-img {
    width: 240px; 
    flex-shrink: 0;
    height: 100%;
    background: #f5f5f5;
    position: relative;
    display: block;
}
.tag-card-img img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
}

/* 右侧内容区 */
.tag-card-body {
    flex-grow: 1;
    padding: 20px 24px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    min-width: 0;
}

/* 标题 */
.tag-card-title {
    font-size: 1.2rem;
    font-weight: 700;
    color: #2d3748;
    text-decoration: none;
    line-height: 1.4;
    margin-bottom: 8px;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}

/* 元数据 */
.tag-card-meta {
    font-size: 13px;
    color: #718096;
    display: flex;
    gap: 15px;
    margin-bottom: 12px;
}

/* 标签云 */
.tag-card-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    height: 24px;
    overflow: hidden;
}
.tag-pill {
    font-size: 11px;
    padding: 2px 8px;
    background: #f7fafc;
    color: #718096;
    border-radius: 4px;
    border: 1px solid #edf2f7;
}
.tag-pill.active {
    background: #e8f5e9;
    color: #388E3C;
    border-color: #c8e6c9;
    font-weight: bold;
}

/* 移动端适配 */
@media (max-width: 650px) {
    .tag-card { flex-direction: column !important; height: auto !important; }
    .tag-card-img { width: 100% !important; height: 180px !important; }
    .tag-card-body { padding: 16px !important; }
    .tag-card-tags { height: auto !important; }
}

/* 加载动画 */
.loading-state { text-align: center; padding: 60px; color: #999; }
.spinner {
    width: 30px; height: 30px; border: 3px solid #eee;
    border-top: 3px solid #4CAF50; border-radius: 50%;
    margin: 0 auto 15px; animation: spin 1s linear infinite;
}
@keyframes spin { 100% { transform: rotate(360deg); } }
</style>

<script>
document.addEventListener('DOMContentLoaded', async () => {
    const params = new URLSearchParams(window.location.search);
    const targetTag = params.get('tag');
    const container = document.getElementById('tag-results');
    const titleSpan = document.querySelector('#tag-title span');

    if (!targetTag) {
        titleSpan.innerText = "全部";
        container.innerHTML = `<div class="loading-state">请从首页点击标签进入。</div>`;
        return;
    }

    titleSpan.innerText = targetTag;
    const normalizedTarget = targetTag.replace('#', '').toLowerCase();

    try {
        const response = await fetch('/searchIndex.json?v=' + Date.now());
        const allData = await response.json();
        
        // 筛选逻辑
        const filtered = allData.filter(item => {
            if (!item.tags) return false;
            const tags = Array.isArray(item.tags) ? item.tags : [item.tags];
            return tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        if (filtered.length === 0) {
            container.innerHTML = `<div class="loading-state" style="border:2px dashed #eee; border-radius:16px;">
                尚未收录 #${targetTag} 相关内容
            </div>`;
            return;
        }

        let html = "";
        filtered.forEach(item => {
            // 图片处理
            let imgUrl = item.cover;
            if (!imgUrl && item.content) {
                const match = item.content.match(/src=["'](.*?)["']/) || item.content.match(/!\[.*?\]\((.*?)\)/);
                if (match) imgUrl = match[1];
            }
            if (!imgUrl) imgUrl = "https://via.placeholder.com/300x200?text=No+Image";

            // Weserv 反代
            if (imgUrl.includes('twimg.com')) {
                imgUrl = `https://images.weserv.nl/?url=${encodeURIComponent(imgUrl)}&w=400`;
            }

            // 日期格式化 (JS端处理)
            let dateStr = '-';
            if (item.date) {
                try {
                    const d = new Date(item.date);
                    dateStr = d.toISOString().split('T')[0]; // 输出 YYYY-MM-DD
                } catch(e) { dateStr = item.date; }
            }

            // 标签高亮
            let tagsHtml = "";
            if (item.tags) {
                tagsHtml = item.tags.map(t => {
                    if (t === 'gardenEntry' || t === 'note') return ''; 
                    const isActive = t.toLowerCase() === normalizedTarget ? 'active' : '';
                    return `<span class="tag-pill ${isActive}">#${t}</span>`;
                }).join('');
            }

            html += `
            <div class="tag-card">
                <a href="${item.url}" class="tag-card-img">
                    <img src="${imgUrl}" loading="lazy" alt="${item.title}">
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

        container.innerHTML = html;

    } catch (e) {
        console.error(e);
        container.innerHTML = `<div class="loading-state" style="color:#ef5350;">索引加载失败，请刷新。</div>`;
    }
});
</script>