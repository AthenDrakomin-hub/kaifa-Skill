---
name: feature-evolution
description: 分析功能变更类型并增量更新文档；当需要对已完成功能进行修改、扩展或重构时使用
---

# 功能迭代变更

## 任务目标
- 本Skill用于：处理已完成功能的增量修改
- 能力包含：变更影响分析、文档增量更新、变更任务生成
- 触发条件：用户对已完成功能提出修改需求

## 前置准备
- 必须存在 `specs/features/[功能名称].md`
- 必须存在 `specs/features/[功能名称]_技术方案.md`
- 必须存在 `specs/features/[功能名称]_任务规划.md`
- 工作目录：项目根目录

## 操作步骤

### 1. 读取所有相关文档
智能体扮演产品架构师角色，读取功能的完整文档：
```
读取 specs/features/[功能名称].md
读取 specs/features/[功能名称]_技术方案.md
读取 specs/features/[功能名称]_任务规划.md
读取 specs/产品概述.md（如果存在）
```

### 2. 分析变更类型
判断变更属于哪种类型：

**变更类型定义**：
- **Tweak（微调）**：小改动，不涉及新需求
  - 示例：调整评论排序方式、修改提示文案
  - 影响：小，通常只改几行代码

- **Extension（扩展）**：加新东西，需要新增验收标准
  - 示例：加点赞功能、支持图片评论
  - 影响：中等，需要新增字段、API、组件

- **Refactor（重构）**：动了核心结构
  - 示例：改了评论存储模型、重构嵌套逻辑
  - 影响：大，可能影响已有功能

### 3. 影响分析
分析变更对已有文档和代码的影响：

```markdown
# 变更分析报告

## 变更需求
用户想要：给评论功能加一个点赞功能

## 变更类型
**类型**：Extension（扩展）
**理由**：需要新增"点赞"相关的验收标准、新增 API 接口和数据库字段，但不破坏现有评论结构

## 影响范围

### 需求文档影响
- 需要新增验收标准：
  - AC-06：用户可点赞评论
  - AC-07：用户可取消点赞
  - AC-08：点赞数实时更新
- 需要更新功能描述

### 技术方案影响
- **数据库**：
  - comments 表新增 like_count 字段
  - 新增 comment_likes 关联表
- **API**：
  - 新增 POST /api/comments/:id/like
  - 新增 DELETE /api/comments/:id/like
- **组件**：
  - CommentItem 新增点赞按钮
  - 显示点赞数

### 已有代码影响
- 需修改文件：
  - src/modules/comments/services/commentRepository.ts（新增点赞方法）
  - src/modules/comments/components/CommentItem.tsx（新增点赞UI）
- 需新增文件：
  - migrations/004_add_comment_likes.sql
  - src/app/api/comments/[id]/like/route.ts
  - src/modules/comments/services/likeService.ts

### 回归风险
**风险等级**：低
**理由**：不影响已有的评论发布和展示逻辑，点赞功能相对独立

## 新增任务
需要新增 3 个开发任务：

### Task-09：创建点赞数据表
- **预计工时**：20分钟
- **涉及文件**：migrations/004_add_comment_likes.sql
- **依赖**：无
- **风险**：低

### Task-10：实现点赞 API
- **预计工时**：35分钟
- **涉及文件**：
  - src/app/api/comments/[id]/like/route.ts
  - src/modules/comments/services/likeService.ts
- **依赖**：Task-09
- **风险**：低

### Task-11：更新评论组件支持点赞
- **预计工时**：35分钟
- **涉及文件**：
  - src/modules/comments/components/CommentItem.tsx
- **依赖**：Task-10
- **风险**：低

**新增任务总工时**：90分钟
```

### 4. 增量更新文档
**不是推倒重写，而是增量更新**：

#### 4.1 更新需求文档
在现有文档基础上追加：

```markdown
## 变更历史

### 2024-01-20：新增点赞功能
**变更类型**：Extension

#### 新增验收标准
- AC-06：用户可点赞评论
  - Given：用户已登录
  - When：用户点击评论的点赞按钮
  - Then：评论点赞数+1，按钮状态变为已点赞
  
- AC-07：用户可取消点赞
  - Given：用户已点赞某评论
  - When：用户再次点击点赞按钮
  - Then：评论点赞数-1，按钮状态恢复未点赞

- AC-08：点赞数实时更新
  - Given：评论有点赞
  - When：用户查看评论列表
  - Then：显示当前点赞数
```

#### 4.2 更新技术方案
在技术方案中追加新章节：

```markdown
## 6. 变更记录

### 6.1 点赞功能扩展（2024-01-20）

#### 新增数据表
```sql
CREATE TABLE comment_likes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  comment_id UUID NOT NULL REFERENCES comments(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(comment_id, user_id)
);
```

#### 新增 API
**POST /api/comments/:id/like**
- 功能：点赞评论
- 权限：需要登录

**DELETE /api/comments/:id/like**
- 功能：取消点赞
- 权限：需要登录
```

#### 4.3 更新任务规划
在任务规划中追加新任务：

```markdown
## 7. 变更任务

### 2024-01-20：点赞功能扩展
- [ ] Task-09：创建点赞数据表 [20min]
- [ ] Task-10：实现点赞 API [35min]
- [ ] Task-11：更新评论组件支持点赞 [35min]

**当前进度**：37.5% (3/11 任务完成)
```

### 5. 生成变更文档
创建专门的变更文档：

```markdown
# 评论功能变更记录

## 变更列表

| 日期 | 变更类型 | 描述 | 影响等级 | 文档链接 |
|------|---------|------|---------|---------|
| 2024-01-20 | Extension | 新增点赞功能 | 低 | [详情](./变更_2024-01-20_点赞功能.md) |

## 详细变更

### 2024-01-20：新增点赞功能

#### 变更原因
用户反馈希望可以点赞评论，表达认同

#### 变更内容
- 新增点赞/取消点赞功能
- 显示评论点赞数
- 点赞状态实时更新

#### 影响范围
- 新增数据库表：comment_likes
- 新增 API：POST/DELETE /api/comments/:id/like
- 修改组件：CommentItem

#### 任务执行
- [x] Task-09：创建点赞数据表
- [x] Task-10：实现点赞 API
- [ ] Task-11：更新评论组件支持点赞
```

### 6. 确认变更方案
向用户展示变更分析：
```
已读取评论功能的全套文档，分析变更影响。

变更分类：扩展（Extension）
原因：需要新增"点赞"相关的验收标准、新增 API 接口和数据库字段，但不破坏现有评论结构。

影响分析：

需求文档：新增 2 条验收标准（AC-06、AC-07）
技术方案：新增 1 个 API，comments 表新增 like_count 字段
已有代码：需修改 2 个文件，新增 3 个文件
回归风险：低，不影响已有的评论发布和展示逻辑

需要新增 3 个开发任务（Task-09 到 Task-11），预计工时 90 分钟。

是否确认生成变更文档？
```

## 输出格式
- 更新文档：`specs/features/[功能名称].md`（增量更新）
- 更新技术方案：`specs/features/[功能名称]_技术方案.md`（追加章节）
- 更新任务规划：`specs/features/[功能名称]_任务规划.md`（追加任务）
- 变更记录：`specs/features/[功能名称]_变更记录.md`（新建）

## 注意事项

### 边界守卫规则
- **当前阶段只能做**：变更分析和文档更新
- **禁止行为**：不直接修改代码、不执行任务
- 如果用户说"直接改代码"，提醒："变更分析完成后，会用 feature-implementation 执行具体任务"

### 项目上下文协议
- 执行前必须读取功能的所有文档
- 变更要符合产品概述的整体方向

### 变更原则
- 增量更新，不推倒重写
- 保留变更历史
- 评估回归风险
- 生成新任务而非修改旧任务

## 使用示例

**用户输入**：
```
/feature-evolution 我想给评论功能加一个点赞功能
```

**智能体响应**：
```
已读取评论功能的全套文档，分析变更影响。

变更分类：扩展（Extension）
原因：需要新增"点赞"相关的验收标准、新增 API 接口和数据库字段，但不破坏现有评论结构。

影响分析：

需求文档：新增 2 条验收标准（AC-06、AC-07）
技术方案：新增 1 个 API，comments 表新增 like_count 字段
已有代码：需修改 2 个文件，新增 3 个文件
回归风险：低，不影响已有的评论发布和展示逻辑

需要新增 3 个开发任务（Task-09 到 Task-11），预计工时 90 分钟。

是否确认生成变更文档？
```

**用户确认**：
```
确认。
```

**智能体响应**：
```
变更文档已生成：

✓ 更新 specs/features/评论功能.md（新增验收标准）
✓ 更新 specs/features/评论功能_技术方案.md（追加点赞功能设计）
✓ 更新 specs/features/评论功能_任务规划.md（新增 Task-09~11）
✓ 创建 specs/features/评论功能_变更记录.md

下一步：请使用 feature-implementation Skill 执行新增任务（Task-09 到 Task-11）。
```

## 变更类型判断示例

### Tweak（微调）
```
用户：评论默认排序改为按时间倒序

分析：
- 类型：Tweak
- 理由：只是调整排序逻辑，无新需求
- 影响：修改 GET /api/comments 的排序逻辑
- 任务：1个（修改排序逻辑）
```

### Extension（扩展）
```
用户：支持图片评论

分析：
- 类型：Extension
- 理由：需要新增图片上传、存储、展示功能
- 影响：新增图片字段、上传API、图片组件
- 任务：3-5个
```

### Refactor（重构）
```
用户：把评论改成无限层级嵌套

分析：
- 类型：Refactor
- 理由：改变评论存储模型，影响核心逻辑
- 影响：重写嵌套逻辑、数据库结构调整、API重构
- 任务：5+个，回归测试量大
```
