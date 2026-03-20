# Claude Code 记忆系统的工作原理

![alt text](image.png)
这就像你给新员工一份入职手册，他读完之后就知道公司的规矩。不同的是，Claude 每次对话都会重新“入职”——所以这份手册必须简洁有效。

![alt text](image-1.png)

# Claude Code 的五层记忆架构
Claude Code 支持五个层级的记忆，就像洋葱一样，从外到内，按层级结构组织——高层级的文件优先加载，为底层文件提供基础：

![alt text](image-2.png)

![alt text](image-3.png)

# 企业策略级记忆设定
企业策略级记忆设定的作用是组织范围内的指令，由 IT/DevOps 统一管理和部署组织。适合内容是，公司编码标准、安全策略、合规要求以及禁止使用的库或模式。通过配置管理系统（MDM、Group Policy、Ansible 等）部署，确保在所有开发者机器上一致分发。

位置：macOS: /Library/Application Support/ClaudeCode/CLAUDE.md
Linux: /etc/claude-code/CLAUDE.md
Windows: C:\Program Files\ClaudeCode\CLAUDE.md
```
# 公司开发策略

## 安全要求
- 禁止在代码中硬编码任何密钥或敏感信息
- 所有 API 调用必须使用 HTTPS
- 用户输入必须经过验证和清理

## 合规要求
- 所有日志必须排除 PII（个人身份信息）
- 数据库连接必须使用加密传输

## 禁止项
- 禁止使用未经审批的第三方库
- 禁止直接访问生产数据库
```

# 用户级内容设定
承载的是你的全局偏好，即跨所有项目生效的个人偏好，如个人代码风格，沟通语言设置，通用工作习惯等。比如说我希望所有的 PPT 都是 16:9，黑体字。这种设置就应该放在此处。

位置：~/.claude/CLAUDE.md

```# 个人偏好

## 沟通方式
- 使用中文回复
- 代码注释使用英文
- 解释简洁直接，不要过多铺垫

## 通用代码风格
- 缩进使用 2 空格
- 优先使用 async/await
- 变量命名使用 camelCase
- 常量命名使用 UPPER_SNAKE_CASE

## 我的常用工具
- 包管理器: uv
- 编辑器: VS Code
- 终端: zsh
```

# 项目级团队共享规范

团队共享规范是团队共享的项目知识，应该提交到 Git。适合存放的内容包括项目架构和技术栈、团队编码规范、重要的设计决策和常用命令。位置：项目根目录的  ./CLAUDE.md


示例（一个后端 API 项目）：

```
# 项目：订单服务 API

## 技术栈
- Node.js 20 + TypeScript
- Fastify（Web 框架）
- Prisma（ORM）
- PostgreSQL + Redis
- Zod（数据验证）

## 目录结构
src/ 
├── routes/ # 路由定义 
├── controllers/ # 请求处理 
├── services/ # 业务逻辑 
├── repositories/ # 数据访问 
├── schemas/ # Zod schemas 
└── types/ # 类型定义

## API 响应格式
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: { code: string; message: string };
}
编码规范
- TypeScript strict 模式
- 禁止使用 any，使用 unknown + 类型守卫
- 所有 API 端点必须有 Zod schema 验证
- 业务错误使用自定义 Error 类
常用命令
- pnpm dev - 启动开发服务器
- pnpm test - 运行测试
- pnpm prisma migrate dev - 运行数据库迁移
```
# 本地级个人工作空间

个人工作空间用于记载个人工作笔记，不提交到 Git，适合内容包括本地环境配置、个人调试技巧、当前工作备注，敏感信息（测试账号等）。位置：项目根目录的./CLAUDE.local.md示例如下。

```
# 本地开发笔记

## 我的环境
- 本地 API: http://localhost:3000
- 测试数据库: order_service_dev
- Redis: localhost:6379

## 测试账号
- admin@test.com / test123
- user@test.com / test123

## 当前工作
- 正在重构支付模块
- 参考 PR #234 的讨论
- 周五前完成

## 调试技巧
- 订单状态机日志: LOG_LEVEL=debug pnpm dev
- 查看 Redis 缓存: redis-cli KEYS "order:*"
```
这里重点强调一下：记得把  CLAUDE.local.md  加入  .gitignore！

# 规则目录：分类组织

最后说一下 rules，这是一个比较高阶的技巧，初学者可以作为知识了解一下，也可以先略过不看。Rules 是按主题组织的规则文件，支持条件作用域（也就是视情况来确定是否加载该记忆内容），适合场景包括 CLAUDE.md 变得太长时，不同文件类型需要不同规范时，以及前后端分离的项目。位置：.claude/rules/*.md目录结构：

位置：.claude/rules/*.md

目录结构：

```
.claude/
└── rules/
    ├── typescript.md      # TypeScript 规范
    ├── testing.md         # 测试规范
    ├── api-design.md      # API 设计规范
    └── security.md        # 安全规范
  ```
  条件作用域示例：.claude/rules/testing.md

```---
paths:
  - "src/**/*.test.ts"
  - "tests/**/*.ts"
---

# 测试规范

## 命名
- 单元测试: `*.test.ts`
- 集成测试: `*.integration.test.ts`

## 结构
使用 Arrange-Act-Assert 模式：

```typescript
describe('OrderService', () => {
  describe('createOrder', () => {
    it('should create order when stock is available', async () => {
      // Arrange
      const mockProduct = createMockProduct({ stock: 10 });

      // Act
      const order = await orderService.createOrder(mockProduct.id, 1);

      // Assert
      expect(order.status).toBe('created');
    });
  });
});

## 覆盖率要求
- 业务逻辑: > 80%
- 工具函数: > 90%
- 路由/控制器: 可以较低
```

真正有价值的 CLAUDE.md，应该长这样。

```
# 项目规范

## TypeScript
- 使用 `interface` 定义对象结构，`type` 用于联合类型
- 禁止 `any`，使用 `unknown` + 类型守卫
- 函数参数 > 3 个时，使用对象参数

## 错误处理
```typescript
// 业务错误
throw new BusinessError('ORDER_NOT_FOUND', '订单不存在');

// 验证错误（Zod 自动抛出）
const data = orderSchema.parse(input);

// controller 中不要 try-catch
// 由全局错误中间件统一处理
```

不是模糊要求“要高质量”，而是给出了如何做才算高质量；不是“注意错误处理”，而是具体的错误模型；不是抽象描述，而是可直接模仿的代码形态。


# 核心原则 3：关键三问题 WHY / WHAT / HOW

WHY —— 为什么要这样做？

```
## 为什么使用 Zod？
- TypeScript 只有编译时类型检查
- API 输入需要运行时验证
- Zod 可以同时生成 TS 类型和验证逻辑
- 错误信息自动生成，对用户友好
```
# WHAT —— 具体要做什么，不要做什么？

```
## 数据库操作规范
- 所有查询通过 Prisma ORM
- 复杂查询封装在 `src/repositories/`
- 禁止在 controller/service 中直接写 SQL
- 事务使用 `prisma.$transaction()`
```
# HOW —— 按什么步骤去做？

```
## 创建新 API 端点

1. 在 `src/schemas/` 创建请求/响应 Zod schema
2. 在 `src/routes/` 添加路由定义
3. 在 `src/controllers/` 实现请求处理
4. 在 `src/services/` 实现业务逻辑
5. 在 `tests/` 添加测试用例

示例参考: `src/routes/orders.ts`
```

核心原则 4：渐进式披露：不要把一切都塞进 CLAUDE.md

示例：
# Step 1：创建基础 CLAUDE.md

先通过 /init 命令自动初始化 CLAUDE.md 文件，或使用下面的命令在项目根目录手动创建记忆文件

```
touch CLAUDE.md
```

再创建以下内容
```
# 项目：电商平台前端

## 技术栈
- React 18 + TypeScript
- Vite 构建
- TanStack Query（数据获取）
- Zustand（状态管理）
- Tailwind CSS

## 目录结构
src/ 
├── components/ # 组件 
│ ├── ui/ # 基础 UI 
│ └── features/ # 功能组件 
├── pages/ # 页面 
├── hooks/ # 自定义 Hooks 
├── stores/ # Zustand stores 
├── api/ # API 调用 
└── types/ # 类型定义

## 组件规范
- 函数组件 + Hooks
- Props 接口命名: `XxxProps`
- 一个组件一个目录: `Button/index.tsx`

## 状态管理
- 服务端状态: TanStack Query
- 客户端状态: Zustand
- 本地状态: useState

## 常用命令
- `pnpm dev` - 开发服务器
- `pnpm build` - 构建
- `pnpm test` - 测试
```

# Step 2：创建本地记忆

```
touch CLAUDE.local.md
echo "CLAUDE.local.md" >> .gitignore
```
然后创建如下的内容。

```
# 本地笔记

## 环境
- API: http://localhost:8080
- Mock: 使用 MSW

## 当前任务
- 重构购物车组件
- 截止: 本周五
```

# Step 3：添加条件规则（可选）

```
mkdir -p .claude/rules
```
然后创建如下.claude/rules/testing.md：

```
---
paths:
  - "src/**/*.test.tsx"
  - "src/**/*.test.ts"
---

# 测试规范

- 使用 Vitest + React Testing Library
- 测试文件放在同目录: `Button.test.tsx`
- 优先测试用户行为，而非实现细节

```typescript
// ✅ 好
expect(screen.getByRole('button')).toBeEnabled();

// ❌ 不好
expect(component.state.isLoading).toBe(false);
```

示例二：
优化已有的 CLAUDE.md

# Step 1：识别核心内容
详细的 API 文档、数据库表结构和部署流程虽然重要，但是完全没有必要每次都读入 Claude 内存，可以移动到单独文件，精简原来的 CLAUDE.md 。

![alt text](image-4.png)

# Step 2：拆分成独立文件

详细的 API 文档、数据库表结构和部署流程虽然重要，但是完全没有必要每次都读入 Claude 内存，可以移动到单独文件，精简原来的 CLAUDE.md

```
## 核心规范
[精简内容]

## 详细参考
- API 端点清单: @docs/api.md
- 数据库 Schema: @prisma/schema.prisma
- 部署配置: @docs/deploy.md
```

# Step 3：使用条件规则

可以考虑进一步把测试规范、前端规范、后端规范拆分到  .claude/rules/，并设置  paths条件。

# 场景三：记忆管理命令

要查看当前记忆，在 Claude Code 中输入：

/memory

就会显示当前加载的所有记忆内容和来源

编辑记忆的命令参数如下。

```
/memory edit         # 编辑项目级 CLAUDE.md
/memory edit user    # 编辑用户级记忆
/memory edit local   # 编辑本地级记忆
```
# Auto Memory 解读

最后的一个重要知识点是，Claude Code 本身拥有自动记忆功能，随着项目的演进和对话的深入，会在 ~/.claude/projects//memory/ 目录下自动生成 Auto Memory，用于记录模型在项目中学习到的模式、调试经验与结构认知。这意味着，Claude Code 的“记忆”并不是单一文件，而是一种多层叠加的上下文注入架构：有些是人为编写的长期规则，有些是组织级强制策略，还有一些是模型自动沉淀的经验笔记。CLAUDE.md 决定“系统被告知什么”，而 Auto Memory 决定“系统在实践中学到了什么”。记忆因此成为一种结构化的工程能力，而不是简单的对话缓存。

