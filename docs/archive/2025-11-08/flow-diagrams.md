# 论文神器 - 流程图文档

本文档包含论文神器项目的核心业务流程图和系统架构图。

## 1. 核心业务流程

### 1.1 主流程：三步骤文章生成

```mermaid
flowchart TD
    Start([开始]) --> Step1[步骤1: 写作设定]

    Step1 --> Input1[选择文章类型<br/>32种类型可选]
    Input1 --> Input2[选择风格特点<br/>46种风格可选]
    Input2 --> Input3[设置篇幅<br/>主节点数 + 子节点数]
    Input3 --> Input4[输入标题/提示词]
    Input4 --> Input5[选择AI模型<br/>GPT-3.5/4/Gemini/Claude]
    Input5 --> History{加载历史?}

    History -->|是| LoadHistory[从localStorage<br/>加载历史创作]
    History -->|否| Step2[步骤2: 段落撰写]
    LoadHistory --> Step2

    Step2 --> GenerateOutline[调用AI生成提纲<br/>JSON格式]
    GenerateOutline --> ParseOutline[解析提纲结构<br/>提取章节和小节]
    ParseOutline --> WriteMode{选择编写模式}

    WriteMode -->|全部编写| BatchWrite[批量生成所有章节]
    WriteMode -->|单独编写| SingleWrite[选择章节单独生成]

    BatchWrite --> WriteLoop[遍历所有章节]
    SingleWrite --> WriteLoop

    WriteLoop --> CallAI[调用AI API<br/>SSE流式生成内容]
    CallAI --> StreamDisplay[实时流式显示内容]
    StreamDisplay --> NeedImage{需要配图?}

    NeedImage -->|是| GenerateImage[调用DALL-E<br/>生成章节配图]
    NeedImage -->|否| SaveContent
    GenerateImage --> SaveContent[保存章节内容]

    SaveContent --> MoreChapters{还有章节?}
    MoreChapters -->|是| WriteLoop
    MoreChapters -->|否| SaveHistory[保存到localStorage<br/>创作历史]

    SaveHistory --> Step3[步骤3: 预览编辑]

    Step3 --> Preview[Markdown预览]
    Preview --> Action{用户操作}

    Action -->|复制| CopyToClipboard[复制到剪贴板]
    Action -->|在线编辑| OpenEditor[新窗口打开<br/>markdown-AI-editor]
    Action -->|返回修改| Step2
    Action -->|重新开始| Step1

    CopyToClipboard --> End([结束])
    OpenEditor --> End

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style Step1 fill:#87CEEB
    style Step2 fill:#87CEEB
    style Step3 fill:#87CEEB
    style CallAI fill:#FFD700
    style GenerateImage fill:#FFD700
```

### 1.2 提纲生成详细流程

```mermaid
flowchart TD
    Start([开始生成提纲]) --> BuildPrompt[构建系统提示词]

    BuildPrompt --> AddContext1[添加文章类型<br/>例: 学术论文]
    AddContext1 --> AddContext2[添加风格特点<br/>例: 严肃严谨]
    AddContext2 --> AddContext3[添加篇幅要求<br/>例: 5主节点, 3子节点]
    AddContext3 --> AddContext4[添加标题/主题]

    AddContext4 --> SystemPrompt[完整系统提示词]
    SystemPrompt --> UserPrompt[用户提示词:<br/>请生成JSON格式提纲]

    UserPrompt --> CallAPI[调用AI API<br/>model: 高级模型<br/>temperature: 0.7]

    CallAPI --> SSEStream[SSE流式响应]
    SSEStream --> BufferData[缓冲数据块]
    BufferData --> ParseJSON{解析JSON}

    ParseJSON -->|失败| RetryParse[重试解析<br/>提取JSON部分]
    RetryParse --> ParseJSON

    ParseJSON -->|成功| ValidateStructure{验证结构}
    ValidateStructure -->|失败| Error[显示错误<br/>请求用户重试]

    ValidateStructure -->|成功| ExtractData[提取提纲数据]
    ExtractData --> BuildTree[构建章节树<br/>title/chapters/sections]

    BuildTree --> DisplayOutline[显示提纲结构]
    DisplayOutline --> EnableWrite[启用撰写功能]
    EnableWrite --> End([提纲生成完成])

    Error --> End

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CallAPI fill:#FFD700
    style Error fill:#FF6B6B
```

### 1.3 章节内容生成详细流程

```mermaid
flowchart TD
    Start([开始生成章节]) --> GetChapter[获取章节信息<br/>标题 + 小节列表]

    GetChapter --> BuildPrompt[构建章节提示词]
    BuildPrompt --> AddOutline[添加整体提纲上下文]
    AddOutline --> AddChapterInfo[添加当前章节要求]
    AddChapterInfo --> AddStyle[添加风格要求]

    AddStyle --> CallAPI[调用AI API<br/>model: 用户选择模型<br/>stream: true]

    CallAPI --> InitSSE[初始化SSE连接]
    InitSSE --> ReadStream[读取流数据]

    ReadStream --> ParseChunk[解析数据块]
    ParseChunk --> ExtractDelta[提取增量内容<br/>delta.content]

    ExtractDelta --> AppendContent[追加到章节内容]
    AppendContent --> UpdateUI[更新UI显示<br/>实时渲染Markdown]

    UpdateUI --> CheckInterrupt{用户中断?}
    CheckInterrupt -->|是| CloseStream[关闭SSE连接]
    CheckInterrupt -->|否| MoreData{还有数据?}

    MoreData -->|是| ReadStream
    MoreData -->|否| StreamComplete[流式响应完成]

    StreamComplete --> CheckImage{需要生成配图?}
    CheckImage -->|否| SaveChapter

    CheckImage -->|是| BuildImagePrompt[构建配图提示词<br/>基于章节内容]
    BuildImagePrompt --> CallDALLE[调用DALL-E API<br/>model: gpt-4-dalle]
    CallDALLE --> GetImageURL[获取图片URL]
    GetImageURL --> InsertImage[插入Markdown图片]
    InsertImage --> SaveChapter[保存章节内容]

    CloseStream --> SaveChapter
    SaveChapter --> UpdateProgress[更新生成进度]
    UpdateProgress --> End([章节生成完成])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CallAPI fill:#FFD700
    style CallDALLE fill:#FFD700
```

## 2. 系统架构流程

### 2.1 整体系统架构

```mermaid
flowchart TB
    subgraph Client["客户端层"]
        Browser[Web浏览器]
        VueApp[Vue 3 应用]
        Components[组件层<br/>ArticleGenerator<br/>PageHeader等]
    end

    subgraph Frontend["前端应用层"]
        Router[Vue Router<br/>路由管理]
        Utils[工具层<br/>chat_transfer<br/>prompts<br/>models]
        API[API接口层<br/>sse.js<br/>public.js]
        Storage[本地存储<br/>localStorage<br/>历史记录]
    end

    subgraph Proxy["代理层"]
        Nginx[Nginx反向代理<br/>或Vercel Proxy]
        ProxyAI[/v1/* → AI服务]
        ProxyEditor[/api/* → 编辑器]
    end

    subgraph Backend["后端服务层"]
        AIService[AI服务<br/>Two-API中转]
        OpenAI[OpenAI API<br/>GPT-3.5/4]
        Gemini[Google Gemini]
        Claude[Anthropic Claude]
        DALLE[DALL-E图片生成]

        EditorService[编辑器服务<br/>markdown-AI-editor]
    end

    Browser --> VueApp
    VueApp --> Components
    Components --> Router
    Components --> Utils
    Components --> API
    Components --> Storage

    API --> Nginx
    Nginx --> ProxyAI
    Nginx --> ProxyEditor

    ProxyAI --> AIService
    AIService --> OpenAI
    AIService --> Gemini
    AIService --> Claude
    AIService --> DALLE

    ProxyEditor --> EditorService

    Storage -.->|读取历史| Components
    Components -.->|保存历史| Storage

    style Client fill:#E3F2FD
    style Frontend fill:#FFF3E0
    style Proxy fill:#F3E5F5
    style Backend fill:#E8F5E9
```

### 2.2 API调用流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant UI as Vue组件
    participant API as API层(sse.js)
    participant Proxy as Nginx代理
    participant AI as AI服务

    User->>UI: 点击生成按钮
    UI->>UI: 构建请求参数<br/>messages, model, stream
    UI->>API: makeSingleChat(data, auth, callback)

    API->>API: 构建HTTP请求<br/>添加Bearer Token
    API->>Proxy: POST /v1/chat/completions<br/>Content-Type: application/json

    Proxy->>AI: 转发请求到AI服务
    AI->>AI: 处理请求<br/>调用对应模型

    AI-->>Proxy: SSE流式响应<br/>data: {delta: {...}}
    Proxy-->>API: 转发SSE流

    loop 流式数据传输
        API->>API: 解析SSE数据块<br/>提取delta.content
        API->>UI: 回调函数<br/>callback(content)
        UI->>UI: 更新UI<br/>追加内容
        UI->>User: 实时显示生成内容
    end

    AI-->>Proxy: data: [DONE]
    Proxy-->>API: 流结束标记
    API->>UI: 回调结束
    UI->>UI: 保存完整内容
    UI->>User: 显示生成完成
```

### 2.3 数据存储流程

```mermaid
flowchart LR
    subgraph Generate["内容生成"]
        G1[生成提纲]
        G2[生成章节内容]
        G3[生成配图]
    end

    subgraph Memory["内存状态"]
        M1[currentOutline<br/>当前提纲]
        M2[articleSections<br/>章节内容数组]
        M3[generationProgress<br/>生成进度]
    end

    subgraph LocalStorage["本地持久化"]
        L1[history_list<br/>历史列表]
        L2[history_detail_{id}<br/>历史详情]
        L3[user_config<br/>用户配置]
    end

    G1 --> M1
    G2 --> M2
    G3 --> M2

    M1 --> Save1[保存操作]
    M2 --> Save1
    M3 --> Save1

    Save1 --> L1
    Save1 --> L2

    L1 --> Load1[加载操作]
    L2 --> Load1
    L3 --> Load1

    Load1 --> M1
    Load1 --> M2
    Load1 --> M3

    style Generate fill:#FFE0B2
    style Memory fill:#B2DFDB
    style LocalStorage fill:#D1C4E9
```

## 3. 部署架构

### 3.1 Docker部署架构

```mermaid
flowchart TB
    subgraph Docker["Docker容器"]
        subgraph Nginx["Nginx服务"]
            Static[静态文件服务<br/>/dist]
            Proxy1[反向代理<br/>/v1/* → AI服务]
            Proxy2[反向代理<br/>/api/* → 编辑器]
        end

        Config[环境变量配置<br/>proxy_url<br/>proxy_key<br/>editor_url<br/>ai_models]
    end

    subgraph Build["构建产物"]
        VueBuild[Vue构建<br/>npm run build]
        DistFolder[/dist文件夹<br/>index.html<br/>js/css/assets]
    end

    subgraph External["外部服务"]
        AIExt[AI服务<br/>OpenAI兼容API]
        EditorExt[编辑器服务<br/>suishouji.qiangtu.com]
    end

    VueBuild --> DistFolder
    DistFolder --> Static

    Config --> Nginx

    Proxy1 --> AIExt
    Proxy2 --> EditorExt

    User([用户浏览器]) --> Nginx

    style Docker fill:#E3F2FD
    style Build fill:#FFF3E0
    style External fill:#E8F5E9
```

### 3.2 Vercel部署架构

```mermaid
flowchart TB
    subgraph Vercel["Vercel平台"]
        Static[静态站点<br/>自动部署Vue构建产物]
        Serverless[Serverless函数<br/>api/proxy.js]
    end

    subgraph GitHub["GitHub仓库"]
        Code[源代码]
        Actions[GitHub Actions<br/>自动构建]
    end

    subgraph External["外部服务"]
        AIExt[AI服务]
        EditorExt[编辑器服务]
    end

    Code --> Actions
    Actions --> Deploy[部署触发]
    Deploy --> Vercel

    User([用户浏览器]) --> Static
    Static --> Serverless
    Serverless --> AIExt
    Serverless --> EditorExt

    style Vercel fill:#E3F2FD
    style GitHub fill:#FFF3E0
    style External fill:#E8F5E9
```

## 4. 数据流图

### 4.1 完整数据流

```mermaid
flowchart LR
    subgraph Input["用户输入"]
        I1[文章类型]
        I2[风格特点]
        I3[篇幅设置]
        I4[标题主题]
        I5[模型选择]
    end

    subgraph Process["处理流程"]
        P1[构建提示词]
        P2[调用AI API]
        P3[SSE流式响应]
        P4[解析处理]
        P5[格式化输出]
    end

    subgraph Output["输出结果"]
        O1[提纲JSON]
        O2[章节Markdown]
        O3[配图URL]
        O4[完整文章]
    end

    subgraph Storage["数据存储"]
        S1[内存状态]
        S2[localStorage]
    end

    I1 --> P1
    I2 --> P1
    I3 --> P1
    I4 --> P1
    I5 --> P2

    P1 --> P2
    P2 --> P3
    P3 --> P4
    P4 --> P5

    P5 --> O1
    P5 --> O2
    P5 --> O3

    O1 --> S1
    O2 --> S1
    O3 --> S1

    S1 --> S2
    S2 -.->|恢复| S1

    O1 --> O4
    O2 --> O4
    O3 --> O4

    style Input fill:#FFE0B2
    style Process fill:#B2DFDB
    style Output fill:#D1C4E9
    style Storage fill:#FFCCBC
```

## 5. 错误处理流程

```mermaid
flowchart TD
    Start([API调用开始]) --> TryRequest[发起请求]

    TryRequest --> CheckResponse{响应状态}

    CheckResponse -->|200 OK| ProcessData[处理数据]
    CheckResponse -->|401| AuthError[认证错误<br/>提示Token无效]
    CheckResponse -->|429| RateLimit[速率限制<br/>提示稍后重试]
    CheckResponse -->|500| ServerError[服务器错误<br/>提示联系管理员]
    CheckResponse -->|网络错误| NetworkError[网络错误<br/>提示检查网络]

    ProcessData --> ValidateData{数据有效?}
    ValidateData -->|是| Success([成功])
    ValidateData -->|否| ParseError[解析错误<br/>提示数据格式异常]

    AuthError --> Retry{允许重试?}
    RateLimit --> Retry
    NetworkError --> Retry
    ParseError --> Retry

    Retry -->|是| Wait[等待延迟]
    Retry -->|否| Fail([失败])

    Wait --> TryRequest
    ServerError --> Fail

    style Start fill:#90EE90
    style Success fill:#90EE90
    style Fail fill:#FF6B6B
    style AuthError fill:#FFD700
    style RateLimit fill:#FFD700
    style ServerError fill:#FF6B6B
    style NetworkError fill:#FFD700
    style ParseError fill:#FFD700
```

---

**文档版本**: v1.0
**更新时间**: 2025-11-08
**图表工具**: Mermaid
