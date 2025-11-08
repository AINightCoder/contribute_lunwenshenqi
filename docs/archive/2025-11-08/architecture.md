# 论文神器 (lunwenshenqi) - 项目架构文档

## 1. 项目概述

### 1.1 项目简介

**论文神器**是一个基于AI的结构化长文生成工具，能够帮助用户快速生成高质量的长篇文章，包括学术论文、新闻报道、叙述性文章等多种类型。

### 1.2 技术栈

- **前端框架**: Vue 3.2.13 (Composition API)
- **UI组件库**: Element Plus 2.7.8
- **Markdown编辑器**: @kangc/v-md-editor
- **HTTP客户端**: Axios 1.4.0
- **AI服务**: OpenAI兼容API (支持多模型)
- **部署方案**: Docker + Nginx / Vercel Serverless

### 1.3 核心特性

- ✅ 结构化生成：先生成提纲，再逐章节编写
- ✅ 多种类型：支持32种文章类型、46种风格特点
- ✅ 多模型支持：GPT-3.5/4、Gemini、Claude等
- ✅ 配图功能：自动调用DALL-E生成章节插图
- ✅ 历史管理：本地保存创作历史，可随时恢复
- ✅ 在线编辑：集成markdown-AI-editor

## 2. 系统架构

### 2.1 目录结构

```
lunwenshenqi/
├── api/                    # API代理层（Vercel部署）
│   └── proxy.js           # 反向代理配置
├── docs/                   # 项目文档
├── public/                 # 静态资源
├── src/                    # 源代码目录
│   ├── api/               # API接口层
│   │   ├── sse.js        # SSE流式对话接口
│   │   ├── public.js     # 公共接口
│   │   ├── model.js      # 模型相关
│   │   └── logs.js       # 日志相关
│   ├── components/        # 公共组件
│   │   ├── PageHeader.vue           # 页面头部
│   │   ├── DateInfo.vue            # 日期信息
│   │   └── ScrollToBottomButton.vue # 滚动按钮
│   ├── router/            # 路由配置
│   │   └── index.js
│   ├── utils/             # 工具类
│   │   ├── token.js      # Token管理
│   │   ├── request.js    # HTTP请求封装
│   │   ├── base.js       # 基础配置
│   │   ├── chat_transfer.js # 对话消息处理
│   │   ├── page.js       # 页面工具
│   │   ├── browser_db.js # 浏览器存储
│   │   ├── prompts.js    # 提示词模板
│   │   ├── models.js     # 模型配置
│   │   └── ...           # 其他工具
│   ├── views/             # 页面组件
│   │   └── chat/
│   │       └── ArticleGenerator.vue  # 核心业务组件
│   ├── App.vue            # 根组件
│   └── main.js            # 入口文件
├── .env                    # 环境变量
├── dockerfile             # Docker构建文件
├── docker-entrypoint.sh   # Docker入口脚本
├── nginx.conf.template    # Nginx配置模板
├── vue.config.js          # Vue CLI配置
└── package.json           # 项目依赖
```

### 2.2 分层架构

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│     (Vue Components / Views)        │
│  - ArticleGenerator.vue             │
│  - PageHeader.vue                   │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Business Logic Layer        │
│        (Utils / Composables)        │
│  - chat_transfer.js                 │
│  - prompts.js                       │
│  - models.js                        │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Data Access Layer           │
│           (API Services)            │
│  - sse.js (SSE流式对话)             │
│  - public.js (公共接口)             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│         Infrastructure Layer        │
│    (Request / Storage / Config)     │
│  - request.js (Axios封装)           │
│  - browser_db.js (localStorage)     │
│  - token.js (认证管理)              │
└─────────────────────────────────────┘
```

## 3. 核心功能模块

### 3.1 文章生成器 (ArticleGenerator.vue)

核心业务组件，实现三步骤论文生成流程：

#### 步骤1: 写作设定
- 文章类型选择（32种类型）
- 风格特点选择（46种风格）
- 篇幅设置（主节点数、子节点数）
- 标题/提示词输入
- 模型选择（GPT-3.5/4、Gemini、Claude等）
- 创作历史加载

#### 步骤2: 段落撰写
- AI自动生成提纲（JSON格式）
- 支持全部编写或单独编写每个章节
- 支持为章节生成配图（gpt-4-dalle）
- 实时显示编写进度
- 支持中断操作

#### 步骤3: 预览编辑
- Markdown预览
- 复制到剪贴板
- 新窗口打开在线编辑器

### 3.2 API接口层

#### sse.js - SSE流式对话接口
```javascript
// 核心接口
makeSingleChat(data, auth, callback)      // 普通对话请求
makeTextFileLineIterator(response)        // SSE流式响应处理
makeSpeechToText(data, auth)             // 语音转文字
makeTextToSpeech(data, auth)             // 文字转语音
```

#### public.js - 公共接口
```javascript
getContent(id)                           // 获取公共内容
getArticleKey()                          // 获取文章编辑Key
updateArticle(title, content, key)       // 更新文章内容
```

### 3.3 工具类模块

#### chat_transfer.js - 对话消息处理
- `initBaseMessage()`: 初始化消息对象
- `fixMessages()`: 修复消息角色顺序（确保user/assistant交替）

#### prompts.js - 提示词模板
提供各种预设的提示词模板，用于引导AI生成内容。

#### models.js - 模型配置
管理支持的AI模型列表和配置。

## 4. 数据流设计

### 4.1 用户创作流程

```
用户输入参数
   ↓
构建系统提示词
   ↓
调用AI生成提纲（JSON）
   ↓
解析提纲结构
   ↓
逐章节调用AI生成内容
   ↓
实时流式显示
   ↓
保存到本地存储
   ↓
Markdown预览/导出
```

### 4.2 消息格式规范

AI对话消息遵循OpenAI格式：
```json
{
  "role": "user|assistant|system",
  "content": "消息内容"
}
```

提纲生成采用JSON格式：
```json
{
  "title": "文章标题",
  "chapters": [
    {
      "title": "章节标题",
      "sections": ["小节1", "小节2"]
    }
  ]
}
```

## 5. 部署方案

### 5.1 Docker部署

使用Docker容器化部署，通过环境变量动态配置：

**关键环境变量**:
- `proxy_url`: AI服务地址
- `proxy_key`: AI服务密钥
- `editor_url`: 编辑器服务地址
- `ai_models`: 支持的AI模型列表

**部署流程**:
```bash
docker build -t lunwenshenqi .
docker run -p 80:80 \
  -e proxy_url=https://api.openai.com \
  -e proxy_key=sk-xxx \
  -e editor_url=https://suishouji.qiangtu.com \
  lunwenshenqi
```

### 5.2 Vercel部署

使用Vercel Serverless部署，通过api/proxy.js实现反向代理。

### 5.3 宝塔面板部署

支持传统的宝塔面板部署方式。

## 6. 安全设计

### 6.1 认证机制

- Token认证（Bearer Token）
- 支持共享Token和自定义Token

### 6.2 代理转发

所有AI API请求通过后端代理转发，避免前端暴露API密钥。

## 7. 性能优化

### 7.1 流式响应

使用SSE（Server-Sent Events）实现流式响应，提升用户体验：
- 内容逐字显示
- 降低等待时间
- 支持中断操作

### 7.2 本地缓存

使用localStorage缓存：
- 创作历史记录
- 用户配置参数
- Token信息

### 7.3 按需生成

支持全部生成和单独生成两种模式：
- 全部生成：批量调用AI完成所有章节
- 单独生成：仅生成指定章节，节省Token

## 8. 扩展性设计

### 8.1 模型扩展

通过配置文件轻松添加新的AI模型：
```javascript
// src/utils/models.js
export const models = [
  { name: 'gpt-3.5-turbo-16k', ... },
  { name: 'gpt-4', ... },
  // 添加新模型
]
```

### 8.2 文章类型扩展

通过配置文件添加新的文章类型和风格：
```javascript
// 在 ArticleGenerator.vue 中
articleTypes: ['学术论文', '新闻报道', ...],
styleFeatures: ['严肃严谨', '引经据典', ...]
```

### 8.3 提示词模板扩展

通过prompts.js添加新的提示词模板。

## 9. 依赖的外部服务

### 9.1 AI服务

- **服务**: OpenAI兼容的API服务（Two-API中转）
- **模型**: GPT-3.5/4、Gemini Pro、Claude 3等
- **功能**: 文本生成、图片生成（DALL-E）

### 9.2 编辑器服务

- **服务**: suishouji.qiangtu.com
- **项目**: markdown-AI-editor
- **功能**: 在线Markdown编辑和AI辅助编辑

## 10. 未来规划

- [ ] 支持更多AI模型
- [ ] 优化提示词模板
- [ ] 增加文章质量评分
- [ ] 支持多语言
- [ ] 支持自定义提示词模板
- [ ] 增加文章导出格式（PDF、Word）
- [ ] 支持团队协作
- [ ] 增加云端同步

---

**文档版本**: v1.0
**更新时间**: 2025-11-08
