# Docker + Nginx 自动化部署（详细步骤）

## 第一步：编写 Dockerfile

创建 `Dockerfile`，使用两阶段构建：

```dockerfile
# 第一阶段：构建
FROM node:20-alpine AS builder
ARG BASE_URL=/
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN VITE_BASE_URL=${BASE_URL} npm run build

# 第二阶段：运行
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

说明：
- **第一阶段**：用 Node.js 20 镜像安装依赖并构建产物（`dist` 目录）
- **第二阶段**：用 Nginx 镜像托管构建产物，只保留最终的静态文件，镜像体积小
- `ARG BASE_URL=/`：构建参数，Docker 部署用 `/`，GitHub Pages 用 `/xiaomi/`

## 第二步：编写 Nginx 配置

创建 `nginx.conf`：

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Vue Router history 模式支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    gzip_min_length 256;
}
```

说明：
- `try_files`：Vue Router 使用 history 模式时，所有路由都回退到 `index.html`
- 静态资源缓存 1 年，带 `immutable` 标记（Vite 构建的文件名带 hash，更新后文件名会变）
- 开启 gzip 压缩，减少传输体积

## 第三步：编写 docker-compose.yml

创建 `docker-compose.yml`：

```yaml
services:
  web:
    build:
      context: .
      args:
        BASE_URL: /
    ports:
      - "8080:80"
    restart: unless-stopped
```

说明：
- `context: .`：构建上下文为当前目录
- `args.BASE_URL: /`：传给 Dockerfile 的构建参数，Docker 部署用根路径
- `ports: "8080:80"`：宿主机 8080 端口映射到容器 80 端口
- `restart: unless-stopped`：容器意外退出时自动重启

## 第四步：编写 .dockerignore

创建 `.dockerignore`，排除不需要打包进镜像的文件：

```
node_modules
dist
.git
.github
.vscode
.claude
*.md
```

作用：减小构建上下文体积，加快构建速度。

## 第五步：构建并启动容器

```bash
docker compose up -d --build
```

这条命令做了以下事情：
1. 读取 `docker-compose.yml` 和 `Dockerfile`
2. 拉取 `node:20-alpine` 和 `nginx:alpine` 基础镜像
3. 在容器内执行 `npm ci` 安装依赖
4. 执行 `npm run build` 构建产物
5. 将构建产物复制到 Nginx 容器中
6. 启动 Nginx 服务
7. 映射 8080 端口到宿主机

## 第六步：验证部署

浏览器访问 http://localhost:8080 ，看到页面即部署成功。

## 常用命令

```bash
docker compose up -d --build   # 重新构建并启动
docker compose down             # 停止并删除容器
docker compose logs -f          # 查看实时日志
docker compose ps               # 查看容器状态
docker compose restart          # 重启容器
```

## 文件结构

```
xiaomi/
├── Dockerfile              # Docker 构建配置
├── nginx.conf              # Nginx 服务器配置
├── docker-compose.yml      # Docker Compose 编排配置
├── .dockerignore           # Docker 构建排除文件
├── vite.config.js          # Vite 配置（base 路径）
├── package.json
├── src/
│   ├── App.vue
│   ├── main.js
│   └── ...
└── public/
```
