# SWU Guide 智能校园导览系统

> 面向校园游览场景的多端应用原型，整合地图、路线、内容管理、AI 问答与建筑识别。

![Flutter](https://img.shields.io/badge/Flutter-mobile-02569B?logo=flutter&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-backend-6DB33F?logo=springboot&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-AI%20service-009688?logo=fastapi&logoColor=white)
![Vue](https://img.shields.io/badge/Vue%203-admin-4FC08D?logo=vuedotjs&logoColor=white)

## 项目概览

校园游览常见的问题不是“缺少信息”，而是信息分散、路线与讲解割裂、现场识别成本高。SWU Guide 尝试把这些能力放进同一个移动端流程：

- 在地图中查看校园地点与路线
- 根据当前位置获取讲解内容
- 通过自然语言咨询校园信息
- 拍照识别建筑并继续追问
- 由管理后台维护景点、讲解和 AI 语料

本仓库是 **4 人团队项目的公开 Fork**。我主要负责 AI 虚拟导游、视觉识别、AI 语料接口，以及功能分支的 Review 与合并。

## 系统架构

~~~mermaid
flowchart LR
    App[Flutter App] --> Java[Spring Boot 业务后端]
    Admin[Vue 3 管理后台] --> Java
    App --> AI[FastAPI AI 微服务]
    Java --> DB[(MySQL)]
    Java --> Redis[(Redis)]
    Java --> AI
    AI --> LLM[OpenAI-compatible Model API]
    AI --> Corpus[(校园语料)]
~~~

| 模块 | 技术 | 主要职责 |
|---|---|---|
| 移动端 | Flutter / Dart | 地图、景点、讲解、路线、AI 对话与识别页面 |
| 管理后台 | Vue 3 / TypeScript | 景点、内容、路线、评论和语料管理 |
| 业务后端 | Spring Boot / MyBatis-Plus | 用户、景点、路线、评论、打卡和语料接口 |
| AI 微服务 | FastAPI / Python | 检索增强问答、建筑识别和场景问答 |
| 数据层 | MySQL / Redis | 业务数据、缓存与位置相关计算 |

## 我负责的部分

### 1. 校园问答

当前实现将问答拆为 4 个步骤：

~~~mermaid
flowchart LR
    A[加载校园语料] --> B[检索相关内容]
    B --> C[组装上下文]
    C --> D[调用大模型生成回答]
    D --> E[返回回答与参考语料]
~~~

已提供 3 个 FastAPI 接口：

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | /api/rag/load-corpus | 加载问答语料 |
| GET | /api/rag/search | 检索相关语料 |
| POST | /api/rag/chat | 返回回答与参考来源 |

当前版本使用轻量关键词召回，便于验证端到端流程；Chroma 向量检索与更完整的语料评测列在后续计划中。

### 2. 建筑识别与场景问答

视觉模块接收 Base64 图片并调用视觉模型，提供：

| 方法 | 路径 | 说明 |
|---|---|---|
| POST | /api/vision/recognize | 返回建筑名称、介绍和位置信息 |
| POST | /api/vision/scene-qa | 围绕图片继续提问 |

移动端相机接入与识别结果卡片仍处于原型阶段，服务端接口已按独立路由与服务层拆分。

### 3. 团队协作

- 按 RAG、Vision 划分路由层与服务层，统一接口返回结构
- 参与功能分支 Review、冲突处理与代码合并
- 与 Flutter、Java 模块约定接口路径和数据结构
- 维护团队分工、接口清单和本地启动说明

## 当前实现状态

| 能力 | 状态 | 说明 |
|---|---|---|
| Flutter 基础页面与导航 | 原型完成 | AI 对话和 AR 页面仍需接入真实接口 |
| Vue 管理后台 | 基础页面已搭建 | 需要继续补充真实数据联调 |
| Spring Boot 业务模块 | 基础接口已搭建 | 本地运行依赖 MySQL 与 Redis |
| RAG 问答服务 | 基础链路可用 | 当前为关键词召回，向量检索待升级 |
| 视觉模型服务 | 接口已实现 | 移动端相机与结果展示待联调 |
| 自动化测试与部署 | 待完善 | 计划补充接口测试和容器化 |

把实现状态写清楚，是为了区分“已经可以演示的链路”和“下一阶段规划”，避免 README 与代码不一致。

## 目录结构

~~~text
SmartCampusGuide/
├── 1_CampusApp_Flutter/    # Flutter 移动端
├── 2_AdminWeb_Vue/         # Vue 3 管理后台
├── 3_Backend_Java/         # Spring Boot 业务后端
├── 4_AIService_Python/     # FastAPI AI 微服务
├── 团队分工方案.md
└── README.md
~~~

## 快速启动

### 1. AI 微服务

~~~bash
cd 4_AIService_Python
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn main:app --reload --port 5000
~~~

在 .env 中填写模型服务配置。API Key 只应保存在本地 .env，不要提交到仓库。

### 2. Java 业务后端

先准备 MySQL 与 Redis，并通过环境变量配置数据库：

~~~powershell
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "your_password"
$env:AI_SERVICE_URL = "http://localhost:5000"

cd 3_Backend_Java
mvn spring-boot:run
~~~

默认端口为 8080。

### 3. Vue 管理后台

~~~bash
cd 2_AdminWeb_Vue
npm install
npm run dev
~~~

### 4. Flutter 移动端

~~~bash
cd 1_CampusApp_Flutter
flutter pub get
flutter run
~~~

## 配置与安全

- 模型 API Key、TTS 密钥和数据库密码均通过环境变量注入
- .env、虚拟环境、Chroma 本地数据和构建产物已加入 .gitignore
- 仓库提供 .env.example，仅保留字段说明，不包含真实凭据
- application.yml 不再提供数据库密码默认值

## 后续计划

- 使用 Chroma / Embedding 升级语料召回，并建立问答评测集
- 完成 Flutter AI 对话与相机识别的真实接口联调
- 为 RAG、Vision 和业务接口补充自动化测试
- 增加 Docker Compose，统一 MySQL、Redis、Java 与 Python 的启动流程
- 补充可公开的演示截图与短视频

## 说明

这是一个教学与团队协作项目，当前以可运行原型和模块化设计为主。若用于真实校园服务，还需要补充权限控制、内容审核、模型评测、监控告警和隐私合规方案。
