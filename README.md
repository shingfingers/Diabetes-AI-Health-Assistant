<div align="center">

# 🩺 糖尿病 AI 健康助手

> **Diabetes AI Health Assistant** —— 基于 **LLM + Agent + RAG + LangGraph** 的智能慢病管理系统：7×24 小时个性化血糖监测、饮食指导、用药管理、运动规划与 AI 健康咨询
>
> 后端 FastAPI + Tortoise-ORM + MySQL，AI 编排 LangGraph + ChromaDB（RAG），前端 Vue 3 + Vite

[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-Agent-orange)](https://github.com/langchain-ai/langgraph)
[![ChromaDB](https://img.shields.io/badge/RAG-ChromaDB-ff69b4)](https://www.trychroma.com/)
[![Vue](https://img.shields.io/badge/Vue-3-brightgreen)](https://vuejs.org/)
[![MySQL](https://img.shields.io/badge/MySQL-8-blue)](https://www.mysql.com/)
[![License](https://img.shields.io/badge/License-MIT-blue)](#)

</div>

---

## 📖 项目简介

糖尿病是全球最常见的慢性病之一，患者需要长期监测血糖、管理饮食、按时用药、合理运动。传统慢病管理方式高度依赖患者自觉性和定期就医，存在 **数据碎片化、缺乏个性化指导、就医频次有限、依从性差** 等痛点。

本系统通过 AI Agent 技术，整合患者的血糖、饮食、用药、运动等多维数据，结合糖尿病医学指南与营养学知识库（RAG），通过 **LangGraph 编排多步 Agent 流程**，为每位患者生成个性化的健康管理方案和实时健康咨询。

---

## ✨ 功能特性

- 📊 **血糖监测与分析**：记录血糖数据，趋势分析，异常预警（高 / 低血糖）
- 🍽️ **智能饮食管理**：根据血糖数据推荐食谱，计算碳水化合物与 GI 值，每日营养统计
- 💊 **用药提醒与管理**：个性化用药方案，用药频率 / 时机管理，在用状态跟踪
- 🏃 **运动健康指导**：运动记录与运动前后血糖对比，评估运动疗法有效性
- 📝 **健康报告生成**：定期生成健康数据报告，由 LLM 输出专家级建议
- 🤖 **AI 智能问诊**：基于 RAG 的医学知识问答，结合患者档案给出个性化回答
- 📈 **数据可视化**：血糖趋势图、饮食营养分析、健康评分仪表盘
- 🧠 **AI 工作流编排**：意图识别 → 患者档案 → 知识检索 → LLM 生成 → 风险评估

---

## 🧠 AI 工作流（LangGraph）

将健康管理拆解为多步节点，通过状态机（`StateGraph`）编排，而非一次性把问题丢给大模型：

```text
意图识别 ──▶ 获取患者档案 ──▶ 知识库检索(RAG) ──▶ LLM 生成建议 ──▶ 风险评估与警告追加
 classify      load_context        retrieve            generate            assess_risk
```

| 节点 | 职责 |
|---|---|
| `classify_input` | 关键词意图识别：血糖 / 饮食 / 用药 / 通用健康咨询 |
| `load_patient_context` | 读取患者档案 + 近 7 天血糖统计摘要 |
| `retrieve_knowledge` | 按意图分类从 ChromaDB 语义检索医学知识（RAG） |
| `analyze_and_generate` | 组合「患者上下文 + 检索结果」调用 LLM 生成个性化建议 |
| `risk_assessment` | 依据血糖风险等级在回复末尾追加风险警告 |

> **RAG 医学知识增强**：知识库内置糖尿病防治指南、营养学、用药、并发症等条目，通过 `paraphrase-multilingual-MiniLM-L12-v2` 向量化后存入 ChromaDB，问答时按余弦相似度召回，避免 LLM 幻觉、让回答有据可依。

---

## 🧱 技术栈

| 层 | 技术 |
|---|---|
| Web 框架 | [FastAPI](https://fastapi.tiangolo.com/) + Uvicorn |
| ORM / 数据库 | [Tortoise-ORM](https://tortoise.readthedocs.io/) + asyncmy + MySQL 8 |
| 数据校验 | Pydantic v2（`from_attributes` 自动序列化模型 `@property`） |
| AI 应用编排 | [LangGraph](https://github.com/langchain-ai/langgraph) `StateGraph` |
| 大模型调用 | [LangChain](https://python.langchain.com/) + OpenAI 兼容协议（DeepSeek / 通义千问 / OpenAI 均可） |
| RAG / 向量库 | [ChromaDB](https://www.trychroma.com/) + sentence-transformers 本地向量模型 |
| 项目管理 | [uv](https://github.com/astral-sh/uv) |
| 前端 | Vue 3 + TypeScript + Vite + Vue Router + Pinia + Axios + Element Plus + ECharts |

---

## 🏗️ 模块说明

### 后端（FastAPI，`app/`）

| 模块 | 说明 |
|---|---|
| `main.py` | 应用入口：注册路由、CORS、生命周期（启动时初始化医学知识库） |
| `app/config.py` | 环境变量加载（`.env`）与 Tortoise-ORM 配置 |
| `app/models/database.py` | Tortoise-ORM 生命周期注册 |
| `app/models/` | 7 张表模型：`Patient` / `BloodSugar` / `DietRecord` / `Medication` / `Exercise` / `HealthReport` / `ChatHistory` |
| `app/schemas/` | 请求 / 响应 Pydantic 模型（数据校验与安全隔离） |
| `app/routers/` | 路由：患者 / 血糖 / 饮食 / 用药 / 运动 / 报告 / 对话 |
| `app/core/health_analyzer.py` | 血糖数学统计、趋势识别、达标率与风险等级评估 |
| `app/core/diet_advisor.py` | 每日营养摄入分析、GI 评估与个性化食谱推荐 |
| `app/core/llm_client.py` | LLM 客户端封装（含对话历史裁剪、异常兜底） |
| `app/core/vector_store.py` | ChromaDB 向量库：文档嵌入存储与语义检索 |
| `app/core/knowledge_base.py` | 医学知识库定义、初始化与检索 |
| `app/core/rag_engine.py` | RAG 引擎：召回 → 阈值过滤 → 引用格式化 |
| `app/core/langgraph_agent.py` | LangGraph Agent 工作流编排与对外服务 `run_agent` |

### 前端（Vue 3 + TS，`frontend/`）

| 页面 / 组件 | 说明 |
|---|---|
| `Dashboard` | 仪表盘：健康评分、血糖趋势、关键指标总览 |
| `PatientManage` | 患者档案管理（增删改查、BMI 展示） |
| `BloodSugar` | 血糖记录录入与趋势图表、统计卡片 |
| `DietManage` | 饮食记录、每日营养统计、食谱推荐 |
| `Medication` | 用药方案管理 |
| `Exercise` | 运动记录与运动前后血糖对比 |
| `HealthReport` | 健康报告生成与历史查询 |
| `AIChat` | AI 智能问诊对话界面（多轮会话） |
| `components/` | `Navbar` / `Sidebar` / `BloodSugarChart` / `ChatMessage` / `DietCard` / `HealthScore` / `MedicationReminder` |
| `api/` `stores/` | Axios 接口封装与 Pinia 状态管理 |

---

## 📂 目录结构

```text
diabetes_assistant/
├── main.py                    # 后端应用入口（FastAPI + Uvicorn）
├── pyproject.toml             # 依赖与项目元信息（uv 管理）
├── .env.example               # 环境变量模板（不含真实密钥）
├── app/                       # 后端源码
│   ├── config.py              #   环境变量 / Tortoise-ORM 配置
│   ├── core/                  #   核心引擎：健康分析 / 饮食建议 / LLM / RAG / Agent
│   ├── models/                #   Tortoise-ORM 数据模型（7 张表）
│   ├── schemas/               #   Pydantic 请求 / 响应模型
│   └── routers/               #   API 路由
├── frontend/                  # 前端源码（Vue 3 + TS + Vite）
│   ├── src/
│   │   ├── views/             #   Dashboard / BloodSugar / Diet / AIChat ...
│   │   ├── components/        #   图表 / 卡片 / 导航等复用组件
│   │   ├── api/               #   Axios 接口封装
│   │   ├── stores/            #   Pinia 状态管理
│   │   └── router/            #   前端路由
│   └── package.json
├── data/chroma_db/            # 向量库本地持久化（首次启动自动生成，已忽略）
└── screenshots/               # README 界面截图（占位，自行补充）
```

---

## 🔌 API 接口一览

统一前缀 `/api`，Swagger 文档见 `/docs`。

| 模块 | 方法与路径 | 说明 |
|---|---|---|
| 患者 | `GET /api/patients/` | 患者列表（分页） |
| | `GET /api/patients/{id}` | 患者详情 |
| | `POST /api/patients/` | 新建患者 |
| | `PUT /api/patients/{id}` | 更新患者 |
| | `DELETE /api/patients/{id}` | 删除患者 |
| 血糖 | `POST /api/blood-sugar/` | 新增血糖记录 |
| | `GET /api/blood-sugar/patient/{id}?days=` | 查询患者近期血糖记录 |
| | `GET /api/blood-sugar/patient/{id}/stats?days=` | 血糖统计（均值 / 达标率 / 趋势 / 风险） |
| | `PUT /api/blood-sugar/{record_id}` | 更新血糖记录 |
| | `DELETE /api/blood-sugar/{record_id}` | 删除血糖记录 |
| 饮食 | `POST /api/diet/` | 新增饮食记录 |
| | `GET /api/diet/patient/{id}/records` | 患者饮食记录 |
| | `GET /api/diet/patient/{id}/daily?date=` | 每日营养统计 |
| | `GET /api/diet/recommend/{meal_type}?patient_id=` | 食谱推荐（结合近期血糖） |
| | `PUT /api/diet/{record_id}` | 更新饮食记录 |
| | `DELETE /api/diet/{record_id}` | 删除饮食记录 |
| 用药 | `POST /api/medication/` | 新增用药记录 |
| | `GET /api/medication/patient/{id}?active_only=` | 患者用药记录 |
| | `PUT /api/medication/{id}` | 更新用药记录 |
| | `DELETE /api/medication/{id}` | 删除用药记录 |
| 运动 | `POST /api/exercise/` | 新增运动记录 |
| | `GET /api/exercise/patient/{id}` | 患者运动记录 |
| | `PUT /api/exercise/{record_id}` | 更新运动记录 |
| | `DELETE /api/exercise/{record_id}` | 删除运动记录 |
| 报告 | `POST /api/reports/generate/{id}?report_type=` | 生成健康报告（周报 / 月报） |
| | `GET /api/reports/patient/{id}` | 历史健康报告 |
| 对话 | `POST /api/chat/message` | AI 智能问诊（触发 LangGraph 工作流） |
| | `GET /api/chat/history/{id}` | 患者对话历史 |

---

## 🗄️ 数据库设计

MySQL 8，库名 `diabetes`，共 7 张表，均通过外键关联 `patients`：

| 表 | 说明 |
|---|---|
| `patients` | 患者基础信息（身高 / 体重 / 病史 / 紧急联系人），含 BMI 计算属性 |
| `blood_sugar` | 血糖记录（数值 / 测量时段 / 测量时间），含血糖状态判定 |
| `diet_records` | 饮食记录（食物 / 热量 / 碳水 / 蛋白 / 脂肪 / GI 值），含 GI 等级判定 |
| `medications` | 用药方案（药品 / 剂量 / 频次 / 时机 / 在用状态） |
| `exercise_records` | 运动记录（项目 / 时长 / 强度 / 运动前后血糖），含血糖变化计算 |
| `health_reports` | 健康报告（周期 / 平均血糖 / 波动标准差 / 达标率 / 风险等级 / AI 建议） |
| `chat_history` | 对话历史（角色 / 内容 / 时间），为多轮上下文提供基础 |

---

## 🚀 部署与运行

### 前置依赖

| 依赖 | 说明 |
|---|---|
| Python 3.12+ | 后端运行环境（推荐用 [uv](https://github.com/astral-sh/uv) 管理） |
| Node.js 18+ | 前端构建 / 运行 |
| MySQL 8 | 主数据库，需提前创建 `diabetes` 库并导入建表 SQL |
| API Key | 任一 OpenAI 兼容的大模型服务（DeepSeek / 通义千问 / OpenAI 等），密钥仅存于 `.env`，**不提交版本库** |

### 快速开始

```bash
cp .env.example .env             # 填写数据库连接与 LLM 配置
```

在 `.env` 中填写 `DB_*` 数据库连接与 `LLM_API_KEY` / `LLM_BASE_URL` / `LLM_MODEL`；并在 MySQL 中创建 `diabetes` 库、导入建表 SQL（表结构见上方「数据库设计」）。

```bash
# 后端
uv sync
uv run uvicorn main:app --port 8000

# 前端
cd frontend
npm install && npm run dev
```

接口文档见 `/docs`。

### 生产部署

后端以 Uvicorn / Gunicorn 多进程方式部署，前端 `npm run build` 产物托管至静态资源服务或 CDN，由 Nginx 反向代理统一对外，并将 `/api` 转发至后端服务。

---

## 🔌 端口速查

- 后端 FastAPI：**8000**
- 前端开发服务器：**3000**
- MySQL：**3306**

---

## 🖼️ 界面展示

#### 仪表盘 / 健康总览
<p>
  ![Uploading 209928f1e77e9c47c6e02e017e7ae9a8.png…]()

</p>

#### 血糖监测与趋势
<p>
  <!-- 放一张血糖页截图：<img src="screenshots/blood-sugar.png" width="800" alt="血糖监测"> -->
</p>

#### 饮食管理
<p>
  <!-- 放一张饮食页截图：<img src="screenshots/diet.png" width="800" alt="饮食管理"> -->
</p>

#### 用药 / 运动管理
<p>
  <!-- <img src="screenshots/medication.png" width="400" alt="用药管理"> -->
  <!-- <img src="screenshots/exercise.png" width="400" alt="运动管理"> -->
</p>

#### 健康报告
<p>
  <!-- 放一张报告页截图：<img src="screenshots/report.png" width="800" alt="健康报告"> -->
</p>

#### AI 智能问诊
<p>
  <!-- 放一张 AI 对话截图：<img src="screenshots/ai-chat.png" width="800" alt="AI 智能问诊"> -->
</p>

---

## ⚠️ 说明

- `.env`（含数据库密码与 API Key）已被 `.gitignore` 忽略，请勿提交真实密钥。
- `data/chroma_db/`（向量库持久化文件）与 `frontend/node_modules/` 同样不纳入版本管理，首次启动 / 安装依赖后自动生成。
- 首次启动会自动初始化向量库（嵌入模型加载一次后常驻，医学知识库向量化落盘）；国内网络下代码默认使用 `hf-mirror.com` 镜像。
- 本项目为学习 / 演示用途，AI 给出的建议不能替代专业医生诊断。
