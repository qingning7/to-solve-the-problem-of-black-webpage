# to-solve-the-problem-of-black-webpage
！本教程适用于由 github pages 部署的网页

！项目类型 ：Vite+React

虽然过程看上去繁琐，但是一步步做下来 100% 成功，本人已亲自验证，按以下操作执行后网页能正常显示。

一、问题本质（一句话）

GitHub Pages 托管的是已打包的静态文件（HTML/CSS/JS），而 Vite+React 项目通常是源码（需要先 npm run build 生成 dist/）。如果把源码直接部署或资源路径错误（base 未设置）就会出现白屏/黑屏。

二：准备（把仓库拉到本地）

方法 1（图形界面）：使用 GitHub Desktop

File → Clone repository → 选择 qingning7/CHRISTMAS-TREE → Clone

方法 2（浏览器下载）：仓库页面 → Code → Download ZIP → 解压

方法 3（命令行）：

git clone https://github.com/你的用户名/仓库名.git
cd 仓库名


确认你在项目根目录：能看到 package.json、vite.config.js（或 vite.config.ts）、src/、index.html。

三：打开终端（进入项目根目录）

VS Code：File → Open Folder → `Ctrl + `` 打开内置终端

文件资源管理器：在项目文件夹空白处 Shift + 右键 → 在此处打开 PowerShell/Terminal

或系统 Terminal：cd /path/to/CHRISTMAS-TREE

确认目录内容：

ls
# 应该看到 package.json vite.config.js src index.html ...

四：安装依赖 & 构建（生成 dist）
直接在终端运行
npm install
npm run build


构建成功会看到类似：

dist/index.html
dist/assets/...
✓ built in ...
另一种检查构建成功的方法：在本地项目文件夹中查看是否多了一个名为 dist 的文件

如果没有 dist： 报错信息复制并粘贴给AI（常见是依赖未安装、node 版本问题、或不在项目根目录）。

五：确保 Vite 的 base 配置正确（关键点）

在 vite.config.js（或 vite.config.ts）中：

export default defineConfig({
  plugins: [react()],
  base: '/REPO_NAME/', // 例如：'/CHRISTMAS-TREE/'
  // ...
})


说明：base 要以 /repo-name/（包含前后斜杠）形式指定，否则构建后的静态资源引用路径会指向错误位置导致 404 → 白屏。

六：部署到 GitHub Pages（两种稳定方法）
方法 A（推荐，自动化）：使用 GitHub Actions（deploy workflow）

在仓库根创建新文件：.github/workflows/deploy.yml

把下面内容粘贴进去（已验证100%可用）：

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


提交（git add . && git commit -m "Add deploy workflow" && git push）

在 GitHub → Actions 中观察 workflow；成功后 Pages 会自动发布。

访问：https://<username>.github.io/<repo>/（例如 https://qingning7.github.io/CHRISTMAS-TREE/）

方法 B（手动）：把 dist 内容放到 docs/ 或 gh-pages 分支

把 dist 内容放到 docs（main 分支）

rm -rf docs
mkdir docs
cp -r dist/* docs/
git add docs
git commit -m "deploy: copy dist to docs"
git push


然后在仓库 Settings → Pages → Branch: main, Folder: /docs → Save。

把 dist 内容推到 gh-pages 分支（手动）
创建 gh-pages 分支并把 dist 的内容作为该分支根目录内容 push（注意不要把外层 dist 目录也上传，只上传里面的文件）。

七：Pages 设置 & 常见部署配置

Pages → Source：选择 gh-pages 分支 或 main / docs（取决于你用的方式）

如果构建产物包含以 _ 开头的文件，建议在构建产物根放一个.nojekyll 文件以禁用 Jekyll。

等待少许时间（通常几秒到几分钟），再访问页面。

八：如何快速定位“黑屏”/空白的问题（诊断清单）

打开浏览器 F12 → Console：看是否有 JS 报错（比如 Uncaught ReferenceError）

Network（刷新）：看 index.html、JS/CSS 是否返回 200 或 404。若 JS 返回 404，通常是 base 配置错误或部署目录错。

在本地打开 dist/index.html（用浏览器 file://）看是否能加载（注意 file:// 有时和服务器行为不同，但能快速验证资源是否存在）。

检查 GitHub Actions logs：看 npm run build 是否成功，upload-pages-artifact 是否上传了 dist。

检查 Pages 设置：Branch / Folder 是否选择正确。

如果使用 React Router（BrowserRouter），页面刷新可能 404 —— 考虑改成 HashRouter 或配置 404 重写到 index.html（Pages 默认不支持 history API 重写）。

九：常见坑与注意事项（总结）

不要把源码直接当静态网站部署（必须先 build）

必须设置 base 为 /repo-name/（否则资源路径错）

workflow 中不要引用不存在的 step outputs（易导致失败）

Vite build 的 chunk size 警告可忽略（仅性能提示）

如果你看见 "/index.css doesn't exist at build time..."：这通常是某些运行时引入或 CSS 插件提示，不是导致黑屏的原因

如果使用 GitHub Pages + SPA 路由，优先使用 HashRouter 或 gh-pages 的 404→index.html rewrite 方案

十：常用命令速查（复制粘贴版）
# 下载仓库（命令行）
git clone https://github.com/qingning7/CHRISTMAS-TREE.git
cd CHRISTMAS-TREE

# 安装 & 构建
npm install
npm run build

# 快速把 dist 内容放到 docs 并推 main
rm -rf docs
mkdir docs
cp -r dist/* docs/
git add docs
git commit -m "deploy: copy dist to docs"
git push

常见问题快速答案

Q：Some chunks are larger than 500 kB 会造成黑屏吗？
A：不会，只是性能/优化建议，可以后续按需拆分代码（dynamic import / manualChunks）。

Q：构建成功但页面还是空白？
A：检查 Pages 源（branch/folder）、base、Network 中是否有 404。
