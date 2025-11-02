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