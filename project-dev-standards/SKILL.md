---
name: project-dev-standards
description: 制定代码风格、命名约定、Git工作流等开发规范；当需要统一团队编码标准或确保AI生成代码的一致性时使用
---

# 开发规范制定

## 任务目标
- 本Skill用于：制定项目的开发规范和编码标准
- 能力包含：代码风格定义、命名约定制定、Git工作流规范、AI协作协议
- 触发条件：已完成项目结构设计，需要制定编码标准

## 前置准备
- 必须存在 `specs/技术栈.md`
- 必须存在 `specs/项目结构.md`
- 工作目录：项目根目录

## 操作步骤

### 1. 读取前置文档
智能体扮演技术委员会角色，读取必要的文档：
```
读取 specs/技术栈.md
读取 specs/项目结构.md
```

### 2. 生成开发规范
基于技术栈生成定制化的开发规范：

```markdown
# 开发规范

## 1. 命名约定

### 文件命名
| 元素 | 风格 | 示例 |
|------|------|------|
| 组件文件 | PascalCase | PostCard.tsx |
| 工具函数文件 | camelCase | formatDate.ts |
| 目录名 | kebab-case | user-profile/ |
| 类型文件 | PascalCase | User.ts |
| 样式文件 | 同组件名 | PostCard.module.css |

### 代码命名
| 元素 | 风格 | 示例 |
|------|------|------|
| 组件名 | PascalCase | PostCard |
| 函数名 | camelCase | getUserInfo |
| 常量 | UPPER_SNAKE_CASE | API_BASE_URL |
| 变量 | camelCase | userList |
| 类型/接口 | PascalCase | User, IUserData |

## 2. 代码风格

### TypeScript 规范

**类型定义**
```typescript
// ✅ 正确 - 明确类型定义
interface User {
  id: string;
  name: string;
  email: string;
}

// ❌ 错误 - 禁止使用 any
const data: any = await fetchUser();

// ✅ 正确 - 明确类型定义
const data: User = await fetchUser();
```

**函数定义**
```typescript
// ✅ 正确 - 明确参数和返回类型
async function getUser(id: string): Promise<User> {
  // ...
}

// ❌ 错误 - 缺少类型注解
async function getUser(id) {
  // ...
}
```

### React 组件规范

**组件定义**
```typescript
// ✅ 正确 - 使用函数组件和明确类型
interface PostCardProps {
  title: string;
  content: string;
  publishedAt: Date;
}

export function PostCard({ title, content, publishedAt }: PostCardProps) {
  return (
    <article>
      <h2>{title}</h2>
      <p>{content}</p>
    </article>
  );
}
```

## 3. Git 工作流

### 分支命名
- **主分支**：main
- **开发分支**：develop
- **功能分支**：feature/功能名
- **修复分支**：fix/问题描述
- **发布分支**：release/版本号

### 提交信息规范
```
<type>(<scope>): <subject>

<body>

<footer>
```

**类型（type）**：
- feat: 新功能
- fix: 修复BUG
- docs: 文档更新
- style: 代码格式调整
- refactor: 重构
- test: 测试相关
- chore: 构建/工具相关

**示例**：
```
feat(posts): 添加文章标签筛选功能

- 新增标签选择组件
- 实现标签筛选逻辑
- 添加相关测试

Closes #123
```

## 4. 错误处理规范

### 统一错误处理
```typescript
// ✅ 正确 - 统一的错误处理
try {
  const data = await fetchData();
  return { success: true, data };
} catch (error) {
  console.error('Failed to fetch data:', error);
  return { success: false, error: '数据获取失败' };
}
```

### 错误边界
- 每个模块必须有错误边界
- API调用必须捕获异常
- 用户友好的错误提示

## 5. 测试规范

### 单元测试
- 所有工具函数必须有单元测试
- 测试文件命名：`*.test.ts` 或 `*.spec.ts`
- 测试覆盖率目标：核心逻辑 > 80%

### 测试示例
```typescript
describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-15');
    expect(formatDate(date)).toBe('2024-01-15');
  });
});
```

## 6. 文档规范

### 代码注释
```typescript
/**
 * 格式化日期
 * @param date - 日期对象
 * @param format - 格式字符串
 * @returns 格式化后的日期字符串
 */
export function formatDate(date: Date, format: string): string {
  // ...
}
```

### README 规范
- 项目简介
- 快速开始
- 开发指南
- 部署说明
- 常见问题

## 7. AI 协作协议

### AI 必须遵守的规则
1. 严格遵循本开发规范
2. 生成代码前先说明设计思路
3. 每次修改都要说明变更内容
4. 遇到不确定的地方主动询问
5. 生成代码后主动提议编写测试

### AI 禁止的行为
1. 禁止使用 any 类型
2. 禁止跳过错误处理
3. 禁止生成不符合命名规范的代码
4. 禁止随意修改已有代码结构

## 8. 开发流程规范

### 功能开发流程
1. 从 develop 分支创建 feature 分支
2. 编写代码和测试
3. 提交符合规范的 commit
4. 创建 Pull Request
5. 通过代码审查后合并

### 代码审查要点
- 代码是否符合规范
- 是否有必要的测试
- 是否有清晰的注释
- 是否处理了边界情况
```

### 3. 确认规范内容
向用户展示规范要点并确认：
```
完整规范共 15 个章节，已全部生成。主要包含：

- 命名约定（文件、变量、类型等）
- TypeScript 和 React 编码规范
- Git 工作流和提交规范
- 错误处理和测试规范
- AI 协作协议

请确认后我保存到 specs/开发规范.md。
```

### 4. 保存文档
将开发规范保存到 `specs/开发规范.md`

## 输出格式
- 文件路径：`specs/开发规范.md`
- 文档格式：结构化的开发规范文档
- 内容要求：具体、可执行、有示例对比

## 注意事项

### 边界守卫规则
- **当前阶段只能做**：开发规范制定和标准定义
- **禁止行为**：不涉及具体代码编写、不涉及功能实现
- 如果用户问"帮我按这个规范写代码"，提醒："规范定好后，我们在功能开发阶段会严格遵循"

### 项目上下文协议
- 执行前必须读取 `specs/技术栈.md` 和 `specs/项目结构.md`
- 规范必须匹配所选技术栈的最佳实践

### 规范制定原则
- 每条规则都要具体到可以直接执行
- 提供正确和错误的代码示例对比
- 考虑AI生成代码的特点和限制

## 使用示例

**用户输入**：
```
/project-dev-standards 请制定开发规范
```

**智能体响应**：
```
已读取技术栈和项目结构文档，以下是定制的开发规范（节选）：

命名约定

元素 | 风格 | 示例
组件文件 | PascalCase | PostCard.tsx
工具函数文件 | camelCase | formatDate.ts
目录名 | kebab-case | user-profile/

TypeScript 严格规则

// 错误 - 禁止使用 any
const data: any = await fetchPost()

// 正确 - 明确类型定义
const data: PostDetail = await fetchPost()

Git 提交规范

feat(posts): 添加文章标签筛选功能
fix(auth): 修复登录状态丢失问题

完整规范共 15 个章节，已全部生成。请确认后我保存到 specs/开发规范.md。
```
