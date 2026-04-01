---
name: feature-implementation
description: 按任务规划逐个执行编码、测试和验证；当需要按照既定方案实现具体功能时使用
---

# 编码实现

## 任务目标
- 本Skill用于：按照任务规划逐个执行开发任务
- 能力包含：代码编写、测试执行、代码审查、进度更新
- 触发条件：已完成任务规划，准备开始编码

## 前置准备
- 必须存在 `specs/features/[功能名称]_任务规划.md`
- 必须存在 `specs/features/[功能名称]_技术方案.md`
- 建议存在 `specs/features/[功能名称].md`
- 工作目录：项目根目录

## 操作步骤

### 1. 读取前置文档
智能体扮演高级开发工程师角色，读取必要的文档：
```
读取 specs/features/[功能名称]_任务规划.md
读取 specs/features/[功能名称]_技术方案.md
读取 specs/features/[功能名称].md（如果存在）
读取 specs/开发规范.md（如果存在）
```

### 2. 确认执行范围
明确要执行的任务或阶段：

```
正在读取相关文档…

需求文档：specs/features/评论功能.md ✓
技术方案：specs/features/评论功能_技术方案.md ✓
任务规划：specs/features/评论功能_任务规划.md ✓

第 1 阶段：数据层，包含 3 个任务。前置检查通过，无依赖阻塞。

即将开始执行 Task-01：创建 comments 表及索引
涉及文件：migrations/003_create_comments.sql
参考：技术方案 3.1 节 - 数据库设计

是否继续？
```

### 3. 执行任务
按顺序执行每个任务：

#### 3.1 任务执行流程
```
开始执行 Task-01：创建 comments 表及索引

【步骤1】创建迁移文件
正在创建 migrations/003_create_comments.sql …

【步骤2】执行迁移
正在执行数据库迁移 …

【步骤3】验证结果
正在验证表结构 …
✓ 表 comments 创建成功
✓ 索引创建成功
✓ 外键约束正常

【步骤4】更新任务状态
已在任务规划文档中标记 Task-01 为完成

Task-01 完成 ✓
```

#### 3.2 测试验证
每个任务完成后立即验证：
```
开始执行 Task-03：数据层单元测试

【步骤1】编写测试用例
正在创建 src/modules/comments/services/__tests__/commentRepository.test.ts …

【步骤2】执行测试
正在运行测试 …

PASS  src/modules/comments/services/__tests__/commentRepository.test.ts
  commentRepository
    ✓ create: should create a new comment (15ms)
    ✓ findById: should find comment by id (12ms)
    ✓ findByPostId: should find comments by post id (18ms)
    ✓ update: should update comment (14ms)
    ✓ delete: should delete comment (11ms)

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Coverage:    85%

Task-03 完成 ✓
```

### 4. 阶段完成报告
一个阶段完成后生成报告：

```markdown
# 第1阶段完成报告

## 完成时间
- 开始时间：YYYY-MM-DD HH:MM:SS
- 完成时间：YYYY-MM-DD HH:MM:SS
- 实际工时：95分钟（预估：105分钟）

## 完成任务
- [x] Task-01：创建 comments 表及索引 [30min]
- [x] Task-02：实现评论数据访问层 CRUD [35min]
- [x] Task-03：数据层单元测试 [30min]

## 代码变更
### 新增文件
- migrations/003_create_comments.sql
- src/modules/comments/services/commentRepository.ts
- src/modules/comments/services/__tests__/commentRepository.test.ts
- src/modules/comments/types/Comment.ts

### 修改文件
- 无

## 测试结果
- 测试用例：5个
- 通过率：100%
- 代码覆盖率：85%

## 遇到的问题
- 无

## 下一步
准备执行第2阶段：接口层（3个任务）
```

### 5. 更新任务状态
在任务规划文档中更新进度：
```markdown
## 6. 进度追踪
- [x] 第1阶段：数据层（3个任务）✓ 完成于 YYYY-MM-DD
- [ ] 第2阶段：接口层（3个任务）
- [ ] 第3阶段：表现层（2个任务）

**当前进度**：37.5% (3/8 任务完成)
```

### 6. 保存完成报告
将阶段完成报告保存到 `docs/开发记录/[功能名称]_第N阶段完成报告.md`

## 输出格式
- 代码文件：按技术方案生成
- 测试文件：每个任务对应的测试
- 完成报告：`docs/开发记录/[功能名称]_第N阶段完成报告.md`
- 进度更新：更新 `specs/features/[功能名称]_任务规划.md`

## 注意事项

### 边界守卫规则
- **当前阶段只能做**：按任务规划执行编码和测试
- **禁止行为**：不随意添加任务、不修改验收标准
- 如果发现任务规划有问题，提醒："任务规划可能需要调整，建议先用 feature-task-planning 更新规划"

### 项目上下文协议
- 执行前必须读取所有相关文档
- 严格遵循开发规范
- 代码必须符合技术方案设计

### 执行原则
- 一个阶段一个阶段执行，不一口气全做完
- 每个任务完成后立即写测试
- 测试通过才能进入下一个任务
- 每个阶段完成后可让用户检查

### 错误处理
- 测试失败：立即修复，不进入下一个任务
- 遇到阻塞：报告用户，等待决策
- 发现设计问题：暂停执行，建议更新技术方案

## 使用示例

**用户输入**：
```
/feature-implementation 完成评论功能的任务 第1阶段
```

**智能体响应**：
```
正在读取相关文档…

需求文档：specs/features/评论功能.md ✓
技术方案：specs/features/评论功能_技术方案.md ✓
任务规划：specs/features/评论功能_任务规划.md ✓

第 1 阶段：数据层，包含 3 个任务。前置检查通过，无依赖阻塞。

即将开始执行 Task-01：创建 comments 表及索引
涉及文件：migrations/003_create_comments.sql
参考：技术方案 3.1 节 - 数据库设计

是否继续？
```

**用户确认后**：
```
开始执行 Task-01：创建 comments 表及索引

【步骤1】创建迁移文件
正在创建 migrations/003_create_comments.sql …

【步骤2】执行迁移
正在执行数据库迁移 …

【步骤3】验证结果
正在验证表结构 …
✓ 表 comments 创建成功
✓ 索引创建成功
✓ 外键约束正常

Task-01 完成 ✓

即将执行 Task-02：实现评论数据访问层 CRUD
涉及文件：src/modules/comments/services/commentRepository.ts

是否继续？
```

**阶段完成**：
```
第1阶段：数据层已全部完成！

完成情况：
- Task-01：创建 comments 表及索引 ✓
- Task-02：实现评论数据访问层 CRUD ✓
- Task-03：数据层单元测试 ✓

测试结果：5/5 通过，覆盖率 85%
完成报告已保存到 docs/开发记录/评论功能_第1阶段完成报告.md

下一步：准备执行第2阶段（接口层）
是否继续？
```

## 异常处理

### 测试失败
```
Task-03 执行失败！

测试失败：
✗ create: should validate content length
  Expected: throw error
  Received: no error thrown

正在回滚代码变更…
已恢复到 Task-03 开始前的状态

建议：检查验证逻辑实现
是否查看失败代码？
```

### 发现设计问题
```
执行 Task-04 时发现技术方案可能存在问题：

问题：技术方案中未考虑评论审核功能
影响：需要新增审核状态字段和相关逻辑

建议：
1. 暂停当前执行
2. 更新技术方案
3. 更新任务规划
4. 重新执行

是否暂停并更新技术方案？
```
