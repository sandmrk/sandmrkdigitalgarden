---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<style>
h1[data-note-icon], .header-meta { display: none !important; }
</style>

<div id="cgfan-tag-page">
    <div style="text-align: center; padding: 40px 0 60px;">
        <div style="background:#e8f5e9; color:#4caf50; display:inline-block; padding:4px 12px; border-radius:20px; font-size:12px; font-weight:800; letter-spacing:1px; margin-bottom:15px;">SELECTION</div>
        <h1 id="tag-title" style="margin:0 !important; font-size:2.5rem; color:#2d3748;">#<span>...</span></h1>
        <p style="color:#718096; margin-top:10px;">CGFAN 灵感索引库</p>
        <a href="/" style="display:inline-block; margin-top:20px; color:#4caf50; text-decoration:none; font-weight:bold; border-bottom:1px solid #4caf50;">← 返回画廊主页</a>
    </div>

    <div id="tag-results" style="max-width:800px; margin:0 auto; min-height:400px; padding-bottom:100px;">
        <div style="text-align:center; padding:50px; color:#999;">
            <div class="spinner"></div>
            <p>正在读取全站索引...</p>
        </div>
    </div>
</div>

<style>
.spinner {
    width: 30px; height: 30px; border: 3px solid #eee;
    border-top: 3px solid #4caf50; border-radius: 50%;
    margin: 0 auto 15px; animation: spin 1s linear infinite;
}
@keyframes spin { 100% { transform: rotate(360deg); } }

.tag-card {
    display: flex; background: #fff; border: 1px solid rgba(0,0,0,0.05);
    border-radius: 16px; overflow: hidden; margin-bottom: 24px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.02); transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}
.tag-card:hover {
    transform: translateY(-4px); border-color: #4caf50;
    box-shadow: 0 12px 25px rgba(76, 175, 80, 0.1);
}
.tag-card-img { width: 140px; height: 140px; flex-shrink: 0; background: #f9f9f9; }
.tag-card-img img { width: 100%; height: 100%; object-fit: cover; }
.tag-card-body { padding: 20px; flex-grow: 1; display: flex; flex-direction: column; justify-content: center; }
.tag-card-title { font-size: 1.15rem; font-weight: 700; color: #2d3748; text-decoration: none; margin-bottom: 8px; line-height: 1.4; }
.tag-card-meta { font-size: 12px; color: #718096; }

@media (max-width: 600px) {
    .tag-card { flex-direction: column; height: auto; }
    .tag-card-img { width: 100%; height: 180px; }
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
        container.innerHTML = `<div style="text-align:center; padding:50px;">请从首页点击标签进入。</div>`;
        return;
    }

    titleSpan.innerText = targetTag;

    try {
        // [核心] 请求插件生成的全站索引
        // 加个时间戳防止缓存 old version
        const response = await fetch('/searchIndex.json?v=' + Date.now());
        if (!response.ok) throw new Error("Index file not found");
        
        const allData = await response.json();
        
        // 筛选数据
        // Digital Garden 的 index 结构通常包含: title, tags, content, url
        const normalizedTarget = targetTag.replace('#', '').toLowerCase();
        
        const filtered = allData.filter(item => {
            if (!item.tags) return false;
            // 兼容 tags 是数组或字符串的情况
            const tags = Array.isArray(item.tags) ? item.tags : [item.tags];
            return tags.some(t => t.toLowerCase() === normalizedTarget);
        });

        if (filtered.length === 0) {
            container.innerHTML = `<div style="text-align:center; padding:80px; color:#999; border:2px dashed #eee; border-radius:20px;">
                索引中未找到 #${targetTag} 相关条目。<br>
                <span style="font-size:12px; opacity:0.7;">(请确认相关笔记已添加标签并 Public)</span>
            </div>`;
            return;
        }

        // 渲染列表
        let htmlContent = "";
        filtered.forEach(item => {
            // [难点突破] 从 content 文本中提取第一张图片作为封面
            // 匹配 markdown 图片 ![...](url) 或 html 图片 <img src="...">
            let imgUrl = "https://via.placeholder.com/150x150?text=No+Image"; // 默认图
            
            if (item.content) {
                const mdMatch = item.content.match(/!\[.*?\]\((.*?)\)/);
                const htmlMatch = item.content.match(/<img.*?src=["'](.*?)["']/);
                
                if (mdMatch && mdMatch[1]) imgUrl = mdMatch[1];
                else if (htmlMatch && htmlMatch[1]) imgUrl = htmlMatch[1];
            }

            // 处理 Weserv 反代
            if (imgUrl.includes('twimg.com')) {
                imgUrl = `https://images.weserv.nl/?url=${encodeURIComponent(imgUrl)}&w=400`;
            }

            // 处理 URL (searchIndex 里的 url 通常是 /notes/xxx/ 或 /xxx)
            const link = item.url || item.path; 

            htmlContent += `
            <div class="tag-card">
                <div class="tag-card-img">
                    <img src="${imgUrl}" loading="lazy" onerror="this.src='https://via.placeholder.com/150?text=CGFAN'">
                </div>
                <div class="tag-card-body">
                    <a href="${link}" class="tag-card-title">${item.title}</a>
                    <div class="tag-card-meta">
                        <span>#${targetTag}</span>
                    </div>
                </div>
            </div>`;
        });

        container.innerHTML = htmlContent;

    } catch (e) {
        console.error("Search Error:", e);
        container.innerHTML = `<div style="text-align:center; padding:50px; color:#ef5350;">
            索引加载失败，请检查网络或 searchIndex.json 是否生成。
        </div>`;
    }
});
</script>