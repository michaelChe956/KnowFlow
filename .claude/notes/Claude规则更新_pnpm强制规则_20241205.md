# Claude 规则更新 - pnpm 强制规则

## 更新日期
2024年12月5日

## 更新内容

### 1. 新增强制规则
在 `CLAUDE.md` 的强制规则部分添加：
- 📦 **前端包管理**：前端项目必须使用 pnpm 作为包管理工具，禁止使用 npm 或 yarn

### 2. 修正现有命令
将文档中的所有 npm 命令替换为 pnpm：
- `npm run build` → `pnpm build`
- `npm run lint` → `pnpm lint`
- 统一使用 pnpm 作为前端包管理工具

### 3. 新增前端开发规范
添加了完整的前端开发强制规则章节：

#### 📦 包管理工具规范
- 强制使用 pnpm
- 禁止使用 npm/yarn
- 统一依赖管理

#### pnpm 常用命令指南
提供了完整的 pnpm 命令参考：
- 安装依赖：`pnpm install`
- 添加包：`pnpm add <package-name>`
- 运行脚本：`pnpm dev/build/test/lint`

#### 前端项目结构说明
- KnowFlow Web：`knowflow/web/` （Vue3 + Element Plus）
- RAGFlow Web：`web/` （React + Umi + Ant Design）

#### 依赖管理规范
- 使用 `pnpm.lock.yaml` 锁定版本
- 定期运行 `pnpm audit` 安全检查
- 团队协作要求

### 4. 特别注意事项
针对现有 package.json 中可能存在的 npm 命令引用：
- pnpm 会自动转换 package.json 中的 npm 脚本
- 无论 package.json 如何写，都使用 pnpm 命令行工具
- 确保团队统一使用 pnpm

## 更新原因
1. **依赖管理一致性**：确保团队使用统一的包管理工具
2. **磁盘空间优化**：pnpm 通过硬链接机制节省磁盘空间
3. **依赖安全性**：pnpm 提供更严格的依赖管理
4. **安装速度**：pnpm 通常比 npm/yarn 更快

## 影响范围
- 所有前端开发工作
- 依赖安装和更新流程
- CI/CD 构建流程
- 团队开发环境配置

## 后续行动
1. 确保团队成员安装 pnpm
2. 检查 CI/CD 流程是否需要更新
3. 监控项目迁移到 pnpm 的过程
4. 处理可能出现的兼容性问题

---
*本文档记录了 Claude 规则的更新过程，便于后续追踪和维护*