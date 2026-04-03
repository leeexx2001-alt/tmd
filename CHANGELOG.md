# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-04-04

### Added

#### Profile 下载功能 (Core Feature)
- 新增用户资料下载功能，支持下载完整个人资料
  - 头像 (avatar) - 高清版本 400x400
  - 横幅 (banner) - 个人主页横幅
  - 简介 (description) - 用户简介文本
  - 资料 (profile.json) - 完整资料 JSON 格式

- Profile 版本管理功能
  - 资料变更时自动备份历史版本
  - 版本文件命名格式: `{类型}_{日期}_{时间}.{扩展名}`

- 新增命令行参数:
  - `--profile` - 与推文下载配合，同时下载用户资料
  - `-profile-user` - 单独指定下载 profile 的用户
  - `-profile-list` - 单独指定下载 profile 的列表ID

#### 列表同步功能
- 新增 `internal/downloading/list_sync.go` - 列表同步模块
- 支持批量同步列表用户信息

#### 代码模块
- 新增 `internal/profile/` 模块
  - `downloader.go` - Profile 下载器 (558 行)
  - `fetcher.go` - Profile 获取器 (257 行)
  - `storage.go` - Profile 存储 (183 行)
  - `types.go` - 类型定义 (158 行)

### Changed

#### README 文档重写
- 重新组织文档结构，添加详细目录
- 新增功能特性说明
- 新增安装与配置指南
- 新增命令行参数详解（表格形式）
- 新增 Profile 下载功能说明
- 新增文件存储结构图示
- 新增 9 个使用场景与示例
- 新增高级设置说明
- 新增参数兼容性速查表
- 新增常见问题解答 (FAQ)
- 新增输出结果格式说明

#### 核心模块增强
- `internal/downloading/features.go` - 重构增强下载功能 (+485 行)
- `internal/downloading/entity.go` - 新增下载实体定义
- `internal/twitter/client.go` - Twitter 客户端增强
- `main.go` - 命令行参数扩展 (+340 行)

### Fixed

- 优化文件下载逻辑
- 改进错误处理机制

### Stats

- 23 files changed
- +4,554 lines / -240 lines

---

## [0.x.x] - Previous Versions

历史版本记录请参考 Git 提交历史:
```bash
git log --oneline
```
