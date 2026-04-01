---
name: bugfix-workflow
description: 强制执行复现-定位-修复-验证的完整修复流程；当遇到需要系统性解决的BUG问题时使用
---

# BUG修复工作流

## 任务目标
- 本Skill用于：系统性解决BUG问题
- 能力包含：问题复现、根因定位、代码修复、回归测试、修复报告
- 触发条件：用户报告BUG或发现异常行为

## 前置准备
- 工作目录：项目根目录
- 建议存在 `docs/BUG修复文档/` 目录（用于存储修复报告）

## 核心规则

### 🚨 硬性规则：未复现，不修复
- **绝对禁止**：在没有复现问题的情况下直接修改代码
- **必须执行**：通过用户提供的信息完整复现问题
- **信息不够**：主动追问，直到能复现为止

## 操作步骤

### 1. 问题接收
接收用户报告的BUG：

```
收到BUG报告：
"评论发布之后页面没有刷新，还是显示旧的评论列表"

我需要先确认几个信息来复现这个问题：
```

### 2. 信息收集（复现阶段）
通过提问收集足够的信息以复现问题：

**必问清单**：
```
1. 发生在哪个页面？
   - 文章详情页？
   - 个人中心？
   - 其他页面？

2. 你做了哪些操作？（尽量细化到每一步）
   - 步骤1：打开文章详情页
   - 步骤2：输入评论内容
   - 步骤3：点击发布按钮
   - ...

3. 发布评论后，页面完全没变化，还是过一会儿才刷新？

4. 打开浏览器控制台，有没有报错信息？
   - 请提供完整的错误信息

5. 这个问题是必现的还是偶发的？
   - 如果偶发，在什么情况下会出现？

6. 浏览器和版本是什么？

7. 有截图或录屏吗？
```

### 3. 问题复现
根据信息尝试复现问题：

```markdown
# BUG复现报告

## BUG编号
BUG-2024-001

## 复现环境
- 浏览器：Chrome 120.0.0
- 操作系统：macOS Sonoma
- 项目版本：v1.2.0

## 复现步骤
1. 打开文章详情页 `/posts/abc-123`
2. 登录用户账号
3. 在评论框输入内容："测试评论"
4. 点击"发布评论"按钮
5. 观察页面变化

## 预期结果
- 评论发布成功
- 评论列表立即刷新，显示新评论
- 评论框清空

## 实际结果
- 评论发布成功（数据库有记录）
- 评论列表未刷新，仍显示旧列表
- 评论框已清空
- 控制台无报错

## 复现状态
✓ 问题已复现

## 复现截图
[附上截图]
```

### 4. 根因定位
分析问题根本原因：

```markdown
# 根因分析

## 问题现象
评论发布成功后，评论列表未刷新

## 定位过程

### 1. 检查API响应
```bash
# 发送评论请求
POST /api/comments
Response: { success: true, data: {...} }
```
✓ API正常返回

### 2. 检查前端逻辑
```typescript
// CommentForm.tsx
const handleSubmit = async (content: string) => {
  await createComment(content);
  // ❌ 缺少：未调用刷新列表的逻辑
};
```
✗ 发现问题：发布评论后未触发列表刷新

### 3. 确认根因
**根本原因**：CommentForm 组件的 handleSubmit 函数在评论发布成功后，没有调用父组件传递的 onCommentPosted 回调函数

**影响范围**：所有使用 CommentForm 的页面

**相关代码**：
- `src/modules/comments/components/CommentForm.tsx` (第45行)
- `src/app/posts/[id]/page.tsx` (第78行)
```

### 5. 修复方案
制定修复方案：

```markdown
# 修复方案

## 修复策略
添加评论发布后的回调触发

## 修复步骤
1. CommentForm 组件添加 onCommentPosted prop
2. handleSubmit 成功后调用 onCommentPosted
3. 父组件传递刷新列表的回调函数

## 代码修改

### 文件1：src/modules/comments/components/CommentForm.tsx
```typescript
// 修改前
interface CommentFormProps {
  postId: string;
}

// 修改后
interface CommentFormProps {
  postId: string;
  onCommentPosted?: () => void; // 新增
}

const handleSubmit = async (content: string) => {
  await createComment(content);
  onCommentPosted?.(); // 新增
};
```

### 文件2：src/app/posts/[id]/page.tsx
```typescript
// 修改前
<CommentForm postId={post.id} />

// 修改后
<CommentForm 
  postId={post.id} 
  onCommentPosted={() => refetchComments()} 
/>
```

## 回归风险
- 风险等级：低
- 理由：只是添加回调触发，不改变已有逻辑
- 影响范围：所有使用 CommentForm 的页面需要传递回调
```

### 6. 执行修复
执行代码修改：

```
开始执行修复：

【步骤1】修改 CommentForm.tsx
正在修改 src/modules/comments/components/CommentForm.tsx …

【步骤2】修改 page.tsx
正在修改 src/app/posts/[id]/page.tsx …

【步骤3】验证修改
正在检查代码语法 … ✓ 通过
正在检查类型 … ✓ 通过

修复完成！
```

### 7. 回归测试
编写并执行测试：

```markdown
# 回归测试

## 单元测试
新增测试用例：

```typescript
// CommentForm.test.tsx
describe('CommentForm', () => {
  it('should call onCommentPosted after successful submission', async () => {
    const onCommentPosted = jest.fn();
    render(<CommentForm postId="123" onCommentPosted={onCommentPosted} />);
    
    await submitComment('test content');
    
    expect(onCommentPosted).toHaveBeenCalled();
  });
});
```

## 测试结果
✓ 新增测试通过
✓ 已有测试全部通过
✓ 代码覆盖率：88%

## 手动测试清单
- [ ] 发布评论后列表刷新
- [ ] 多次连续发布评论正常
- [ ] 其他页面使用 CommentForm 正常
```

### 8. 验证步骤
提供详细的手动验证步骤：

```markdown
# 手动验证步骤

请按以下步骤验证修复：

## 验证环境
- 使用测试账号登录
- 访问测试文章：/posts/test-article

## 验证步骤

### 步骤1：基本功能验证
1. 打开文章详情页
2. 输入评论："测试评论1"
3. 点击"发布评论"按钮
4. ✓ 确认：评论列表立即刷新，显示"测试评论1"

### 步骤2：连续评论验证
1. 继续输入评论："测试评论2"
2. 点击"发布评论"按钮
3. ✓ 确认：评论列表更新，显示"测试评论2"在列表顶部

### 步骤3：其他页面验证
1. 访问其他文章详情页
2. 发布评论
3. ✓ 确认：评论列表正常刷新

### 步骤4：异常情况验证
1. 网络断开情况下发布评论
2. ✓ 确认：显示错误提示，列表不变

## 验证结果
[ ] 已通过所有验证步骤
```

### 9. 修复报告
生成完整的修复报告：

```markdown
# BUG修复报告

## 基本信息
- **BUG编号**：BUG-2024-001
- **报告时间**：2024-01-20 14:30
- **修复时间**：2024-01-20 15:45
- **修复人员**：AI Assistant
- **严重程度**：中

## 问题描述
评论发布成功后，评论列表不刷新，用户需要手动刷新页面才能看到新评论

## 复现步骤
1. 打开文章详情页
2. 发布评论
3. 观察评论列表

## 根本原因
CommentForm 组件在评论发布成功后，未调用父组件传递的回调函数触发列表刷新

## 修复内容
### 代码修改
- `src/modules/comments/components/CommentForm.tsx`：添加 onCommentPosted prop 并在提交成功后调用
- `src/app/posts/[id]/page.tsx`：传递 refetchComments 作为回调

### 新增测试
- `src/modules/comments/components/__tests__/CommentForm.test.tsx`：新增回调触发测试

## 测试结果
- 单元测试：5/5 通过
- 集成测试：3/3 通过
- 手动测试：全部通过

## 影响范围
- 影响文件：2个
- 影响页面：所有使用 CommentForm 的页面
- 回归风险：低

## 预防措施
1. 添加单元测试覆盖回调触发场景
2. 在开发规范中强调组件间通信要使用回调模式
3. Code Review 时检查类似问题

## 相关文档
- [复现报告](./BUG-2024-001_复现.md)
- [修复代码](../../src/modules/comments/components/CommentForm.tsx)
- [测试代码](../../src/modules/comments/components/__tests__/CommentForm.test.tsx)
```

### 10. 保存文档
将修复报告保存到 `docs/BUG修复文档/BUG-YYYY-XXX.md`

## 输出格式
- 复现报告：`docs/BUG修复文档/BUG-XXX_复现.md`
- 修复报告：`docs/BUG修复文档/BUG-XXX.md`
- 单元测试：针对该BUG的测试用例
- 代码修复：实际的代码修改

## 注意事项

### 边界守卫规则
- **当前阶段只能做**：BUG复现、定位、修复、验证
- **禁止行为**：未复现就直接修改代码
- 如果信息不够，必须追问

### 强制流程
1. 信息收集 → 复现问题
2. 定位根因 → 制定方案
3. 执行修复 → 编写测试
4. 手动验证 → 生成报告

### 信息收集技巧
- 每次只问一个关键问题
- 问题要具体，不要开放式
- 引导用户提供截图、日志
- 区分必现和偶发

## 使用示例

**用户输入**：
```
/bugfix-workflow 评论发布之后页面没有刷新，还是显示旧的评论列表
```

**智能体响应**：
```
收到，我需要先确认几个信息来复现这个问题：

1. 发生在哪个页面？（文章详情页？个人中心？）
2. 你做了哪些操作？（尽量细化到每一步）
3. 发布评论后，页面完全没变化，还是过一会儿才刷新？
4. 打开浏览器控制台，有没有报错信息？
5. 这个问题是必现的还是偶发的？

请先回答这几个问题，我复现之后才开始定位和修复。
```

**用户提供信息后**：
```
好的，根据你提供的信息，我已经复现了这个问题：

复现步骤：
1. 打开文章详情页 `/posts/abc-123`
2. 发布评论
3. 观察评论列表 → 列表未刷新

正在定位根本原因…
[分析过程]

找到根本原因：CommentForm 组件发布评论后未触发列表刷新回调

修复方案：
1. 添加 onCommentPosted prop
2. 提交成功后调用回调
3. 父组件传递刷新函数

是否执行修复？
```

**修复完成后**：
```
修复完成！请按以下步骤验证：

1. 打开文章详情页
2. 发布评论
3. 确认列表立即刷新

手动验证步骤：
- [ ] 发布评论后列表刷新
- [ ] 连续发布评论正常
- [ ] 其他页面正常

修复报告已保存到 docs/BUG修复文档/BUG-2024-001.md
```

## 常见BUG类型处理

### 前端BUG
- 重点：复现步骤、浏览器环境、控制台日志
- 定位：检查组件逻辑、状态管理、API调用

### 后端BUG
- 重点：请求参数、响应数据、服务端日志
- 定位：检查API逻辑、数据库查询、错误处理

### 性能BUG
- 重点：数据量、操作频率、环境配置
- 定位：使用性能分析工具、检查算法复杂度

### 兼容性BUG
- 重点：浏览器版本、操作系统、设备类型
- 定位：检查特定环境的兼容性问题
