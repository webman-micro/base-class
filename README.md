# 说明

本包为开发中用到的基类，仅支持TP ORM

# 过滤条件设计

![](/images/filter.png)

如上图所示，一个复杂的查询条件是可以分组逻辑or后与另外一个分组整体逻辑or，要实现这样逻辑有两种方案。

| 方案               | 说明                           |
|------------------|------------------------------|
| 1、多维数组递归组装       | 缺点数据很难验证和处理                  |
| 2、一维数组 + 一个字符串规则 | ❤️优势处理简单，需要严格限制字符串规则，防止sql注入 |

> 当前我选择了方案2

## 方案参数说明

多条数据参数设计：

```json
{
    "filter": {
        "param": {
            "id#1" : ["-eq", 2],
            "id#2" : ["-eq", 3],
            "phone" : ["-eq", 18888888888],
            "name" : ["-lk", "xxx"]
        },
        "rule": "(id {id#1} or phone {phone}) or (name {name} and id {id#2})"
    },
    "order": [
        {
            "field": "id",
            "order": "asc"
        }
    ],
    "page": [
        1,
        200
    ]
}
```

### 1. filter参数设计

filter 必须使用 {"param": {}, "rule": ""} 参数结构

1.1 param 格式必须为 id : ["-eq", 2]（字段: [过滤方法, 过滤参数]）

| 过滤方法         | 说明                     |
|--------------|------------------------|
| -gt          | > 大于                   |
| -egt         | >= 大于等于                |
| -lt          | >= 小于                  |
| -lk          | LIKE 包含                |
| -not-lk      | NOT LIKE 不包含           |
| -eq          | = 等于                   |
| -neq         | <> 不等于                 |
| -find_in_set | find_in_set 在字符串里面     |
| -bw          | BETWEEN 在两个参数之间范围      |
| -not-bw      | NOT BETWEEN 不在两个参数之间范围 |
| -in          | IN 在列表之中               |
| -not-in      | NOT IN 不在列表之中          |

1.2 字段名为验证方法 queryFilter:id,name 后面跟着的rule是当前允许查询的字段

>  可能存在一个字段被表达式多次使用且参数和过滤方法不一样，可以采用如下方式：

> param key 规则：字段名#数字

```json
{
  "param": {
    "id#1" : ["-eq", 2],
    "id#2" : ["-eq", 3]
  }
}
```

2.1 rule 只能包含，param里面字段名允许的字段和 () {} 空格

> 用()分组，{字段}这里会被参数替换的部分，字段等于param参数的key值

```(id {id#1} or phone {phone}) or (name {name} and id {id#2})```