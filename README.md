# 写在前面
* 项目是为pt圈的朋友们便利应用所写，源码随docker释出
* 不欢迎商业应用
* 安全方面，在公网使用请自己多加小心

## latest update:
* 20250919: arm64, amd64 架构 buildx
* previous: 基本成型


> 下面有请 gemini 为你介绍如何安装使用

# 一、 项目 Docker 快速启动指南

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
    - `TORDB_API_KEY=some_api_key`: 设置一个自己和torll2访问 TORDB 时需要的密码(API Key)
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


## 其他常用命令

- **查看所有服务日志**: `docker compose logs -f`
- **停止并移除容器**: `docker compose down`
- **仅停止服务**: `docker compose stop`
- **仅启动服务**: `docker compose start`


---

# 二、设置与使用

本部分将指导你在 Docker 服务成功启动后，如何配置 `torll2` 的各项功能，使其成为一个全自动的媒体管理系统。

## 核心概念

-   **torll2**: 主应用，负责任务调度、RSS解析、连接下载器和媒体库管理。
-   **tordb**: 辅助服务，提供电影、剧集等元数据信息。`torll2` 通过查询它来获取媒体信息。
-   **qBittorrent**: 下载客户端。`torll2` 会将下载任务发送给它。
-   **[rcp 脚本](https://github.com/ccf-2012/rcp)**: 一个在下载机上运行的“信使”。当 qBittorrent 下载完成后，会调用此脚本，由它向 `torll2` 请求数据，然后执行后续的整理（重命名、硬链接等）操作。
-   **[torfilter 油猴脚本](https://greasyfork.org/zh-CN/scripts/451748)**: 在站点网页上发起过滤、查重和下载的脚本。
---

## 配置步骤

请按照以下顺序，在 `torll2` 的 Web UI ([http://localhost:6006](http://localhost:6006)) 中进行配置。

### 步骤 1: 连接 Torll2 与 Tordb

这是让系统能够识别媒体信息的关键一步。

1.  在 `torll2` 界面中，导航至 **设置** -> **TORCP 服务设置**。
2.  填写以下信息：
    -   **TorDB URL**: `http://tordb:6009`
        > **说明**: `tordb` 是 Docker 网络内部的服务名，`torll2` 通过这个地址访问 `tordb` 服务。
    -   **TorDB API Key**: 填写你在 `.env` 文件中为 `TORDB_API_KEY` 设置的值。
3.  点击保存。

### 步骤 2: 配置下载器

1.  导航至 **下载** -> **下载客户端**。
2.  点击 **添加下载器**，并填入你的 qBittorrent 客户端信息（WebUI 地址、用户名、密码）。
3.  这里有一个远端映射路径 `Local Map Path` 此路径是 torll2 所在主机访问媒体文件的根目录，用于后续的文件管理（如删除、读取等）。在查找媒体文件时，是由此路径与媒体库中存储的相对路径拼合而成的。比如可以通过本地网络 nfs mount 过来，或上传网盘后rclone(等) mount过来，或者生成 strm 实现访问。

### 步骤 3: 在下载器所在机器上配置 rcp 脚本

这一步是为了实现下载完成后，与 `torll2` 通信获取信息后，按要求对文件进行重命名和分类。

1.  **下载脚本**: 从 [rcp 脚本仓库](https://github.com/ccf-2012/rcp) 下载到你**运行 qBittorrent 的机器**上（例如，你的 NAS）。
2.  **修改 `rcp.sh`**:
    -   用文本编辑器打开 `rcp` 目录下的 `rcp.sh` 文件。
    -   修改 `cd` 后面的路径，使其指向 `rcp` 目录在你下载机上的**绝对路径**。
    -   确认执行 `rcp.py` 的 `python` 命令路径是否正确。很多设备的默认 Python 版本较低，请确保使用 Python 3.10+ 的解释器。

    ```sh
    #!/usr/bin/bash
    # 脚本所在的绝对路径
    cd /path/to/your/rcp 
    # 使用正确的 Python 解释器路径执行 rcp.py
    /opt/bin/python rcp.py $1 -t $2 -u $3 -n $4 >> rcp.log 2>> rcp2e.log
    ```

3.  **修改 `config.ini`**:
    -   在 `rcp` 目录中，将 `config.ini.template` 复制为 `config.ini`。
    -   修改 `config.ini` 文件，填入 `torll2` 的 `url` 和 `api_key`。
        -   `url`: `torll2` 服务的地址，例如 `http://192.168.1.100:6006`。
        -   `api_key`: `torll2` 自动生成的 API Key（请从 `docker compose logs torll2` 日志中获取）。

4.  **配置 qBittorrent**:
    -   在 qBittorrent 的 **设置** -> **下载** -> **“Torrent 完成时运行外部程序”** 中，填入以下命令（请使用 `rcp.sh` 的绝对路径）：

    ```sh
    sh /path/to/your/rcp/rcp.sh "%F" "%I" "%L" "%N"
    ```

### 步骤 4: 添加索引站点

1.  导航至 **索引** -> **站点设置**。
2.  点击 **添加站点**，从预设列表中选择你的 PT 站点，并配置:
  * 你的站点 `Cookie` 
  * `速览URL` - 这是用于浏览站点时的起始 url，在点选过滤按钮时会基于此 url 进行拼接

### 步骤 5: 添加 RSS 订阅

1.  导航至 **RSS** -> **RSS源**。
2.  点击 **添加FEED**，填入从站点获取的 RSS 订阅链接。
3.  根据需要配置 **Filter** (过滤器)，可以配置多个过滤规则，只有全部 filter 通过才收录，通过 JSON 格式的规则实现精准下载。过滤规则示例见下：

    **过滤器示例:**
    ```json
    [
        {
          "tag": "中字剧集",
          "title_not_regex": "x264|720p",
          "subtitle_not_regex": "第\d+.*集",
          "size_gb_min": 2,
          "size_gb_max": 15
        }
    ]
    ```
    **可用过滤器字段:**
-   `title_regex`, `title_not_regex`: 对种子**标题**进行正则匹配或排除。
-   `subtitle_regex`, `subtitle_not_regex`: 对种子**副标题**进行正则匹配或排除。
-   `rsstags_regex`, `rsstags_not_regex`: 对站点的**种子标签** (Tags) 进行正则匹配或排除。
-   `rsscat_regex`, `rsscat_not_regex`: 对站点的**种子分类** (Category) 进行正则匹配或排除。
-   `size_gb_min`, `size_gb_max`: 对种子**大小**设置 GB 单位的上下限。
-   `rate_min`: 对 IMDB / Douban **评分**设置最小值要求。


### 步骤 6 (可选): 配置通知服务

在 **设置** -> **通知** 中，你可以根据需要配置 Telegram 或 Emby 通知，以便在下载完成或出错时收到提醒。

### 步骤 7 (可选): 安装油猴脚本

详见 [torfilter 油猴脚本](https://greasyfork.org/zh-CN/scripts/451748)

---

至此，系统已基本配置完毕，可以开始全自动工作了。

# 三、 如何更新

为了确保你的应用保持最新，请按照以下步骤操作。

1.  **拉取最新的代码和配置**
    在你的项目目录中（即 `docker-compose.yml` 文件所在的目录），运行 `git` 命令来获取最新的更新。

    ```bash
    git pull
    ```

2.  **拉取最新的 Docker 镜像**
    这个命令会从 Docker Hub 或其他镜像仓库拉取 `docker-compose.yml` 文件中定义的所有服务的最新版本。

    ```bash
    docker compose pull
    ```

3.  **使用新镜像重新启动服务**
    以下命令会停止当前正在运行的服务，并使用刚刚拉取的最新镜像来重新创建和启动它们。`--build` 标志会确保在需要时重新构建本地镜像。

    ```bash
    docker compose up --build -d
    ```

完成以上步骤后，你的服务就会以最新版本运行。