# Yii2 Core Validators Mapping

本文件基于 Yii2 官方 API 与官方 Guide 整理，用于把字段说明稳定映射成 `rules()`。

## Built-in Alias Mapping

| Alias | Class / Meaning | 常见用途 | 关键参数 | 生成 rules 时的注意点 |
| --- | --- | --- | --- | --- |
| `boolean` | `yii\validators\BooleanValidator` | 布尔值 | `trueValue` `falseValue` `strict` | HTML/Form 提交通常是字符串，默认不要随便开 `strict` |
| `captcha` | `yii\captcha\CaptchaValidator` | 验证码 | `caseSensitive` `captchaAction` `skipOnEmpty` | 这是 built-in validator，但类在 `yii\captcha` 命名空间 |
| `compare` | `yii\validators\CompareValidator` | 两字段或字段与常量比较 | `compareAttribute` `compareValue` `operator` `type` | 官方说明直接比较只适合字符串和数字；日期比较要配合 `date` |
| `date` | `yii\validators\DateValidator` | 日期 | `format` `timestampAttribute` `min` `max` | 适合 `Y-m-d` 这类纯日期输入 |
| `datetime` | `yii\validators\DateValidator` shortcut | 日期时间 | `format` `timestampAttribute` | 内置别名，本质还是 `DateValidator` |
| `time` | `yii\validators\DateValidator` shortcut | 时间 | `format` `timestampAttribute` | 内置别名，本质还是 `DateValidator` |
| `default` | `yii\validators\DefaultValueValidator` | 空值时赋默认值 | `value` | 它不做校验，只改值 |
| `double` | `yii\validators\NumberValidator` | 小数 | `min` `max` | 与 `number` 等价；默认优先用 `number` |
| `each` | `yii\validators\EachValidator` | 数组元素逐个校验 | `rule` `allowMessageFromRule` | 字段本身必须是数组，否则失败 |
| `email` | `yii\validators\EmailValidator` | 邮箱 | `allowName` `checkDNS` `enableIDN` | `checkDNS` 可能受 DNS 波动影响 |
| `exist` | `yii\validators\ExistValidator` | 查库存在性 | `targetClass` `targetAttribute` `targetRelation` `filter` `allowArray` | 只在 AR 上下文清晰时生成 |
| `file` | `yii\validators\FileValidator` | 文件上传 | `extensions` `mimeTypes` `minSize` `maxSize` `maxFiles` | 用于 `UploadedFile` |
| `filter` | `yii\validators\FilterValidator` | 值转换 / 归一化 | `filter` `skipOnArray` | 它不做校验，只改值 |
| `image` | `yii\validators\ImageValidator` | 图片上传 | `minWidth` `maxWidth` `minHeight` `maxHeight` | 继承 `file`，可同时配 `extensions` |
| `ip` | `yii\validators\IpValidator` | IP / 子网 | `ipv4` `ipv6` `subnet` `normalize` `negation` `expandIPv6` `ranges` | 适合 IP 白名单、网段输入 |
| `in` | `yii\validators\RangeValidator` | 枚举值 | `range` `strict` `not` `allowArray` | 枚举场景优先用它，不要手写 `match` 模拟 |
| `integer` | `yii\validators\NumberValidator` shortcut | 整数 | `min` `max` | 内置为 `NumberValidator + integerOnly => true` |
| `match` | `yii\validators\RegularExpressionValidator` | 正则格式 | `pattern` `not` | `pattern` 必填 |
| `number` | `yii\validators\NumberValidator` | 数字 | `min` `max` | 包括整数和小数；整数优先用 `integer` |
| `required` | `yii\validators\RequiredValidator` | 必填 / 必须等于某值 | `requiredValue` `strict` | `requiredValue` 未设置时校验“非空” |
| `safe` | `yii\validators\SafeValidator` | 仅允许批量赋值 | 无 | 它不做校验，不能替代“可选字段” |
| `string` | `yii\validators\StringValidator` | 字符串与长度 | `length` `min` `max` `encoding` | 文本字段最常用 |
| `trim` | `yii\validators\TrimValidator` | 去首尾空格 | 默认内置 `skipOnArray => true` | 适合字符串预处理，应放在 `required` 前 |
| `unique` | `yii\validators\UniqueValidator` | 查库唯一性 | `targetClass` `targetAttribute` `filter` | 只在 AR/表结构明确时生成 |
| `url` | `yii\validators\UrlValidator` | URL | `validSchemes` `defaultScheme` `enableIDN` | 只校验 scheme 和 host 是否合理，不负责 XSS 防护 |

## Rule Ordering Heuristic

默认推荐顺序：

1. `trim` / `filter`
2. `default`
3. `required`
4. 基础类型与格式
5. `each`
6. `compare`
7. `exist` / `unique`
8. `safe`

原因：

- `trim`、`filter`、`default` 会改值，应该先发生
- `required` 应在值被规范化后再判断
- `compare` 依赖前面字段已完成基础校验
- `exist`、`unique` 放后面，避免无意义查库

## Parameter-to-Rule Quick Mapping

### 字符串

- 普通文本：`trim + string`
- 必填文本：`trim + required + string`
- 固定长度：`string, 'length' => N`
- 长度区间：`string, 'min' => X, 'max' => Y`

### 整数 / 数字

- 正整数 ID：`integer, 'min' => 1`
- 金额 / 比率：`number, 'min' => 0`
- 分页：`default + integer + min/max`

### 布尔 / 枚举 / 正则

- 布尔：`boolean`
- 状态枚举：`in, 'range' => [...]`
- 自定义编码格式：`match, 'pattern' => ...`

### 数组

- 整数数组：`each, 'rule' => ['integer']`
- 字符串数组：`each, 'rule' => ['string', 'max' => ...]`
- 枚举数组：可用 `in, 'range' => [...], 'allowArray' => true`，也可用 `each + in`

### 日期

- 可空日期：`default => null + date`
- 固定边界：`date + min/max`
- 两字段日期区间：先 `date` 转成可比较值，再 `compare`

### 数据库

- 外键存在：`exist`
- 唯一索引：`unique`
- 组合字段：`targetAttribute` 用数组

没有明确的 `targetClass` / `targetRelation` / 字段映射时，不要猜。

## High-Risk Misuses

- `safe` 只代表可批量赋值，不代表“字段合法”
- `default`、`filter`、`trim` 都可能改值
- `compare` 不要直接拿来做日期字符串比较，除非格式天然可排序且需求明确
- `each` 只保证数组元素规则，不会自动替代数组本身的业务约束
- `exist` / `unique` 会查库，不适合在没有 AR 上下文时硬写
- `url` 不是安全防护工具，只是 URL 格式校验

## Official Sources

- API: https://www.yiiframework.com/doc/api/2.0/yii-validators-validator
- Guide: https://www.yiiframework.com/doc/guide/2.0/en/tutorial-core-validators
