# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🔒 强制规则

**必须使用中文回答所有问题和对话**

在使用此项目时，Claude Code 必须遵循以下强制规则：
- 🇨🇳 **语言要求**：所有回答、解释、注释和沟通必须使用中文
- ❌ **禁止使用英文**：不得使用任何英文回复（代码本身的技术术语除外）
- 📝 **文档语言**：所有生成的文档、注释和说明都必须是中文
- 💬 **交流方式**：与用户的所有交互必须使用中文进行
- 📦 **前端包管理**：前端项目必须使用 pnpm 作为包管理工具，禁止使用 npm 或 yarn

这是一项不可协商的强制要求，违反此规则将被视为严重的使用错误。

## 项目概述

KnowFlow 是基于 RAGFlow 的企业级智能知识库解决方案，采用分布式微服务架构。项目作为 RAGFlow 的增强扩展，通过独立服务组件提供用户管理、团队协作、高级文档解析等功能。

## 核心架构

### 系统组成
- **RAGFlow 核心服务**：原生的检索增强生成框架
- **KnowFlow 扩展服务**：企业级功能增强，包括用户管理、权限控制、文档解析等
- **MinerU/Dots 解析引擎**：先进的文档智能解析服务
- **Gotenberg 文档转换**：支持多种文档格式转换

### 端口配置
- RAGFlow 前端：80/443
- RAGFlow 后端：9380
- KnowFlow 后端：5000
- Gotenberg 服务：3000
- MinerU API 服务：8888
- MinerU VLM 服务：30000
- Dots 服务：8000

## 常用开发命令

### Docker 部署（推荐）

#### 启动完整服务
```bash
# 有 GPU 环境
docker compose -f docker/docker-compose-gpu.yml up -d

# 无 GPU 环境
docker compose -f docker/docker-compose.yml up -d
```

#### 启动文档解析服务（任选一种）
```bash
# MinerU 服务（推荐）
docker run -d \
    --gpus all \
    -p 8888:8888 \
    -p 30000:30000 \
    -e MINERU_MODEL_SOURCE=local \
    -e SGLANG_MEM_FRACTION_STATIC=0.8 \
    --name mineru-sglang \
    zxwei/mineru-api-full:v2.1.11

# Dots 服务
cd knowflow/dots
./deploy.sh
```

#### 管理员权限设置
```bash
./docker/set_superuser.sh set admin@example.com
```

### 源码部署

#### KnowFlow 后端
```bash
cd knowflow/server
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
cd ../
./scripts/install.sh --local
python3 app.py
```

#### RAGFlow 前端
```bash
cd web
pnpm install
pnpm dev
```

#### RAGFlow 后端
```bash
source .venv/bin/activate
export PYTHONPATH=$(pwd)
./local_entrypoint.sh
```

### 测试和构建

#### 前端测试和构建
```bash
# KnowFlow Web（基于 Vue3）
cd knowflow/web
pnpm test
pnpm build
pnpm lint

# RAGFlow Web（基于 Umi/React）
cd web
pnpm test
pnpm build
pnpm lint
```

#### 代码质量检查
```bash
# ESLint 检查
cd knowflow/web
pnpm lint

cd web
pnpm lint
```

## 项目结构

### 后端服务
- `knowflow/server/`：KnowFlow 后端服务（Flask）
  - `app.py`：应用入口
  - `models/`：数据模型（RBAC、用户管理等）
  - `routes/`：API 路由
  - `services/`：业务逻辑服务
- `api/`：RAGFlow API 服务

### 前端应用
- `knowflow/web/`：KnowFlow 前端（Vue3 + Element Plus）
- `web/`：RAGFlow 前端（React + Umi + Ant Design）

### 文档解析服务
- `knowflow/mineru/`：MinerU 解析服务
- `knowflow/dots/`：Dots 解析服务
- `knowflow/server/services/knowledgebases/`：文档解析和知识库管理

### 配置和部署
- `docker/`：Docker Compose 部署配置
- `knowflow/server/services/config/`：服务配置文件
- `conf/`：RAGFlow 配置

## 核心功能模块

### 1. 用户权限管理（RBAC）
- 基于角色的访问控制
- 用户、团队、权限管理
- JWT 认证机制

### 2. 文档解析引擎
- **MinerU 解析**：OCR 文字识别、图像提取、图文混排
- **Dots 解析**：小红书文档解析、复杂版式支持
- **多种分块策略**：文档结构分块、标题分块、父子分块等

### 3. 知识库管理
- 文档上传和解析
- 多模态内容理解
- 向量化和检索

### 4. 企业集成
- 企业微信集成
- LDAP/SSO 支持
- API Token 管理

## 开发注意事项

### 配置管理
- 主要配置文件：`knowflow/server/services/config/settings.yaml`
- 环境变量：`docker/.env`
- RAGFlow 配置：`conf/service_conf.yaml`

### 数据库
- 使用 MySQL 作为主数据库
- 与 RAGFlow 共享数据层
- 包含用户、权限、知识库等表

### 服务依赖
- KnowFlow 依赖 RAGFlow 核心服务
- 需要配置正确的基础设施地址（MySQL、Redis、MinIO、Elasticsearch）
- MinerU/Dots 服务需要独立部署

### 开发环境
- Python 3.9+（后端）
- Node.js 16+（前端）
- pnpm（前端包管理）
- Docker & Docker Compose（部署）

### 前端开发强制规则

#### 📦 包管理工具
- **强制使用 pnpm**：所有前端项目必须使用 pnpm 作为包管理工具
- **禁止使用 npm/yarn**：严禁在前端项目中使用 npm 或 yarn 命令
- **统一依赖管理**：pnpm 能确保依赖版本一致性和磁盘空间优化

#### pnpm 常用命令
```bash
# 安装依赖
pnpm install

# 添加依赖
pnpm add <package-name>          # 生产依赖
pnpm add -D <package-name>       # 开发依赖

# 运行脚本
pnpm dev                         # 开发服务器
pnpm build                       # 生产构建
pnpm test                        # 运行测试
pnpm lint                        # 代码检查

# 全局操作
pnpm -g add <package-name>       # 全局安装
pnpm -g list                     # 全局包列表
```

#### 前端项目结构
- **KnowFlow Web**：`knowflow/web/` （Vue3 + Element Plus）
- **RAGFlow Web**：`web/` （React + Umi + Ant Design）

#### 依赖管理规范
- 所有包安装必须通过 pnpm
- 使用 `pnpm.lock.yaml` 锁定依赖版本
- 定期运行 `pnpm audit` 检查安全漏洞
- 遵循语义化版本控制原则

#### ⚠️ 重要注意事项
- **package.json 脚本**：如果 package.json 中的 scripts 仍包含 npm 命令，使用 pnpm 时会自动转换
- **脚本执行**：即使 package.json 中写的是 `npm run dev`，也可以用 `pnpm dev` 执行
- **统一原则**：无论 package.json 中如何写，都要使用 pnpm 命令行工具
- **团队协作**：确保所有团队成员都安装并使用 pnpm

## 默认账户信息
- 邮箱：admin@gmail.com
- 密码：admin

首次登录后请及时修改默认密码。

---

## 📋 Claude 使用规范

### 语言使用
- 所有回复、解释和建议必须使用中文
- 代码注释和文档说明应用中文编写
- 错误信息和调试提示使用中文描述
- 项目相关的所有交流使用中文进行

### 技术支持
- 如遇到问题，请用中文描述具体需求
- 代码审查和优化建议用中文提供
- 架构设计和最佳实践指导使用中文

### 文档生成
- 自动生成的代码注释使用中文
- API 文档和使用说明用中文编写
- 部署指南和配置说明用中文描述

### 📁 文档存储强制规则

**所有 Claude 生成的文档必须存放在 `.claude` 目录下**

为了统一管理和维护项目文档，所有 Claude 生成的文档类型必须按以下规则存储：

#### 核心文档类型
- **需求文档** → `.claude/docs/`
  - 功能需求说明
  - 用户需求分析
  - 业务需求文档

- **方案设计** → `.claude/designs/`
  - 架构设计方案
  - 技术方案文档
  - 实现方案设计
  - 接口设计文档

- **README 文档** → `.claude/readmes/`
  - 项目说明文档
  - 模块使用指南
  - 部署说明文档

#### 其他文档类型
- **系统架构** → `.claude/architecture/`
  - 系统架构分析
  - 技术架构文档
  - 微服务架构设计

- **开发笔记** → `.claude/notes/`
  - 开发过程记录
  - 临时技术笔记
  - 问题解决记录

- **分析报告** → `.claude/analysis/`
  - 代码分析报告
  - 技术调研报告
  - 性能分析文档

- **开发日志** → `.claude/logs/`
  - 开发日志记录
  - 问题追踪文档
  - 变更记录

#### 文档命名规范
- 使用中文命名，清晰表达文档内容
- 格式：`文档名称_版本号_日期.md`
- 示例：`用户认证系统设计方案_v1.0_20241205.md`

#### 强制要求
- ❌ **禁止**将文档随意放置在项目根目录或其他位置
- ✅ **必须**将所有 Claude 生成的文档放入对应的 `.claude` 子目录
- 📝 **建议**定期整理和归档文档，保持目录结构清晰
- 🔍 **便于**团队成员快速查找和维护相关文档

**目录结构示例：**
```
.claude/
├── docs/           # 需求文档
├── designs/        # 方案设计
├── readmes/        # README 文档
├── architecture/   # 系统架构
├── notes/          # 开发笔记
├── analysis/       # 分析报告
└── logs/           # 开发日志
```