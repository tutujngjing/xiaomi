# Vue 3 + Vite 项目

## 本地开发

```bash
npm install
npm run dev
```

## CI/CD 部署记录

以下是本项目配置 GitHub Pages 自动部署的完整步骤：

### 1. 登录 GitHub CLI

```bash
gh auth login
```

选择：

- GitHub.com
- HTTPS
- 浏览器登录或粘贴 Token

### 2. 申请 workflow 权限

因为部署文件 `.github/workflows/deploy.yml` 属于 GitHub Actions 工作流文件，需要额外权限才能推送：

```bash
gh auth refresh -s workflow -h github.com
```

在浏览器中确认授权即可。

### 3. 提交代码

```bash
git add .
git commit -m "init: vue3 + vite project with CI/CD"
```

### 4. 创建远程仓库并推送

```bash
gh repo create xiaomi --public --source=. --push
```

这条命令做了三件事：

- 在 GitHub 上创建了 `xiaomi` 仓库（公开）
- 关联本地仓库为远程源
- 推送代码到 `main` 分支

### 5. 开启 GitHub Pages（Actions 源）

```bash
gh api -X POST repos/tutujngjing/xiaomi/pages -f build_type=workflow
```

这一步将 GitHub Pages 的部署源设置为 GitHub Actions，而不是默认的分支文件。

### 6. 自动部署

推送代码后，GitHub Actions 会自动执行 `.github/workflows/deploy.yml` 中定义的流程：

1. checkout 代码
2. 安装 Node.js 20
3. `npm ci` 安装依赖
4. `npm run build` 构建产物
5. 上传 `dist` 目录
6. 部署到 GitHub Pages

部署完成后，网站地址为：**https://tutujngjing.github.io/xiaomi/**

### 7. 查看部署状态

```bash
gh run list          # 查看最近的 CI 运行记录
gh run watch <id>    # 实时查看某次运行的状态
```

## 后续更新

以后每次 push 到 `main` 分支，GitHub Actions 都会自动构建和部署，无需手动操作：

```bash
git add .
git commit -m "your changes"
git push origin main
```
