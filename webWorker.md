##### webWorker
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
      worker.postMessage({cdd-  5
      ]\L
        t/
        ype: "test",
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