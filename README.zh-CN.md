# webpack-html

**基于 Webpack 4 的 HTML 多页面项目模板，支持 TypeScript、Sass 和页面入口自动发现。**

[English](./README.md) | 简体中文

通过文件夹组织独立 HTML 页面，复用公共样式和脚本，并使用 Webpack 打包资源。适合传统多页面网站、移动端 H5 页面、活动落地页和小型前端原型；它不是单页面应用框架。

> **旧版工具链提示：** 本项目使用 Webpack 4、TypeScript 3 和 `node-sass` 4，更适合作为旧项目维护参考或改造起点。不保证兼容当前版本的 Node.js，安装前请阅读[兼容性说明](#兼容性说明)。

## 目录

- [功能特性](#功能特性)
- [快速开始](#快速开始)
- [新增页面](#新增页面)
- [项目结构](#项目结构)
- [常用命令](#常用命令)
- [配置说明](#配置说明)
- [构建与部署](#构建与部署)
- [兼容性说明](#兼容性说明)
- [参与贡献](#参与贡献)
- [许可证](#许可证)

## 功能特性

- **按目录发现多页面入口：** `src/` 下每个页面目录提供 `index.ts` 入口和 `index.html` 模板。
- **自动生成 HTML：** 使用 `html-webpack-plugin` 输出 `<页面名>.html`，并注入公共及页面专属资源。
- **公共入口：** 根目录的 `main` 入口引入 `reset-css`，所有生成页面都会加载该入口。
- **TypeScript 与 JavaScript：** 构建流程配置了 TypeScript 编译和 Babel 转译。
- **CSS 与 Sass：** 开发时注入样式，构建时提取独立 CSS，并通过 Autoprefixer 为最近两个版本的浏览器添加前缀。
- **px 自动转换 rem：** `px2rem-loader` 的 `remUnit` 为 `16`，即 `16px` 转换为 `1rem`。
- **图片处理：** GIF、PNG、JPEG 使用 `url-loader`，内联阈值为 20 KB，输出图片文件名带哈希。
- **本地开发：** Webpack Dev Server 使用 `3000` 端口，Nodemon 监听 `src/` 下 JavaScript 和 TypeScript 文件变化。
- **开发规范：** 包含 ESLint、Prettier、lint-staged 和 commitlint 配置。

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/Fullsize/webpack-html.git
cd webpack-html
```

请使用与旧版依赖兼容的 Node.js，并选择 npm 或 Yarn。项目脚本使用 POSIX 风格环境变量；Windows 用户可使用 WSL / Git Bash，或通过 `cross-env` 改写脚本。

### 2. 创建第一个页面

仓库目前**没有预置 `src/` 目录**。入口配置会在启动时读取该目录，因此需要先创建页面：

```bash
mkdir -p src/home
```

先创建 `src/home/index.html`：

```html
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>首页 | webpack-html</title>
    <meta name="description" content="使用 Webpack 和 TypeScript 构建的多页面网站。" />
  </head>
  <body>
    <h1 id="greeting">你好，webpack-html！</h1>
  </body>
</html>
```

再创建 `src/home/index.ts`：

```ts
const greeting = document.getElementById("greeting");

if (greeting) {
  greeting.textContent = "来自 TypeScript 的问候！";
}

export {};
```

### 3. 安装依赖并启动

```bash
npm install
npm start
```

也可以选择 Yarn：

```bash
yarn install
yarn start
```

访问 **http://localhost:3000/home.html**。根地址展示的是单独的根目录 `index.html` 模板，不是 `home` 页面。浏览器不会自动打开。

如果安装或配置加载失败，请查看[兼容性说明](#兼容性说明)。以上命令描述的是项目配置的使用流程，不代表已验证其在你的环境中兼容运行。

## 新增页面

每个页面对应一个目录：

```text
src/
├── home/
│   ├── index.html
│   ├── index.ts
│   └── style.scss
└── about/
    ├── index.html
    └── index.ts
```

构建后会在输出目录根路径生成 `home.html` 和 `about.html`。页面入口必须命名为 **`index.ts`**，不是 `index.js`；如果需要修改，应调整 `webpack/entriesConfig.ts`。

在页面入口中导入样式，才能将其加入打包流程：

```ts
import "./style.scss";
```

```scss
/* src/home/style.scss */
#greeting {
  margin: 16px;
  font-size: 32px;
}
```

按照当前转换配置，上面的值会变为 `1rem` 和 `2rem`。实际显示尺寸取决于文档根元素字号；项目没有配置响应式根字号脚本。

**页面约定：**

- `src/` 顶层只放页面目录；扫描器会把每个顶层条目当作页面。
- 开发服务运行时，先创建 HTML 模板，再创建 TypeScript 入口，避免触发构建时找不到模板。
- 公共资源放在 `static/`，在需要的位置引用或导入；该目录**不会**被整体复制到 `dist/`。
- 不要将页面命名为 `uni`，这是公共入口的保留名称；也应避免命名为 `index`，因为根模板已经生成 `index.html`。
- 新页面未被识别时，重启开发进程。

## 项目结构

```text
webpack-html/
├── src/                       # 页面目录，需要自行创建
├── static/                    # 公共源资源
├── service/api-client.ts      # Axios 请求辅助模块
├── utils/index.ts             # URL 查询参数辅助函数
├── webpack/
│   ├── entriesConfig.ts       # 扫描页面目录与模板
│   └── webpack.config.ts      # 入口、loader、插件和开发服务配置
├── config.js                  # 按环境区分的 API 配置
├── index.html                 # 根页面 HTML 模板
├── main.js / main.ts          # 公共入口候选文件，均引入 reset-css
├── tsconfig.json              # TypeScript 编译配置
├── package.json               # 依赖、命令和 Git hooks
└── dist/                      # 构建输出目录
```

## 常用命令

| npm | Yarn | 用途 |
| --- | --- | --- |
| `npm install` | `yarn install` | 安装依赖 |
| `npm start` | `yarn start` | 启动 Nodemon 和开发服务器 |
| `npm run build` | `yarn build` | 构建并输出到 `dist/` |

`npm test` 目前只是返回错误的占位命令，项目尚未配置自动化测试。

## 配置说明

| 配置项 | 文件位置 | 当前值 / 行为 |
| --- | --- | --- |
| 页面发现 | `webpack/entriesConfig.ts` | `src/<页面名>/index.ts` 与 `index.html` |
| 开发服务器 | `webpack/webpack.config.ts` → `devServer` | `0.0.0.0:3000`，实时刷新，不自动打开浏览器 |
| 构建输出 | `webpack/webpack.config.ts` → `output` | `dist/`，JavaScript 输出到 `js/[name]-[hash].js` |
| 路径别名 | `webpack/webpack.config.ts` → `resolve.alias` | `@` 指向 `src/`，`~` 指向仓库根目录 |
| rem 转换 | `webpack/webpack.config.ts` → `px2rem-loader` | `remUnit: 16` |
| 图片处理 | `webpack/webpack.config.ts` → `url-loader` | 20 KB 内联阈值，输出图片位于 `static/img/` |
| API 地址 | `config.js` | 开发环境占位地址：`https://localhost:80` |

Webpack 路径别名尚未同步到 `tsconfig.json`（`paths` 为空）。如果要在 TypeScript 中使用别名导入，需要补充对应的 TypeScript 路径映射。

开发命令设置 `ENVIRONMENT=development`，构建命令设置 `ENVIRONMENT=build`。目前 `config.js` **只定义了开发环境**；如果在构建中使用 `service/api-client.ts`，需要先补充对应环境配置。该模块还使用 `prod` 判断是否添加调试参数，采用它之前应统一环境名称。

## 构建与部署

```bash
npm run build
```

构建会清理上一次输出，并将 HTML、JavaScript、提取后的 CSS 和引用的资源输出到 `dist/`。将 **`dist/` 中的内容**部署到静态 Web 服务器或静态托管平台即可。每个页面有独立地址，例如 `/home.html`、`/about.html`。

部署前建议：

- 为每个 HTML 模板设置明确的标题、页面描述和有意义的正文。本模板不会自动生成 SEO 元数据、站点地图或 `robots.txt`。
- 部署在子目录或使用 CDN 时，检查 `output.publicPath`。
- 检查 source map 配置：目前开发和构建都使用 `devtool: "eval-source-map"`，生产环境应按需求调整。
- 使用 Axios 辅助模块时，替换占位 API 配置。

## 兼容性说明

- **旧版依赖：** `node-sass` 4 依赖原生绑定，可能无法在新版 Node.js 或 Apple Silicon 上安装。请核对它支持的 Node.js 与平台组合，或迁移到 Dart Sass 和兼容的 loader。旧版兼容 Node.js 已停止维护，不应用于生产服务。
- **TypeScript 配置加载：** Webpack 配置文件使用 `.ts` 扩展名，但 `package.json` 没有声明 `ts-node`。如果 webpack-cli 提示无法加载配置，可添加兼容的 TypeScript 配置加载器（例如 `ts-node`），并为 Webpack 配置设置 CommonJS 编译；也可以将配置文件改为 JavaScript。
- **缺少 `src/`：** 出现涉及 `src` 的 `ENOENT` 错误时，请先创建页面目录再启动。
- **缺少页面入口或模板：** `src/` 下每个条目都必须包含 `index.ts` 和 `index.html`。
- **实时刷新不等于 HMR：** 当前配置中的模块热替换功能处于注释状态。
- **依赖维护：** 用于新的生产项目之前，应审查并更新依赖树。

## 参与贡献

欢迎在 [Fullsize/webpack-html](https://github.com/Fullsize/webpack-html) 提交 Issue 或 Pull Request。报告问题时，请附上操作系统、Node.js 版本、包管理器、复现步骤和错误输出。

保持改动范围清晰，遵循现有 lint 规范，并使用 [Conventional Commits](https://www.conventionalcommits.org/) 格式提交。修改文档时，请同步更新本文件和 [README.md](./README.md)。

## 许可证

`package.json` 声明许可证为 **ISC**。仓库目前没有独立许可证文件，如需在再分发前确认完整许可文本，请联系维护者。
