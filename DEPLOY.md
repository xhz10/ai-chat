# 部署指南（AI Chat 前端项目）

本项目基于 Vite + Vue3，推荐使用 nvm 管理 Node 版本，确保部署安全、环境一致。

---

## 1. 环境准备

### 1.1 安装 nvm（如已安装可跳过）

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# 或参考官方文档：https://github.com/nvm-sh/nvm
```

### 1.2 切换 Node 版本

项目根目录已包含 `.nvmrc` 文件，推荐版本：**16.17.1**

```bash
cd /path/to/your/project
nvm install # 自动安装 .nvmrc 指定版本
nvm use     # 切换到该版本
```

---

## 2. 安装依赖

```bash
npm install
```

---

## 3. 本地开发（可选）

```bash
npm run dev
```

---

## 4. 构建生产包

```bash
npm run build
```

构建完成后，静态文件会生成在 `dist/` 目录。

---

## 5. 生产环境部署

你可以用任意静态文件服务器（如 nginx、http-server、vercel、netlify 等）部署 `dist/` 目录。

### 5.1 使用 http-server 快速本地预览

```bash
npm install -g http-server
http-server dist
```

### 5.2 使用 nginx 部署（示例配置）

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /path/to/your/project/dist;
    index index.html;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 6. 常见问题

- **端口冲突**：如 5173 端口被占用，可在 `package.json` 的 dev 命令中指定其他端口。
- **后端跨域**：如需与后端联调，确保后端已正确设置 CORS。
- **Node 版本不符**：务必用 `nvm use` 保证 Node 版本一致。

---

## 7. 其他建议

- 推荐将 `.nvmrc`、`DEPLOY.md` 一并提交到仓库，方便团队协作。
- 如需自动化部署，可结合 CI/CD 工具（如 GitHub Actions、Jenkins 等）。

---

如有疑问，欢迎随时联系开发者。 