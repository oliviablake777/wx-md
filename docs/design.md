# Markdown-to-WeChat-Renderer 技术架构设计

本文档旨在为 Markdown-to-WeChat-Renderer 组件项目确立清晰、统一的技术架构和开发规范，以确保项目的高效开发、后期维护和功能扩展。

## 1. 技术选型

为了构建一个现代化、高性能且易于维护的组件，我们选择以下技术栈：

- **核心框架**：**React 18+**
  - **理由**：React 拥有庞大的社区、丰富的生态和组件化的开发模式，非常适合构建可复用的 UI 组件。其 Hooks API 能让我们更便捷地管理组件状态和副作用。

- **构建工具**：**Vite**
  - **理由**：Vite 提供了极速的冷启动和模块热更新（HMR）能力，能显著提升开发体验。其基于 Rollup 的打包机制也为生产环境输出了高度优化的代码。

- **样式方案**：**CSS Modules + PostCSS**
  - **理由**：CSS Modules 保证了样式的局部作用域，避免了全局样式污染。结合 PostCSS，我们可以使用最新的 CSS 特性，并自动添加浏览器前缀，增强兼容性。

- **代码高亮**：**`react-syntax-highlighter`**
  - **理由**：这是一个功能强大且灵活的代码高亮库，支持多种编程语言和丰富的高亮主题（如 `github-dark`），完全满足需求文档中的要求。

- **Markdown 解析**：**`react-markdown` + `remark-gfm`**
  - **理由**：`react-markdown` 是一个流行的 Markdown 转 React 组件的库，安全性高，可定制性强。配合 `remark-gfm` 插件，可以支持 GitHub Flavored Markdown (GFM)，如表格、删除线等。

- **HTML 内联样式转换**：**`juice`**
  - **理由**：`juice` 是一个成熟的库，专门用于将 CSS 规则转换为 HTML 内联样式。这是实现“复制到公众号”功能的核心，能确保样式在微信编辑器中正确显示。

## 2. 目录结构规范

项目将采用模块化的目录结构，以实现高内聚、低耦合的设计目标。

\`\`\`markdown
/md-wx-renderer
├── docs/                      # 项目文档
│   ├── requirement_analysis.md
│   └── technical_architecture.md
├── public/                    # 静态资源
├── src/                       # 源码目录
│   ├── assets/                # 静态资源（图片、字体等）
│   ├── components/            # 可复用组件
│   │   ├── Previewer/         # 预览核心组件
│   │   ├── SettingsPanel/     # 设置面板组件
│   │   └── CodeBlock/         # 定制化代码块组件
│   ├── constants/             # 常量
│   ├── hooks/                 # 自定义 Hooks
│   ├── styles/                # 全局样式与主题
│   │   ├── themes/            # 主题样式文件 (CSS Modules)
│   │   └── global.css         # 全局基础样式
│   ├── utils/                 # 工具函数
│   ├── App.jsx                # 应用主组件
│   └── main.jsx               # 入口文件
├── .eslintrc.cjs              # ESLint 配置文件
├── .prettierrc                # Prettier 配置文件
├── package.json               # 项目依赖与脚本
└── vite.config.js             # Vite 配置文件
\`\`\`

## 3. 编码规范

- **代码风格**：遵循 **Prettier** 的默认规则，实现自动化格式统一。
- **代码质量**：使用 **ESLint** 配合 `eslint-plugin-react` 和 `eslint-plugin-react-hooks`，遵循官方推荐的最佳实践，确保代码质量。
- **命名规范**：
  - **组件**：采用大驼峰命名法（PascalCase），如 `SettingsPanel`。
  - **函数/变量**：采用小驼峰命名法（camelCase），如 `handleCopy`。
  - **CSS 类名**：当使用 CSS Modules 时，采用小驼峰命名法。
- **注释**：对复杂的业务逻辑、算法或非直观的代码块，添加必要的注释。

## 4. 微信公众号样式兼容方案

微信公众号编辑器对外部 HTML 的样式支持有严格限制，核心要求是 **所有样式必须以内联 `style` 属性的形式存在**。

### 实现流程：

1.  **用户点击“复制”按钮**：触发 `handleCopy` 函数。
2.  **获取渲染后 HTML**：通过 `ref` 获取预览区域 `div` 的 `innerHTML`。
3.  **获取当前主题 CSS**：根据用户选择的主题，动态加载对应主题的 CSS 文件内容（字符串形式）。
4.  **调用 `juice` 进行转换**：
    - 将获取到的 HTML 字符串和主题 CSS 字符串作为参数传递给 `juice` 库。
    - `juice` 会解析 CSS，并将所有匹配的样式规则应用为对应 HTML 元素的 `style` 属性。
    - 例如，CSS 中的 `.title { color: blue; }` 会被转换为 `<h1 class="title" style="color: blue;">...</h1>`。
5.  **处理代码块高亮**：`react-syntax-highlighter` 默认会生成带 `style` 的 `<span>` 标签，这部分样式可以直接被微信识别，无需额外处理。
6.  **复制到剪贴板**：使用 `navigator.clipboard.writeText` API 将处理后的 HTML 字符串复制到用户剪贴板。

通过此方案，可以确保所有视觉样式（包括主题颜色、字体、边距以及代码高亮）都能被完整、正确地复制和粘贴到微信公众号编辑器中，实现所见即所得的效果。