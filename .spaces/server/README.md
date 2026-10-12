# server-hono 搭建教程

将 `apps/server`（NestJS + Fastify）的横切架构迁移到 Hono 的 step-by-step 实施文档。
每一步都是完整代码，按顺序复制粘贴即可跑通。

## 架构决策（讨论结论，先读）

| 决策点 | 结论 | 理由 |
|---|---|---|
| 位置 | monorepo 下新建 `apps/server-hono` | 不动现有 Nest 版，迁移验证完再切换 |
| 工具链 | **vite-plus 1.1.0**（`vp run/test/fmt/check`） | 与 monorepo 约定一致；只管编译测试格式化，与运行时无关 |
| 运行时 | **Node 为主，Bun 可选** | 双平台靠分层实现：代码只写运行时无关 API，两个薄入口 `main.node.ts` / `main.bun.ts` |
| Redis 客户端 | `ioredis` | Node/Bun 双平台都能跑；Bun 原生客户端和 BullMQ 适配器留作后续优化 |
| 队列 | 暂不迁移；需要时上 BullMQ v6（已支持 Bun） | v6 后端可插拔（Redis/PostgreSQL），不再是 Bun 障碍 |
| 数据库 | Drizzle ORM + PostgreSQL | 与 Nest 版一致，schema 从简（users + access_grants） |
| API 文档 | `@hono/zod-openapi` + `@hono/swagger-ui` | zod schema 即 OpenAPI，UI 挂 `/api-docs` |
| 类型复用 | workspace `shared` 包 | `ApiFailure / ApiErrorDetail / AccessScope / ScopeType` 直接 import |

### 双平台分层图

```
工具链层  vite-plus ────────────── 编译/测试/lint，与运行时无关
代码层    src/** ──────────────── hono + zod + drizzle + ioredis（全是跨运行时库）
入口层    main.node.ts → @hono/node-server     ← 唯一感知运行时的两个文件
          main.bun.ts  → Bun.serve
```

### 双平台已知风险点

- `argon2` 是 napi 原生模块：Node 无问题；Bun 对 napi 基本兼容，**首次用 Bun 入口时先验证登录接口**，不行就换 `@node-rs/argon2`（同样 napi 但预编译更全）或 `bcryptjs`（纯 JS）。

## 最终目录结构

```
apps/server-hono/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── drizzle.config.ts
├── .env
└── src/
    ├── main.node.ts                 # Node 入口
    ├── main.bun.ts                  # Bun 入口（可选）
    ├── app.ts                       # createApp：中间件 + 路由 + OpenAPI
    ├── config/env.ts                # 环境变量
    ├── common/
    │   ├── errors/app-error.ts      # AppErrorDefinition + AppException
    │   ├── errors/common.errors.ts  # COMMON_ERRORS
    │   ├── http/on-error.ts         # onError/notFound：归一化 + 脱敏日志
    │   └── access/scope.ts          # 作用域推导
    │   └── access/access-policy.ts  # accessPolicy 中间件工厂
    ├── db/
    │   ├── index.ts                 # pg.Pool + drizzle
    │   └── schema.ts                # users + access_grants
    └── modules/
        ├── auth/   auth.errors.ts / session.service.ts / user.repository.ts
        │           session-auth.ts / auth.routes.ts
        ├── access/ access.errors.ts / access.service.ts
        └── companies/ companies.routes.ts   # 示例业务模块
```

## 步骤索引

1. [01-初始化与依赖](./01-初始化与依赖.md)
2. [02-错误体系](./02-错误体系.md)
3. [03-全局错误处理](./03-全局错误处理.md)
4. [04-数据层](./04-数据层.md)
5. [05-认证与权限](./05-认证与权限.md)
6. [06-组装与 OpenAPI](./06-组装与OpenAPI.md)
7. [07-启动与验证](./07-启动与验证.md)

## 前置条件

- pnpm（monorepo 根目录统一安装）
- PostgreSQL（`docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:16`）
- Redis（`docker run -d -p 6379:6379 redis:7`）
- 可选：Bun（仅验证双入口时需要）

开始 → [01-初始化与依赖](./01-初始化与依赖.md)
