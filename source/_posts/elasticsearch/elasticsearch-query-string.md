---
title: Elasticsearch 查询 - query string
date: 2020-07-02 09:17:03
tags: [ElasticSearch]
---

使用查询解析器来解析其内容的查询。下面是一个例子：

```
GET /_search
{
    "query": {
        "query_string": {
            "default_field": "content",
            "query": "this AND that OR thus"
        }
    }
}
```

`query_string` 查询解析输入并在运算符周围分割文本。每个文本部分彼此独立地分析。例如以下查询：

```
GET /_search
{
    "query": {
        "query_string": {
            "default_field": "content",
            "query": "(new york city) OR (big apple)"
        }
    }
}
```

将分成 `new york city` 或 `big apple`，然后通过为该字段配置的分析器独立地分析每个部分。

> 空格不被视为运算符，这意味着 new york city 将 “按原样” 传递给为该字段配置的分析器。如果该字段是关键字字段，则分析器将创建单个术语 new york city，并且查询构建器将在查询中使用此术语。如果要分别查询每个术语，则需要在术语周围添加显式运算符（例如 new AND york AND city）。

当提供多个字段时，也可以修改如何使用类型参数在每个文本部分内组合不同字段查询。这里描述了可能的模式，默认是 Bestfield。query_string 顶级参数包括：

* query: 要解析的实际查询。参见[查询字符串语法](https://www.elastic.co/guide/en/elasticsearch/reference/6.4/query-dsl-query-string-query.html#query-string-syntax)。
* default_field: 如果未指定前缀字段，则查询字词的默认字段。默认为 `index.query.default_field` 索引设置，而索引设置默认为 `*.*` 提取映射中符合术语查询条件的所有字段，并过滤元数据字段。然后组合所有提取的字段以在没有提供前缀字段时构建查询。
* default_operator: 如果未显式指定运算符，则使用默认运算符。例如，使用默认运算符 OR，查询 capital of Hungary 将转换为 capital OR of OR Hungary，并且使用默认运算符 AND，将相同的查询转换为 capital AND of AND Hungary。默认值为 OR。
* analyzer：用于分析查询字符串的分析器名称。
* quote_analyzer：分析器的名称，用于分析查询字符串中的引用短语。对于这些部件，它将覆盖使用 analyzer 参数或 search_quote_analyzer 设置设置的其他分析器。
* allow_leading_wildcard: 设置时，* 或 ? 允许作为第一个字符。默认为 true。
* enable_position_increments：设置为 true 在结果查询中启用位置增量。默认为 true。
* fuzzy_max_expansions：控制模糊查询将扩展到的术语数。默认为 50
* fuzziness：设置模糊查询的模糊性。默认为 AUTO。
* fuzzy_prefix_length：设置模糊查询的前缀长度。默认是 0.
* fuzzy_transpositions：设置为 false 禁用模糊转置（ab->ba）。默认是 true。
* phrase_slop: 设置短语的默认斜率。如果为 0，则需要精确的短语匹配。默认值是 0。
* boost：设置查询的提升值。默认为 1.0。
* auto_generate_phrase_queries：默认为 false。
* analyze_wildcard: 默认情况下，不分析查询字符串中的通配符。通过将此值设置为 true，将尽最大努力分析这些值。
* max_determinized_states: 限制允许创建的 regexp 查询的自动机状态数。这可以防止太难（例如指数级）的 regexp。默认为 10000。
* minimum_should_match: 一个值，用于控制生成的布尔查询中应该匹配的 "should" 子句的数量。它可以是绝对值（2），百分比（30%）或两者的组合。
* lenient：如果设置为 true 将导致基于格式的失败（如向数字字段提供文本）将被忽略。
* time_zone: 时区应用于与日期相关的任何范围查询。
* quote_field_suffix: 附加到查询字符串的引用部分的字段的后缀。这允许使用具有不同分析链的字段进行精确匹配。
* auto_generate_synonyms_phrase_query: 是否应为多项同义词自动生成短语查询。默认为 true。
* all_fields：执行上可以查询映射检测到的所有字段的查询。


## Default Field

如果未在查询字符串语法中明确指定要搜索的字段，`index.query.default_field` 则将使用该字段来派生要搜索的字段。如果 `index.query.default_field` 未指定，query_string 则将自动尝试确定索引映射中可查询的现有字段，并对这些字段执行搜索。请注意，这不包括嵌套文档，使用嵌套查询来搜索这些文档。


## Multi Field

该 query_string 查询还可以指定查询的字段。可以通过 "fields" 参数提供字段：

```
field1: query_term OR field2: query_term | ...
```

query_string 针对多个字段运行查询的想法是将每个查询字词扩展为 OR 子句，如下所示：

例如，以下查询：

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": ["content", "name"],
            "query": "this AND that"
        }
    }
}
```

匹配相同的单词

```
GET /_search
{
    "query": {
        "query_string": "(content:this OR name:this) AND (content:that OR name:that)"
    }
}
```

由于从单个搜索项生成了多个查询，因此使用 `dis_max` 带有 `tie_breaker` 的查询自动组合它们。例如（name 使用 ^5 符号表示增强 5）：

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": ["content", "name^5"],
            "query": "this AND that OR thus",
            "tie_breaker": 0
        }
    }
}
```

简单通配符也可以用于搜索文档的特定内部元素。例如，如果我们有一个 city 包含多个字段（或带有字段的内部对象）的对象，我们可以自动搜索所有 “城市” 字段：

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": ["city.*"],
            "query": "this AND that OR thus"
        }
    }
}
```

另一种选择是在查询字符串本身中提供通配符字段搜索（正确转义*符号），例如 `city.\*:something`：

```
GET /_search
{
    "query": {
        "query_string": {
            "query": "city.\\*:(this AND that OR thus)"
        }
    }
}
```

> 由于 \(反斜杠)是json字符串中的特殊字符，因此需要对其进行转义，因此上面的两个反斜杠 query_string。

query_string 对多个字段运行查询时，允许使用以下附加参数：

* type: 应如何组合字段以构建文本查询。

fields 参数还可以包括基于通配符字段名称，允许自动扩展到相关字段（包括动态引入的字段）。例如：

```
GET /_search
{
    "query": {
        "query_string": {
            "fields": ["content", "name.*^5"],
            "query": "this AND that OR thus"
        }
    }
}
```


## Query stirng 语法

查询字符串被解析为一些列术语和运算符。术语可以是单个单词 - quick 或 brown - 或短语，由双引号括起来 - "quick brown" 以相同的顺序搜索短语中的所有单词。

operator 允许你自定义搜索 - 可用选项如下：

### Field names

如 query_string 查询中所述 default_field，搜索搜索词，但可以在查询语法中指定其他字段：

* 其中 status 字段包含 active：
  - `status:active`
* 其中 title 字段包含 quick 或 brown。如果省略 OR 运算符，将使用默认运算符：
  - `title:(quick OR brown)`
  - `title:(quick brown)`
* 其中 author 字段包含精确短语 "john smith"
  - `author:"john smith"`
* book 中任何字段包含 quick 或 brown（注意我们需要对 * 使用反斜杠转义）
  - `book.\*:(quick brown)`
* 该字段 title 具有任何非 null 值：
  - `_exists_:title`


### Wildcards 通配符

通配符可以在单个术语上运行，使用 ? 替换单个字符，使用 *  替换零个或多个字符：

`qu?ck bro *`

* 请注意，通配符查询可能使用大量的内存，并且执行得很糟糕，只要想想需要查询多少项来匹配查询字符串 `a*b*c*`
* 纯通配符 \* 被重写为 exists。因此，通配符 "field:*" 将匹配具有空值的文档，如下所示：`{"field": ""}`，如果字段丢失或使用显式空值设置则不匹配：`{"field": null}`
* 允许在单词的开头（例如 "*ing"）使用通配符特别重，因为需要检查索引中的所有术语，以防它们匹配。可以通过设置 `allow_leading_wildcard` 为禁用前导通配符 false。


### Regular expressions 正则表达式

正则表达式模式可以通过将它们包装在 forward-slashes("/") 中嵌入查询字符串中：

`name:/jo?n(ath[oa]n)/`

正则表达式语法中解释了受支持的正则表达式语法。

> 该 `allow_leading_wildcard` 参数对正则表达式没有任何控制权。如下所示的查询字符串将强制 Elasticsearch 访问索引中的每个术语：/.*n/ 谨慎使用！


### Fuzziness 模糊

我们可以使用 "fuzzy" 运算符搜索与我们的搜索字词类似但不完全相同的字词：

```
quikc~ brwn~ foks~
```


### Ranges 范围

可以为日期，数字或字符串字段指定范围。包含范围用方括号指定，`[min TO max]` 排他范围用大括号指定 `{min TO max}`。

* All days in 2012:
  - `date:[2012-01-01 TO 2012-12-31]`
* Numbers 1..5
  - `count:[1 TO 5]` 
* Tags between alpha and omega，excluding alpha and omega:
  - `tag:{alpha TO omega}`
* Numbers from 10 upwards
  - `count:[10 TO *]`
* Dates before 2012
  - `date:{* TO 2012-01-01}`

可以组合使用大括号和方括号：

* 数字从 1 到 5 但不包括 5:
  - `count:[1 TO 5}`

* 一边无范围的范围可以使用如下语法：
  - `age:>10`
  - `age:>=10`
  - `age:<10`
  - `age:<=10`

要将上限和下限与简化语法结合使用，你需要将两个子句与 AND 运算符连接：

```
age:(>=10 AND <20)
age:(+>=10 +<20)
```


### 使用 Boost 提升权重

使用 boost 运算符 ^ 使一个术语比另一个术语更相关。例如，如果我们想要找到关于 foxes 的所有文档，但我们对 quick foxes 特别感兴趣：

```
quick^2 fox
```

默认 boost 值为 1，但可以是任何正浮点数。0 到 1 之间的提升会降低相关性。

提升也可以应用于短语或群组：

```
"john smith"^2 (foo bar)^4
```


### Boolean operators 布尔运算符

默认情况下，只要一个术语匹配，所有术语都是可选的。搜索 foo bar baz 将查找包含一个或多个 foo 或 bar 或 baz 的任何文档。我们已经讨论了 default_operator 上面的内容，它允许你强制要求所有的术语，但也有一些布尔运算符可以在查询字符串本身中使用，以提供更多的控制。

首选运算符+（此术语必须存在）和-（此术语不得出现）。所有其他条款都是可选的。例如，这个查询：

```
quick brown +fox -news
```

说明：

* fox 必须存在
* news 一定不能存在
* quick 和 brown 是可选的 - 它们的存在增加了相关性

熟悉的布尔运算符 AND，OR 以及 NOT （也写作 &&, || 和 !）也支持，但要小心，他们不遵守通常的优先级规则，所以每当多个运算符一起使用时，应使用括号。例如，以前的查询可以重写为：

```
((quick AND fox) OR (brown AND fox) OR fox) AND NOT news
```

此表单现在可以正确复制原始查询中的逻辑，但相关性评分与原始查询几乎没有相似之处。

相反，使用查询 bool 查询重写相同 match 查询将如下所示：

```
{
    "bool": {
        "must": {
            "match": "fox"
        },
        "should": {
            "match": "quick brown"
        },
        "must_not": {
            "match": "news"
        }
    }
}
```


### Grouping 分组

可以将多个术语或子句与括号组合在一起，以形成子查询：

```
(quick OR brown) AND fox
```

组可用于定位特定字段，或用于提升子查询的结果：

```
status:(active OR pending) title:(full text search)^2
```


### Reserved characters 保留字符

如果你需要在查询本身中使用任何作为运算符的字符（而不是运算符），那么你应该使用前导反斜杠来转义它们。例如，要搜索 (1+1)=2，你需要将查询编写为 `\(1\+1\)\=2`。

保留的字符是：`+ - = && || > < ! ( ) { } [ ] ^ " ~ * ? : \ /`

无法正确转义这些特殊字符可能会导致语法错误，从而阻止你的查询运行。

> `<and>` 根本无法转义。阻止它们尝试创建范围查询的唯一方法是安全从查询字符串中删除它们。


### Empty Query 空查询

如果查询字符串为空或仅包含空格，则查询将生成空结果集。

