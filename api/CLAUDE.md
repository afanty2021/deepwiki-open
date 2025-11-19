[根目录](../../CLAUDE.md) > **api**

# API 模块文档

## 变更记录 (Changelog)

- 2025-11-19 14:15:30 - 深度扫描完成，覆盖率提升至 85%+
- 2025-11-19 13:38:12 - 初始化 API 模块文档生成

## 模块职责

API 模块是 DeepWiki-Open 的后端服务核心，负责：
- 提供 FastAPI RESTful API 接口和 WebSocket 实时通信
- 处理多平台仓库克隆和智能代码分析
- 管理七大 AI 提供商的无缝集成
- 实现高性能检索增强生成 (RAG) 系统
- 支持多模态 AI 处理 (文本、图像、代码)
- 提供企业级向量化和搜索服务
- 实现实时流式响应和进度更新

## 入口与启动

### 主入口文件
- **`main.py`**: 应用程序入口点，配置环境变量和启动 Uvicorn 服务器
- **启动命令**: `python -m api.main`
- **默认端口**: 8001
- **健康检查**: `GET /health`

### 配置管理架构
- **环境变量**: 通过 `.env` 文件或环境变量动态加载
- **配置文件**: `config/` 目录下的 JSON 配置文件
- **日志系统**: `logging_config.py` 配置多级日志输出
- **动态配置**: 支持运行时配置更新

## 对外接口

### 核心 API 端点

#### 1. 认证与授权
```python
GET  /auth/status          # 检查认证模式
POST /auth/validate        # 验证认证码
```

#### 2. 模型管理
```python
GET /models/config         # 获取所有模型配置
GET /lang/config          # 获取语言配置
```

#### 3. 聊天与对话
```python
WebSocket /chat/stream     # 实时流式聊天 (推荐)
POST /chat/completion      # HTTP 聊天接口
```

#### 4. Wiki 项目管理
```python
GET  /wiki/projects        # 获取项目列表
POST /wiki/cache          # 保存项目缓存
GET  /wiki/export         # 导出 Wiki (Markdown/JSON)
```

#### 5. 仓库处理
```python
POST /repo/clone          # 克隆仓库
POST /repo/analyze        # 分析仓库结构
```

### WebSocket 实时通信
- **文件**: `websocket_wiki.py`
- **功能**: 实时流式响应、进度更新、错误处理
- **支持的客户端**: 前端 `utils/websocketClient.ts`
- **特性**: 自动重连、心跳检测、错误恢复

## 关键依赖与配置

### Python 依赖架构 (`pyproject.toml`)

```toml
# 核心框架
fastapi = ">=0.95.0"          # Web 框架和 API
uvicorn = ">=0.21.1"          # ASGI 服务器
pydantic = ">=2.0.0"          # 数据验证和序列化

# AI 客户端生态系统
google-generativeai = ">=0.3.0"  # Google Gemini 模型
openai = ">=1.76.2"              # OpenAI GPT/DALL-E
ollama = ">=0.4.8"               # 本地 Ollama 模型
boto3 = ">=1.34.0"               # AWS Bedrock
azure-identity = ">=1.12.0"      # Azure 认证
azure-core = ">=1.24.0"          # Azure 核心

# 数据处理与存储
faiss-cpu = ">=1.7.4"         # 高性能向量搜索
tiktoken = ">=0.5.0"          # 精确 Token 计算
adalflow = ">=0.1.0"          # AI 应用框架
numpy = ">=1.24.0"            # 数值计算基础

# 网络与异步
websockets = ">=11.0.3"       # WebSocket 实时通信
aiohttp = ">=3.8.4"           # 异步 HTTP 客户端
requests = ">=2.28.0"         # HTTP 请求库

# 模板与配置
jinja2 = ">=3.1.2"            # 模板引擎
python-dotenv = ">=1.0.0"     # 环境变量管理
```

### 配置文件系统架构

```
api/config/
├── embedder.json           # 嵌入模型配置 (3种选项)
├── generator.json          # 生成模型配置 (7大提供商)
├── repo.json              # 仓库处理和过滤规则
└── lang.json              # 多语言支持配置 (10种语言)
```

## 核心模块详细分析

### 1. AI 客户端生态系统

#### **OpenAI 客户端** (`openai_client.py`)
- **功能覆盖**: 文本生成、嵌入向量、图像生成、多模态处理
- **特色功能**:
  - 智能流式转换 (自动非流式转流式)
  - 多模态输入支持 (文本+图像)
  - 完整 Token 使用统计
  - 自动重试和错误恢复
- **支持模型**: GPT-4、GPT-5、DALL-E 3、o1、o3 系列

#### **Google 嵌入客户端** (`google_embedder_client.py`)
- **专用功能**: 高性能文本嵌入
- **配置**: 批量大小 100，任务类型 SEMANTIC_SIMILARITY
- **支持模型**: text-embedding-004, embedding-001
- **特色**: 原生批量处理、响应解析优化

#### **OpenRouter 客户端** (`openrouter_client.py`)
- **统一访问**: 数百个 AI 模型的单一接口
- **特色功能**:
  - XML 内容智能处理和验证
  - 异步流式响应
  - 动态错误生成和恢复
  - 内容格式优化
- **支持**: OpenAI、Anthropic、DeepSeek 等多系列模型

#### **Azure AI 客户端** (`azureai_client.py`)
- **企业级**: Azure OpenAI 服务集成
- **认证支持**: API 密钥 + Azure AD 双认证
- **功能**: 完整的聊天完成和嵌入支持
- **特色**: 企业级安全性和合规性

#### **AWS Bedrock 客户端** (`bedrock_client.py`)
- **云原生**: AWS 基础模型服务
- **提供商支持**: Anthropic Claude、Amazon Titan、Cohere、AI21
- **认证**: IAM 角色和访问密钥支持
- **特色**: 动态请求格式化、提供商特定优化

#### **Dashscope 客户端** (`dashscope_client.py`)
- **阿里云**: 通义千问系列模型
- **特色功能**:
  - 自定义批量处理器 (最大 25)
  - 嵌入缓存系统
  - 向量一致性验证
  - 文档处理管道
- **支持模型**: Qwen-plus、Qwen-turbo、DeepSeek-r1

#### **Ollama 补丁** (`ollama_patch.py`)
- **本地模型**: 开源模型本地运行
- **功能**:
  - 模型可用性检测
  - 单文档处理优化
  - 向量维度一致性
  - 错误跳过和恢复
- **支持模型**: Llama、Qwen、Nomic 等开源模型

### 2. 数据处理与管道

#### **数据管道** (`data_pipeline.py`)
- **仓库克隆**: 支持 GitHub/GitLab/Bitbucket
- **智能过滤**:
  - 目录排除规则 (30+ 默认规则)
  - 文件模式匹配
  - 自定义包含/排除
- **Token 管理**: 多编码器支持，8192 token 限制
- **数据库**: FAISS 向量存储集成

#### **RAG 系统** (`rag.py`)
- **检索增强**: 智能文档检索和生成
- **对话管理**:
  - 自定义对话实现 (修复索引错误)
  - 历史上下文管理
  - 长对话优化
- **向量检索**: FAISS 高性能相似性搜索
- **提示工程**: 多语言智能提示模板

### 3. WebSocket 实时通信

#### **WebSocket 处理器** (`websocket_wiki.py`)
- **实时流式**: 低延迟响应
- **错误处理**:
  - 连接断开恢复
  - 异常情况处理
  - 用户友好错误消息
- **功能**:
  - 大小输入检测 (8000 token 限制)
  - 自定义文件过滤
  - 多语言支持
  - 进度实时更新

### 4. 工具与优化

#### **嵌入工具** (`tools/embedder.py`)
- **动态选择**: 基于配置的自动嵌入器选择
- **类型支持**: OpenAI、Google、Ollama 三种嵌入
- **向后兼容**: 遗留参数支持
- **配置驱动**: 运行时配置更新

#### **提示管理** (`prompts.py`)
- **系统提示**: RAG 专用系统提示模板
- **多语言**: 10 种语言支持
- **格式化**: Markdown 输出格式
- **模板引擎**: Jinja2 模板系统

## 数据模型与 API 规范

### 核心 Pydantic 模型

```python
class WikiPage(BaseModel):
    """Wiki 页面数据模型"""
    id: str
    title: str
    content: str
    filePaths: List[str]
    importance: Literal['high', 'medium', 'low']
    relatedPages: List[str]

class ChatCompletionRequest(BaseModel):
    """聊天请求模型"""
    repo_url: str
    messages: List[ChatMessage]
    provider: str = "google"
    model: Optional[str] = None
    language: str = "en"
    # 高级过滤选项
    excluded_dirs: Optional[str] = None
    excluded_files: Optional[str] = None
    included_dirs: Optional[str] = None
    included_files: Optional[str] = None

class ModelConfig(BaseModel):
    """模型配置模型"""
    providers: List[Provider]
    defaultProvider: str
```

## API 配置详解

### 模型配置 (`generator.json`)
- **7 大提供商**: Google、OpenAI、OpenRouter、Ollama、Bedrock、Azure、Dashscope
- **20+ 模型**: 涵盖最新 AI 模型
- **动态参数**: temperature、top_p、top_k 等可配置
- **自定义模型**: 支持用户自定义模型集成

### 嵌入配置 (`embedder.json`)
- **3 种嵌入**: OpenAI (text-embedding-3-small)、Google (text-embedding-004)、Ollama (nomic-embed-text)
- **批处理优化**: OpenAI(500)、Google(100)
- **维度配置**: 可配置向量维度 (默认 256)
- **任务类型**: SEMANTIC_SIMILARITY 等任务优化

### 仓库配置 (`repo.json`)
- **智能过滤**:
  - 排除目录: node_modules、.git、__pycache__ 等
  - 排除文件: .lock、.env、二进制文件等
- **大小限制**: 默认 50GB
- **支持平台**: GitHub、GitLab、Bitbucket

## 性能与优化

### 向量化性能
- **FAISS 集成**: 高性能向量相似性搜索
- **批量处理**: 不同提供商的批量大小优化
- **缓存机制**: Dashscope 专用嵌入缓存
- **内存优化**: 流式处理大数据集

### 并发处理
- **异步支持**: 所有客户端支持异步调用
- **WebSocket**: 实时并发连接处理
- **错误隔离**: 单个请求错误不影响整体服务
- **资源管理**: 自动连接池和资源清理

## 测试与质量保证

### 测试架构
```bash
tests/
├── unit/                   # 单元测试
│   ├── test_all_embedders.py
│   └── test_google_embedder.py
├── integration/            # 集成测试
│   └── test_full_integration.py
├── api/                   # API 测试
│   └── test_api.py
└── run_tests.py          # 测试运行器
```

### 代码质量
- **类型提示**: 完整的 Python 类型注解
- **数据验证**: Pydantic 模型验证
- **错误处理**: 全面的异常处理和日志记录
- **文档**: 详细的 docstring 和注释

## 运行环境与部署

### 开发环境
```bash
# 安装依赖
poetry install

# 启动开发服务器
python -m api.main

# 运行测试
pytest tests/
```

### 生产环境
```bash
# Docker 部署
docker-compose up

# 或直接运行
python -m api.main
```

### 环境变量配置
```bash
# AI 提供商密钥
GOOGLE_API_KEY=your_google_api_key
OPENAI_API_KEY=your_openai_api_key
OPENROUTER_API_KEY=your_openrouter_api_key
AZURE_OPENAI_API_KEY=your_azure_api_key
AZURE_OPENAI_ENDPOINT=your_azure_endpoint
DASHSCOPE_API_KEY=your_dashscope_api_key

# AWS 配置
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_REGION=us-east-1
AWS_ROLE_ARN=your_role_arn

# Ollama 配置
OLLAMA_HOST=http://localhost:11434

# 系统配置
PORT=8001
DEEPWIKI_AUTH_MODE=false
DEEPWIKI_AUTH_CODE=your_auth_code
DEEPWIKI_EMBEDDER_TYPE=openai
DEEPWIKI_CONFIG_DIR=/path/to/config
```

## 常见问题与解决方案

### Q: 如何添加新的 AI 提供商？
A:
1. 创建新的客户端类继承 `ModelClient`
2. 在 `config/generator.json` 中添加配置
3. 在 `config.py` 中注册客户端类
4. 添加相应的环境变量支持

### Q: 如何优化大仓库处理性能？
A:
1. 调整 `config/repo.json` 中的过滤规则
2. 使用自定义排除目录减少文件数量
3. 调整批处理大小和 chunk 大小
4. 考虑使用更快的嵌入模型

### Q: 如何处理 Token 限制问题？
A:
1. 系统自动检测 >8000 token 的输入
2. 调整 `text_splitter` 配置中的 chunk_size
3. 使用更高效的嵌入模型
4. 启用缓存机制减少重复计算

### Q: 如何配置 WebSocket 连接？
A:
1. 使用 `/chat/stream` WebSocket 端点
2. 发送 JSON 格式的 `ChatCompletionRequest`
3. 处理实时流式响应
4. 实现自动重连机制

### Q: 如何扩展语言支持？
A:
1. 在 `config/lang.json` 中添加新语言
2. 在 `src/messages/` 目录添加翻译文件
3. 更新提示模板中的语言检测逻辑
4. 测试多语言生成功能

## 相关文件清单

### 核心文件
- `main.py` - 应用入口和服务器启动
- `api.py` - FastAPI 路由和端点定义 (800+ 行)
- `config.py` - 配置管理和环境变量处理 (400+ 行)

### AI 客户端生态系统
- `openai_client.py` - OpenAI 客户端 (630+ 行，多模态)
- `google_embedder_client.py` - Google 嵌入客户端 (230+ 行)
- `openrouter_client.py` - OpenRouter 统一客户端 (526+ 行)
- `azureai_client.py` - Azure 企业级客户端 (487+ 行)
- `bedrock_client.py` - AWS Bedrock 云原生客户端 (318+ 行)
- `dashscope_client.py` - 阿里云客户端 (914+ 行，功能最丰富)
- `ollama_patch.py` - 本地模型支持补丁 (105+ 行)

### 数据处理与管道
- `data_pipeline.py` - 数据处理管道 (400+ 行)
- `rag.py` - RAG 检索增强生成 (300+ 行)
- `websocket_wiki.py` - WebSocket 实时通信 (400+ 行)
- `simple_chat.py` - 简单聊天接口
- `prompts.py` - 提示词模板管理

### 配置与工具
- `config/` - 配置文件目录 (4 个 JSON 配置)
- `tools/embedder.py` - 嵌入工具 (50+ 行)
- `logging_config.py` - 日志配置
- `__init__.py` - 包初始化

### 测试文件
- `tests/` - 完整测试套件
  - `unit/test_all_embedders.py` - 嵌入器单元测试
  - `integration/test_full_integration.py` - 集成测试
  - `api/test_api.py` - API 端点测试

### 配置文件
- `pyproject.toml` - Python 依赖和项目配置
- `poetry.lock` - 依赖版本锁定
- `pytest.ini` - 测试配置
- `docker-compose.yml` - Docker 部署配置