---
{"dg-publish":true,"permalink":"/tag/","title":"标签筛选"}
---


<div class="tag-page-header">
  <div class="tag-badge">SELECTION</div>
  <h1 style="margin-top:10px !important;">#<span id="target-tag-name">...</span></h1>
  <p class="tag-subtitle">正在为您聚合相关灵感与提示词</p>
  <a href="/" class="back-link">← 返回画廊主页</a>
</div>

<div class="grid-view vertical-mode" id="tag-results-container">
  <div id="loading-spinner">
    <div class="spinner"></div>
    <p>正在读取灵感缓存...</p>
  </div>
</div>

<style>
// 在 tag.md 的脚本开头加入这一行
console.log("正在尝试读取标签：", targetTag);
console.log("本地存储的所有数据：", localStorage.getItem('cgfan_gallery_data'));
  /* 强制垂直列表布局 */
  .vertical-mode { margin-top: 40px; }
  .vertical-mode table {
    display: flex !important;
    flex-direction: column !important;
    gap: 20px !important;
    max-width: 800px !important;
    margin: 0 auto !important;
    border: none !important;
  }
  .vertical-mode table tbody { display: contents !important; }
  
  .vertical-mode table tr {
    display: flex !important;
    flex-direction: row !important; /* 左图右文 */
    height: 180px !important;
    background: #fff !important;
    border: 1px solid rgba(0,0,0,0.06) !important;
    border-radius: 16px !important;
    overflow: hidden !important;
    margin-bottom: 0 !important;
    transition: transform 0.2s ease !important;
    box-shadow: 0 4px 6px rgba(0,0,0,0.02);
  }
  
  .vertical-mode table tr:hover {
    transform: translateX(8px) !important;
    border-color: #4caf50 !important;
    box-shadow: 0 10px 15px rgba(76, 175, 80, 0.1);
  }

  /* 图片区域 */
  .vertical-mode td:first-child {
    width: 280px !important;
    height: 180px !important;
    flex-shrink: 0;
    padding: 0 !important;
    border: none !important;
  }
  
  .vertical-mode td:first-child img {
    width: 100% !important;
    height: 100% !important;
    object-fit: cover !important;
    display: block !important;
    border-radius: 0 !important;
  }

  /* 文字区域 */
  .vertical-mode td:nth-child(2) {
    padding: 20px 30px !important;
    display: flex !important;
    flex-direction: column !important;
    justify-content: center !important;
    border: none !important;
    flex-grow: 1;
  }

  /* 隐藏多余列 */
  .vertical-mode td:nth-child(3), 
  .vertical-mode td:nth-child(4), 
  .vertical-mode td:last-child {
    display: none !important;
  }

  /* 装饰样式 */
  .tag-page-header { text-align: center; padding-top: 20px; }
  .tag-badge {
    display: inline-block; padding: 4px 12px; background: #e8f5e9;
    color: #4caf50; border-radius: 20px; font-size: 12px; font-weight: 800; letter-spacing: 1px;
  }
  .tag-subtitle { color: #888; font-size: 0.9em; margin-bottom: 20px; }
  .back-link { color: #4caf50; text-decoration: none; font-weight: 600; border-bottom: 2px solid transparent; }
  .back-link:hover { border-bottom-color: #4caf50; }

  .spinner {
    width: 30px; height: 30px; border: 3px solid #e8f5e9;
    border-top: 3px solid #4caf50; border-radius: 50%;
    margin: 0 auto 15px; animation: spin 1s linear infinite;
  }
  @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

  /* 移动端适配 */
  @media (max-width: 600px) {
    .vertical-mode table tr { flex-direction: column !important; height: auto !important; }
    .vertical-mode td:first-child { width: 100% !important; height: 160px !important; }
    .vertical-mode td:nth-child(2) { padding: 20px !important; height: auto !important; }
  }
</style>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // 获取 URL 参数
    const params = new URLSearchParams(window.location.search);
    const targetTag = params.get('tag');
    const container = document.getElementById('tag-results-container');
    const titleEl = document.getElementById('target-tag-name');
    
    // 如果没有标签参数，显示提示
    if (!targetTag) {
        titleEl.innerText = "全部";
        container.innerHTML = "<div style='text-align:center;padding:50px;'>请从首页点击标签进入。</div>";
        return;
    }
    
    titleEl.innerText = targetTag;

    // 1. 读取首页写入的缓存
    const rawData = localStorage.getItem('cgfan_gallery_data');
    
    if (!rawData) {
        // 如果缓存不存在，尝试去 fetch 首页 (回退方案)
        console.warn("缓存未找到，尝试 Fetch...");
        fallbackFetch(targetTag);
        return;
    }

    // 2. 解析数据并渲染
    try {
        const allData = JSON.parse(rawData);
        renderList(allData, targetTag);
    } catch (e) {
        console.error("缓存解析失败", e);
        fallbackFetch(targetTag);
    }

    // 渲染函数
    function renderList(data, tag) {
        // 模糊匹配标签 (忽略大小写)
        const filtered = data.filter(p => {
            if (!p.tags) return false;
            // 处理标签数组或字符串情况
            const tags = Array.isArray(p.tags) ? p.tags : [p.tags];
            return tags.some(t => t.toLowerCase() === tag.toLowerCase());
        });

        if (filtered.length === 0) {
            container.innerHTML = `<div style="text-align:center;padding:80px;color:#999;">
                未找到关于 #${tag} 的内容<br>
                <a href="/" style="color:#4caf50">返回首页刷新缓存</a>
            </div>`;
            return;
        }

        let htmlRows = "";
        filtered.forEach(p => {
            // 图片反代处理
            const imgUrl = p.cover.includes('twimg.com') && !p.cover.includes('weserv')
                ? `https://images.weserv.nl/?url=${encodeURIComponent(p.cover)}&w=400`
                : p.cover;
            
            // 确保标题链接正确 (处理 notes 路径)
            // 假设你的笔记路径结构是 notes/文件名/
            const noteSlug = p.path.split('/').pop().replace('.md', '');
            const link = `/notes/${noteSlug}/`; 

            htmlRows += `
            <tr>
                <td><img src="${imgUrl}" loading="lazy" alt="${p.title}"></td>
                <td>
                    <a href="${link}" style="display:block;font-weight:700;font-size:1.2rem;text-decoration:none;color:#333;margin-bottom:12px;">${p.title}</a>
                    <div style="font-size:0.85rem;color:#666;line-height:1.6;">
                        <span style="margin-right:10px">👤 ${p.author}</span>
                        <span>📅 ${p.created}</span>
                    </div>
                </td>
            </tr>`;
        });

        container.innerHTML = `<table><tbody>${htmlRows}</tbody></table>`;
    }

    // 回退方案：如果缓存没有，去首页抓
    async function fallbackFetch(tag) {
        try {
            const resp = await fetch('/');
            const text = await resp.text();
            // 简单提示用户去首页
            container.innerHTML = `<div style="text-align:center;padding:60px;">
                <p>数据缓存已更新，请<a href="/" style="font-weight:bold;color:#4caf50">点击这里返回首页</a>加载数据。</p>
                <p style="font-size:0.8em;color:#aaa;margin-top:10px;">(系统需要从首页获取一次最新数据)</p>
            </div>`;
        } catch (e) {
            container.innerHTML = "数据加载失败。";
        }
    }
});
</script>