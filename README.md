# to-solve-the-problem-of-black-webpage

[View Chinese Version / 中文版](#chinese-version)

This guide provides a solution for GitHub Pages displaying a blank screen (white or black) after deploying a Vite + React project.

While the process may seem detailed, following these steps ensures a 100% success rate for correct deployment and resource loading.

Troubleshooting Guide: Vite + React Deployment to GitHub Pages (Blank Screen Fix)

## I. The Core Issue

GitHub Pages hosts static files (HTML/CSS/JS). A Vite+React project contains source code that must be compiled via npm run build into a dist/ folder. If you deploy the source code directly or fail to configure the base path, the browser cannot find the assets, resulting in a blank screen.

## II. Preparation: Clone the Repository

Option 1: GitHub Desktop File → Clone repository → Select your repository → Clone.

Option 2: Browser Download Repo Page → Code → Download ZIP → Extract.

Option 3: Command Line

```
git clone https://github.com/USERNAME/REPO_NAME.git
cd REO_NAME
```

Confirm you are in the root directory (you should see package.json, vite.config.js, src/, and index.html).

## III. Open Terminal in Project Root

- VS Code: File → Open Folder → Press Ctrl + ~ to open the terminal.
- Terminal/PowerShell: Navigate to the folder and run ls to ensure you see the project files.

## IV. Install Dependencies & Build

```
npm install
npm run build
```

Upon success, a dist/ folder will be created containing index.html and an assets/ folder.

## V. Set the Correct Base Path (CRITICAL)

Open vite.config.js (or vite.config.ts) and add the base property:

```
export default defineConfig({
  plugins: [react()],
  base: '/REPO_NAME/', // Replace REPO_NAME with your actual GitHub repository name
})
```

Why? Without this, the HTML will look for assets at the root domain (e.g., user.github.io/assets/...) instead of your project folder (e.g., user.github.io/repo/assets/...), leading to 404 errors.

## VI. Deployment (using guthub actions)

1.Create a file at: .github/workflows/deploy.yml
2.Paste the following configuration:

```
name: Deploy Vite to GitHub Pages

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

3.Push to GitHub:

```
git add .
git commit -m "Add deploy workflow"
git push
```

4.Go to GitHub → Actions to monitor the progress. Once finished, go to Settings → Pages. Ensure the Build and deployment > Source is set to "GitHub Actions". Congratulations, you are ought to visit your website successfully.

<a name="chinese-version"></a>

# 中文版

本教程适用于由 github pages 部署的网页打开后显示为空（黑屏）

项目类型 ：Vite+React

虽然过程看上去繁琐，但是一步步做下来 100% 成功，本人已亲自验证，按以下操作执行后网页能正常显示。

## 一、问题本质

GitHub Pages 托管的是已打包的静态文件（HTML/CSS/JS），而 Vite+React 项目通常是源码（需要先 npm run build 生成 dist/）。如果把源码直接部署或资源路径错误（base 未设置）就会出现白屏/黑屏。

## 二：准备（把仓库拉到本地）

方法 1（图形界面）：使用 GitHub Desktop

File → Clone repository → 选择需要拉到本地的 repository → Clone

方法 2（浏览器下载）：仓库页面 → Code → Download ZIP → 解压

方法 3（命令行）：

```
git clone https://github.com/你的用户名/仓库名.git
cd 仓库名
```


确认你在项目根目录：能看到 package.json、vite.config.js（或 vite.config.ts）、src/、index.html。

## 三：打开终端（进入项目根目录）

VS Code：File → Open Folder → `Ctrl + `` 打开内置终端

文件资源管理器：在项目文件夹空白处 Shift + 右键 → 在此处打开 PowerShell/Terminal

或系统 Terminal：cd /path/to/项目名称

确认目录内容：

```
ls
```

应该看到 package.json vite.config.js src index.html ...

## 四：安装依赖（生成 dist）
直接在终端运行
```
npm install
npm run build
```


构建成功会看到类似：

```
dist/index.html
dist/assets/...
✓ built in ...
```

另一种检查构建成功的方法：在本地项目文件夹中查看是否多了一个名为 dist 的文件

如果没有 dist： 报错信息复制并粘贴给AI（常见是依赖未安装、node 版本问题、或不在项目根目录）。

## 五：确保 Vite 的 base 配置正确（重要！！！）

在 vite.config.js（或 vite.config.ts）中：

```
export default defineConfig({
  plugins: [react()],
  base: '/REPO_NAME/', // 例如：'/CHRISTMAS-TREE/'
  // ...
})
```


说明：base 要以 /repo-name/（包含前后斜杠）形式指定，否则构建后的静态资源引用路径会指向错误位置导致 404 → 白屏。

## 六：部署到 GitHub Pages：使用 GitHub Actions（deploy workflow）

在仓库根创建新文件：.github/workflows/deploy.yml

把下面内容粘贴进去（已验证100%可用）：

```yaml
name: Deploy Vite to GitHub Pages

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm run build
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

提交：

```bash
git add .
git commit -m "Add deploy workflow"
git push
```


提交（git add . && git commit -m "Add deploy workflow" && git push）

在 GitHub → Actions 中观察 workflow；成功后 Pages 会自动发布。在 Pages 中可以正常访问网页。

