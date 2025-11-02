# 写在前面
* 项目是为pt圈的朋友们便利应用所写，源码随docker释出
* 不欢迎商业应用
* 安全方面，在公网使用请自己多加小心

# 文档
* 以Github Pages托管，请访问：[安装与使用文档](https://ccf-2012.github.io/torll2-doc/)


## latest update:
* 2025-11-2:  安全性审查加固，所有端点作认证；远程机器删除硬链文件；媒体标记(正在看，不想看，已看完...) ；根据标记清理空间；一些默认值，方便 Docker 使用者；
* 2025-10-28: 下载模块改为单例后台排队，解决卡死以及竞态等问题；数据库少量修改，更新需要删除 mysql_data volume 或者手工进后台 `alembic upgrade head`；
* 2025-10-26: 搜索功能：支持 Prowlarr；刮削：local, agent后台刷新机制大量修正；
* 2025-10-08: 远程 rcp_agent 实现远程下载器中的种子改名硬链、修改、重建；
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
    - `TORLL2_API_KEY` 设置 `torll2` 的 API KEY，将给 rcp, torfilter 使用。
    - `MYSQL_HOST`: 使用这里的 docker-compose.yml 建的话，设为 `mysql` 。
    - `TORDB_API_KEY`: 设置一个自己和torll2访问 TORDB 时需要的密码(API Key)
    - `TORDB_TMDB_API_KEY`: 填入你的 The Movie Database (TMDB) 的 API Key。你可以从 [TMDB 官网](https://www.themoviedb.org/settings/api) 免费申请。

3. 修改 `docker-compose.yml` 中的一行，将 Emby Media 的路径 mount 给 Docker 内
```yml
services:
  torll2:
  #...  
    volumes:
      - <your host emby path>:/media  # <-- 修改这里将 /media 指向你的宿主机上的 emby 硬链生成位置
```

## 步骤 2: 启动服务

在项目根目录（即 `docker-compose.yml` 所在的目录）打开终端，运行以下命令：

```bash
# 该命令会自动构建镜像并在后台启动所有服务
docker compose up --build -d
```

首次启动会需要一些时间来下载和构建镜像。完成后，服务将在后台运行。

## 步骤 3: 获取 torll2 的 API Key

上面 `TORLL2_API_KEY` 设置 `torll2` 的 API KEY，将给 rcp, torfilter 使用。
如果没有设，则`torll2` 服务在首次启动时会自动为你生成一个 API Key。你需要通过查看容器日志来获取它。
请**复制并妥善保管**这个 API Key，你将在访问 `torll2` 的 API 时用到它。

## 步骤 4: 访问应用

现在，你可以通过浏览器访问你的应用了：

- **torll2**: http://<your server ip >:6006
  - 使用你在 `.env` 文件中设置的 `TORLL2_ADMIN_USER` 和 `TORLL2_ADMIN_PASSWORD` 登录。

- **tordb**: http://<your server ip>:6009


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
4.  处理模式，有 3 种，分别为 local, agent, legacy:
    1.  由 torll2 直接控制本地选 local，这通常需要 torll2 直接运行，在 Docker 中运行使用此模式较麻烦；
    2.  torll2 在 Docker 中直接控制下载器，或者下载器在远程，选 agent
    3.  由 qbittorrent 完成后调用脚本发起硬链，选 legacy，这个模式对于远端下载器或Docker外下载器，无法修改和删除硬链
    4.  详见 [下载器处理模式](https://ccf-2012.github.io/torll2-doc/features/downloader-modes/)



### 步骤 3: 在下载器所在机器上配置  agent 
参见：[下载器处理模式](https://ccf-2012.github.io/torll2-doc/features/downloader-modes/)
如果是 Docker 部署的，则以 agent 模式控制较简单，在下载器所在机器上：
1. 下载 rcp
```sh
git clone https://github.com/ccf-2012/rcp
cd rcp
```

2. 编辑一个 config.ini, 内容为：
```ini
[torll]
# torll2服务的URL地址，地址按自己的设，后面路径不动
url = http://<your.torll2.host:6006>/api/v1/torcp/info
# torll2服务的API Key，由torll2 启动时得到
api_key = <api key get from torll2>
# qbit 的名字，与在 torll2 中配置的下载器名字对上
qbitname = <qb name set in torll2>

[emby]
# Emby/Jellyfin媒体库的根目录
root_path = <your media path >
```

3. 启动 `rcp_agent`

```sh
# 启一个 screen 
python rcp_agent.py
```


### 步骤 4: 添加索引站点

1.  导航至 **索引** -> **站点设置**。
2.  点击 **添加站点**，从预设列表中选择你的 PT 站点，并配置:
  * 你的站点 `Cookie` 
  * `速览URL` - 这是用于浏览站点时的起始 url，在点选过滤按钮时会基于此 url 进行拼接

### 步骤 5: 添加 RSS 订阅

1.  导航至 **RSS** -> **RSS源**。
2.  点击 **添加FEED**，填入从站点获取的 RSS 订阅链接。
3.  过滤规则请参阅文末的 **过滤器规则说明** 章节。


### 步骤 6 (可选): 配置通知服务

在 **设置** -> **通知** 中，你可以根据需要配置 Telegram 或 Emby 通知，以便在下载完成或出错时收到提醒。

### 步骤 7 (可选): 安装油猴脚本

详见 [torfilter 油猴脚本](https://greasyfork.org/zh-CN/scripts/451748)

---

至此，系统已基本配置完毕，可以开始全自动工作了。

## 过滤器规则说明 (Filter Rules)

系统中所有的过滤功能（包括 RSS 订阅和追剧）都使用一套统一的、基于 JSON 的规则引擎。这套引擎非常灵活，可以通过组合不同的规则来实现精确的资源筛选。

### 规则基本结构

一个完整的过滤器由一个 `mode` 和一个 `rules` 列表组成。

```json
{
  "mode": "all",
  "rules": [
    { "field": "title", "operator": "contains", "value": "1080p" },
    { "field": "size_gb", "operator": "gt", "value": 5 }
  ]
}
```

*   `mode`: 定义 `rules` 列表中多条规则的组合方式。
    *   `"all"`: **与 (AND)** 逻辑，所有规则都必须匹配。
    *   `"any"`: **或 (OR)** 逻辑，只需匹配任意一条规则即可。
*   `rules`: 一个规则对象的列表，每个对象定义了一条独立的匹配逻辑。

### 规则对象

每个规则对象包含三个部分：`field`, `operator`, 和 `value`。

```json
{ "field": "...", "operator": "...", "value": "..." }
```

#### `field` (字段)

指定要对项目的哪个信息进行匹配。

*   **可用字段**:
    *   `title`: 种子主标题
    *   `subtitle`: 种子副标题
    *   `extitle`: (来自tordb的)扩展标题
    *   `media_title`: (来自tordb的)媒体标题
    *   `site`: 站点标识 (例如 `hdsky`)
    *   `size_gb`: 种子大小 (单位: GB)
    *   `tags`: 种子的标签 (字符串)
    *   `category`: 种子的分类 (字符串)
*   **多字段匹配**: `field` 的值可以是一个列表，表示对列表中的多个字段进行匹配，满足**任意一个**即可。
    *   `"field": ["title", "subtitle"]`

#### `operator` (操作符)

定义如何进行比较。

| 操作符 | 说明 | 示例 `value` |
| :--- | :--- | :--- |
| `contains` | 包含 (不区分大小写) | `"keyword"` |
| `not_contains` | 不包含 (不区分大小写) | `"keyword"` |
| `regex` | 正则表达式匹配 | `"^Movie.*2023$"` |
| `not_regex` | 正则表达式不匹配 | `"S\d+E\d+"` |
| `gt` | 大于 (Greater Than) | `10` |
| `lt` | 小于 (Less Than) | `50` |
| `eq` | 等于 (Equal to) | `25` |
| `in` | 存在于列表中 | `["siteA", "siteB"]` |
| `not_in` | 不存在于列表中 | `["siteC", "siteD"]` |

#### `value` (值)

用于与字段内容进行比较的值，其类型需与操作符的要求相匹配。

### 应用示例

#### 示例 1: RSS 过滤器

需求：匹配所有**标题**包含 `1080p` 但不包含 `x265`，且**大小**在 5GB 到 20GB 之间的种子。

```json
{
  "mode": "all",
  "rules": [
    { "field": "title", "operator": "contains", "value": "1080p" },
    { "field": "title", "operator": "not_contains", "value": "x265" },
    { "field": "size_gb", "operator": "gt", "value": 5 },
    { "field": "size_gb", "operator": "lt", "value": 20 }
  ]
}
```
*注意：对于 RSS，其配置本身是一个过滤器列表，列表中的过滤器是 "OR" 关系。上面的示例是列表中的一个元素。*

#### 示例 1.5: 旧格式转换示例 (Legacy Format Conversion)

为了帮助理解，这里是一个旧版 RSS 过滤器格式转换为新格式的直接示例。

*   **旧格式**:
    ```json
    [
      {
        "size_gb_max": 55,
        "title_not_regex": "S\\d+E\\d+|720p",
        "subtitle_not_regex": "第\\d+\\s*集"
      }
    ]
    ```

*   **等效的新格式**:
    ```json
    [
      {
        "mode": "all",
        "rules": [
          { "field": "size_gb", "operator": "lt", "value": 55 },
          { "field": "title", "operator": "not_regex", "value": "S\\\\d+E\\\\d+|720p" },
          { "field": "subtitle", "operator": "not_regex", "value": "第\\\\d+\\s*集" }
        ]
      }
    ]
    ```


#### 示例 2: 追剧过滤器

需求：订阅美剧《人生切割术》(Severance)，只想要 `HDSky` 或 `PterClub` 站的 `2160p` 资源，但不要 `REMUX` 版本。

在追剧模块中，**订阅名称本身会作为一个正则匹配规则自动加入**。我们只需要配置额外的规则即可。

*   **订阅名称 (Name)**: `Severance`
*   **规则 (Rules) JSON**:
```json
{
  "mode": "all",
  "rules": [
    { "field": "site", "operator": "in", "value": ["hdsky", "pterclub"] },
    { "field": "title", "operator": "contains", "value": "2160p" },
    { "field": "title", "operator": "not_contains", "value": "REMUX" }
  ]
}
```
系统在匹配时，会自动将订阅名 `Severance` 也作为一个匹配条件（`"field": ["title", "extitle", "media_title"], "operator": "regex", "value": "severance"`）加入到 `rules` 列表中。




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



