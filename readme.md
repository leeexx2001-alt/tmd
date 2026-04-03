# TMD - Twitter Media Downloader

Twitter 推文媒体下载工具，支持下载图片、视频、GIF 等多媒体内容。

本项目的代码基于 [unkmonster/tmd](https://github.com/unkmonster/tmd) 项目，修改了部分代码，添加了新的功能特性。新增的功能见 [CHANGELOG.md文件](CHANGELOG.md)

## 功能特性

- 推文媒体下载（图片、视频、GIF）
- 用户推文批量下载
- 列表用户同步下载
- 用户关注列表同步
- **Profile 下载** - 下载用户头像、横幅、简介、完整资料
- **推文 JSON 保存** - 保存推文完整信息为 JSON/TXT 格式
- 多账号支持，自动负载均衡
- 并发下载，下载速度快
- 断点续传，节省时间和流量
- 磁盘空间检测，防止下载中断
- Windows 文件名自动清理

## 目录结构

```
├── main.go                 # 程序入口
├── go.mod                  # Go 模块依赖
├── go.sum
├── cmd/                    # 命令行参数定义
├── internal/               # 内部包
│   ├── database/           # 数据库操作
│   ├── downloading/        # 下载核心逻辑
│   ├── profile/            # Profile 下载模块
│   ├── twitter/            # Twitter API 封装
│   └── utils/              # 工具函数
├── ui/                     # UI 界面
└── README.md
```

## 安装

### 方式一：下载预编译版本

从 [GitHub Releases](https://github.com/leeexx2001-alt/tmd/releases) 下载对应平台的可执行文件。

### 方式二：从源码编译

```bash
git clone https://github.com/leeexx2001-alt/tmd.git
cd tmd
go build -o tmd.exe .
```

## 配置

首次运行会引导创建配置文件 `config.yaml`：

```yaml
# Twitter 账号配置（支持多账号）
twitter:
  accounts:
    - username: "your_username1"
      password: "your_password1"
      email: "your_email1@example.com"
      email_password: "your_email_password1"
    - username: "your_username2"
      password: "your_password2"
      email: "your_email2@example.com"
      email_password: "your_email_password2"

# 代理配置（可选）
proxy: "http://127.0.0.1:7890"

# 下载配置
download:
  dir: "./downloads"          # 下载目录
  max_file_name_len: 155      # 文件名最大长度 (50-250)
```

## 使用方法

### 基础命令

```bash
# 下载特定用户的推文媒体
./tmd.exe -user "username"

# 下载特定列表的媒体
./tmd.exe -list 123456

# 下载已关注用户的媒体
./tmd.exe -following

# 恢复未完成的下载
./tmd.exe -recover
```

### 命令行参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `-user` | string | - | 指定要下载的用户（可重复） |
| `-list` | uint64 | - | 指定要下载的列表 ID（可重复） |
| `-following` | bool | false | 下载已关注用户的媒体 |
| `-recover` | bool | false | 恢复之前未完成的下载 |
| `-profile` | bool | true | 下载用户 Profile（头像、横幅等） |
| `-noprofile` | bool | false | 跳过 Profile 下载 |
| `-profile-user` | string | - | 单独下载 Profile 的用户（可重复） |
| `-profile-list` | uint64 | - | 下载列表所有成员的 Profile（可重复） |
| `-mark-downloaded` | bool | false | 仅标记用户为已下载，不下载内容 |
| `-mark-time` | string | - | 标记时间戳，格式：2006-01-02T15:04:05 |
| `-auto-follow` | bool | false | 自动关注受保护的列表用户 |
| `-db` | string | - | 指定数据库文件路径 |
| `-max-file-name-len` | int | 155 | 文件名最大长度 (50-250) |
| `-config` | string | - | 配置文件路径 |

### Profile 下载功能

Profile 下载功能可以保存用户的完整资料信息：

**下载内容：**
- `avatar.jpg/png/gif/webp` - 高清头像（400x400）
- `banner.jpg/png/gif/webp` - 个人主页横幅
- `description.txt` - 用户简介
- `profile.json` - 完整资料 JSON

**版本管理：**
当用户更新头像或资料时，旧文件会自动备份到 `.versions/` 目录。

**存储结构：**
```
users/{UserName(screenName)}/.loongtweet/.profile/
├── avatar.jpg           # 当前头像
├── banner.jpg           # 当前横幅
├── description.txt      # 当前简介
├── profile.json         # 当前资料
└── .versions/          # 历史版本备份
    ├── avatar_20240115_103045.jpg
    └── profile_20240115_103045.json
```

**使用示例：**
```bash
# 下载用户推文时同时下载 Profile
./tmd.exe -user "username"

# 单独下载用户的 Profile（不下载推文媒体）
./tmd.exe -profile-user "username"

# 下载列表所有成员的 Profile
./tmd.exe -profile-list 123456

# 跳过 Profile 下载
./tmd.exe -user "username" -noprofile
```

### 推文 JSON 保存

每次下载推文媒体时，会同时保存推文的完整信息到 `.loongtweet/` 子目录：

**保存内容：**
- `{tweet_id}.json` - 推文完整信息的格式化 JSON
- `{tweet_id}.txt` - 人类可读的文本格式

**JSON 内容：**
- 推文文本、时间戳、URL
- 用户信息
- 媒体信息（已清理冗余字段）
- 完整的原始数据

**用途：**
- 即使下载失败也能记录推文信息
- 便于调试和分析
- 可用于数据备份和迁移

**存储结构：**
```
users/{UserName}/.loongtweet/
├── {tweet_id}.json    # 推文 JSON（完整数据）
└── {tweet_id}.txt     # 推文文本（人类可读）
```

### 标记已下载

使用 `-mark-downloaded` 可以将用户标记为已下载，不下载任何内容。这对于：
- 手动备份了推文，需要跳过已备份的用户
- 只想更新数据库中的下载状态

```bash
# 标记用户为已下载（使用当前时间）
./tmd.exe -user "username" -mark-downloaded

# 标记用户为已下载（使用指定时间）
./tmd.exe -user "username" -mark-downloaded -mark-time "2024-01-15T10:30:00"

# 标记用户为已下载（全量下载，下次运行会下载所有推文）
./tmd.exe -user "username" -mark-downloaded -mark-time "null"
```

### 多账号支持

程序支持配置多个 Twitter 账号，会自动进行负载均衡：

- 优先使用备用账号（非受保护用户）
- 受保护用户专用主账号
- 自动跳过有限制的客户端

### 并发控制

- `MaxDownloadRoutine`: 最大并发下载数（默认：CPU 核心数 × 10，最大 100）
- `userTweetMaxConcurrent`: 用户推文最大并发（默认 35）

## 文件存储结构

```
downloads/
├── users/                              # 用户推文
│   └── {UserName(screenName)}/
│       ├── tweet1.jpg
│       ├── tweet2.mp4
│       └── .loongtweet/               # 推文完整信息
│           ├── {tweet_id}.json
│           └── {tweet_id}.txt
│       └── .profile/                   # 用户 Profile
│           ├── avatar.jpg
│           ├── banner.jpg
│           ├── description.txt
│           ├── profile.json
│           └── .versions/              # 版本备份
└── lists/                              # 列表推文
    └── {ListName}/
        └── {UserName(screenName)}/     # 符号链接到 users/
```

## 高级设置

### 环境变量

```bash
# 代理设置（可选）
HTTPS_PROXY=http://127.0.0.1:7890
HTTP_PROXY=http://127.0.0.1:7890

# 日志级别
LOG_LEVEL=debug
```

### 配置文件

配置文件位于 `config.yaml`，支持以下高级选项：

```yaml
twitter:
  accounts:
    - username: "your_username"
      password: "your_password"
      email: "your_email@example.com"
      email_password: "your_email_password"

proxy: "http://127.0.0.1:7890"

download:
  dir: "./downloads"
  max_file_name_len: 155
  skip_unchanged: true        # 跳过未变化的文件
  enable_versioning: true     # 启用 Profile 版本管理
  avatar_quality: "400x400"  # 头像质量
```

## 常见问题

### Q: 下载失败，显示 "account locked"

A: 账号可能被锁定，尝试在浏览器中登录 Twitter 解锁后，重新运行程序。

### Q: 下载速度慢

A: 尝试减少并发数，或检查网络代理设置。

### Q: 文件名过长导致保存失败

A: 使用 `-max-file-name-len` 参数调整文件名最大长度：

```bash
./tmd.exe -user "username" -max-file-name-len 100
```

### Q: 如何完全重新下载某个用户？

A: 使用 `-mark-time "null"` 清除下载记录：

```bash
./tmd.exe -user "username" -mark-downloaded -mark-time "null"
./tmd.exe -user "username"
```

## 编译

```bash
# Windows
go build -ldflags="-s -w" -o tmd.exe .

# Linux
GOOS=linux GOARCH=amd64 go build -o tmd .

# macOS
GOOS=darwin GOARCH=amd64 go build -o tmd .
```

## 致谢

本项目基于 [unkmonster/tmd](https://github.com/unkmonster/tmd) 开发。

## License

MIT License
