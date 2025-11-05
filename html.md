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


##### 输入浏览器网址发生了什么？

1、检测域名合法
2、DNS解析
3、发送HTTP三次握手
4、返回html构建DOM树和CS树
5、构建完成渲染页面
6、四次挥手

一、URL解析
输入处理：当用户在地址栏输入内容并按下回车后，浏览器首先会判断输入的内容是搜索关键词还是一个合法的 URL。
如果是搜索关键词：浏览器会使用默认的搜索引擎来合成一个新的搜索 URL。例如，输入 apple，可能会变成 https://www.google.com/search?q=apple。
如果是合法的 URL：浏览器会为其加上协议（如 http:或 https:）和端口等，合成完整的 URL。例如，输入 www.example.com可能会被补全为 https://www.example.com/。

二、DNS解析
查询过程（递归查询 + 迭代查询）：
浏览器缓存：浏览器首先检查自身缓存中是否有该域名对应的 IP 地址。
操作系统缓存：如果浏览器缓存没有，会去检查操作系统的 hosts 文件和 DNS 缓存（如 Windows 的 DNS Client 服务）。
路由器缓存：请求会发往路由器，检查路由器缓存。
ISP DNS 服务器：如果以上都没有，会向互联网服务提供商（ISP）指定的 DNS 服务器（如 8.8.8.8）发起递归查询。
根域名服务器 -> 顶级域服务器 -> 权威域名服务器：ISP 的 DNS 服务器会从根域名服务器开始，进行迭代查询，最终在负责该域名的权威域名服务器上找到对应的 IP 地址。
获取 IP：拿到 IP 地址后，会一路缓存回来，最终返回给浏览器

三、构建TCP三次握手
因为 HTTP 协议是使用 TCP 作为传输层协议的，所以必须先建立可靠的 TCP 连接。
三次握手：
第一次：客户端（浏览器）向服务器发送一个 SYN 包（同步序列编号），请求建立连接。
第二次：服务器收到 SYN 包后，返回一个 SYN-ACK 包（同步确认应答），表示同意建立连接。
第三次：客户端再向服务器发送一个 ACK 包，确认连接已建立。
至此，TCP 连接建立成功，客户端和服务器可以开始可靠地传输数据。
（如果是 HTTPS）：在 HTTP 数据传输开始之前，还会进行一次 TLS 握手。
协商加密套件、交换密钥、验证服务器证书等，建立起安全的加密通道。

四、发送HTTP请求
构建并发送请求报文：TCP 连接建立后，浏览器会构建一个 HTTP 请求报文，并通过该连接发送给服务器。请求报文包括：
请求行：包含请求方法（GET, POST 等）、URL 路径、HTTP 版本。
请求头：包含浏览器信息（User-Agent）、可接受的内容类型（Accept）、Cookie、缓存控制（Cache-Control）等大量信息。
请求体：对于 POST 等方法，会包含要发送给服务器的数据（如表单数据）

五、服务器处理请求并返回响应
服务器处理：服务器接收到请求后，会根据 URL 路径和请求头信息，由后端的 Web 服务器（如 Nginx、Apache）、应用服务器（如 Node.js、Tomcat）和数据库等协同工作，生成一个 HTTP 响应。
返回响应报文：服务器将响应发送回浏览器。响应报文包括：
状态行：包含 HTTP 版本、状态码（如 200 OK、404 Not Found）。
响应头：包含内容类型（Content-Type）、内容长度（Content-Length）、设置 Cookie（Set-Cookie）等信息。
响应体：通常是请求的 HTML 文档内容

六、浏览器接收响应、解析和渲染
这是前端最需要关注的部分。
1. 处理响应：
浏览器根据响应头的状态码判断请求是否成功（如 200）。
根据 Content-Type字段（如 text/html）判断响应体是什么类型。
2. 解析 HTML，构建 DOM 树：
浏览器从响应体中读取 HTML 原始字节流。
根据指定的编码（如 UTF-8）将字节流转换成字符串。
将字符串通过词法分析转换成一系列的 Tokens（标签令牌）。
接着 语法分析，根据 Tokens 构建出节点（Node），并最终形成一棵DOM（文档对象模型）树。
3. 解析 CSS，构建 CSSOM 树：
在构建 DOM 树的过程中，如果遇到 <link>标签引入的外部 CSS 文件，或者 <style>标签内的内部 CSS，浏览器会并行地开始解析 CSS。
与 HTML 解析类似，最终会构建出一棵 CSSOM（CSS 对象模型）树，它描述了页面所有元素的样式规则。
4. 合并 DOM 和 CSSOM，构建渲染树：
将 DOM 树和 CSSOM 树合并成一棵 渲染树。这棵树只包含需要在页面上显示的元素（例如，不包含 display: none的元素）
5. 布局：
根据渲染树，计算每个节点在视口内的确切位置和大小。这个阶段也称为 “重排”
6. 绘制：
将布局计算的每个节点转换成屏幕上的实际像素。包括绘制颜色、文字、图像、边框、阴影等。
绘制通常是在多个图层上完成的。
7. 合成与显示：
浏览器将各图层分别绘制并合成到一起。最终，页面内容显示在屏幕上。
关键点：
遇到 JavaScript 会阻塞：当 HTML 解析器遇到 <script>标签（没有 async或 defer属性）时，会暂停 DOM 的构建，先下载（如果是外部脚本）并执行 JavaScript 代码。因为 JS 可能会修改 DOM 或 CSSOM。
优化策略：这就是为什么我们常把 <script>标签放在 body 底部，或者使用 async/defer属性的原因

七、TCP 连接断开（四次挥手）
四次挥手：当页面完全加载后，如果不需要保持连接（HTTP/1.1 默认 Connection: keep-alive会保持连接以供复用），浏览器和服务器会断开 TCP 连接。
第一次：浏览器发送 FIN 包，表示数据发送完毕，请求关闭连接。
第二次：服务器返回 ACK 包，确认收到关闭请求。
第三次：服务器发送 FIN 包，表示服务器也数据发送完毕，准备关闭。
第四次：浏览器返回 ACK 包，确认关闭。等待一段时间后，双方连接正式关闭。

##### 什么是重排重绘
###### 重排：布局改变，需要重新计算几何属性
几何属性：
width, height
padding, margin, border
top, left, right, bottom
font-size, line-height

添加或删除 DOM 元素（display: none会触发重排，visibility: hidden只会触发重绘）
元素位置改变（如改变 position的值）
修改 display属性（如 block改为 inline-block）
激活 CSS 伪类（如 :hover如果改变了布局）
计算或获取元素的几何信息（这非常关键！）
offsetTop, offsetLeft, offsetWidth, offsetHeight
scrollTop, scrollLeft, scrollWidth, scrollHeight
clientTop, clientLeft, clientWidth, clientHeight
getComputedStyle()或 currentStyle（IE）

改变浏览器窗口大小（resize事件）
改变网页中的文字大小
内容变化（如用户输入文本）

###### 重绘：外观改变，但不影响布局

color
border-style, border-radius
visibility（注意：与 display:none不同，visibility:hidden只触发重绘）
text-decoration
background相关属性（background-color, background-image等）
outline相关属性
box-shadow

JavaScript-> 2. 样式计算-> 3. 布局-> 4. 绘制-> 5. 合成
重排：也叫 回流。当渲染树中的一部分（或全部）因为元素的尺寸、布局、隐藏等改变而需要重新构建。这个过程发生在上述流程的第 3 步（布局）。重排一定会触发后续的重绘。
重绘：当元素发生的改变只影响其外观、风格，而不会影响其布局时，浏览器会重新绘制受影响的元素。这个过程发生在上述流程的第 4 步（绘制）。


##### DOCTYPE的作用
文件头声明
告诉浏览器当前HTML文件使用哪个版本的html标准，确保浏览器以正确的方式渲染页面

##### 块级元素和行内元素的区别？分别有哪些
块级元素：独占一行，前后有换行，默认宽度100%，可以设置宽高，完整的盒模型，可以包含块级元素盒行内元素
行内元素：与其他行内元素在同一行展示，宽度由内容决定，不可设置宽高，只能包含行内元素盒文本，水平margin/padding有效，垂直无效

块级元素：div,h1-h6,table,form,
行内元素：span,a,strong,button,label,img,video,audio,svg,canvas,input,select,

行内块元素：
像行内元素一样水平排列，像块级元素一样可以设置宽高，完整的和模型