# -------------------------------------------------------------------
# Stage 1: 前端构建 (使用 Node.js 构建 React 应用)
FROM node:20-alpine AS frontend-builder

# 设置工作目录为前端代码目录
WORKDIR /app/web

# 复制前端依赖文件
# 【路径调整】现在必须以 'backend/' 开头 (相对于根目录的 Context)
COPY backend/web/package.json .
COPY backend/web/package-lock.json .
RUN rm -f package-lock.json && rm -rf node_modules && npm install --omit=dev --legacy-peer-deps

# 安装依赖
RUN npm install

# 复制前端源代码
COPY backend/web/ .

# 执行前端构建，生成静态文件到 dist 目录
RUN npm run build


# -------------------------------------------------------------------
# Stage 2: 后端构建 (使用 Go 编译应用，嵌入前端资产)
FROM golang:1.24-alpine AS backend-builder

# 设置 CGO_ENABLED=0 进行静态编译
ENV CGO_ENABLED=0

# 设置 Go 工作目录
WORKDIR /app/api

# 复制 Go 模块文件
# 【路径调整】现在必须以 'backend/' 开头
COPY backend/api/go.mod .
COPY backend/api/go.sum .

# 下载 Go 依赖
RUN go mod download

# 复制后端 Go 源代码
COPY backend/api/ .

# **关键步骤：从前端构建阶段复制构建好的静态文件**
# 复制到 api 目录下的 web/dist
COPY --from=frontend-builder /app/web/dist ./dist

# 编译应用，输出名为 siyuan-share-api 的可执行文件
RUN go build -ldflags="-s -w" -o /go/bin/siyuan-share-api .


# -------------------------------------------------------------------
# Stage 3: 运行阶段 (创建最终的精简镜像)
FROM alpine:latest

# 安装 ca-certificates
RUN apk update && apk add --no-cache ca-certificates

# 设置工作目录
WORKDIR /app

# 从构建阶段复制编译好的二进制文件
COPY --from=backend-builder /go/bin/siyuan-share-api .

# 暴露服务端口 (8080)
EXPOSE 8080

# 容器启动命令
ENTRYPOINT ["./siyuan-share-api"]
