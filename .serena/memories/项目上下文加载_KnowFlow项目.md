# KnowFlow 项目上下文加载完成

## 项目基本信息
- **项目名称**: KnowFlow
- **项目类型**: 基于 RAGFlow 的企业级智能知识库解决方案
- **架构模式**: 分布式微服务架构
- **主要技术栈**: Python (Flask) + Vue3/React + TypeScript
- **包管理工具**: 强制使用 pnpm (前端)

## 核心服务组件
1. **RAGFlow 核心服务**: 检索增强生成框架 (端口: 80/443, 9380)
2. **KnowFlow 扩展服务**: 企业级功能增强 (端口: 5000)
3. **MinerU 解析引擎**: 文档智能解析服务 (端口: 8888, 30000)
4. **Dots 解析服务**: 复杂版式文档解析 (端口: 8000)
5. **Gotenberg 文档转换**: 多格式文档转换 (端口: 3000)

## 主要功能模块
- 用户权限管理 (RBAC)
- 文档解析引擎 (MinerU/Dots)
- 知识库管理
- 企业集成 (企业微信、LDAP/SSO)

## 开发环境要求
- Python 3.9+ (后端)
- Node.js 16+ (前端)
- pnpm (前端包管理，强制要求)
- Docker & Docker Compose (部署)

## 项目结构
- `knowflow/server/`: KnowFlow 后端服务 (Flask)
- `knowflow/web/`: KnowFlow 前端 (Vue3 + Element Plus)
- `web/`: RAGFlow 前端 (React + Umi + Ant Design)
- `api/`: RAGFlow API 服务
- `docker/`: 部署配置
- `knowflow/mineru/`, `knowflow/dots/`: 文档解析服务

## 重要配置文件
- `knowflow/server/services/config/settings.yaml`: KnowFlow 主配置
- `docker/.env`: 环境变量配置
- `conf/service_conf.yaml`: RAGFlow 配置

## 数据库
- 使用 MySQL 作为主数据库
- 与 RAGFlow 共享数据层

## 默认账户
- 邮箱: admin@gmail.com
- 密码: admin

## Claude 交互规范
- 必须使用中文进行所有交流和文档编写
- 所有生成的文档必须存储在 `.claude/` 目录下
- 前端开发强制使用 pnpm 包管理工具

## 会话状态
- 项目已激活: KnowFlow
- 上下文已加载完成
- 内存已初始化
- 准备进行开发工作