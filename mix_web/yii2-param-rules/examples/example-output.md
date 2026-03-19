# Example Output

## Input

用户给出的参数说明：

- `company_id`：必填，整数，必须大于 0，并且要存在于 `Company::class` 的 `id`
- `keyword`：可选，字符串，去首尾空格，最大 50 字符
- `status`：可选，枚举值为 `draft / published / archived`
- `page`：可选，整数，默认 1，最小 1
- `page_size`：可选，整数，默认 20，范围 1-100
- `start_date`：可选，格式 `Y-m-d`
- `end_date`：可选，格式 `Y-m-d`，且不能早于 `start_date`
- `tag_ids`：可选，整数数组，每个元素都要大于 0
- `email`：可选，邮箱
- `callback_url`：可选，URL，只允许 `https`
- `is_export`：可选，布尔值，默认 `false`

## Output

```php
public function rules()
{
    return [
        [['keyword'], 'trim'],

        [['page'], 'default', 'value' => 1],
        [['page_size'], 'default', 'value' => 20],
        [['is_export'], 'default', 'value' => false],
        [['start_date', 'end_date', 'email', 'callback_url', 'status', 'tag_ids'], 'default', 'value' => null],

        [['company_id'], 'required'],

        [['company_id', 'page', 'page_size'], 'integer', 'min' => 1],
        ['page_size', 'integer', 'max' => 100],
        [['is_export'], 'boolean'],

        [['keyword'], 'string', 'max' => 50],
        [['status'], 'in', 'range' => ['draft', 'published', 'archived']],
        [['email'], 'email'],
        [['callback_url'], 'url', 'validSchemes' => ['https']],

        [['start_date'], 'date', 'format' => 'php:Y-m-d', 'timestampAttribute' => 'start_date_ts'],
        [['end_date'], 'date', 'format' => 'php:Y-m-d', 'timestampAttribute' => 'end_date_ts'],
        ['start_date_ts', 'compare', 'compareAttribute' => 'end_date_ts', 'operator' => '<=', 'type' => 'number', 'enableClientValidation' => false],

        [['tag_ids'], 'each', 'rule' => ['integer', 'min' => 1]],

        [['company_id'], 'exist', 'targetClass' => Company::class, 'targetAttribute' => ['company_id' => 'id']],
    ];
}
```

## Notes

- 这个输出假设模型中允许增加 `start_date_ts` 和 `end_date_ts` 辅助属性。
- 如果当前模型不方便新增辅助属性，应改成用户可接受的其它日期比较实现，而不是硬写不可运行的 `compare`。
