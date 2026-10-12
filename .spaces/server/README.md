# Hono Server 搭建教程

从 NestJS + Fastify 模板（错误体系 + 会话认证 + RBAC 鉴权）迁移到 Hono 的完整 step-by-step 教程。
每一步都是完整代码，照着复制粘贴即可跑通。

## 最终你会得到什么

| 能力 | 实现 |
|---|---|
| 统一错误体系 | 错误码注册表 + `AppException` + 全局 `onError`（脱敏日志、只记 5xx、`x-request-id`、Bearer 质询） |
| 入参校验 | `zod` + `@hono/zod-validator`，校验失败输出统一 `ApiFailure` |
| 会话认证 | Bearer 不透明 token + Redis 滑动续期，`sessionAuth` 全局中间件 |
| 权限鉴权 | `accessPolicy({...})` 路由级中间件，声明式策略 + 默认拒绝，支持 platform/company/project 三级作用域 |

## 最终目录结构

```
hono/
├── package.json
├── tsconfig.json
├── .env
└── src/
    ├── main.ts                        # 启动入口
    ├── app.ts                         # createApp：全局中间件 + onError + 路由挂载
    ├── shared/
    │   └── api-failure.ts             # 统一错误响应类型
    ├── common/
    │   ├── errors/
    │   │   ├── app-error.ts           # AppErrorDefinition + AppException
    │   │   └── common.errors.ts       # COMMON_ERRORS 注册表
    │   ├── http/
    │   │   └── on-error.ts            # 全局错误归一化 + 脱敏日志
    │   └── access/
    │       ├── scope.ts               # 从路由参数推导作用域
    │       └── access-policy.ts       # accessPolicy 中间件工厂
    └── modules/
        ├── auth/
        │   ├── auth.errors.ts
        │   ├── session.service.ts     # token 生成/digest + Redis 存取
        │   ├── session-auth.ts        # sessionAuth 中间件
        │   └── auth.routes.ts         # login / logout / me
        ├── access/
        │   ├── access.errors.ts
        │   └── access.service.ts      # 权限判定（含 {scope} 占位符解析）
        └── companies/
            └── companies.routes.ts    # 示例业务模块
```

## 步骤索引

1. [01-初始化项目](./01-初始化项目.md) — 建目录、装依赖、tsconfig、.env
2. [02-错误体系](./02-错误体系.md) — ApiFailure 类型 + AppException + COMMON_ERRORS（框架无关，最先搭）
3. [03-全局错误处理](./03-全局错误处理.md) — onError 归一化 + 日志脱敏
4. [04-认证](./04-认证.md) — Redis 会话 + sessionAuth 中间件 + 登录路由
5. [05-权限](./05-权限.md) — accessPolicy 中间件 + 三级作用域
6. [06-组装与示例](./06-组装与示例.md) — app.ts / main.ts / 示例业务路由
7. [07-启动与验证](./07-启动与验证.md) — 启动 + curl 验证每个场景

## 前置条件

- Node.js ≥ 20，pnpm
- 本地 Redis（`redis://localhost:6379`），没有的话：`docker run -d -p 6379:6379 redis:7`

开始 → [01-初始化项目](./01-初始化项目.md)
