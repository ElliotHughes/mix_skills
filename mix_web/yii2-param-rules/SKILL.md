---
name: yii2-param-rules
description: 基于 Yii2 官方 core validators，为接口参数、Form/Model 字段说明直接生成可落地的 `rules()`；适用于 required、string、integer、number、boolean、date/datetime/time、enum、array、compare、exist、unique、email、url、ip、file、image、default、trim、filter 等参数校验场景。
---

## Execution Contract

If this skill is triggered, the assistant MUST begin the response with:

=== USING SKILL: yii2-param-rules ===

---

## When to Use This Skill

当出现以下任意请求时必须使用本 Skill：

- 根据接口参数说明生成 Yii2 `rules()`
- 根据字段定义生成 `Model::rules()` 或 `Form::rules()`
- 补参数校验规则、请求校验规则、搜索条件校验规则
- 把接口文档 / JSON 参数表 / 需求描述转换成 Yii2 validator rules
- 需要判断某个字段该用 `string`、`integer`、`date`、`each`、`exist`、`unique` 等哪种规则

## Goal

把“参数定义”稳定转换为可直接使用的 Yii2 `rules()`，并尽量满足以下要求：

1. 优先使用 Yii2 core validator alias，而不是手写完整类名
2. 规则顺序合理，避免先后顺序导致行为错误
3. 不凭空发明业务规则、表结构或关联关系
4. 输出尽量可以直接粘贴到 `rules()` 方法中使用

## Required Reference

触发本 Skill 后，先读取：

- `references/validators.md`

该文件整理了 Yii2 官方 `Validator::$builtInValidators` 与 Core Validators 指南中的映射、常见参数和易错点。

## Workflow

### Step 1: 先把参数说明标准化

至少识别出以下信息：

- 字段名
- 是否必填
- 数据类型：`string / integer / number / boolean / array / date / datetime / time / file / image`
- 格式限制：邮箱、URL、IP、正则、枚举
- 范围限制：`min / max / length / range`
- 默认值
- 是否需要去空格或类型转换
- 是否依赖别的字段比较
- 是否需要查库存在性或唯一性

如果用户只给了半结构化描述，先整理成字段清单，再生成规则。

### Step 2: 按 Yii2 习惯决定规则顺序

默认按下面顺序输出：

1. `trim` / `filter`
2. `default`
3. `required`
4. 类型与格式校验：`boolean`、`integer`、`number`、`string`、`date`、`email`、`url`、`ip`、`match`、`in`
5. 数组元素校验：`each`
6. 跨字段比较：`compare`
7. 查库规则：`exist`、`unique`
8. 仅需放行但不校验的字段：`safe`

不要把会修改值的规则放到依赖原值的规则后面。

### Step 3: 选择正确的 validator

优先按下面原则选：

- 字符串长度用 `string`
- 整数用 `integer`
- 小数用 `number`，`double` 仅在用户明确要求时使用
- 布尔值用 `boolean`
- 枚举值用 `in`
- 正则格式用 `match`
- 数组字段本身是数组时，用 `each` 约束每个元素
- 日期相关优先用 `date` / `datetime` / `time`
- 两字段比较用 `compare`
- 查关联是否存在用 `exist`
- 查唯一性用 `unique`
- 上传文件用 `file`
- 图片上传用 `image`
- 仅赋默认值用 `default`
- 仅转换值用 `filter`
- 仅去首尾空格用 `trim`
- 仅允许批量赋值、不做校验时才用 `safe`

### Step 4: 生成规则时必须遵守的约束

- 默认输出 Yii2 alias，如 `'required'`、`'integer'`、`'string'`
- 不要无依据生成 `targetClass`、`targetRelation`、表名、字段名
- `default` 和 `filter` 会改值，不是“纯校验”
- `safe` 不是“可选字段”的同义词，它只是允许批量赋值
- `compare` 直接比较时只适合字符串和数字
- 日期区间比较优先使用 `date + timestampAttribute + compare`
- `trim` 应放在字符串必填校验之前
- 如果字段允许空且需要保存 `null`，优先考虑 `default` => `null`
- 如果用户没有要求类型强转，不要默认加 `filter => intval / boolval`

### Step 5: 输出格式

默认输出完整方法：

```php
public function rules()
{
    return [
        // ...
    ];
}
```

仅当用户明确要求“只要数组”时，再只输出 `return [...]` 内部内容。

输出后如存在缺失信息，只补一小段“假设 / 待确认”，例如：

- `exist` 的 `targetClass` 暂按 `Company::class` 处理
- 日期格式暂按 `php:Y-m-d`
- `status` 枚举值来自需求文档而不是代码常量

## Default Patterns

### 常见字符串参数

```php
[['keyword', 'name'], 'trim'],
[['keyword', 'name'], 'string', 'max' => 100],
```

### 必填字符串

```php
[['username'], 'trim'],
[['username'], 'required'],
[['username'], 'string', 'min' => 4, 'max' => 32],
```

### 分页参数

```php
['page', 'default', 'value' => 1],
['page_size', 'default', 'value' => 20],
[['page', 'page_size'], 'integer', 'min' => 1],
['page_size', 'integer', 'max' => 100],
```

### 枚举参数

```php
['status', 'in', 'range' => ['draft', 'published', 'archived']],
```

### 整数数组

```php
['tag_ids', 'each', 'rule' => ['integer', 'min' => 1]],
```

### 可空日期

```php
[['start_date', 'end_date'], 'default', 'value' => null],
[['start_date', 'end_date'], 'date', 'format' => 'php:Y-m-d'],
```

### 日期区间比较

```php
['start_date', 'date', 'format' => 'php:Y-m-d', 'timestampAttribute' => 'start_date_ts'],
['end_date', 'date', 'format' => 'php:Y-m-d', 'timestampAttribute' => 'end_date_ts'],
['start_date_ts', 'compare', 'compareAttribute' => 'end_date_ts', 'operator' => '<=', 'type' => 'number', 'enableClientValidation' => false],
```

如果模型里没有辅助属性，不要擅自生成 `_ts` 字段，除非用户允许你一起补属性定义。

## What to Avoid

- 不要把所有字段都标成 `safe`
- 不要把数组字段直接写成 `integer`
- 不要对日期范围直接用字符串 `compare`，除非格式天然可比且用户明确接受
- 不要在没有 AR 上下文时强行生成 `exist` / `unique`
- 不要把 `default` 当成“字段可选”的唯一表达
- 不要省略 `in` 的 `range`
- 不要生成无法运行的伪代码式 rules

## Checklist

- [ ] 每个字段的类型与业务约束都映射到了具体 validator
- [ ] 规则顺序不会造成明显行为问题
- [ ] `exist` / `unique` 的目标信息有来源
- [ ] 数组、日期、枚举、比较等特殊场景已单独处理
- [ ] 输出是可直接粘贴的 Yii2 `rules()` 代码
