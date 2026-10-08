# webpack-html

**A Webpack 4 multi-page HTML starter with TypeScript, Sass, and automatic page discovery.**

English | [简体中文](./README.zh-CN.md)

Build independent HTML pages from a folder-based structure, share common styles and scripts, and bundle assets with Webpack. This repository is intended for traditional multi-page websites, mobile H5 pages, landing pages, and small front-end prototypes—not as a single-page application framework.

> **Legacy toolchain:** this project uses Webpack 4, TypeScript 3, and `node-sass` 4. It is best treated as a reference or a starting point for maintaining older projects. Compatibility with current Node.js releases is not guaranteed; review [compatibility notes](#compatibility-notes) before installing.

## Contents

- [Features](#features)
- [Quick start](#quick-start)
- [Create a page](#create-a-page)
- [Project structure](#project-structure)
- [Commands](#commands)
- [Configuration](#configuration)
- [Build and deployment](#build-and-deployment)
- [Compatibility notes](#compatibility-notes)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Folder-based multi-page entries:** each page folder under `src/` supplies an `index.ts` entry and an `index.html` template.
- **Generated HTML:** `html-webpack-plugin` produces `<page-name>.html` and injects the shared and page-specific bundles.
- **Shared entry:** the root `main` entry imports `reset-css` and is included in every generated page.
- **TypeScript and JavaScript:** TypeScript compilation and Babel transpilation are configured in the build pipeline.
- **CSS and Sass:** style injection during development, separate CSS files during builds, and Autoprefixer for the last two browser versions.
- **Pixel-to-rem conversion:** `px2rem-loader` is configured with `remUnit: 16` (`16px` → `1rem`).
- **Image processing:** GIF, PNG, and JPEG files use `url-loader`, with a 20 KB inline threshold and hashed filenames for emitted images.
- **Local development:** Webpack Dev Server runs on port `3000`; Nodemon watches JavaScript and TypeScript changes under `src/`.
- **Code conventions:** ESLint, Prettier, lint-staged, and commitlint configuration is included.

## Quick start

### 1. Clone the repository

```bash
git clone https://github.com/Fullsize/webpack-html.git
cd webpack-html
```

Use a Node.js version compatible with the legacy dependencies, and either npm or Yarn. The scripts use POSIX-style environment variables; on Windows, use WSL/Git Bash or adapt the scripts with `cross-env`.

### 2. Create the first page

The repository does **not** include a `src/` directory. Create a page before starting the server, because the entry configuration reads this directory at startup:

```bash
mkdir -p src/home
```

Create `src/home/index.html` first:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Home | webpack-html</title>
    <meta name="description" content="A multi-page website built with Webpack and TypeScript." />
  </head>
  <body>
    <h1 id="greeting">Hello, webpack-html!</h1>
  </body>
</html>
```

Then create `src/home/index.ts`:

```ts
const greeting = document.getElementById("greeting");

if (greeting) {
  greeting.textContent = "Hello from TypeScript!";
}

export {};
```

### 3. Install dependencies and start

```bash
npm install
npm start
```

Or use Yarn instead:

```bash
yarn install
yarn start
```

Open **http://localhost:3000/home.html**. The root URL serves the separate root `index.html` template, not the `home` page. The browser does not open automatically.

If installation or configuration loading fails, see [compatibility notes](#compatibility-notes). The commands above describe the configured workflow, not a verified compatibility guarantee for your environment.

## Create a page

Add a folder for every page:

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

This produces `home.html` and `about.html` at the output root. Page entry filenames must be **`index.ts`**, not `index.js`, unless you change `webpack/entriesConfig.ts`.

Import a stylesheet from the page entry to include it in the bundle:

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

With the current conversion settings, these values become `1rem` and `2rem`. The rendered size depends on the document's root font size; the project does not set up a responsive root-font-size script.

**Page conventions:**

- Keep only page directories at the top level of `src/`; the scanner treats every top-level item as a page.
- Create the HTML template before the TypeScript entry when the dev server is running, to avoid rebuilding with a missing template.
- Put shared resources in `static/` and reference or import them where needed. The directory is **not** copied wholesale into `dist/`.
- Do not name a page `uni`; that name is reserved for the shared entry. Avoid `index` as a page name because the root template already generates `index.html`.
- Restart the development process if newly added pages are not discovered.

## Project structure

```text
webpack-html/
├── src/                       # Page directories (create this yourself)
├── static/                    # Shared source assets
├── service/api-client.ts      # Axios client helper
├── utils/index.ts             # Query-string helper
├── webpack/
│   ├── entriesConfig.ts       # Discovers page folders and templates
│   └── webpack.config.ts      # Entries, loaders, plugins, and dev server
├── config.js                  # API configuration by environment
├── index.html                 # Root HTML template
├── main.js / main.ts          # Shared entry candidates; both import reset-css
├── tsconfig.json              # TypeScript compiler settings
├── package.json               # Dependencies, scripts, and Git hooks
└── dist/                      # Generated build output
```

## Commands

| npm | Yarn | Purpose |
| --- | --- | --- |
| `npm install` | `yarn install` | Install dependencies |
| `npm start` | `yarn start` | Start Nodemon and the development server |
| `npm run build` | `yarn build` | Generate output in `dist/` |

`npm test` is a placeholder that exits with an error. No automated test suite is currently configured.

## Configuration

| Setting | Location | Current value / behavior |
| --- | --- | --- |
| Page discovery | `webpack/entriesConfig.ts` | `src/<page>/index.ts` + `index.html` |
| Server | `webpack/webpack.config.ts` → `devServer` | `0.0.0.0:3000`, live reload, no automatic browser opening |
| Output | `webpack/webpack.config.ts` → `output` | `dist/`, JavaScript in `js/[name]-[hash].js` |
| Import aliases | `webpack/webpack.config.ts` → `resolve.alias` | `@` → `src/`, `~` → repository root |
| rem conversion | `webpack/webpack.config.ts` → `px2rem-loader` | `remUnit: 16` |
| Images | `webpack/webpack.config.ts` → `url-loader` | 20 KB inline threshold; emitted files under `static/img/` |
| API URL | `config.js` | Development placeholder: `https://localhost:80` |

Webpack aliases are not mirrored in `tsconfig.json` (`paths` is empty). Add matching TypeScript paths if you want to use aliases in TypeScript imports.

The scripts set `ENVIRONMENT=development` for the dev server and `ENVIRONMENT=build` for builds. `config.js` currently defines **only** the development environment; add a matching configuration before using `service/api-client.ts` in a build. That helper also checks `prod` for its debug parameter, so align the environment names if you adopt it.

## Build and deployment

```bash
npm run build
```

The build cleans the previous output and generates HTML, JavaScript, extracted CSS, and referenced assets in `dist/`. Publish the **contents of `dist/`** to a static web server or static hosting service. Each page has its own URL, such as `/home.html` or `/about.html`.

Before deploying:

- Give each HTML template a descriptive title, meta description, and meaningful content. This starter does not automatically generate SEO metadata, a sitemap, or `robots.txt`.
- Review `output.publicPath` if hosting under a subdirectory or serving assets from a CDN.
- Review source-map settings: `devtool` is currently `eval-source-map` for both development and builds; change it for production as appropriate.
- Replace the placeholder API configuration if the application uses the Axios helper.

## Compatibility notes

- **Legacy dependencies:** `node-sass` 4 relies on native bindings and may fail to install on newer Node.js versions or Apple Silicon. Check its supported Node.js/platform combinations, or migrate to Dart Sass and compatible loaders. Older compatible Node.js versions are end-of-life and should not be used for production services.
- **TypeScript configuration loading:** the Webpack configuration files end in `.ts`, but `ts-node` is not declared in `package.json`. If webpack-cli reports that it cannot load the configuration, add a compatible TypeScript configuration loader (such as `ts-node`) and configure CommonJS compilation for the Webpack config, or convert the config files to JavaScript.
- **Missing `src/`:** an `ENOENT` error mentioning `src` means the page directory needs to be created before startup.
- **Missing page entry/template:** every item under `src/` must provide both `index.ts` and `index.html`.
- **Live reload, not HMR:** hot module replacement is commented out in the current configuration.
- **Dependency maintenance:** review and update the dependency tree before using this starter for a new production project.

## Contributing

Issues and pull requests are welcome at [Fullsize/webpack-html](https://github.com/Fullsize/webpack-html). For bug reports, include your operating system, Node.js version, package manager, reproduction steps, and error output.

Keep changes focused, follow the existing linting conventions, and use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages. When updating documentation, keep this file and [README.zh-CN.md](./README.zh-CN.md) in sync.

## License

`package.json` declares the **ISC** license. A standalone license file is not currently included; consult the maintainer if you need the full license text before redistribution.
