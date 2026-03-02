---
name: operate-log-guide
description: 新增操作日志（Operate Log）接入规范与完整步骤（PHP + Strategy 模式 + MongoDB）
---

## 🔐 Execution Contract

If this skill is triggered, the assistant MUST begin the response with:

=== USING SKILL: operate-log-guide ===

Failure to output this marker means the skill was not used.

---


## When to Use This Skill

当出现以下任意请求时必须使用本 Skill：

- 新增操作日志
- 添加 operateLog
- 新增 action 日志
- 增加 SysOperateLogModel 记录
- 新增成员管理日志

# 新增操作日志（Operate Log）接入 Skill

## 目标
为系统新增一种操作日志记录：从业务代码触发 `operateLog()`，最终写入 MongoDB，并支持前端按 `type / operate_type` 筛选展示。

## 核心概念
- **action**：业务触发动作标识（snake_case 字符串）
- **type**：日志大类（成员管理 / 账号管理 等），用于前端大类筛选
- **operate_type**：操作类型（与 action 值保持一致），用于前端细分筛选
- **operate_log**：业务详情字段（数组 → JSON 存 Mongo）

## 涉及文件（按顺序改）
1. `modules/sys/services/optLog/UserOptLogService.php`：定义 `ACTION_*` 常量
2. `modules/common/models/sys/SysOperateLogModel.php`：定义 `OPERATE_TYPE_*` 常量
3. `modules/sys/services/optLog/strategies/<XxxStrategy>.php`：注册 action + 编写 handler
4. `modules/sys/services/optLog/factories/LogStrategyFactory.php`：仅新建 Strategy 时注册策略类
5. 业务调用处：调用 `NewUserOptLogService::operateLog()`

---

## Step 1：定义 ACTION 常量
文件：`UserOptLogService.php`

在对应分类注释下新增常量：

```php
// 成员管理
const ACTION_ADD_MEMBER = 'add_member';
const ACTION_BIND_MEMBER_TFA = 'bind_member_tfa';   // <-- 新增
```

命名规范：
- 常量名：`ACTION_<动作>_<对象>`
- 值：`snake_case` 字符串

---

## Step 2：定义 OPERATE_TYPE 常量
文件：`SysOperateLogModel.php`

```php
const OPERATE_TYPE_ADD_MEMBER = 'add_member';
const OPERATE_TYPE_BIND_MEMBER_TFA = 'bind_member_tfa';  // <-- 新增
```

命名规范：
- 常量名：`OPERATE_TYPE_<动作>_<对象>`
- 值：与 ACTION 的值保持一致

---

## Step 3：在 Strategy 中注册和处理

### 3a：选择已有 Strategy 或新建
已有策略（优先复用）：

| Strategy | 日志类型 (type) | 适用场景 |
|---|---|---|
| `MemberManagementStrategy` | `SysOperateLogModel::TYPE_MEMBER_MGMT` | 成员管理相关 |
| `AccountManagementStrategy` | `SysOperateLogModel::TYPE_ACCOUNT_MGMT` | 账号/环境管理相关 |

如果新操作属于已有分类，直接在已有 Strategy 中添加；否则新建 Strategy（见 Step 4）。

### 3b：注册 action
在 `getSupportedActions()` 数组中添加：

```php
public function getSupportedActions(): array
{
    return [
        // ...已有的
        UserOptLogService::ACTION_BIND_MEMBER_TFA,  // <-- 新增
    ];
}
```

### 3c：在 handle() 中添加 case
```php
public function handle(array $logData): array
{
    $baseData = $this->getBaseLogData($logData);
    $action = $logData['action'];

    switch ($action) {
        // ...已有的 case
        case UserOptLogService::ACTION_BIND_MEMBER_TFA:
            return $this->handleBindMemberTfa($baseData, $logData);
        default:
            return [];
    }
}
```

### 3d：编写 handler 方法
```php
private function handleBindMemberTfa($baseData, $logData)
{
    // 从 $logData 中取出需要记录的业务字段
    $operateLog = [
        'target_user_id' => $logData['target_user_id'] ?? 0,
        'target_email'   => $logData['target_email'] ?? '',
    ];

    return $this->buildLogData(
        $baseData,
        SysOperateLogModel::TYPE_MEMBER_MGMT,                // type: 日志大类
        SysOperateLogModel::OPERATE_TYPE_BIND_MEMBER_TFA,    // operate_type: 操作类型
        $operateLog                                          // operate_log: 业务详情 (JSON)
    );
}
```

说明：
- `$baseData` 由 `AbstractLogStrategy::getBaseLogData()` 自动填充，包含：`company_id`, `sys_user_id`, `sys_user_name`, `ip`, `country`, `region`, `city`, `created_time`
- `$operateLog` 是自定义业务字段，会被 JSON 编码后存入 `operate_log`
- `type` 和 `operate_type` 用于前端筛选和展示

---

## Step 4：（仅新建 Strategy 时）注册策略类
文件：`LogStrategyFactory.php`

```php
private static function getStrategyClasses(): array
{
    return [
        AccountManagementStrategy::class,
        MemberManagementStrategy::class,
        NewStrategy::class,  // <-- 新增
    ];
}
```

新 Strategy 需继承 `AbstractLogStrategy` 并实现 `getSupportedActions()` 和 `handle()`：

```php
<?php
namespace app\modules\sys\services\optLog\strategies;

use app\modules\common\models\sys\SysOperateLogModel;
use app\modules\sys\services\optLog\UserOptLogService;

class NewStrategy extends AbstractLogStrategy
{
    public function getSupportedActions(): array
    {
        return [
            UserOptLogService::ACTION_XXX,
        ];
    }

    public function handle(array $logData): array
    {
        $baseData = $this->getBaseLogData($logData);
        // ...
    }
}
```

---

## Step 5：在业务代码中调用
```php
use app\modules\sys\services\optLog\NewUserOptLogService;
use app\modules\sys\services\optLog\UserOptLogService;

// 必传字段：company_id, sys_user_id, action
// 其余字段根据 handler 中定义的 $operateLog 按需传入
try {
    NewUserOptLogService::getInstance()->operateLog([
        'company_id'     => $operatorInfo['company_id'],
        'sys_user_id'    => $operatorInfo['id'],          // 操作人 ID
        'action'         => UserOptLogService::ACTION_BIND_MEMBER_TFA,
        'target_user_id' => $targetUserId,                // 自定义业务字段
        'target_email'   => $targetEmail,                 // 自定义业务字段
    ]);
} catch (\Exception $e) {
    // 日志记录失败不影响主流程
}
```

---

## 数据流向（用于排查）
```
业务代码调用 operateLog()
    -> NewUserOptLogService::operateLog()
        -> LogStrategyFactory::getStrategy($action)  // 根据 action 找到 Strategy
        -> Strategy::handle($logData)                // 组装日志数据
        -> NewUserOptLogService::insertToMongo()     // 写入 MongoDB
```

---

## 验收 Checklist
- [ ] ACTION 常量新增（命名/值符合规范）
- [ ] OPERATE_TYPE 常量新增（值与 action 一致）
- [ ] action 已加入 Strategy 的 `getSupportedActions()`
- [ ] `handle()` switch 能命中并返回非空数组
- [ ] handler 的 `type / operate_type / operate_log` 正确
- [ ] 业务代码使用 try-catch，日志失败不阻断主流程
- [ ] Mongo 中能看到新日志记录，前端筛选可用
