# 写在前面
* 项目是为pt圈的朋友们便利应用所写，源码随docker发布
* 不欢迎商业应用
* 个人编程练习实践，有缘碰上时会维护一下
* 安全方面完全没底，在公网使用请自己多加小心

> 下面有请 gemini 为你介绍如何安装使用

# 一 项目 Docker 快速启动指南

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

#  二 设置与使用

## 概述
torll2 与 [tordb](https://github.com/ccf-2012/tordb) 配合使用，所有关于影视信息的查询（例如 TMDb 数据）都通过 tordb 服务完成。

1. 设置-TORCP 服务设置，设置 TorcpDB(即TorDB) URL 和 API Key，以使系统可与 TorDB 接上，并设定改名硬链的参数；
2. 下载-下载客户端，设置下载器；需要到 [rcp ](https://github.com/ccf-2012/rcp) 放在下载器所在机器；qBittorrent 下载完成运行程序指向此目录，详见后面说明；
3. 索引-站点设置-添加站点；
4. RSS-RSS源-添加FEED；
5. (optional) 设置-通知-Telegram Settings 以及 Enable Emby Notifications

> 至此，系统可以基本运转起来了


## 下载模块 
* 当前仅支持 qBittorrent
* 下载模块设计为支持多机，可配置多个 qBittorrent 下载器
* 为实现下载后自动整理，需在 qBittorrent 中配置“下载完成时运行外部程序”

1.  将项目中的 `rcp` 目录完整复制到下载机（例如 NAS）。
2.  修改 `rcp/rcp.sh` 脚本，确保 `cd` 和 `python` 命令的路径正确。
    -   **注意**: 很多设备的默认 Python 版本可能不满足 3.10+ 的要求。请确认并使用正确的 Python 解释器路径。

```sh
#!/usr/bin/bash
# 脚本所在的绝对路径
cd /your/path/to/rcp 
# 使用正确的 Python 解释器路径执行 rcp.py
/opt/bin/python rcp.py $1 -t $2 -u $3 -n $4 >> rcp.log 2>> rcp2e.log
```
3.  复制 `rcp/config.ini.template` 为 `config.ini`，修改其中的 `url` 和 `api_key`。
4.  在 qBittorrent 设置中填入命令：

```sh
sh /your/path/to/rcp/rcp.sh "%F" "%I" "%L" "%N"
```
#### 下载器中的远端映射路径
* 远端整理完成的硬链文件，要让运行Emby的主服务器访问到，比如可以通过本地网络 nfs mount 过来，或上传网盘rclone(等) mount过来，或者生成strm实现访问。
* 在下载器设置中，需配置 `Local Map Path`。此路径是 torll2 所在主机访问媒体文件的根目录，用于后续的文件管理（如删除、读取等）。在查找媒体文件时，由此路径与媒体库中存储的相对路径拼合，与此路径相关的有：
  1. 在编辑媒体信息时，需要操作媒体文件，使用拼合路径；
  2. 在媒体库删除一个条目时，会同步删除 qbit 中的种子信息，和媒体库中 硬链/mount/strm 的文件，会使用拼合路径；


## 索引模块
* 索引模块包含 速览、浏览、搜索及站点设置
* 站点设置中从预设站点中选择，配置自己的cookie 及 速览url，所配速览url是在定时刷新时取种子的页面
* 速览模块是查看已经缓存的本地数据库，多站聚合显示最新的种子
* 浏览模块现取各站种子列表，解析后以相似的形式展示，本地不缓存
* 搜索模块聚合搜索各站的种子，历史各次搜索可回溯浏览


## RSS 模块

* 在 RSS 添加页面中，设置Name，URL, LinkType, Interval, Tag, 是否下载
  * URL是从站点上生成的，尽量勾选类型、副标题、Size，标签
  * LinkType大部分内站都是 NexusPHP，另支持了少数外站
  * 是否下载，意思是Filter通过的种子，是只入库，还是发起下载，打勾就会下载
  * Tag以后使用
* filter部分，是通过 JSON 格式配置的
**配置示例:**
```json
{
  "filters": [
    {
      "tag": "中字剧集",
      "title_not_regex": "x264|720p",
      "subtitle_not_regex": "第\\d+.*集",
      "size_gb_min": 2,
      "size_gb_max": 15
    }
  ]
}
```
**可用过滤器字段:**
-   `title_regex`, `title_not_regex`: 对种子**标题**进行正则匹配或排除。
-   `subtitle_regex`, `subtitle_not_regex`: 对种子**副标题**进行正则匹配或排除。
-   `rsstags_regex`, `rsstags_not_regex`: 对站点的**种子标签** (Tags) 进行正则匹配或排除。
-   `rsscat_regex`, `rsscat_not_regex`: 对站点的**种子分类** (Category) 进行正则匹配或排除。
-   `size_gb_min`, `size_gb_max`: 对种子**大小**设置 GB 单位的上下限。
-   `rate_min`: 对 IMDB / Douban **评分**设置最小值要求。

### 外站 rss / 非 nexusphp 站点
* 非 nexusphp 站点 rss 根据其 rss 信息结构有以下几类：
  1.  **hdbits**: "link" 是下载链接，"title" 干净，"guid" 存种子数字id, "description"中可能有 imdb
  2.  **passthepopcorn**: "link" 是下载链接，"title" 需要解析, "comments" 中为info link, "description" 中可能有 imdb
  3.  **broadcasthe**: "link" 是下载链接，"title" 需要解析, "guid" 中为info link, "description" 中可能有 imdb
  4.  **blutopia**: "link" 是下载链接，"title" 干净，"guid" 存种子数字id, 有"contentlength", 有"category", "description"中可能有 imdb, tmdb
  5.  **filelist**: "link" 是下载链接，"title" 为标题+标签，"description" 中可能有 imdb，可能有 size，category


## 媒体库模块
* 媒体库显示受管理的媒体条目，包括其识别刮削后的TMDb信息、下载器及其中的hash，在本地映射的路径
* 如果识别不对，可在此进行手工修正
* 删除时可选仅删数据库记录、下载器中种子、改名硬链后的映射