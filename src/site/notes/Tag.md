---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
h1[data-note-icon], .header-meta { display: none !important; }
</style>

<div id="cgfan-tag-page">
    <div style="text-align: center; padding: 40px 0 50px;">
        <div class="selection-badge">SELECTION</div>
        <h1 id="tag-title" style="margin:10px 0 !important; font-size:2.5rem; color:#2d3748;">#<span>...</span></h1>
        <p style="color:#718096; margin-top:8px; font-size:0.95rem;">CGFAN 灵感索引库</p>
        <a href="/" class="back-link">← 返回画廊主页</a>
    </div>

    <div id="tag-results">
        <div class="loading-state">
            <div class="spinner"></div>
            <p>正在读取全站索引...</p>
        </div>
    </div>
</div>

<style>
/* 基础变量 */
:root {
    --cg-green: #4CAF50;
    --cg-green-dark: #388E3C;
    --cg-bg-card: #ffffff;
    --cg-text-main: #2d3748;
    --cg-text-sub: #718096;
}

/* 顶部样式 */
.selection-badge {
    display: inline-block; padding: 4px 12px; background: #e8f5e9;
    color: var(--cg-green); border-radius: 20px; font-size: 12px; font-weight: 800; letter-spacing: 1px;
}
.back-link {
    display: inline-block; margin-top: 20px; color: var(--cg-green); 
    text-decoration: none; font-weight: bold; border-bottom: 2px solid transparent;
}
.back-link:hover { border-bottom-color: var(--cg-green); }

/* 卡片容器 */
#tag-results { max-width: 800px; margin: 0 auto; padding-bottom: 100px; min-height: 400px; }

/* 卡片本体 - 左图右文 */
.tag-card {
    display: flex;
    background: var(--cg-bg-card);
    border: 1px solid rgba(0,0,0,0.06);
    border-radius: 16px;
    overflow: hidden;
    margin-bottom: 24px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.02);
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    height: 200px; /* 固定高度确保整齐 */
}

.tag-card:hover {
    transform: translateY(-4px) translateX(4px);
    border-color: var(--cg-green);
    box-shadow: 0 12px 30px rgba(76, 175, 80, 0.1);
}

/* 左侧图片区 */
.tag-card-img {
    width: 280px; /* 宽图片模式 */
    height: 100%;
    flex-shrink: 0;
    background: #f7fafc;
    position: relative;
    overflow: hidden;
}
.tag-card-img img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 0.5s ease;
}
.tag-card:hover .tag-card-img img { transform: scale(1.05); }

/* 右侧内容区 */
.tag-card-body {
    padding: 24px;
    flex-grow: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between; /* 上下撑开 */
    min-width: 0; /* 防止 flex 子元素溢出 */
}

/* 标题 */
.tag-card-title {
    font-size: 1.25rem;
    font-weight: 700;
    color: var(--cg-text-main);
    text-decoration: none;
    line-height: 1.4;
    display: -webkit-box;
    -webkit-line-clamp: 2; /* 最多显示2行 */
    -webkit-box-orient: vertical;
    overflow: hidden;
}
.tag-card-title:hover { color: var(--cg-green); }

/* 作者与日期 */
.tag-card-meta {
    font-size: 0.85rem;
    color: var(--cg-text-sub);
    margin-top: 8px;
    display: flex;
    align-items: center;
    gap: 15px;
}
.tag-card-meta span { display: flex; align-items: center; gap: 4px; }

/* 标签组 */
.tag-card-tags {
    margin-top: 15px;
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    height: 28px; /* 限制高度，防止太多 */
    overflow: hidden;
}
.tag-pill {
    font-size: 12px;
    padding: 2px 8px;
    background: #f0f2f5;
    color: #666;
    border-radius: 4px;
    text-decoration: none;
    transition: all 0.2s;
}
.tag-pill:hover { background: #e2e8f0; }

/* 高亮标签样式 */
.tag-pill.active {
    background: #e8f5e9;
    color: var(--cg-green-dark);
    font-weight: bold;
}

/* 加载动画 */
.loading-state { text-align: center; padding: 60px; color: #999; }
.spinner {
    width: 30px; height: 30px; border: 3px solid #eee;
    border-top: 3px solid var(--cg-green); border-radius: 50%;
    margin: 0 auto 15px; animation: spin 1s linear infinite;
}
@keyframes spin { 100% { transform: rotate(360deg); } }

/* 移动端适配 */
@media (max-width: 650px) {
    .tag-card { flex-direction: column; height: auto; }
    .tag-card-img { width: 100%; height: 180px; }
    .tag-card-body { padding: 16px; }
    .tag-card-tags { height: auto; } /* 移动端展示所有标签 */
}
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
        // 请求全站索引 (加时间戳防缓存)
        const response = await fetch('/searchIndex.json?v=' + Date.now());
        if (!response.ok) throw new Error("Index load failed");
        
        const allData = await response.json();
        
        // 筛选数据
        const filtered = allData.filter(item => {
            if (!item.tags) return false;
            // 兼容性处理
            const tags = Array.isArray(item.tags) ? item.tags : [item.tags];
            return tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        if (filtered.length === 0) {
            container.innerHTML = `<div class="loading-state" style="border:2px dashed #eee; border-radius:16px;">
                尚未收录 #${targetTag} 相关内容<br>
                <a href="/" class="back-link">返回首页</a>
            </div>`;
            return;
        }

        // 生成 HTML
        let html = "";
        filtered.forEach(item => {
            // 1. 图片处理逻辑：优先用 cover 字段，没有则正则抓取内容
            let imgUrl = item.cover;
            if (!imgUrl && item.content) {
                const imgMatch = item.content.match(/src=["'](.*?)["']/) || item.content.match(/!\[.*?\]\((.*?)\)/);
                if (imgMatch) imgUrl = imgMatch[1];
            }
            if (!imgUrl) imgUrl = "https://via.placeholder.com/400x300?text=CGFAN"; // 默认图

            // Weserv 反代
            if (imgUrl.includes('twimg.com')) {
                imgUrl = `https://images.weserv.nl/?url=${encodeURIComponent(imgUrl)}&w=600`;
            }

            // 2. 标签处理逻辑：生成 Pills 并高亮当前标签
            let tagsHtml = "";
            if (item.tags && Array.isArray(item.tags)) {
                tagsHtml = item.tags.map(t => {
                    if (t === 'gardenEntry' || t === 'note') return ''; // 过滤系统标签
                    const isActive = t.toLowerCase() === normalizedTarget ? 'active' : '';
                    return `<span class="tag-pill ${isActive}">#${t}</span>`;
                }).join('');
            }

            // 3. 链接处理
            const link = item.url || item.path; 

            html += `
            <div class="tag-card">
                <a href="${link}" class="tag-card-img">
                    <img src="${imgUrl}" loading="lazy" alt="${item.title}">
                </a>
                <div class="tag-card-body">
                    <div>
                        <a href="${link}" class="tag-card-title">${item.title}</a>
                        <div class="tag-card-meta">
                            <span>👤 ${item.author || 'CGFan'}</span>
                            <span>📅 ${item.date || '-'}</span>
                        </div>
                    </div>
                    <div class="tag-card-tags">
                        ${tagsHtml}
                    </div>
                </div>
            </div>`;
        });

        container.innerHTML = html;

    } catch (e) {
        console.error(e);
        container.innerHTML = `<div class="loading-state" style="color:#ef5350;">索引加载异常，请刷新重试。</div>`;
    }
});
</script>