# 论文神器 - 系统架构详细设计

## 1. 技术栈架构

### 1.1 技术栈全景图

```mermaid
graph TB
    subgraph Frontend["前端技术栈"]
        Vue[Vue 3.2.13<br/>Composition API]
        ElementPlus[Element Plus 2.7.8<br/>UI组件库]
        VueRouter[Vue Router 4.x<br/>路由管理]
        MDEditor[@kangc/v-md-editor<br/>Markdown编辑器]
        Axios[Axios 1.4.0<br/>HTTP客户端]
    end

    subgraph Build["构建工具链"]
        VueCLI[Vue CLI<br/>项目脚手架]
        Webpack[Webpack<br/>模块打包]
        Babel[Babel<br/>ES6+转译]
        PostCSS[PostCSS<br/>CSS处理]
    end

    subgraph Deploy["部署方案"]
        Docker[Docker<br/>容器化]
        Nginx[Nginx<br/>Web服务器]
        Vercel[Vercel<br/>Serverless]
    end

    subgraph Backend["后端服务"]
        TwoAPI[Two-API<br/>AI服务中转]
        OpenAI[OpenAI API]
        Gemini[Google Gemini]
        Claude[Anthropic Claude]
    end

    Vue --> ElementPlus
    Vue --> VueRouter
    Vue --> MDEditor
    Vue --> Axios

    Vue --> VueCLI
    VueCLI --> Webpack
    Webpack --> Babel
    Webpack --> PostCSS

    Webpack --> Docker
    Docker --> Nginx
    Webpack --> Vercel

    Axios --> TwoAPI
    TwoAPI --> OpenAI
    TwoAPI --> Gemini
    TwoAPI --> Claude

    style Frontend fill:#E3F2FD
    style Build fill:#FFF3E0
    style Deploy fill:#E8F5E9
    style Backend fill:#F3E5F5
```

### 1.2 依赖关系

```mermaid
graph LR
    subgraph Core["核心依赖"]
        Vue3[vue@3.2.13]
        VueRouter[vue-router@4.x]
    end

    subgraph UI["UI依赖"]
        ElementPlus[element-plus@2.7.8]
        ElementIcons[@element-plus/icons-vue]
        MDEditor[@kangc/v-md-editor]
        Prism[prismjs]
    end

    subgraph HTTP["网络依赖"]
        Axios[axios@1.4.0]
        EventSource[eventsource-parser]
    end

    subgraph Dev["开发依赖"]
        VueCLI[@vue/cli-service]
        Sass[sass]
        ESLint[eslint]
    end

    Vue3 --> VueRouter
    Vue3 --> ElementPlus
    ElementPlus --> ElementIcons
    Vue3 --> MDEditor
    MDEditor --> Prism

    Vue3 --> Axios
    Axios --> EventSource

    Vue3 --> VueCLI
    VueCLI --> Sass
    VueCLI --> ESLint

    style Core fill:#90EE90
    style UI fill:#87CEEB
    style HTTP fill:#FFD700
    style Dev fill:#DDA0DD
```

## 2. 组件架构

### 2.1 组件层次结构

```mermaid
graph TD
    App[App.vue<br/>根组件]

    App --> PageHeader[PageHeader.vue<br/>页面头部]
    App --> RouterView[RouterView<br/>路由视图]

    RouterView --> ArticleGenerator[ArticleGenerator.vue<br/>核心业务组件]

    PageHeader --> Logo[Logo显示]
    PageHeader --> DarkMode[暗黑模式切换]
    PageHeader --> GitHub[GitHub链接]

    ArticleGenerator --> Steps[Steps步骤条<br/>Element Plus]
    ArticleGenerator --> Step1[步骤1: 写作设定<br/>配置表单]
    ArticleGenerator --> Step2[步骤2: 段落撰写<br/>提纲展示+内容生成]
    ArticleGenerator --> Step3[步骤3: 预览编辑<br/>Markdown预览]

    Step1 --> TypeSelect[文章类型选择]
    Step1 --> StyleSelect[风格特点选择]
    Step1 --> SizeInput[篇幅设置]
    Step1 --> TitleInput[标题输入]
    Step1 --> ModelSelect[模型选择]
    Step1 --> HistoryLoad[历史记录加载]

    Step2 --> OutlineDisplay[提纲树形展示]
    Step2 --> WriteButtons[编写操作按钮]
    Step2 --> ProgressBar[生成进度条]
    Step2 --> ContentDisplay[内容实时显示]

    Step3 --> MDPreview[Markdown预览<br/>v-md-editor]
    Step3 --> CopyButton[复制按钮]
    Step3 --> EditorButton[在线编辑按钮]
    Step3 --> ScrollButton[ScrollToBottomButton<br/>滚动按钮]

    style App fill:#FFB6C1
    style ArticleGenerator fill:#87CEEB
    style Step1 fill:#90EE90
    style Step2 fill:#FFD700
    style Step3 fill:#DDA0DD
```

### 2.2 组件通信

```mermaid
sequenceDiagram
    participant User as 用户
    participant Header as PageHeader
    participant AG as ArticleGenerator
    participant API as API层
    participant Storage as LocalStorage

    User->>Header: 切换暗黑模式
    Header->>Header: 更新主题状态
    Header->>AG: emit theme-change

    User->>AG: 填写表单（步骤1）
    AG->>AG: 更新formData状态

    User->>AG: 点击生成提纲
    AG->>API: makeSingleChat(提纲请求)
    API-->>AG: SSE流式返回JSON
    AG->>AG: 解析并存储outline

    User->>AG: 点击全部编写
    loop 遍历所有章节
        AG->>API: makeSingleChat(章节请求)
        API-->>AG: SSE流式返回内容
        AG->>AG: 实时更新articleSections
    end

    AG->>Storage: 保存历史记录
    Storage-->>AG: 保存成功

    User->>AG: 切换到预览步骤
    AG->>AG: 渲染Markdown

    User->>AG: 点击复制
    AG->>AG: 复制到剪贴板
    AG->>User: 显示成功提示
```

## 3. 数据架构

### 3.1 数据模型

```mermaid
classDiagram
    class FormData {
        +String articleType
        +Array styleFeatures
        +Number mainNodes
        +Number subNodes
        +String title
        +String prompt
        +String model
    }

    class Outline {
        +String title
        +Array chapters
    }

    class Chapter {
        +String title
        +Array sections
        +String content
        +String image
        +Boolean generated
    }

    class History {
        +String id
        +Date createdAt
        +FormData formData
        +Outline outline
        +Array articleSections
    }

    class APIRequest {
        +String model
        +Array messages
        +Boolean stream
        +Number temperature
    }

    class APIResponse {
        +String id
        +String object
        +Array choices
        +Object usage
    }

    FormData --> Outline : generates
    Outline --> Chapter : contains
    History --> FormData : includes
    History --> Outline : includes
    History --> Chapter : includes
    FormData --> APIRequest : builds
    APIRequest --> APIResponse : receives
```

### 3.2 状态管理

```mermaid
stateDiagram-v2
    [*] --> Idle: 初始化

    Idle --> ConfiguringStep1: 进入步骤1
    ConfiguringStep1 --> ConfiguringStep1: 修改配置
    ConfiguringStep1 --> LoadingHistory: 加载历史
    LoadingHistory --> ConfiguringStep1: 历史加载完成
    ConfiguringStep1 --> GeneratingOutline: 点击下一步

    GeneratingOutline --> OutlineReady: 提纲生成成功
    GeneratingOutline --> ConfiguringStep1: 生成失败

    OutlineReady --> WritingStep2: 进入步骤2
    WritingStep2 --> GeneratingContent: 开始编写
    GeneratingContent --> GeneratingContent: 正在生成章节
    GeneratingContent --> GeneratingImage: 需要配图
    GeneratingImage --> GeneratingContent: 配图完成
    GeneratingContent --> ContentReady: 所有章节完成
    GeneratingContent --> WritingStep2: 中断生成

    ContentReady --> PreviewStep3: 进入步骤3
    PreviewStep3 --> PreviewStep3: 预览/编辑操作
    PreviewStep3 --> WritingStep2: 返回修改
    PreviewStep3 --> ConfiguringStep1: 重新开始
    PreviewStep3 --> [*]: 完成

    state GeneratingContent {
        [*] --> FetchingChapter
        FetchingChapter --> StreamingContent
        StreamingContent --> ChapterComplete
        ChapterComplete --> [*]
    }
```

## 4. 模块架构

### 4.1 模块依赖图

```mermaid
graph TB
    subgraph Views["视图层"]
        ArticleGenerator
    end

    subgraph Components["组件层"]
        PageHeader
        DateInfo
        ScrollButton
    end

    subgraph Utils["工具层"]
        ChatTransfer[chat_transfer.js<br/>消息处理]
        Prompts[prompts.js<br/>提示词模板]
        Models[models.js<br/>模型配置]
        Page[page.js<br/>页面工具]
        Date[date.js<br/>日期格式化]
        Txt[txt.js<br/>文本处理]
        Enums[enums.js<br/>枚举常量]
    end

    subgraph API["API层"]
        SSE[sse.js<br/>流式对话]
        Public[public.js<br/>公共接口]
    end

    subgraph Infrastructure["基础设施层"]
        Request[request.js<br/>HTTP封装]
        BrowserDB[browser_db.js<br/>存储封装]
        Token[token.js<br/>Token管理]
        Base[base.js<br/>基础配置]
    end

    ArticleGenerator --> PageHeader
    ArticleGenerator --> ScrollButton
    ArticleGenerator --> ChatTransfer
    ArticleGenerator --> Prompts
    ArticleGenerator --> Models
    ArticleGenerator --> Page
    ArticleGenerator --> Date
    ArticleGenerator --> SSE
    ArticleGenerator --> Public
    ArticleGenerator --> BrowserDB

    PageHeader --> DateInfo

    SSE --> Request
    Public --> Request
    SSE --> ChatTransfer

    Request --> Base
    Request --> Token
    BrowserDB --> Base

    style Views fill:#E3F2FD
    style Components fill:#FFF3E0
    style Utils fill:#E8F5E9
    style API fill:#F3E5F5
    style Infrastructure fill:#FFE0B2
```

### 4.2 核心模块详解

#### utils/chat_transfer.js - 消息处理模块

```mermaid
graph LR
    subgraph Functions["核心功能"]
        Init[initBaseMessage<br/>初始化消息对象]
        Fix[fixMessages<br/>修复消息顺序]
        Validate[validateRole<br/>验证角色]
    end

    subgraph Logic["处理逻辑"]
        L1[确保user/assistant交替]
        L2[移除连续相同角色消息]
        L3[保持system消息在最前]
    end

    Input[输入: messages数组] --> Fix
    Fix --> L1
    Fix --> L2
    Fix --> L3
    L1 --> Output[输出: 修复后的messages]
    L2 --> Output
    L3 --> Output

    Init --> CreateMsg[创建标准消息对象]
    CreateMsg --> SetRole[设置role: user/assistant/system]
    SetRole --> SetContent[设置content: 文本内容]
```

#### api/sse.js - SSE流式对话模块

```mermaid
graph TB
    subgraph Public["公共接口"]
        MakeSingleChat[makeSingleChat<br/>单次对话]
        MakeSpeech[makeSpeechToText<br/>语音转文字]
        MakeTTS[makeTextToSpeech<br/>文字转语音]
    end

    subgraph Internal["内部函数"]
        MakeIterator[makeTextFileLineIterator<br/>SSE流解析器]
        ParseChunk[解析数据块]
        ExtractDelta[提取delta内容]
    end

    subgraph Process["处理流程"]
        P1[构建请求体]
        P2[发送POST请求]
        P3[获取ReadableStream]
        P4[逐行读取数据]
        P5[解析JSON]
        P6[提取内容]
        P7[回调函数]
    end

    MakeSingleChat --> P1
    P1 --> P2
    P2 --> P3
    P3 --> MakeIterator
    MakeIterator --> P4
    P4 --> ParseChunk
    ParseChunk --> P5
    P5 --> ExtractDelta
    ExtractDelta --> P6
    P6 --> P7

    style Public fill:#90EE90
    style Internal fill:#FFD700
    style Process fill:#87CEEB
```

## 5. 安全架构

### 5.1 认证流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant Frontend as 前端应用
    participant Proxy as Nginx代理
    participant AI as AI服务

    User->>Frontend: 访问应用
    Frontend->>Frontend: 检查localStorage<br/>是否有自定义Token

    alt 有自定义Token
        Frontend->>Frontend: 使用用户Token
    else 无自定义Token
        Frontend->>Frontend: 使用默认共享Token
    end

    User->>Frontend: 发起AI请求
    Frontend->>Frontend: 添加Bearer Token到Header

    Frontend->>Proxy: POST /v1/chat/completions<br/>Authorization: Bearer {token}

    Proxy->>Proxy: 可选：替换/验证Token
    Proxy->>AI: 转发请求<br/>Authorization: Bearer {proxy_key}

    AI->>AI: 验证Token

    alt Token有效
        AI-->>Proxy: 200 OK + 响应数据
        Proxy-->>Frontend: 转发响应
        Frontend-->>User: 显示结果
    else Token无效
        AI-->>Proxy: 401 Unauthorized
        Proxy-->>Frontend: 401错误
        Frontend-->>User: 提示Token无效
    end
```

### 5.2 数据安全

```mermaid
graph TD
    subgraph ClientSide["客户端安全"]
        C1[HTTPS加密传输]
        C2[Token不明文显示]
        C3[localStorage加密存储]
        C4[XSS防护<br/>Vue自动转义]
    end

    subgraph ServerSide["服务端安全"]
        S1[反向代理隐藏API]
        S2[环境变量存储密钥]
        S3[CORS跨域控制]
        S4[请求速率限制]
    end

    subgraph Data["数据安全"]
        D1[历史记录本地存储]
        D2[不上传敏感信息]
        D3[AI响应内容过滤]
    end

    ClientSide --> ServerSide
    ServerSide --> Data

    style ClientSide fill:#E3F2FD
    style ServerSide fill:#E8F5E9
    style Data fill:#FFF3E0
```

## 6. 性能架构

### 6.1 性能优化策略

```mermaid
mindmap
    root((性能优化))
        前端优化
            代码分割
                路由懒加载
                组件异步加载
            资源优化
                图片懒加载
                Gzip压缩
                CDN加速
            渲染优化
                虚拟滚动
                防抖节流
                Markdown增量渲染
        网络优化
            SSE流式传输
                降低首字延迟
                提升用户体验
            HTTP/2
                多路复用
                头部压缩
            缓存策略
                localStorage缓存
                浏览器缓存
                Service Worker
        后端优化
            反向代理
                请求合并
                连接池
            负载均衡
                多实例部署
                健康检查
```

### 6.2 缓存架构

```mermaid
graph TB
    subgraph Browser["浏览器层"]
        B1[Memory Cache<br/>运行时状态]
        B2[LocalStorage<br/>持久化数据]
        B3[HTTP Cache<br/>静态资源]
    end

    subgraph Application["应用层"]
        A1[Vue响应式状态]
        A2[历史记录缓存]
        A3[配置缓存]
    end

    subgraph Network["网络层"]
        N1[Nginx缓存]
        N2[CDN缓存]
    end

    A1 --> B1
    A2 --> B2
    A3 --> B2

    StaticFiles[静态文件请求] --> B3
    B3 -->|未命中| N1
    N1 -->|未命中| N2
    N2 -->|未命中| Origin[源站]

    APIRequest[API请求] --> A2
    A2 -->|未命中| Backend[后端服务]

    style Browser fill:#E3F2FD
    style Application fill:#FFF3E0
    style Network fill:#E8F5E9
```

## 7. 可扩展架构

### 7.1 插件化设计

```mermaid
graph TB
    subgraph Core["核心系统"]
        App[应用核心]
        Config[配置中心]
    end

    subgraph Plugins["可扩展插件"]
        P1[模型插件<br/>新增AI模型]
        P2[类型插件<br/>新增文章类型]
        P3[风格插件<br/>新增写作风格]
        P4[提示词插件<br/>自定义Prompt]
        P5[导出插件<br/>新增导出格式]
    end

    subgraph Registry["插件注册"]
        R1[模型注册表<br/>models.js]
        R2[类型注册表<br/>articleTypes]
        R3[提示词注册表<br/>prompts.js]
    end

    Config --> App

    P1 --> R1
    P2 --> R2
    P3 --> R2
    P4 --> R3
    P5 --> App

    R1 --> App
    R2 --> App
    R3 --> App

    style Core fill:#FFB6C1
    style Plugins fill:#90EE90
    style Registry fill:#FFD700
```

### 7.2 扩展点设计

```mermaid
graph LR
    subgraph Extension["扩展点"]
        E1[BeforeGenerate<br/>生成前钩子]
        E2[AfterGenerate<br/>生成后钩子]
        E3[ContentTransform<br/>内容转换器]
        E4[CustomPrompt<br/>自定义提示词]
        E5[CustomExport<br/>自定义导出]
    end

    subgraph Usage["使用场景"]
        U1[内容预处理]
        U2[质量检测]
        U3[格式转换]
        U4[特殊需求Prompt]
        U5[多格式导出]
    end

    E1 --> U1
    E2 --> U2
    E3 --> U3
    E4 --> U4
    E5 --> U5

    style Extension fill:#E3F2FD
    style Usage fill:#FFF3E0
```

## 8. 监控架构

### 8.1 日志收集

```mermaid
graph TB
    subgraph Sources["日志源"]
        S1[前端错误日志]
        S2[API调用日志]
        S3[性能指标]
        S4[用户行为日志]
    end

    subgraph Collection["收集层"]
        C1[Console日志]
        C2[Error Boundary]
        C3[Performance API]
    end

    subgraph Storage["存储层"]
        ST1[LocalStorage<br/>临时存储]
        ST2[远程日志服务<br/>可选]
    end

    S1 --> C2
    S2 --> C1
    S3 --> C3
    S4 --> C1

    C1 --> ST1
    C2 --> ST1
    C3 --> ST1

    ST1 -.->|可选上报| ST2

    style Sources fill:#E3F2FD
    style Collection fill:#FFF3E0
    style Storage fill:#E8F5E9
```

---

**文档版本**: v1.0
**更新时间**: 2025-11-08
**适用版本**: lunwenshenqi v1.x
