---
name: external-member-notify
description: 规划并实现团队外部成员加入/离开通知需求，优先复用已有邀请入团队、成员删除、模板通知和异步事件链路；适用于接口改造、事件补发、模板映射和验收设计。
---

## Execution Contract

If this skill is triggered, the assistant MUST begin the response with:

=== USING SKILL: external-member-notify ===

---

## When to Use This Skill

当出现以下任意请求时必须使用本 Skill：

- 规划团队成员加入/离开通知需求
- 新增外部成员加入团队通知
- 新增外部成员离开团队通知
- 评估团队通知接口改动
- 参考已有逻辑复用完成通知需求

## Goal

在不破坏现有接口兼容性的前提下，完成“仅外部成员生效”的团队加入/离开通知需求，并优先复用：

- 邀请链接加入团队成功链路
- 成员删除成功链路
- 现有通知发送封装
- 现有模板中心与多语言逻辑
- 现有异步事件 / outbox / 队列能力

## Scope

仅处理以下场景：

- 外部成员通过邀请链接成功加入团队后触发加入通知
- 外部成员被团队管理员彻底删除成功后触发离开通知

不处理：

- 内部成员加入或删除
- 团队被删除导致成员无法访问团队
- 删除失败、回滚、半成功状态下的通知

## Business Rules

### 成员范围

- 外部成员：通过邀请加入团队的成员
- 内部成员：不触发通知

如果系统已有 `member_type`、`join_source`、`source_type` 等稳定字段，直接复用；不要重新发明判定规则。

### 触发时机

- 加入团队通知：邀请入团队成功后触发
- 离开团队通知：彻底删除外部成员成功后触发
- 一律在主事务成功提交后异步发送通知

### 模板变量

- `user_name`：邮箱 > 手机号 > 用户 ID
- `team_name`：团队名称
- `operator`：邮箱 > 手机号 > 用户 ID
- 系统操作：中文显示“系统”，英文显示“System”
- `operator_date`：统一展示为 UTC+8

### 模板映射

加入团队通知：

- 邮件中文：313
- 邮件英文：314
- 站内信中文：315
- 站内信英文：316

离开团队通知：

- 邮件中文：317
- 邮件英文：318
- 站内信中文：319
- 站内信英文：320

## Reuse Priority

必须按以下顺序复用已有能力，只有找不到现成逻辑时才新增：

1. 现有邀请接受成功后的后置处理
2. 现有删除团队成员成功后的后置处理
3. 现有通知服务 / 消息中心封装
4. 现有模板选择与多语言逻辑
5. 现有异步事件或 outbox 机制

明确禁止：

- 为通知单独复制一套邀请入团逻辑
- 在主接口里同步发送邮件和站内信
- 仅为显示名称新增一套重复的格式化逻辑

## Interface Change Strategy

默认策略：优先内部复用，最小化外部接口改动。

### 1. 邀请入团队成功接口

优先复用现有邀请接受接口，不改请求和响应结构。

仅新增服务端后置动作：

- 成功入团
- 判定成员为外部成员
- 写 outbox 或发布 `team.member_joined_by_invite.v1`

### 2. 删除团队成员接口

优先复用现有删除成员接口。

如果系统已经能明确区分“彻底删除”和“普通移除”，则不改接口，仅在彻底删除成功后补发事件。

如果系统当前无法区分删除语义，则只补最小必要字段，例如：

```json
{
  "delete_mode": "hard"
}
```

只有 `delete_mode = hard` 且目标成员为外部成员时才触发离开通知。

### 3. 成员查询 / 成员详情接口

仅当系统缺少稳定的外部成员标识时，才补充只读字段：

```json
{
  "member_type": "external",
  "join_source": "invite"
}
```

已有等价字段时不新增。

### 4. 通知发送接口

必须优先复用现有通知封装。

如果当前通知服务已经支持“模板 ID + 变量 + 渠道 + 语言”模式，则只补新场景调用，不新增新接口。

只有在现有通知服务不支持渠道或模板变量时，才新增统一内部接口，例如：

```json
{
  "biz_scene": "team_member_joined",
  "receiver_user_id": "u_123",
  "language": "zh-CN",
  "channels": ["mail", "inbox"],
  "template_ids": {
    "mail": 313,
    "inbox": 315
  },
  "template_vars": {
    "user_name": "user@example.com",
    "team_name": "Example Team",
    "operator": "系统",
    "operator_date": "2026-03-09 18:00:00"
  },
  "idempotency_key": "team_member_joined:t_1:u_1:evt_1"
}
```

## Recommended Event Contracts

优先复用已有领域事件模型；没有时新增以下事件。

### 加入团队事件

```json
{
  "event_name": "team.member_joined_by_invite.v1",
  "event_id": "evt_xxx",
  "occurred_at": "2026-03-09T10:00:00Z",
  "team_id": "t_123",
  "team_name": "Example Team",
  "member_user_id": "u_123",
  "member_email": "user@example.com",
  "member_phone": "13800000000",
  "member_type": "external",
  "join_source": "invite",
  "operator_type": "system",
  "operator_user_id": "",
  "operator_email": "",
  "operator_phone": ""
}
```

### 离开团队事件

```json
{
  "event_name": "team.external_member_hard_deleted.v1",
  "event_id": "evt_xxx",
  "occurred_at": "2026-03-09T10:05:00Z",
  "team_id": "t_123",
  "team_name": "Example Team",
  "member_user_id": "u_123",
  "member_email": "user@example.com",
  "member_phone": "13800000000",
  "member_type": "external",
  "delete_mode": "hard",
  "operator_type": "user",
  "operator_user_id": "u_999",
  "operator_email": "admin@example.com",
  "operator_phone": ""
}
```

字段要求：

- 事件中尽量携带用户和团队快照，避免消费时查不到已删除数据
- `event_id` 用于通知幂等
- `occurred_at` 记录真实操作完成时间，不用消息消费时间

## Implementation Workflow

### Step 1: 先找已有逻辑

至少确认以下问题：

- 邀请入团队成功后当前有没有后置逻辑
- 删除成员成功后当前有没有后置逻辑
- 外部成员当前靠什么字段判定
- 通知服务当前是否已支持模板 ID 与多语言
- 是否已有 outbox / MQ / 异步任务框架

如果以上任一能力已经存在，直接复用，不要重做。

### Step 2: 固化触发条件

加入通知必须同时满足：

- 走邀请加入链路
- 入团成功
- 目标成员是外部成员

离开通知必须同时满足：

- 走成员删除链路
- 删除成功
- 删除模式为彻底删除
- 目标成员是外部成员

### Step 3: 事件后置化

在主事务提交后执行以下动作：

- 写 outbox
- 发布事件
- 或投递异步任务

禁止在事务内直接调用邮件或站内信接口。

### Step 4: 通知编排

通知消费侧完成：

- 选择用户语言
- 根据语言匹配模板 ID
- 填充 `user_name` / `team_name` / `operator` / `operator_date`
- 同时发送邮件和站内信
- 记录幂等键，支持重试

### Step 5: 验收回归

至少验证：

- 外部成员邀请加入触发通知
- 内部成员不触发
- 外部成员彻底删除触发通知
- 普通移除不触发
- 删除失败不触发
- 团队删除联动不触发
- 系统操作人显示正确
- 无邮箱时回退手机号，再回退用户 ID
- 所有模板时间均为 UTC+8

## Review Checklist

- [ ] 已明确复用的邀请、删除、通知、模板、多语言、异步链路
- [ ] 未为本需求新增重复业务流程
- [ ] 对外接口仅做最小必要改动
- [ ] 删除语义不清时已补 `delete_mode` 或等价字段
- [ ] 事件只在成功提交后发送
- [ ] 事件体已包含必要快照字段
- [ ] 通知发送具备幂等和重试能力
- [ ] 模板 ID 313-320 映射完整
- [ ] 内部成员和团队删除场景被明确排除
