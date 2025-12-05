# KnowFlow 项目 Cursor 本地调试详细方案

## 📋 项目概述

**项目名称**: KnowFlow 企业级智能知识库解决方案
**架构类型**: 分布式微服务架构
**调试目标**: 使用 Cursor IDE 进行本地断点调试
**方案类型**: 混合调试方案（Docker + 本地调试）

## 🎯 调试方案选择

### 推荐方案：混合调试模式
- **基础服务**: 使用 Docker 运行（MySQL、Redis、MinIO、Elasticsearch、文档解析服务）
- **应用服务**: 本地调试运行（RAGFlow + KnowFlow 后端和前端）
- **优势**: 简化环境配置，保持调试灵活性，降低系统复杂度

## 🛠️ 环境准备阶段

### 1. 系统要求检查
```bash
# 检查 Python 版本（需要 3.9+）
python3 --version

# 检查 Node.js 版本（需要 16+）
node --version

# 检查 pnpm 安装
pnpm --version

# 检查 Docker 和 Docker Compose
docker --version
docker-compose --version
```

### 2. 端口占用检查
确保以下端口未被占用：
- 80/443: RAGFlow 前端
- 3000: Gotenberg 文档转换
- 5000: KnowFlow 后端
- 8000: Dots 解析服务
- 8888: MinerU API 服务
- 9380: RAGFlow 后端
- 30000: MinerU VLM 服务
- 3306: MySQL
- 6379: Redis
- 9200: Elasticsearch
- 9000: MinIO

### 3. 项目目录准备
```bash
# 进入项目根目录
cd /Users/michaelche/Documents/git-folder/github-folder/KnowFlow

# 备份原始配置文件
cp docker/.env docker/.env.backup
cp knowflow/server/services/config/settings.yaml knowflow/server/services/config/settings.yaml.backup
```

## 🐳 基础设施启动阶段

### 1. 启动基础 Docker 服务
```bash
# 启动基础服务（MySQL、Redis、MinIO、Elasticsearch）
docker compose -f docker/docker-compose.yml up -d mysql redis minio elasticsearch

# 验证服务状态
docker compose -f docker/docker-compose.yml ps
```

### 2. 启动文档解析服务
```bash
# 启动 Gotenberg 服务
docker run -d --name gotenberg \
  -p 3000:3000 \
  gotenberg/gotenberg:7

# 启动 MinerU 服务（推荐方式）
docker run -d \
    --gpus all \
    -p 8888:8888 \
    -p 30000:30000 \
    -e MINERU_MODEL_SOURCE=local \
    -e SGLANG_MEM_FRACTION_STATIC=0.8 \
    --name mineru-sglang \
    zxwei/mineru-api-full:v2.1.11

# 或者启动 Dots 服务（二选一）
cd knowflow/dots
./deploy.sh
```

### 3. 数据库初始化
```bash
# 等待数据库服务完全启动
sleep 30

# 运行数据库初始化脚本
./docker/init_db.sh

# 设置管理员账户
./docker/set_superuser.sh set admin@example.com
```

### 4. 服务连通性验证
```bash
# 测试数据库连接
docker exec -it knowflow-mysql mysql -uroot -p

# 测试 Redis 连接
docker exec -it knowflow-redis redis-cli ping

# 测试 MinIO 连接
curl http://localhost:9000

# 测试 Elasticsearch 连接
curl http://localhost:9200
```

## ⚙️ 配置文件修改阶段

### 1. 环境变量配置
编辑 `docker/.env` 文件：
```env
# 数据库配置
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DB_NAME=ragflow

# Redis 配置
REDIS_HOST=localhost
REDIS_PORT=6379

# MinIO 配置
MINIO_HOST=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# Elasticsearch 配置
ES_HOST=localhost
ES_PORT=9200

# RAGFlow 配置
RAGFLOW_HOST=0.0.0.0
RAGFLOW_PORT=9380

# KnowFlow 配置
KNOWFLOW_HOST=0.0.0.0
KNOWFLOW_PORT=5000
```

### 2. KnowFlow 配置修改
编辑 `knowflow/server/services/config/settings.yaml`：
```yaml
database:
  host: localhost
  port: 3306
  user: root
  password: rootpassword
  database: ragflow

redis:
  host: localhost
  port: 6379

minio:
  host: localhost
  port: 9000
  access_key: minioadmin
  secret_key: minioadmin

elasticsearch:
  host: localhost
  port: 9200

ragflow:
  api_url: http://localhost:9380
```

### 3. RAGFlow 配置修改
编辑 `conf/service_conf.yaml`：
```yaml
mysql:
  host: localhost
  port: 3306
  user: root
  password: rootpassword
  database: ragflow

redis:
  host: localhost
  port: 6379

minio:
  host: localhost
  port: 9000
  access_key: minioadmin
  secret_key: minioadmin

es:
  hosts: ["http://localhost:9200"]
```

## 🔧 Cursor 调试配置阶段

### 1. 创建 `.vscode/launch.json` 配置文件

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug RAGFlow Backend",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/api/db.py",
            "args": ["-p", "9380"],
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}",
            "env": {
                "PYTHONPATH": "${workspaceFolder}",
                "RAGFLOW_DEBUG": "true"
            },
            "justMyCode": false
        },
        {
            "name": "Debug KnowFlow Backend",
            "type": "python",
            "request": "launch",
            "program": "${workspaceFolder}/knowflow/server/app.py",
            "console": "integratedTerminal",
            "cwd": "${workspaceFolder}/knowflow/server",
            "env": {
                "FLASK_DEBUG": "true",
                "FLASK_ENV": "development"
            },
            "justMyCode": false
        },
        {
            "name": "Debug RAGFlow Frontend",
            "type": "node",
            "request": "launch",
            "program": "${workspaceFolder}/web/node_modules/.bin/umi",
            "args": ["dev"],
            "cwd": "${workspaceFolder}/web",
            "console": "integratedTerminal",
            "env": {
                "NODE_ENV": "development"
            }
        },
        {
            "name": "Debug KnowFlow Frontend",
            "type": "node",
            "request": "launch",
            "program": "${workspaceFolder}/knowflow/web/node_modules/.bin/vite",
            "args": ["--mode", "development"],
            "cwd": "${workspaceFolder}/knowflow/web",
            "console": "integratedTerminal",
            "env": {
                "NODE_ENV": "development"
            }
        }
    ],
    "compounds": [
        {
            "name": "Debug Full Stack (RAGFlow + KnowFlow)",
            "configurations": [
                "Debug RAGFlow Backend",
                "Debug KnowFlow Backend",
                "Debug RAGFlow Frontend",
                "Debug KnowFlow Frontend"
            ],
            "stopAll": true
        },
        {
            "name": "Debug Backend Services Only",
            "configurations": [
                "Debug RAGFlow Backend",
                "Debug KnowFlow Backend"
            ],
            "stopAll": true
        }
    ]
}
```

### 2. 创建 `.vscode/tasks.json` 配置文件

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Start Docker Services",
            "type": "shell",
            "command": "docker-compose",
            "args": ["-f", "docker/docker-compose.yml", "up", "-d", "mysql", "redis", "minio", "elasticsearch"],
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            }
        },
        {
            "label": "Start Document Services",
            "type": "shell",
            "command": "docker",
            "args": ["run", "-d", "--name", "gotenberg", "-p", "3000:3000", "gotenberg/gotenberg:7"],
            "group": "build",
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            }
        },
        {
            "label": "Install RAGFlow Dependencies",
            "type": "shell",
            "command": "pnpm",
            "args": ["install"],
            "options": {
                "cwd": "${workspaceFolder}/web"
            },
            "group": "build"
        },
        {
            "label": "Install KnowFlow Dependencies",
            "type": "shell",
            "command": "pnpm",
            "args": ["install"],
            "options": {
                "cwd": "${workspaceFolder}/knowflow/web"
            },
            "group": "build"
        }
    ]
}
```

### 3. 创建 `.vscode/settings.json` 配置文件

```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "python.formatting.provider": "black",
    "typescript.preferences.importModuleSpecifier": "relative",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    },
    "files.associations": {
        "*.yaml": "yaml",
        "*.yml": "yaml"
    },
    "search.exclude": {
        "**/node_modules": true,
        "**/.venv": true,
        "**/docker": true,
        "**/dist": true,
        "**/build": true
    }
}
```

## 🚀 应用服务调试启动阶段

### 1. Python 环境准备
```bash
# 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 安装 RAGFlow 后端依赖
pip install -r requirements.txt

# 安装 KnowFlow 后端依赖
cd knowflow/server
pip install -r requirements.txt
cd ../..
```

### 2. 前端依赖安装
```bash
# 安装 RAGFlow 前端依赖
cd web
pnpm install
cd ..

# 安装 KnowFlow 前端依赖
cd knowflow/web
pnpm install
cd ../..
```

### 3. 分步调试启动

#### 步骤 1: 启动 RAGFlow 后端调试
1. 在 Cursor 中打开 `api/db.py` 文件
2. 设置断点在关键函数上
3. 按 `F5` 或使用调试面板选择 "Debug RAGFlow Backend"
4. 观察调试控制台输出

#### 步骤 2: 启动 KnowFlow 后端调试
1. 在 Cursor 中打开 `knowflow/server/app.py` 文件
2. 设置断点在路由处理函数上
3. 按 `F5` 或使用调试面板选择 "Debug KnowFlow Backend"
4. 验证后端 API 连接

#### 步骤 3: 启动前端调试
1. 分别选择 "Debug RAGFlow Frontend" 和 "Debug KnowFlow Frontend"
2. 在浏览器中访问对应的前端页面
3. 使用开发者工具进行前端调试

#### 步骤 4: 全栈调试
1. 使用 "Debug Full Stack (RAGFlow + KnowFlow)" 复合配置
2. 同时启动所有服务进行集成调试

## 🔍 调试验证阶段

### 1. 后端 API 测试
```bash
# 测试 RAGFlow API
curl -X GET http://localhost:9380/v1/ping

# 测试 KnowFlow API
curl -X GET http://localhost:5000/health
```

### 2. 前端页面访问
- RAGFlow 前端: http://localhost:8000 (或其他配置的端口)
- KnowFlow 前端: http://localhost:5173 (Vite 默认端口)

### 3. 数据库连接验证
```bash
# 连接数据库查看表结构
docker exec -it knowflow-mysql mysql -uroot -p -e "USE ragflow; SHOW TABLES;"
```

## 🛠️ 常见问题排查

### 1. 端口冲突问题
```bash
# 查看端口占用
lsof -i :端口号

# 修改配置文件中的端口设置
# 或者终止占用端口的进程
kill -9 <PID>
```

### 2. 数据库连接失败
- 检查 MySQL 服务是否启动
- 验证数据库连接参数
- 确认数据库初始化完成

### 3. 前端构建失败
- 检查 Node.js 版本兼容性
- 清理 node_modules 重新安装
- 检查 pnpm 版本

### 4. 文档解析服务不可用
- 验证 Docker 容器运行状态
- 检查 MinerU/Dots 服务端口
- 查看服务日志输出

### 5. 调试断点不生效
- 检查 Python 虚拟环境配置
- 验证调试配置文件路径
- 确认代码映射正确

## 📝 调试最佳实践

### 1. 调试工作流
1. **先启动基础服务**：确保 Docker 服务正常运行
2. **分步验证**：逐个启动服务进行验证
3. **日志监控**：实时查看服务日志输出
4. **断点设置**：在关键业务逻辑设置断点

### 2. 调试技巧
- 使用条件断点进行复杂场景调试
- 利用监视窗口跟踪变量状态
- 使用调用堆栈分析程序执行流程
- 配置日志级别获取详细调试信息

### 3. 性能监控
- 监控内存使用情况
- 跟踪 API 响应时间
- 观察数据库查询性能
- 分析前端加载时间

## 🎯 快速启动命令序列

```bash
# 一键启动所有基础服务
docker compose -f docker/docker-compose.yml up -d

# 启动文档解析服务
docker run -d --name gotenberg -p 3000:3000 gotenberg/gotenberg:7

# 激活 Python 虚拟环境
source .venv/bin/activate

# 安装依赖
pip install -r requirements.txt
cd web && pnpm install && cd ../knowflow/web && pnpm install && cd ../..

# 在 Cursor 中按 F5 开始调试
```

## 📋 调试检查清单

### 启动前检查
- [ ] Python 版本 ≥ 3.9
- [ ] Node.js 版本 ≥ 16
- [ ] pnpm 已安装
- [ ] Docker 服务运行正常
- [ ] 端口未被占用
- [ ] 配置文件已备份

### 启动过程检查
- [ ] 基础 Docker 服务启动成功
- [ ] 数据库初始化完成
- [ ] 文档解析服务运行正常
- [ ] Python 虚拟环境激活
- [ ] 前端依赖安装成功

### 调试验证检查
- [ ] 后端 API 响应正常
- [ ] 前端页面可访问
- [ ] 断点调试功能正常
- [ ] 服务间通信正常
- [ ] 数据库读写正常

## 🔗 相关资源

- [RAGFlow 官方文档](https://ragflow.io/)
- [MinerU 项目地址](https://github.com/opendatalab/MinerU)
- [Cursor 官方文档](https://cursor.sh/)
- [Python 调试指南](https://code.visualstudio.com/docs/python/debugging)
- [Vue.js 调试指南](https://vuejs.org/guide/scaling-up/debugging.html)

---

**注意事项**:
- 首次调试可能需要额外时间初始化数据库
- 建议定期清理 Docker 容器避免资源占用
- 调试过程中注意观察日志输出定位问题
- 保持配置文件的一致性避免环境差异