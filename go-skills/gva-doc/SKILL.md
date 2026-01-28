---
name: gva-doc
description: Gin-Vue-Admin (GVA) documentation reference skill. Use when developing with GVA framework, including backend Go/Gin development, frontend Vue.js, authentication/authorization (JWT+Casbin), code generator, deployment, or troubleshooting GVA-related issues. Provides quick access to official GVA documentation for CRUD generation, RBAC permissions, config management, and more.
metadata:
  version: "2.8.8"
---

# GVA Documentation Reference

Quick access to Gin-Vue-Admin official documentation. GVA is a full-stack admin framework built with Go (Gin) and Vue.js, featuring auto code generation, JWT+Casbin auth, and multi-database support.

## Documentation Index

All docs located in `references/gin-vue-admin.com/docs/guide/`.

### Getting Started

| Topic | File | Description |
|-------|------|-------------|
| Project Overview | `introduce/project.md` | Architecture, tech stack, features |
| Environment Setup | `start-quickly/env.md` | Node.js, Go, MySQL requirements |
| Initialization | `start-quickly/initialization.md` | First-time setup and database init |
| Swagger | `start-quickly/swagger.md` | API documentation setup |
| VSCode Config | `start-quickly/vscode.md` | IDE configuration |
| FAQ | `manual/qa.md` | Common questions |

### Code Generator

| Topic | File | Description |
|-------|------|-------------|
| Usage Guide | `generator/server.md` | Complete code generator tutorial |
| Auto Package | `generator/package.md` | Automatic package generation |
| Form Generator | `generator/web.md` | Web form builder |

### Backend (Go/Gin)

| Topic | File | Description |
|-------|------|-------------|
| Overview | `server/index.md` | Backend directory structure |
| Config | `server/config.md` | config.yaml reference |
| JWT Auth | `server/authentication.md` | Authentication system |
| Casbin RBAC | `server/authorization.md` | Role-based access control |
| Code Generator | `server/code-generator.md` | Backend code generation |
| Database Design | `server/database-design.md` | Table schemas and relations |
| GORM | `server/gorm.md` | ORM usage |
| Multi-DB | `server/multiple-databases.md` | MySQL, PostgreSQL, SQLite |
| OSS | `server/oss.md` | Object storage integration |
| Timer | `server/timer.md` | Scheduled tasks |
| MCP Integration | `server/mcp.md` | AI assistant integration |
| Viper | `server/core/viper.md` | Config management |
| Zap | `server/core/zap.md` | Logging system |

### Frontend (Vue.js)

| Topic | File | Description |
|-------|------|-------------|
| Overview | `web/index.md` | Frontend architecture |
| Environment | `web/env.md` | Environment variables |
| Button Auth | `web/button-auth.md` | Button-level permissions |
| Dictionary | `web/dictionary.md` | Dictionary methods |
| Theme | `web/menu-theme.md` | Custom themes |
| Icons | `web/auto-icon.md` | Custom icons |
| TypeScript | `web/typescript.md` | TS configuration |
| Excel Export | `web/export-excel.md` | Data export |

### Deployment

| Topic | File | Description |
|-------|------|-------------|
| Overview | `deployment/index.md` | Deployment strategies |
| Docker | `deployment/docker.md` | Docker deployment |
| Docker Compose | `deployment/docker-compose.md` | Multi-container setup |
| Kubernetes | `deployment/k8s.md` | K8s deployment |

### Plugin System

| Topic | File | Description |
|-------|------|-------------|
| Installation | `plugin/install.md` | Plugin installation |
| Development | `plugin/develop.md` | Creating plugins |

### Best Practices & Troubleshooting

| Topic | File | Description |
|-------|------|-------------|
| Dev Standards | `best-practices/development-standards.md` | Coding standards |
| Common Issues | `troubleshooting/common-issues.md` | Troubleshooting guide |

## Usage

**Prerequisites**: If `references/gin-vue-admin.com/docs` directory is empty or missing, initialize the submodule first:

```sh
git submodule update --init --recursive skills/gva-doc/references/gin-vue-admin.com
```

Read relevant documentation files based on the task:

```sh
# Example: Read authentication docs
Read references/gin-vue-admin.com/docs/guide/server/authentication.md

# Example: Read code generator guide
Read references/gin-vue-admin.com/docs/guide/generator/server.md
```

## Key Concepts

- **AutoCode**: Generate CRUD code in 1 minute via UI
- **JWT + Casbin**: Authentication with role-based access control
- **Multi-DB**: Supports MySQL, PostgreSQL, SQLite, MSSQL
- **Plugin System**: Extensible architecture for custom features
