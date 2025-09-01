# 实操案例：应用 AI 架构提示

本文档提供具体的实操案例，展示如何结合使用本仓库中的三种 AI 提示来解决真实的软件工程问题。

---

### 案例一：重构一个单体的 Node.js 后端服务

这个案例将带你走过一个完整的重构流程：将一个典型的、混乱的单体 Node.js 应用，通过 AI 提示的引导，重构为模块化、可维护的系统。

#### 问题背景：混乱的泥潭 (The Ball of Mud)

我们有一个简单的电子商务后端服务，使用 Express.js 构建。起初很简单，但随着业务发展，几乎所有的逻辑都堆积在了 `index.js` 文件里，形成了一个难以维护的“代码泥潭”。

**原始的 `index.js` (部分代码):**

```javascript
const express = require('express');
const { MongoClient } = require('mongodb');

const app = express();
app.use(express.json());

const mongoUri = 'mongodb://localhost:27017';
const client = new MongoClient(mongoUri);

// 路由、业务逻辑、数据库操作全部混合在一起
app.post('/users', async (req, res) => {
  try {
    await client.connect();
    const db = client.db('ecommerce');
    const usersCollection = db.collection('users');

    const { email, password, name } = req.body;

    // 业务逻辑：检查邮箱是否已存在
    const existingUser = await usersCollection.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email already exists' });
    }

    // 业务逻辑：哈希密码 (此处为伪代码)
    const hashedPassword = `hashed_${password}`;

    const result = await usersCollection.insertOne({ email, password: hashedPassword, name });
    res.status(201).json({ message: 'User created', userId: result.insertedId });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal server error' });
  } finally {
    await client.close();
  }
});

// ... 此处还有几十个类似的、处理产品、订单等的路由 ...

app.listen(3000, () => console.log('Server running on port 3000'));
```

**存在的问题:**
- **紧密耦合**: Express 路由、MongoDB 数据库逻辑和业务规则完全混在一起。
- **难以测试**: 无法独立测试业务逻辑，必须启动整个服务并连接真实数据库。
- **难以扩展**: 添加新功能或修改现有功能都非常痛苦，容易引发意想不到的副作用。
- **违反单一职责**: 一个文件做了所有事情。

#### 步骤一：使用“架构顾问提示”进行规划

我们的目标是解耦这个系统。我们首先使用 `claude-prompt.md` (高级系统架构顾问) 来帮助我们进行高层设计。

**我们向 AI 发出的提示:**
> [粘贴 `claude-prompt.md` 的内容]
>
> **我的问题:**
> 我正在构建一个 Node.js 的电子商务后端服务。目前，所有的逻辑（用户、产品、订单）都混在一个文件里。请使用黑盒原则帮我设计一个模块化的架构。请识别出核心的“原生体”(primitives)和模块边界。

**AI 可能的回答 (摘要):**

> 好的，这是一个经典的重构场景。让我们运用黑盒原则来设计一个清晰的架构。
>
> **1. 核心原生体 (Primitives):**
> 系统中的核心数据流是：
> - `User`: { id, email, name, hashedPassword }
> - `Product`: { id, name, price, description }
> - `Order`: { id, userId, productIds, amount, status }
>
> **2. 黑盒模块边界:**
> 我建议将系统分解为以下几个核心模块，每个模块都是一个黑盒：
> - **`UserService`**: 负责所有与用户相关的业务逻辑（创建、查找、更新等）。它不应该知道数据库是如何实现的。
> - **`ProductService`**: 负责产品相关的业务逻辑。
> - **`OrderService`**: 负责订单相关的业务逻辑。
> - **`DatabaseAdapter`**: 这是一个关键的抽象层。它包装了所有与数据库的交互（例如 MongoDB）。其他的服务将通过这个适配器来存取数据，而不需要知道底层用的是什么数据库。
> - **`ApiRouter`**: 负责处理 HTTP 请求和响应，将请求路由到相应的服务。它只做路由和数据校验，不包含业务逻辑。
>
> **3. 接口设计 (示例 - UserService):**
> `UserService` 应该暴露一个清晰的接口，例如：
> ```typescript
> interface IUserService {
>   createUser(email, password, name): Promise<User>;
>   findUserById(id): Promise<User | null>;
> }
> ```
> 这样，我们就为下一步的重构制定了清晰的蓝图。

#### 步骤二：使用“代码重构提示”进行实现

现在我们有了架构蓝图。我们选取 `UserService` 作为第一个重构目标。我们使用 `claude-code-prompt.md` (系统架构专家) 来帮助我们编写具体的代码。

**我们向 AI 发出的提示:**
> [粘贴 `claude-code-prompt.md` 的内容]
>
> **我的代码 (原始 `index.js` 中的用户创建部分):**
> ```javascript
> // ... 粘贴 app.post('/users', ...) 的代码 ...
> ```
> **我的要求:**
> 根据我们之前规划的架构，请将这段代码重构为一个独立的 `UserService` 模块。它应该通过一个 `DatabaseAdapter` 接口来操作数据库，从而隐藏实现细节。

**AI 可能的回答 (摘要与代码):**

> 好的，这是一个很好的重构点。我们将把业务逻辑提取到 `UserService` 中，并通过依赖注入的方式传入一个数据库适配器。
>
> **1. 定义数据库适配器接口 (`db-adapter.js`):**
> ```javascript
> // 这是一个接口定义，可以用 TypeScript 的 interface 或 JSDoc 来明确
> class DatabaseAdapter {
>   async findOne(collection, query) { throw new Error("Not implemented"); }
>   async insertOne(collection, data) { throw new Error("Not implemented"); }
> }
> ```
>
> **2. 重构后的 `user-service.js`:**
> ```javascript
> class UserService {
>   constructor(dbAdapter) {
>     this.db = dbAdapter;
>     this.collection = 'users';
>   }
>
>   async createUser(email, password, name) {
>     // 业务逻辑：检查邮箱是否已存在
>     const existingUser = await this.db.findOne(this.collection, { email });
>     if (existingUser) {
>       // 只抛出业务相关的错误
>       throw new Error('Email already exists');
>     }
>
>     // 业务逻辑：哈希密码
>     const hashedPassword = this.hashPassword(password);
>
>     const newUser = { email, password: hashedPassword, name };
>     const result = await this.db.insertOne(this.collection, newUser);
>     return { id: result.insertedId, ...newUser };
>   }
>
>   hashPassword(password) {
>     // 真实的哈希逻辑
>     return `hashed_${password}`;
>   }
> }
>
> module.exports = UserService;
> ```
> 这个 `UserService` 现在是完全独立的，它不关心数据库是 MongoDB 还是 PostgreSQL，只依赖于 `DatabaseAdapter` 的接口。

#### 步骤三：使用“开发助手提示”编写测试

重构后的 `UserService` 非常容易测试。我们现在使用 `cursor-prompt.md` (模块化架构开发助手) 来为它编写单元测试。

**我们向 AI 发出的提示:**
> [粘贴 `cursor-prompt.md` 的内容]
>
> **我的代码 (`user-service.js`):**
> ```javascript
> // ... 粘贴上面 UserService 的代码 ...
> ```
> **我的要求:**
> 请为 `UserService` 编写单元测试。测试应该遵循黑盒原则，只测试其公共接口。请使用一个模拟的 `DatabaseAdapter` 来隔离测试，不要依赖真实数据库。

**AI 可能的回答 (Jest 测试代码):**
> 好的。为了测试 `UserService`，我们将创建一个模拟的 `DatabaseAdapter`，这样我们就可以在不触及任何数据库的情况下，完全控制测试场景。
>
> **`user-service.test.js`:**
> ```javascript
> const UserService = require('./user-service');
>
> // 1. 创建一个模拟的数据库适配器
> const mockDbAdapter = {
>   findOne: jest.fn(),
>   insertOne: jest.fn(),
> };
>
> // 在每次测试前重置模拟函数
> beforeEach(() => {
>   mockDbAdapter.findOne.mockClear();
>   mockDbAdapter.insertOne.mockClear();
> });
>
> describe('UserService', () => {
>   const userService = new UserService(mockDbAdapter);
>
>   it('should create a user successfully if email does not exist', async () => {
>     // 安排 (Arrange)
>     mockDbAdapter.findOne.mockResolvedValue(null); // 模拟用户不存在
>     mockDbAdapter.insertOne.mockResolvedValue({ insertedId: 'user-123' });
>
>     // 行动 (Act)
>     const user = await userService.createUser('test@example.com', 'password123', 'Test User');
>
>     // 断言 (Assert)
>     expect(mockDbAdapter.findOne).toHaveBeenCalledWith('users', { email: 'test@example.com' });
>     expect(mockDbAdapter.insertOne).toHaveBeenCalledWith('users', expect.any(Object));
>     expect(user).toBeDefined();
>     expect(user.id).toBe('user-123');
>   });
>
>   it('should throw an error if email already exists', async () => {
>     // 安排 (Arrange)
>     mockDbAdapter.findOne.mockResolvedValue({ id: 'user-456', email: 'exists@example.com' });
>
>     // 行动 & 断言 (Act & Assert)
>     await expect(
>       userService.createUser('exists@example.com', 'password123', 'Another User')
>     ).rejects.toThrow('Email already exists');
>
>     expect(mockDbAdapter.insertOne).not.toHaveBeenCalled();
>   });
> });
> ```

#### 总结

通过这三个步骤，我们成功地：
1.  **规划**了一个清晰、模块化的架构。
2.  **重构**了代码，将业务逻辑从基础设施中分离出来，创建了可替换的黑盒模块。
3.  **测试**了新模块的接口，确保了其行为的正确性，且测试是快速、可靠的。

现在，我们可以对 `ProductService` 和 `OrderService` 重复这个过程，最终得到一个干净、可维护、易于扩展的系统。这就是 AI 架构提示在实践中的力量。
