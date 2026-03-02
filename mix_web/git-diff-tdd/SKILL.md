---
description: 分析当前分支与 origin/master 的接口差异，生成符合 Yii2
  架构规范的技术设计与工时评估文档
name: branch-api-diff-design
tools: git
version: 2
---

# Branch API Diff Design Skill

# 语言规范

-   默认输出语言：zh-CN
-   所有技术分析、风险评估、工时评估使用中文
-   所有错误 message 必须为 en-US
-   严禁输出中文错误 message
-   输出必须结构化、可直接用于技术评审

# 使用场景

当出现以下情况必须使用本 Skill：

-   分支准备合并至 master
-   涉及接口变更
-   涉及参数变更
-   涉及数据库变更
-   涉及错误码变更
-   需要做风险与工时评估

# 执行步骤

## Step 1

git fetch origin

## Step 2

git diff --name-status origin/master...HEAD

# 分析优先级

必须按顺序分析：

1.  controllers
2.  validator
3.  services
4.  models
5.  response/ErrorCode.php
6.  config
7.  其他文件

# 重点目录

-   modules/\*/controllers/
-   modules/\*/validator/
-   modules/\*/services/
-   modules/common/models/
-   response/ErrorCode.php
-   config/

# Breaking Change 判定

以下属于破坏性变更：

-   删除接口
-   删除字段
-   新增必填字段
-   字段类型改变
-   返回结构改变
-   错误码语义改变

若存在，必须标记：

⚠️ 该分支包含破坏性变更

# 输出结构（强制）

# 技术设计文档

## 一、变更背景

-   需求来源
-   业务目标
-   风险等级（低 / 中 / 高）

## 二、接口变更说明

| 模块 \| 控制器 \| 方法 \| 变更类型 \| 是否破坏兼容 \| 影响范围 \|

## 三、参数变更对比

### 旧版本

``` json
{}
```

### 新版本

``` json
{}
```

必须说明兼容性结论。

## 四、返回结构校验

必须保持：

{ "code": 0, "msg": "","data": {} }

## 五、数据库影响评估

-   是否修改表结构
-   是否需要 migration
-   是否需要数据回填
-   是否影响读写分离
-   是否影响索引

## 六、错误码审查

确认：

-   全部在 ErrorCode.php
-   无硬编码
-   message 为英文

## 七、风险分析

必须覆盖：

-   向后兼容风险
-   数据一致性风险
-   性能风险
-   权限风险
-   部署风险

## 八、部署方案

1.  是否需要 migration
2.  是否灰度发布
3.  是否清理缓存
4.  回滚步骤
5.  回滚影响

# 九、工时评估（后端主程模型）

必须对每个接口单独评估。

## Complexity Points 计算

### 基础类型

-   新增接口：3
-   修改接口：2
-   小范围 bug 修复：1
-   删除或不兼容修改：4

### 参数复杂度

-   1-3 个字段：+1

-   4-10 个字段：+2

-   10 或复杂校验：+3

### 输出变更

-   data 扩展字段：+1
-   结构变化：+2
-   错误码体系变化：+2

### 权限复杂度

-   普通鉴权：+1
-   权限模型变化：+2

### 数据库复杂度

-   查询：+1
-   写入/事务：+2
-   表结构变更：+4
-   多表复杂关系：+3

### 外部依赖

-   跨模块：+1
-   支付/RPA/客户端协议：+3

### 不确定性

-   需求模糊：+2
-   灰度或兼容复杂：+2

## 工时映射

CP 1-2：2h\
CP 3-4：4h\
CP 5-6：8h\
CP 7-8：12h\
CP 9-10：16h\
CP 11+：24h+

必须输出三档：

-   乐观
-   常规
-   保守

## 输出表格

| 模块 \| 控制器 \| 方法 \| 变更摘要 \| CP \| 乐观(h) \| 常规(h) \|
  保守(h) \| 风险说明 \|

最后输出：

-   总工时（常规）
-   折算人日
-   是否建议拆分任务

# 强制校验清单

-   所有异常使用 ServiceException
-   无硬编码错误码
-   错误 message 英文
-   Validator default 在前
-   统一 OutPut 输出
-   读写分离规范
-   无 echo/exit

# 结论必须明确

-   是否建议合并
-   是否存在破坏性变更
-   是否建议升级 API 版本
-   是否建议拆分发布

# End
