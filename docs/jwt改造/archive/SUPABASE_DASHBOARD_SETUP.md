# GymBro APP Supabase数据库设置指南

**版本**: v1.0  
**更新时间**: 2025-09-29  
**适用范围**: Supabase Dashboard SQL Editor

## 📋 概述

本指南提供了在Supabase Dashboard中设置GymBro APP完整数据库结构的步骤。数据库结构基于Android Room数据库自动生成，包含29个表和完整的RLS策略。

## 🗂️ 数据库结构概览

### 核心模块表
- **用户管理**: `users`, `user_profiles`, `user_settings`
- **聊天系统**: `chat_sessions`, `chat_raw`, `message_embedding`
- **运动数据**: `exercise`, `exercise_search_history`, `exercise_usage_stats`
- **训练计划**: `workout_plans`, `plan_days`, `plan_templates`
- **训练会话**: `workout_sessions`, `session_exercises`, `session_sets`
- **统计数据**: `daily_stats`, `exercise_history_stats`
- **模板系统**: `workout_templates`, `template_exercises`, `template_versions`

### 支持功能表
- **搜索**: `search_content`, `exercise_fts`, `chat_fts`
- **日历**: `calendar_events`
- **令牌**: `tokens`
- **向量搜索**: `chat_vec`
- **会话摘要**: `session_summary`
- **记忆记录**: `memory_records`
- **自动保存**: `session_autosave`

## 🚀 快速设置步骤

### 步骤1：准备Supabase项目
1. 登录 [Supabase Dashboard](https://supabase.com/dashboard)
2. 选择您的GymBro项目
3. 导航到 **SQL Editor**

### 步骤2：执行数据库结构脚本
```sql
-- 在SQL Editor中执行以下文件内容
-- 文件位置: docs/jwt改造/GYMBRO_COMPLETE_SUPABASE_SCHEMA.sql
```

**重要提示**:
- 脚本包含1192行SQL代码
- 执行时间约2-5分钟
- 建议分段执行以避免超时

### 步骤3：验证设置结果
执行以下查询验证表创建：
```sql
-- 验证所有表已创建
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public' 
ORDER BY table_name;

-- 应该返回29个表
```

## 🔐 安全策略说明

### RLS策略矩阵
| 用户类型 | 数据访问 | 特殊限制 |
|----------|----------|----------|
| `authenticated` | 仅自己数据 | 基于 `auth.uid()` |
| `anonymous` | 仅自己数据 | 禁止公开分享 |
| `service_role` | 完全访问 | 后端服务专用 |

### 匿名用户支持
- ✅ 支持匿名用户基本功能
- ❌ 禁止创建公开分享
- 🗑️ 30天自动数据清理
- 📊 审计字段追踪用户类型

## 📊 关键特性

### 1. 自动UUID生成
```sql
-- 所有主键ID字段自动生成UUID
id UUID DEFAULT gen_random_uuid() PRIMARY KEY
```

### 2. 时间戳管理
```sql
-- 自动设置创建和更新时间
created_at TIMESTAMPTZ DEFAULT NOW(),
updated_at TIMESTAMPTZ DEFAULT NOW()
```

### 3. JSON字段支持
```sql
-- 设置和配置使用JSONB类型
settingsJson JSONB,
fitnessGoalsJson JSONB,
metadata JSONB DEFAULT '{}'
```

### 4. 全文搜索支持
- `exercise_fts`: 运动全文搜索
- `chat_fts`: 聊天全文搜索
- `search_content`: 通用搜索内容

### 5. 向量搜索支持
- `chat_vec`: 聊天向量存储
- `message_embedding`: 消息嵌入向量
- `user_profiles.vector`: 用户档案向量

## 🔧 高级配置

### 定时数据清理
```sql
-- 设置每日清理匿名用户数据（需要pg_cron扩展）
SELECT cron.schedule('cleanup-anonymous-data', '0 2 * * *', 
    'SELECT cleanup_anonymous_user_data(30);');
```

### 性能优化索引
脚本自动创建以下索引：
- 用户ID索引（所有用户相关表）
- 时间戳索引（支持时间范围查询）
- 状态字段索引（支持状态筛选）
- 用户类型审计索引（支持匿名用户分析）

## 🚨 注意事项

### 执行前检查
- [ ] 确认Supabase项目已创建
- [ ] 确认有足够的数据库配额
- [ ] 备份现有数据（如有）

### 执行期间
- ⏱️ 脚本执行需要2-5分钟
- 🔄 如遇超时，可分段执行
- 📝 注意观察执行日志

### 执行后验证
- [ ] 验证29个表已创建
- [ ] 验证RLS策略已启用
- [ ] 验证索引已创建
- [ ] 测试基本CRUD操作

## 🔍 故障排除

### 常见问题

#### 1. 扩展未启用
**错误**: `extension "uuid-ossp" does not exist`
**解决**: 在SQL Editor中执行：
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

#### 2. 权限不足
**错误**: `permission denied for schema public`
**解决**: 确认使用service_role密钥执行

#### 3. 表已存在
**错误**: `relation "users" already exists`
**解决**: 脚本使用`IF NOT EXISTS`，可安全重复执行

#### 4. 超时错误
**错误**: `Query timeout`
**解决**: 分段执行脚本，每次执行100-200行

### 分段执行建议
如遇超时，可按以下顺序分段执行：

1. **扩展和基础表** (第1-400行)
2. **索引创建** (第401-600行)
3. **外键约束** (第601-700行)
4. **RLS策略** (第701-1000行)
5. **匿名用户支持** (第1001-1192行)

## 📞 支持

如遇到问题，请检查：
1. Supabase项目状态
2. 数据库配额使用情况
3. SQL语法错误日志
4. 网络连接状态

## 📚 相关文档

- [Supabase RLS文档](https://supabase.com/docs/guides/auth/row-level-security)
- [PostgreSQL数据类型](https://www.postgresql.org/docs/current/datatype.html)
- [GymBro匿名用户实现报告](./ANON_IMPLEMENTATION_FINAL_REPORT.md)

---

**重要提醒**: 执行SQL脚本前请确保理解其影响，建议在测试环境中先行验证。
