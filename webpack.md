##### Webpack
webpack是一个js的静态模块打包器，将应用程序的文件按照入口文件开始构建依赖关系图，然后生成一个或者多个打包文件。


核心概念：
entry:构建依赖图起点
output:打包后的文件输出位置
loader：转换非js文件为有效模块
plugin：执行更广泛的任务，如优化打包，资源管理
mode：生产或者开发环境


##### 什么是loader?举例常用的loader和作用
loader是将非js文件转换为webpack能够处理的模块

css-loader:处理css中的@import和url()
file-loader：处理文件资源
style-loader：将css插入到dom中
babel-loader：转换es6+为es5
url-loader：类似 file-loader，但可设置文件大小限制转为 base64


##### 什么是plugin?与loader的区别
loader:用于转换特殊的文件，作用在单文件上
plugin:在构建的过程中，利用钩子函数执行扩展功能

常用plugin：
htmlWebpackPlugin:生成html文件，并自动注入到打包后的资源
CleanWebpackPlugin:清除输出目录打包资源
MiniCssExtractPlugin：提取css到单独的文件
DefinePlugin：定义全局变量


##### 如何优化webpack构建性能
1、合理使用exclude和includes,较少不必要的文件处理
2、使用cache-loader：缓存构建结果
3、代码分割：使用splitChunks分割公共代码
4、多线程构建：使用 thread-loader或 HappyPack
5、tree shaking


##### tree shaking是什么？
消除无用代码（dead code）的优化技术。
实现条件：
使用 ES6 模块语法（import/export）
在 package.json中设置 "sideEffects": false
在生产模式下（mode: 'production'）自动启用

##### 如何实现代码分割
1、​​入口点分割​​：配置多个入口点
2、​​动态导入​​：使用 import()语法
3、​​SplitChunksPlugin​​：提取公共依赖

```
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendor: {
        test: /[\\/]node_modules[\\/]/,
        name: 'vendors',
        chunks: 'all'
      }
    }
  }
}
```

##### webpack热更新原理
1、webpack dev server启动本地服务
2、建立websocket连接，监听本地文件变化
3、文件变化时，重新编译文件
4、通过websocket推送更新给浏览器
5、浏览器更新替换模块，不刷新页面

##### 写过什么loader?写过什么plugin？

```

const Regex = /woa\.com/
const base64UrlPlugin = ({ types: t }) => ({
  visitor: {
    StringLiteral(path) {
      const { value } = path.node
      if (Regex.test(value)) {
        const base64Url = Buffer.from(value).toString('base64')
        console.log('【替换】', value, '=>', base64Url)
        const atobCallExpression = t.callExpression(
          t.identifier('atob'),
          [t.stringLiteral(base64Url)],
        )
        path.replaceWith(atobCallExpression)
      }
    },
  },
})
```


##### webpack打包流程和相关钩子函数
初始化参数->确定入口->确定依赖->开始编译->编译模块->编译完成->输出资源->打包完成
代码模拟
```
// 模拟webpack核心流程
class WebpackProcess {
  constructor(options) {
    this.options = options;
    this.compiler = null;
    this.compilation = null;
  }
  
  run() {
    // 1. 初始化阶段
    this.initialize();
    
    // 2. 编译阶段
    this.compile();
    
    // 3. 输出阶段
    this.emit();
    
    // 4. 完成阶段
    this.finish();
  }
  
  initialize() {
    // 参数标准化、创建Compiler实例
    console.log('1. 初始化参数和插件');
  }
  
  compile() {
    // 创建Compilation、构建模块依赖图
    console.log('2. 编译模块，生成AST和依赖');
  }
  
  emit() {
    // 生成最终文件，写入文件系统
    console.log('3. 输出资源到dist目录');
  }
  
  finish() {
    // 统计信息、清理工作
    console.log('4. 打包完成');
  }
}
```

一、初始化阶段钩子
```
compiler.hooks.environment.tap('MyPlugin', () => {
  console.log('environment: 准备环境');
});

compiler.hooks.afterEnvironment.tap('MyPlugin', () => {
  console.log('afterEnvironment: 环境准备完成');
});

compiler.hooks.entryOption.tap('MyPlugin', (context, entry) => {
  console.log('entryOption: 处理入口配置', entry);
});

compiler.hooks.afterPlugins.tap('MyPlugin', (compiler) => {
  console.log('afterPlugins: 插件注册完成');
});

compiler.hooks.afterResolvers.tap('MyPlugin', (compiler) => {
  console.log('afterResolvers: 解析器设置完成');
});
```
二、编译过程钩子
```
compiler.hooks.beforeRun.tap('MyPlugin', (compiler) => {
  console.log('beforeRun: 开始执行编译前');
});

compiler.hooks.run.tap('MyPlugin', (compiler) => {
  console.log('run: 开始编译');
});

compiler.hooks.watchRun.tap('MyPlugin', (compiler) => {
  console.log('watchRun: 监听模式开始编译');
});

compiler.hooks.normalModuleFactory.tap('MyPlugin', (nmf) => {
  console.log('normalModuleFactory: 创建普通模块工厂');
});

compiler.hooks.contextModuleFactory.tap('MyPlugin', (cmf) => {
  console.log('contextModuleFactory: 创建上下文模块工厂');
});

compiler.hooks.beforeCompile.tap('MyPlugin', (params) => {
  console.log('beforeCompile: 准备编译');
});

compiler.hooks.compile.tap('MyPlugin', (params) => {
  console.log('compile: 开始创建compilation');
});

compiler.hooks.thisCompilation.tap('MyPlugin', (compilation, params) => {
  console.log('thisCompilation: 创建新的compilation');
});

compiler.hooks.compilation.tap('MyPlugin', (compilation, params) => {
  console.log('compilation: compilation创建完成');
});
```

三、输出阶段钩子
```
compiler.hooks.shouldEmit.tap('MyPlugin', (compilation) => {
  console.log('shouldEmit: 检查是否应该输出资源');
  return true; // 返回false可阻止输出
});

compiler.hooks.emit.tap('MyPlugin', (compilation) => {
  console.log('emit: 生成资源到输出目录前');
});

compiler.hooks.afterEmit.tap('MyPlugin', (compilation) => {
  console.log('afterEmit: 资源已输出到目录');
});

compiler.hooks.assetEmitted.tap('MyPlugin', (file, info) => {
  console.log('assetEmitted: 单个资源输出完成', file);
});
```
四、完成阶段钩子
```
compiler.hooks.done.tap('MyPlugin', (stats) => {
  console.log('done: 编译完成', stats.hasErrors());
});

compiler.hooks.failed.tap('MyPlugin', (error) => {
  console.log('failed: 编译失败', error);
});

compiler.hooks.invalid.tap('MyPlugin', (fileName, changeTime) => {
  console.log('invalid: 监听模式下文件变化', fileName);
});

compiler.hooks.watchClose.tap('MyPlugin', () => {
  console.log('watchClose: 监听模式停止');
});
```
五、模块构建钩子
```
compilation.hooks.buildModule.tap('MyPlugin', (module) => {
  console.log('buildModule: 开始构建模块', module.request);
});

compilation.hooks.succeedModule.tap('MyPlugin', (module) => {
  console.log('succeedModule: 模块构建成功', module.request);
});

compilation.hooks.failedModule.tap('MyPlugin', (module, error) => {
  console.log('failedModule: 模块构建失败', module.request, error);
});

compilation.hooks.finishModules.tap('MyPlugin', (modules) => {
  console.log('finishModules: 所有模块构建完成');
});
```
六、优化阶段钩子
```
compilation.hooks.optimize.tap('MyPlugin', () => {
  console.log('optimize: 开始优化阶段');
});

compilation.hooks.optimizeModules.tap('MyPlugin', (modules) => {
  console.log('optimizeModules: 优化模块');
});

compilation.hooks.optimizeChunks.tap('MyPlugin', (chunks) => {
  console.log('optimizeChunks: 优化代码块');
});

compilation.hooks.optimizeTree.tap('MyPlugin', (chunks, modules) => {
  console.log('optimizeTree: 异步优化依赖树');
});

compilation.hooks.optimizeChunkAssets.tap('MyPlugin', (chunks) => {
  console.log('optimizeChunkAssets: 优化代码块资源');
});

compilation.hooks.optimizeAssets.tap('MyPlugin', (assets) => {
  console.log('optimizeAssets: 优化所有资源');
});
```

钩子执行顺序
```
// 1. 初始化阶段
environment → afterEnvironment → entryOption → afterPlugins → afterResolvers

// 2. 编译开始
beforeRun → run → normalModuleFactory → contextModuleFactory → beforeCompile → compile

// 3. 编译过程
compilation → thisCompilation → make → buildModule → succeedModule/failedModule

// 4. 优化阶段
finishModules → optimize → optimizeModules → optimizeChunks → optimizeTree

// 5. 输出阶段
shouldEmit → emit → assetEmitted → afterEmit

// 6. 完成阶段
done / failed
```


##### Babel
源代码 → 解析（Parse）→ 转换（Transform）→ 生成（Generate）→ 目标代码
     ↓          ↓             ↓             ↓
   ES6+       AST         修改AST         ES5
   JSX       树结构       插件处理       普通JS
TypeScript             预设应用

```
// 完整的编译过程
const babel = require('@babel/core');

const result = babel.transformSync(code, {
  // 1. 解析阶段配置
  parserOpts: {
    sourceType: 'module',
    plugins: ['jsx', 'typescript']
  },
  
  // 2. 转换阶段配置
  presets: ['@babel/preset-env'],
  plugins: ['@babel/plugin-transform-arrow-functions'],
  
  // 3. 生成阶段配置
  generatorOpts: {
    retainLines: true,
    comments: false
  }
});
```

自定义插件：访问字符串
```
const base64UrlPlugin = ({ types: t }) => ({
  visitor: {
    StringLiteral(path) {
      const { value } = path.node
      if (Regex.test(value)) {
        const base64Url = Buffer.from(value).toString('base64')
        console.log('【替换】', value, '=>', base64Url)
        const atobCallExpression = t.callExpression(
          t.identifier('atob'),
          [t.stringLiteral(base64Url)],
        )
        path.replaceWith(atobCallExpression)
      }
    },
  },
})
```