##### webWorker
优势：
1、不会阻塞 UI 渲染和用户交互
2、可以执行计算密集型任务
3、真正实现多线程编程

限制：
1、无法访问 DOM​ - Worker 不能直接操作页面元素
2、无法使用 window 对象​ - 使用 self代替
3、通信通过消息传递​ - 数据需要序列化
4、同源策略​ - Worker 脚本必须与主页面同源
```
const createWorker = () => {
  try {
    // 直接使用相对路径
    const workerUrl = new URL("./worker.js", import.meta.url);
    console.log("Worker URL:", workerUrl.toString());

    const newWorker = new Worker(workerUrl, {
      name: "LogWorker",
    });

    // 设置错误处理
    newWorker.onerror = (error) => {
      console.error("Worker 错误:", error);
    };

    // 设置消息处理
    newWorker.onmessage = (event) => {
      console.log("Worker 接收到响应:", event.data);
      // 转发消息给回调函数
      if (messageCallback) {
        messageCallback(event.data);
      }
    };

    console.log("Worker 创建成功");
    return newWorker;
  } catch (error) {
    console.error("Worker 创建失败:", error);
    return null;
  }
};

// 获取 Worker 实例
export const getWorker = () => {
  if (!worker) {
    worker = createWorker();
    if (worker) {
      // 发送测试消息
      worker.postMessage({
        type: "test",
        payload: {
          type: "test",
          message: "Worker 初始化测试",
          timestamp: new Date().toISOString(),
        },
      });
    }
  }
  return worker;
};
```