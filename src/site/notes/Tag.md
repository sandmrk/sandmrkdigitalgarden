---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
h1[data-note-icon], .header-meta { display: none !important; }
</style>

<div id="cgfan-tag-container">
    <div style="text-align: center; padding: 40px 0 60px;">
        <div style="background:#e8f5e9; color:#4caf50; display:inline-block; padding:4px 12px; border-radius:20px; font-size:12px; font-weight:800; letter-spacing:1px; margin-bottom:15px;">SELECTION</div>
        <h1 id="tag-title-display" style="margin:0 !important; font-size:2.5rem; color:#2d3748;">#<span style="color:#4caf50;">...</span></h1>
        <p style="color:#718096; margin-top:10px;">CGFAN 灵感收藏库</p>
        <a href="/" style="display:inline-block; margin-top:20px; color:#4caf50; text-decoration:none; font-weight:bold; border-bottom:1px solid #4caf50;">← 返回画廊主页</a>
    </div>

    <div id="tag-result-list" style="max-width:800px; margin:0 auto; min-height:300px;">
        <div style="text-align:center; padding:50px; color:#999;">正在检索库中数据...</div>
    </div>
</div>

```dataviewjs
// 获取 URL 参数
const params = new URLSearchParams(window.location.search);
const targetTag = params.get('tag');

// 获取容器
const container = document.getElementById('tag-result-list');
const titleSpan = document.querySelector('#tag-title-display span');

if (!targetTag) {
    titleSpan.innerText = "全部";
    container.innerHTML = `<div style="text-align:center; padding:50px;">请从首页点击标签进入。</div>`;
} else {
    // 更新标题
    titleSpan.innerText = targetTag;
    
    // === 核心：直接查询 Dataview 数据库 ===
    // 这里不需要 fetch，直接查 dv.pages
    const allPages = dv.pages('""').filter(p => p["dg-publish"] === true && p.cover).sort(p => p.created, 'desc');
    
    // 模糊匹配标签 (忽略大小写，忽略 # 前缀)
    const normalizedTarget = targetTag.replace('#', '').toLowerCase();
    const filtered = allPages.filter(p => {
        if (!p.tags) return false;
        // 处理 Proxy 数组
        const tags = Array.from(p.tags); 
        return tags.some(t => t.replace('#', '').toLowerCase() === normalizedTarget);
    });

    if (filtered.length === 0) {
        container.innerHTML = `<div style="text-align:center; padding:80px; border:2px dashed #eee; border-radius:20px; color:#999;">
            尚未收录 #${targetTag} 相关的灵感。<br><br>
            <a href="/" style="background:#4caf50; color:white; padding:8px 20px; border-radius:8px; text-decoration:none;">去看看别的</a>
        </div>`;
    } else {
        // 构建 HTML
        let htmlContent = "";
        for (let p of filtered) {
            // 图片反代处理
            let imgUrl = p.cover;
            if (imgUrl.includes('twimg.com')) {
                imgUrl = `https://images.weserv.nl/?url=${encodeURIComponent(p.cover)}&w=400`;
            }
            
            // 路径处理
            const slug = p.file.path.split('/').pop().replace('.md', '');
            const link = `/notes/${slug}/`;

            htmlContent += `
            <div class="tag-card" style="display:flex; margin-bottom:24px; background:#fff; border:1px solid rgba(0,0,0,0.05); border-radius:16px; overflow:hidden; box-shadow:0 4px 15px rgba(0,0,0,0.02); transition:transform 0.2s;">
                <div style="width:140px; height:140px; flex-shrink:0; background:#f0f0f0;">
                    <img src="${imgUrl}" style="width:100%; height:100%; object-fit:cover;" loading="lazy">
                </div>
                <div style="padding:20px; flex-grow:1; display:flex; flex-direction:column; justify-content:center;">
                    <a href="${link}" style="font-size:1.1rem; font-weight:bold; color:#2d3748; text-decoration:none; margin-bottom:8px; display:block;">${p.title || p.file.name}</a>
                    <div style="font-size:12px; color:#718096; display:flex; gap:15px;">
                        <span>👤 ${p.author || '未知'}</span>
                        <span>📅 ${p.created ? dv.date(p.created).toFormat("yyyy-MM-dd") : '-'}</span>
                    </div>
                </div>
            </div>`;
        }
        
        container.innerHTML = htmlContent;
        
        // 渲染后微调样式 (JS 动态注入)
        const style = document.createElement('style');
        style.innerHTML = `
            .tag-card:hover { transform: translateY(-3px); border-color: #4caf50 !important; }
            @media (max-width: 600px) {
                .tag-card { flex-direction: column !important; height: auto !important; }
                .tag-card > div:first-child { width: 100% !important; height: 180px !important; }
            }
        `;
        document.head.appendChild(style);
    }
}