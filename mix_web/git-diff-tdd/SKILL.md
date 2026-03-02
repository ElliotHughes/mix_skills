---
name: branch-api-diff-design
description: 分析当前分支与 origin/master 的接口差异，并生成符合 Yii2 架构规范的技术设计文档
tools: git
---

# Branch API Diff Design Skill

## 语言要求

- 默认输出语言：zh-CN
- 所有技术说明使用中文
- 所有错误信息必须保持 en-US
- 禁止输出中文错误 message
- 文档必须结构化、工程化、可直接用于评审

---

# 适用场景

在以下情况必须使用本 Skill：

- 当前分支准备合并到 master
- 需要输出接口改动技术方案
- 需要做 API 变更影响评估
- 需要做破坏性变更分析
- 需要做发布风险评估

---

# 执行步骤

## Step 1：同步 master

git fetch origin

## Step 2：获取差异文件

git diff --name-status origin/master...HEAD

---

# 差异分析优先级

按以下顺序分析：

1. controllers
2. validator
3. services
4. models
5. response/ErrorCode.php
6. config
7. 其他文件

---

# 重点分析目录

- modules/*/controllers/
- modules/*/validator/
- modules/*/services/
- modules/common/models/
- response/ErrorCode.php
- config/

---

# 必须识别的变更类型

## 1️⃣ 控制器层

路径：
modules/<module>/controllers/*Controller.php

检查：

- 是否新增 action
- 是否删除 action
- 是否修改参数
- 是否修改返回结构
- 是否修改继承基类：
  - BaseController
  - AuthBaseController
  - ClientBaseController

分析：

- 是否破坏兼容
- 是否影响客户端
- 是否影响权限模型

---

## 2️⃣ 参数变更识别

重点检查：

- 新增字段
- 删除字段
- 必填 → 非必填
- 非必填 → 必填
- 类型变化
- 默认值变化
- 校验顺序是否正确（default 必须在前）

判断：

- 是否属于 Breaking Change
- 是否影响旧客户端

---

## 3️⃣ Service 层变更

路径：
modules/*/services/

检查：

- 新增异常
- 修改业务流程
- 是否新增事务
- 是否修改数据库写入逻辑

强制规则：

所有异常必须：

throw new ServiceException("Error message", ErrorCode::XXX);

禁止：

- 硬编码错误码
- 中文错误 message

---

## 4️⃣ Model 层

路径：
modules/common/models/

检查：

- 字段变化
- relation 变化
- 是否影响数据库结构

若影响 DB：

必须输出：

- migration 方案
- 数据回填策略
- 回滚策略

---

## 5️⃣ 错误码检查

文件：
response/ErrorCode.php

必须检查：

- 新增错误码是否定义
- 是否存在硬编码数字
- message 是否为 en-US

---

# 输出格式（强制）

必须严格按照以下结构输出：

---

# 技术设计文档

## 一、变更背景

- 业务目标
- 需求来源
- 风险等级（低 / 中 / 高）

---

## 二、接口变更说明

| 模块 | 控制器 | 方法 | 变更类型 | 是否破坏兼容 | 影响范围 |
|------|--------|------|----------|--------------|----------|

变更类型枚举：

- 新增接口
- 删除接口
- 参数新增
- 参数删除
- 参数类型修改
- 返回结构修改
- 权限模型变化

---

## 三、参数变更对比

### 旧版本

```json
{}