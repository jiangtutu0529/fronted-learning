##### js,css,html加载顺序
HTML下载 → HTML解析 → 构建DOM树
         ↓
        CSS下载 → 构建CSSOM树
         ↓
    JavaScript下载/执行
         ↓
    DOMContentLoaded事件
         ↓
    页面渲染
         ↓
    load事件

    ```
    <!DOCTYPE html>
<html>
<head>
    <!-- 遇到CSS时不会阻塞DOM解析，但会阻塞渲染 -->
    <link rel="stylesheet" href="style.css">
    
    <!-- 遇到普通script会阻塞后续DOM解析 -->
    <script src="script1.js"></script>
</head>
<body>
    <div>内容</div>
    
    <!-- 异步script不会阻塞DOM解析 -->
    <script src="script2.js" async></script>
    
    <!-- 延迟script在DOM解析完成后执行 -->
    <script src="script3.js" defer></script>
</body>
</html>
    ```

##### 时间节点
```
// DOM解析完成（不等待样式表、图片加载）
document.addEventListener('DOMContentLoaded', function() {
    console.log('DOM解析完成');
});

// 页面完全加载（包括所有资源）
window.addEventListener('load', function() {
    console.log('页面完全加载');
});
```

##### 资源阻塞
1. css阻塞
   ```
   <!-- CSS不会阻塞DOM解析，但会阻塞渲染 -->
<link rel="stylesheet" href="style.css">

<!-- 关键CSS内联，避免阻塞 -->
<style>
    /* 关键样式 */
    .critical { display: block; }
</style>
   ```
2. js阻塞
```
<!-- 阻塞后续DOM解析 -->
<script src="blocking.js"></script>

<!-- 异步加载，不阻塞DOM解析 -->
<script src="async.js" async></script>

<!-- 延迟执行，DOM解析完成后执行 -->
<script src="defer.js" defer></script>

<!-- 动态加载，不阻塞 -->
<script>
    const script = document.createElement('script');
    script.src = 'dynamic.js';
    document.head.appendChild(script);
</script> 
```

##### 现代加载
```
<!-- type="module" 默认具有defer行为 -->
<script type="module" src="app.js"></script>

<!-- 异步模块 -->
<script type="module" async src="chart.js"></script>
```

##### async和defer的区别？
答案：
async：异步下载，下载完成后立即执行（不保证顺序）
defer：异步下载，DOM解析完成后按顺序执行
普通script：同步下载和执行，阻塞DOM解析