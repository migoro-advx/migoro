# 見頃（Migoro）

**見頃** 是一个「花叶实况」地图社区：拍照 AI 识花、发布带地理位置的赏花实况，并在地图上通过 30 天时间拨盘浏览各地的开花动态。

> 「見頃」（migoro）意为"正是观赏的好时节"。产品全站仅支持简体中文，用户界面统一展示汉字「見頃」。

## 🌸 核心体验

- **拍照识花**：上传照片调用 StepFun 视觉大模型（`step-1o-turbo-vision`）识别花卉，并自动匹配平台花品字典（标准名/学名/别名/典型花期/是否在季）
- **一键发帖**：识花后即可发布带地理位置与花期状态的赏花实况帖子，照片存储于腾讯云 COS
- **地图浏览**：MapTiler 地图按可视范围（Web Mercator bbox）加载各地实况，底部半圆时间拨盘支持拖拽/滚轮回溯最近 30 天的开花动态
- **iMessage 花卉助手**：在 iMessage 里像和朋友聊天一样问花——"最近有什么花在开、附近能看什么花、这是什么花"，还支持直接发照片识花、发位置找花

## 🏗 系统架构

```
 iMessage 用户                    浏览器 / PWA
      ↕ Photon Spectrum               ↕
┌───────────────┐            ┌───────────────────┐
│ agent          │            │ web                │
│ Node.js Agent  │            │ TanStack Start     │
│ LLM 函数调用循环 │            │ (Cloudflare Workers)│
└───────┬───────┘            └─────────┬─────────┘
        │        HTTP (公开 API)        │
        └──────────────┬───────────────┘
                       ↓
              ┌─────────────────┐
              │ backend          │
              │ Spring Boot 3    │
              └───┬───┬───┬───┬─┘
                  ↓   ↓   ↓   ↓
          PostgreSQL COS StepFun Clerk
          (Flyway)  (图片) (识花)  (鉴权)
```

## 📦 仓库结构

本仓库为 monorepo 超级仓库（superproject），通过 git submodule 管理三个子模块，各模块的完整文档见各自 README：

| 模块 | 说明 | 技术栈 |
|------|------|--------|
| [`web/`](web/README.md) | 前端（PWA） | TanStack Start · React 19 · Tailwind v4 · MapTiler SDK · Clerk · Cloudflare Workers |
| [`backend/`](backend/README.md) | 后端服务 | Java 17 · Spring Boot 3.3.5 · MyBatis-Plus · PostgreSQL/Flyway · 腾讯云 COS · StepFun |
| [`agent/`](agent/README.md) | iMessage AI Agent | Node.js · Photon Spectrum · LLM function calling |

## 🚀 快速开始

### 克隆仓库

```bash
git clone --recurse-submodules git@github.com:migoro-advx/migoro.git

# 已克隆但缺少子模块时
git submodule update --init
```

### 启动各模块

环境变量与前置条件（数据库、COS、Clerk、StepFun 等）的配置细节见各子模块 README，此处仅列启动命令：

| 模块 | 命令 | 地址 |
|------|------|------|
| backend | `mvn spring-boot:run` | `http://localhost:8080` |
| web | `pnpm install && pnpm dev` | `http://localhost:3000` |
| agent | `npm install && npm start` | —（连接 iMessage 线路） |

## 🔗 跨端硬约定

三个模块必须共同遵守以下契约：

- **雪花 ID 一律字符串传输**（请求与响应），防止 JS `Number` 精度丢失，全链路禁止数值转换
- **坐标统一 Web Mercator（EPSG:3857，米）**：后端不做坐标转换，前端与 agent 各自完成 WGS84 转换
- **帖子软删除**：状态 `PUBLISHED / HIDDEN / DELETED`，公开接口仅返回 `PUBLISHED`
- **仅支持简体中文（zh-CN）**：无多语言计划，所有用户可见文案为简体中文

## 🔧 子模块维护

子模块更新流程：进入子模块目录拉取最新提交，再回到超级仓库提交指针变更。

```bash
cd backend && git pull origin main && cd ..
git add backend
git commit -m "chore: bump backend submodule"
```

> 注意：`web`、`backend` 跟踪 `main` 分支，`agent` 跟踪 `master` 分支（见 [`.gitmodules`](.gitmodules)）。
