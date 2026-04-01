---
name: feature-tech-design
description: 基于需求文档设计API接口、数据库表和核心逻辑；当需要制定功能的技术实现方案时使用
---

# 技术方案设计

## 任务目标
- 本Skill用于：基于需求文档设计技术实现方案
- 能力包含：API接口设计、数据库设计、核心逻辑设计、方案对比
- 触发条件：已完成功能需求澄清，需要制定技术方案

## 前置准备
- 必须存在 `specs/features/[功能名称].md`
- 建议存在 `specs/技术栈.md`
- 建议存在 `specs/项目结构.md`
- 工作目录：项目根目录

## 操作步骤

### 1. 读取前置文档
智能体扮演系统架构师角色，读取必要的文档：
```
读取 specs/features/[功能名称].md
读取 specs/技术栈.md（如果存在）
读取 specs/项目结构.md（如果存在）
读取 specs/开发规范.md（如果存在）
```

### 2. 分析需求和技术约束
基于需求文档分析：
- **功能需求**：核心功能和优先级
- **非功能需求**：性能、安全要求
- **技术约束**：已选技术栈、架构模式

### 3. 设计技术方案

#### 3.1 数据库设计
```markdown
### 1. 数据库设计

#### 表结构
```sql
CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  like_count INTEGER DEFAULT 0,
  status VARCHAR(20) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_parent_id ON comments(parent_id);
CREATE INDEX idx_comments_created_at ON comments(created_at DESC);
```

#### 字段说明
| 字段 | 类型 | 说明 | 约束 |
|-----|------|------|------|
| id | UUID | 主键 | NOT NULL |
| post_id | UUID | 文章ID | 外键，NOT NULL |
| user_id | UUID | 评论用户ID | 外键，NOT NULL |
| parent_id | UUID | 父评论ID | 外键，可为空 |
| content | TEXT | 评论内容 | NOT NULL，最大500字符 |
| like_count | INTEGER | 点赞数 | DEFAULT 0 |
| status | VARCHAR(20) | 状态 | active/deleted |
| created_at | TIMESTAMP | 创建时间 | DEFAULT NOW() |
```

#### 3.2 API接口设计
```markdown
### 2. API接口设计

#### POST /api/comments
**功能**：发表评论
**权限**：需要登录

**请求体**：
```json
{
  "post_id": "uuid",
  "parent_id": "uuid | null",
  "content": "string"
}
```

**响应**：
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "content": "string",
    "user": {
      "id": "uuid",
      "name": "string",
      "avatar": "string"
    },
    "created_at": "timestamp"
  }
}
```

**错误码**：
- 400：参数错误
- 401：未登录
- 403：无权限
- 500：服务器错误

#### GET /api/comments
**功能**：获取评论列表
**权限**：公开

**查询参数**：
- post_id（必需）：文章ID
- page：页码，默认1
- page_size：每页数量，默认20

**响应**：
```json
{
  "success": true,
  "data": {
    "total": 100,
    "page": 1,
    "page_size": 20,
    "items": [
      {
        "id": "uuid",
        "content": "string",
        "user": {...},
        "replies": [...],
        "like_count": 0,
        "created_at": "timestamp"
      }
    ]
  }
}
```
```

#### 3.3 核心逻辑设计
```markdown
### 3. 核心逻辑设计

#### 评论发布流程
```
用户提交评论
    ↓
验证登录状态 → 未登录 → 返回401
    ↓
验证内容长度 → 超过500字 → 返回400
    ↓
敏感词过滤 → 包含敏感词 → 返回400
    ↓
保存到数据库
    ↓
更新文章评论计数
    ↓
返回评论详情
```

#### 嵌套回复处理
- 最多支持2层嵌套
- parent_id为空：顶级评论
- parent_id不为空：回复评论
- 查询时递归获取子评论

#### 点赞功能
- 使用独立的likes表
- 用户只能点赞一次
- 使用事务更新点赞计数
```

#### 3.4 方案对比
```markdown
### 4. 方案对比

#### 嵌套回复实现方式

**方案A：邻接表模型（parent_id）**
- 优点：实现简单，查询直观
- 缺点：多层嵌套时递归查询性能差
- 适用：2层嵌套足够

**方案B：路径枚举模型（path字段）**
- 优点：查询性能好，排序方便
- 缺点：路径字段维护稍复杂
- 适用：多层嵌套场景

**推荐**：方案A
**理由**：需求限制了最多2层嵌套，邻接表完全够用，实现更简单
```

### 4. 验收标准映射
逐条检查验收标准的技术实现：

```markdown
### 5. 验收标准映射

| 验收标准 | 技术实现 | API接口 |
|---------|---------|--------|
| AC-01: 登录用户可发表评论 | POST /api/comments | POST /api/comments |
| AC-02: 评论字数限制 | content.length <= 500 | 验证层实现 |
| AC-03: 支持回复评论 | parent_id字段 | POST /api/comments |
| AC-04: 评论列表分页 | page, page_size参数 | GET /api/comments |
| AC-05: 访客可查看评论 | 无需认证 | GET /api/comments |
```

### 5. 生成技术方案文档
将所有内容整合为完整的技术方案文档，保存到 `specs/features/[功能名称]_技术方案.md`

### 6. 确认技术方案
向用户展示关键设计决策：
```
已读取需求文档 specs/features/评论功能.md，开始设计技术方案。

关于评论的嵌套回复，有两种实现方式：

选项 A：邻接表模型（每条评论记录 parent_id）
优点：实现简单，查询直观
缺点：多层嵌套时递归查询性能差

选项 B：路径枚举模型（每条评论记录完整路径 path）
优点：查询性能好，排序方便
缺点：路径字段维护稍复杂

推荐：A，因为需求限制了最多 2 层嵌套，邻接表完全够用

确认后我继续设计数据库表结构和 API 接口。
```

## 输出格式
- 文件路径：`specs/features/[功能名称]_技术方案.md`
- 文档格式：结构化的技术方案文档
- 内容要求：具体的SQL、API定义、流程图

## 注意事项

### 边界守卫规则
- **当前阶段只能做**：技术方案设计和数据库设计
- **禁止行为**：不涉及具体代码编写、不涉及任务拆解
- 如果用户问"开始写代码"，提醒："技术方案确定后，我们会在任务规划阶段拆解具体任务"

### 项目上下文协议
- 执行前必须读取 `specs/features/[功能名称].md`
- 基于需求文档设计技术方案
- 符合 `specs/技术栈.md` 和 `specs/开发规范.md` 的要求

### 设计原则
- 每条验收标准都有对应的技术实现
- 提供多种方案时给出推荐和理由
- 考虑性能、安全、可维护性

## 使用示例

**用户输入**：
```
/feature-tech-design 评论功能
```

**智能体响应**：
```
已读取需求文档 specs/features/评论功能.md，开始设计技术方案。

关于评论的嵌套回复，有两种实现方式：

选项 A：邻接表模型（每条评论记录 parent_id）
优点：实现简单，查询直观
缺点：多层嵌套时递归查询性能差

选项 B：路径枚举模型（每条评论记录完整路径 path）
优点：查询性能好，排序方便
缺点：路径字段维护稍复杂

推荐：A，因为需求限制了最多 2 层嵌套，邻接表完全够用

确认后我继续设计数据库表结构和 API 接口。
```

**用户确认后**：
```
好的，继续设计数据库表结构：

CREATE TABLE comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES posts(id),
  user_id UUID NOT NULL REFERENCES users(id),
  parent_id UUID REFERENCES comments(id),
  content TEXT NOT NULL,
  like_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW()
);

API接口设计：
- POST /api/comments：发表评论
- GET /api/comments：获取评论列表
- DELETE /api/comments/:id：删除评论

详细技术方案已保存到 specs/features/评论功能_技术方案.md
```
