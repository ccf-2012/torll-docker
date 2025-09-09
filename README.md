# 项目 Docker 快速启动指南

欢迎使用！本指南将帮助你通过 Docker 快速启动 `torll2` 和 `tordb` 服务。

## 步骤 1: 准备配置文件

1.  将项目中的 `.env.example` 文件复制一份，并重命名为 `.env`。

    ```bash
    cp .env.example .env
    ```

2.  打开 `.env` 文件，根据你的需要修改以下变量：

    - `MYSQL_ROOT_PASSWORD`: 为数据库设置一个**强密码**。
    - `TORLL2_ADMIN_USER`: 设置 `torll2` 的初始**管理员用户名**。
    - `TORLL2_ADMIN_PASSWORD`: 设置 `torll2` 的初始**管理员密码**。
    - `TORDB_TMDB_API_KEY`: 填入你的 The Movie Database (TMDB) 的 API Key。你可以从 [TMDB 官网](https://www.themoviedb.org/settings/api) 免费申请。

## 步骤 2: 启动服务

在项目根目录（即 `docker-compose.yml` 所在的目录）打开终端，运行以下命令：

```bash
# 该命令会自动构建镜像并在后台启动所有服务
docker compose up --build -d
```

首次启动会需要一些时间来下载和构建镜像。完成后，服务将在后台运行。

## 步骤 3: 获取 torll2 的 API Key

`torll2` 服务在首次启动时会自动为你生成一个 API Key。你需要通过查看容器日志来获取它。

运行以下命令：

```bash
docker compose logs torll2
```

在日志输出中，你应该能找到类似下面的一行信息：

```
INFO:     Generated API Key: [xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx]
```

请**复制并妥善保管**这个 API Key，你将在访问 `torll2` 的 API 时用到它。

## 步骤 4: 访问应用

现在，你可以通过浏览器访问你的应用了：

- **torll2**: [http://localhost:6006](http://localhost:6006)
  - 使用你在 `.env` 文件中设置的 `TORLL2_ADMIN_USER` 和 `TORLL2_ADMIN_PASSWORD` 登录。

- **tordb**: [http://localhost:6009](http://localhost:6009)

---

## 其他常用命令

- **查看所有服务日志**: `docker-compose logs -f`
- **停止并移除容器**: `docker-compose down`
- **仅停止服务**: `docker-compose stop`
- **仅启动服务**: `docker-compose start`