##### css in js
将 CSS 样式直接写入 JavaScript 或 TypeScript 代码中的技术，代表性的库包括 styled-components, Styled System, Emotion 等
优势：
1、组件级样式和局部作用域
  这是最根本的优势。CSS-in-JS 自动实现了样式的局部作用域，彻底解决了全局样式污染的问题。
  传统 CSS 的问题： 在传统的 CSS 中，每一个样式类都是全局的。如果不小心在兩個不同的组件中使用了相同的类名（如 .button），一个组件的样式就会意外地影响另一个组件。你需要依靠命名约定（如 BEM：.myComponent__button--primary）来避免冲突，但这依赖于开发者的自觉性，且命名会变得冗长复杂。

  CSS-in-JS 的解决方案： 样式被定义在组件内部，库会自动生成一个唯一的类名。这个类名只作用于当前组件，不会影响其他任何部分。
2、动态样式
可以轻松地根据组件的 props或全局主题（theme）来动态改变样式。这使得创建可配置的、灵活的UI组件变得非常简单，无需在CSS中定义多个类并通过JavaScript来切换。
```
// 创建一个按钮，其颜色和大小由 props 决定
const Button = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: ${props => {
    switch (props.size) {
      case 'large': return '16px 24px';
      case 'small': return '4px 8px';
      default: return '8px 16px';
    }
  }};
  border: none;
  border-radius: 4px;
`;

// 使用组件
function App() {
  return (
    <div>
      <Button primary size="large">主要大按钮</Button>
      <Button size="small">默认小按钮</Button>
      <Button primary>主要默认按钮</Button>
    </div>
  );
}
```
3、更好的开发者体验（DX）与可维护性
  I、样式与逻辑共置： 组件的样式和 JavaScript 逻辑放在同一个文件中（或紧邻的文件中），遵循了“高内聚”原则。当你需要修改一个组件时，所有相关代码（结构、逻辑、样式）都在一处，无需在多个文件（Component.jsx, Component.css, Component.module.css）之间来回切换。
  II、自动厂商前缀： 库会自动为需要浏览器厂商前缀的 CSS 属性（如 user-select, transition）添加前缀，无需开发者手动编写。
  III、自动清除无用样式： 如果组件被卸载，其相关的样式标签也会自动从 DOM 中移除，避免了传统项目中CSS文件不断膨胀的问题。
4、 主题切换轻松实现
CSS-in-JS 与 React 的 Context API 结合得非常好，可以极其方便地实现全局主题（如深色/浅色模式）
```
// 1. 定义主题
const theme = {
  light: {
    colors: {
      primary: 'blue',
      background: 'white',
      text: 'black'
    }
  },
  dark: {
    colors: {
      primary: 'lightblue',
      background: 'black',
      text: 'white'
    }
  }
};

// 2. 用 ThemeProvider 包裹应用
import { ThemeProvider } from 'styled-components';

function App() {
  const [isDarkMode, setIsDarkMode] = useState(false);
  const currentTheme = isDarkMode ? theme.dark : theme.light;

  return (
    <ThemeProvider theme={currentTheme}>
      <Button onClick={() => setIsDarkMode(!isDarkMode)}>
        切换主题
      </Button>
      <Content />
    </ThemeProvider>
  );
}

// 3. 在任何被包裹的组件中都可以使用主题
const Content = styled.div`
  background-color: ${props => props.theme.colors.background};
  color: ${props => props.theme.colors.text};
  padding: 20px;
`;

const ThemedButton = styled(Button)`
  background-color: ${props => props.theme.colors.primary};
`;
```
5、提升性能
CSS-in-JS 的样式是跟组件代码捆绑在一起的。当进行代码分割时（例如使用 React.lazy），只有被加载的组件所需的样式才会被注入到页面中，实现了样式的自动代码分割，减少了初始 CSS 包的大小

缺点：
1、运行时开销：样式需要在 JavaScript 中解析和注入，这会增加一些运行时成本。不过，对于大多数应用来说，这个开销可以忽略不计。一些库（如编译时CSS-in-JS）也在努力解决这个问题
2、学习曲线： 团队需要学习新的 API 和约定。
3、捆绑体积增大： 需要引入一个额外的库。
4、不利于非JS环境： 服务器端渲染（SSR）或静态站点生成（SSG）时需要额外配置。