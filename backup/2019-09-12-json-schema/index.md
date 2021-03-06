---
title: JSON Schema
description: A JSON based format for defining the structure of JSON data
date: '2019-09-12T02:46:50.792Z'
---

JSON Schema(`media type: application/schema+json`) 是一种**基于 JSON 格式**、**描述 JSON 数据结构**的规范。JSON Schema 描述了 JSON 数据的结构层次、字段类型、键值约束，可以用于数据校验、文档生成、超链接导航（hyperlink navigation）和基于 JSON 数据的交互控制（比如输入关键字的联想），其精准的数据定义可以辅助开发者和开发工具安全、高效地处理 JSON 数据。

---

## Core

这一部分介绍 JSON Schema 的核心定义（Core Definitions）和专用术语（Terminology），对应 JSON Schema 版本列表中的 Core 部分——内容以社区中支持最广泛的 JSON Schema Draft 04 为主。

#### 内置类型

JSON Schema 定义了以下七种类型描述 JSON 数据：

- `type: "array"`，表示当前数据是一个数组
- `type: "boolean"`，表示当前数据是一个布尔值
- `type: "integer"`，表示当前数据是一个整数
- `type: "number"`，表示当前数据是一个数值，此类型是 `type: "integer"` 的超集
- `type: "null"`，表示当前数据是 `null`
- `type: "object"`，表示当前数据是一个 Object
- `type: "string"`，表示当前数据是一个字符串

```json
{
    "title": "Person",
    "properties": {
        "name": {
            "type": "string"
        }
    }
}
```

#### 值相等

判断 JSON 中两个数据的相等性，需要满足以下条件：

1. 如果两个数据都是 `null`，则相等，反之进行下一层判断
1. 如果两个数据都是布尔值且值相等，则相等，反之进行下一层判断
1. 如果两个数据都是字符串且值相等，则相等，反之进行下一层判断
1. 如果两个数据都是数值且值相等，则相等，反之进行下一层判断
1. 如果两个数据都是数组，数组长度相同且相同索引位置的数据相等，则相等，反之进行下一层判断
1. 如果两个数据都是对象，对象属性数量相同且相同属性对应的值相等，则相等

#### "$schema"

JSON Schema 中的 `$schema` 关键字必须在 JSON Schema 文档的根结点上，值必须符合 [URI](https://tools.ietf.org/html/rfc3986) 和 [json-reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) 规范，必须是绝对路径且经过字符串标准化处理（normalize）。这一关键字有两个作用，一是标识当前 JSON Schema 的版本，二是表明从 `$schema` 所在的根结点位置开始，是一个独立的 JSON Schema 文档，实现多份 JSON Schema 文档嵌套表示的能力。

```json
{
    "id": "",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
}
```

#### Resolution Scope

JSON Schema 基于 [json-reference](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) 实现引用解析，同时进行了两点扩展：

- JSON Schema 使用 `id` 关键字表示引用关系
- JSON Schema 支持任意字符串的 `#`（Hash Fragment）引用，比如 `#anothoer-schema`

通过设置 `id` 属性，相当于为 `id` 所在的 JSON Schema 文档区域设置了一个解析区域（Resolution Scope）。JSON Schema 文档默认的解析区域是 JSON Schema 的 URI，如果 JSON Schema 文档并非通过 URI 下载而来，则为 `$schema` 的值或为空。

`id` 的值必须是字符串和有效的 URI，必须经过字符串的标准化处理（normalize），不能为空或者空 `#`。当 JSON Schema 文档解析器遇到 `id` 时，根据 `id` 生成的 URI 成为新的解析区域，并且需要 JSON Schema 文档的生成源保证 `id` 的唯一性。以下面代码为例：

```json
{
    "id": "http://x.y.z/rootschema.json#",
    "schema1": {
        "id": "#foo"
    },
    "schema2": {
        "id": "otherschema.json",
        "nested": {
            "id": "#bar"
        },
        "alsonested": {
            "id": "t/inner.json#a"
        }
    },
    "schema3": {
        "id": "some://where.else/completely#"
    }
}
```

根据上面的 JSON Schema 文档示例，可以解析出以下解析区域（Resolution Scope）：

- `# (document root)`，[http://x.y.z/rootschema.json#]()
- `#/schema1`, [http://x.y.z/rootschema.json#foo]()
- `#/schema2`, [http://x.y.z/otherschema.json#]()
- `#/schema2/nested`, [http://x.y.z/otherschema.json#bar]()
- `#/schema2/alsonested`, [http://x.y.z/t/inner.json#a]()
- `#/schema3`, [some://where.else/completely#]()

有两种机制可以生成解析区域：`Canonical dereferencing` 和 `Inline dereferencing`。以下面代码为例：

```json
{
    "id": "http://my.site/myschema#",
    "definitions": {
        "schema1": {
            "id": "schema1",
            "type": "integer"
        },
        "schema2", {
            "type": "array",
            "items": { "$ref": "schema1" }
        }
    }
}
```

当解析器时遇到 `"id": "schema1"` 时，首先生成解析区域 `"http://my.site/schema1#"`，处理这一解析区域可以使用两种方法：

- `Canonical dereferencing`，解析器直接基于 URI 请求数据内容
- `inline dereferencing`，解析器通过查找发现 `"http://my.site/schema1#"` 在当前 JSON Schema 文档内，则直接使用

---

## Validation

这一部分介绍校验 JSON Schema 文档有效性的详细规则，对应 JSON Schema 版本列表中的 Validation 部分——内容以社区中支持最广泛的 JSON Schema Draft 04 为主。

#### 数值

- `multipleOf` 是数值且值大于 0
    - 对应的 JSON 数值除以 `multipleOf` 的数值，结果必须是整数
- `maximum` 是数值，`exclusiveMaximum` 是布尔值，如果设置了 `exclusiveMaximum`，则必须设置 `maximum`
    - 如果未设置 `exclusiveMaximum` 或者该值为 false（默认值），则 JSON 数值必须小于等于 `maximum` 的值
    - 如果 `exclusiveMaximum` 的值为 true，则 JSON 数值必须小于 `maximum` 的值
- `minimum` 是数值，`exclusiveMinimum` 是布尔值，如果设置了 `exclusiveMinimum`，则必须设置 `minimum`
    - 如果未设置 `exclusiveMinimum` 或者该值为 false（默认值），则 JSON 数值必须大于等于 `minimum` 的值
    - 如果 `exclusiveMinimum` 的值为 true，则 JSON 数值必须大于 `minimum` 的值
- `maxLength` 和 `minLength` 是整数且大于等于 0
    - JSON 字符串的长度必须大于 `minLength`（默认值为 0）
    - JSON 字符串的长度必须小于 `maxLength`
- `pattern` 是字符串且为正则表达式
    - JSON 字符串必须匹配正则表达式的规则

#### 数组

- `additionalItems` 是布尔值或者 Object，如果是 Object，则必须是一个规范的 JSON Schema 文档；`items` 是 Object 或数组，如果是 Object，则必须是规范的 JSON Schema 文档，如果是数组，则数组的每个元素都必须是 Object 且为规范的 JSON Schema 文档
    - 如果 `items` 值为空或 Object，则任何 JSON 数据均为有效值，忽略 `additionalItems` 的值
    - 如果 `additionalItems` 的值为 true 或 Object，则任何 JSON 数据均为有效值
    - 如果 `additionalItems` 的值为 true 且 `items` 值为数组，则 JSON 数组的元素数要小于等于 `items` 数组的长度且匹配 JSON Schema 约束
- `maxItems` 和 `minItems` 是整数且大于等于 0（默认值为 0）
    - JSON 数组的长度必须小于等于 `maxItems` 的值
    - JSON 数组的长度必须大于等于 `minItems` 的值
- `uniqueItems` 是布尔值（默认值为 false）
    - 如果 `uniqueItems` 为 false，则任何 JSON 数组数据均为有效值
    - 如果 `uniqueItems` 为 true，JSON 数组中的任一元素必须具有唯一性

#### Object

- `maxProperties` 和 `minProperties` 是整数且大于等于 0
    - JSON Object 的属性数量必须小于等于 `maxProperties` 的值
    - JSON Object 的属性数量必须大于等于 `minProperties` 的值（默认值为 0）
- `required` 是数组且至少有一个元素，每个元素必须是字符串且具有唯一性
    - JSON Object 的属性必须包含 `required` 数组中的所有元素
- `dependencies` 是 Object，且该 Object 的每个属性值均为 Object 或数组
    - 如果 `dependencies` 是 Object，则 Object 为标准的 JSON Schema，称之为 `Schema Dependency`
    - 如果 `dependencies` 是数组，则数组元素为字符串、至少有一个元素、每个元素具有唯一性，称之为 `Property Dependency`
    - 对于 `Schema Dependency`，JSON 数据需要匹配对应 key-value 键值对的 Schema 约束
    - 对于 `Property Dependency`，JSON 数据的属性列表中需要包含依赖的属性
- `additionalProperties` 是布尔值或 Object（默认值为 false），如果是 Object，则必须是标准的 JSON Schema 文档
    - 如果 `additionalProperties` 为 true 或 Schema Object，则任何 JSON 数据均为有效值
- `properties` 是 Object（默认值为 `{}`），其每一个属性值都是 Object 且为标准的 JSON Schema 文档
- `patternProperties` 是 Object（默认值为 `{}`），且该 Object 的 key 为标准的正则表达式，其每一个属性值均为 Object 且为标准的 JSON Schema 文档

如果 `additionalProperties` 为 false，则需要进一步校验 `properties` 和 `patternProperties` 的属性集合，假设：

- `s` 表示待校验的属性集合
- `p` 表示 `properties` 指定的属性集合
- `pp` 表示 `patternProperties` 匹配的属性集合

首先从 `s` 集合中移除所有 `p` 集合的属性，然后从 `s` 集合中移除所有匹配 `pp` 集合的属性，最后检查 `s` 集合是否为空，如果为空，则待校验的 JSON 数据有效。

#### 所有类型

以下属性为所有 JSON Schema 类型拥有：

- `enum` 是数组且至少有一个元素，每个元素在数组中具有唯一性，每个元素可以是任意类型
    - JSON 数据匹配 `enum` 数组中的任一项即有效
- `type` 是字符串或数组，如果是数组则每一元素均为字符串且具有唯一性，字符串必须为 Core 规范中定义的七种类型之一
    - JSON 数据的类型必须符合 `type` 的定义
- `allOf` 是数组且至少有一个元素，每个元素均为 Object 且为标准的 JSON Schema 文档
    - JSON 数据必须匹配 `allOf` 数组中的每一条 Schema 定义
- `anyOf` 是数组且至少有一个元素，每个元素均为 Object 且为标准的 JSON Schema 文档
    - JSON 数据必须至少匹配 `anyOf` 数组中的一个 Schema 定义
- `oneOf` 是数组且至少有一个元素，每个元素均为 Object 且为标准的 JSON Schema 文档
    - JSON 数据必须匹配 `oneOf` 数组中的一条 Schema 定义，匹配项必须有且只有一个
- `not` 是 Object 且为标准的 JSON Schema
    - JSON 数据必须不匹配 `not` 定义的 Schema
- `definitions` 是 Object，且每个属性值均为标准的 JSON Schema 对象，校验 JSON 数据时无须校验 `definitions` 定义的 Schema，其作用更多复用 Schema

```json
{
    "type": "array",
    "items": {
        "$ref": "#/definitions/positiveInteger"
    },
    "definitions": {
        "positiveInteger": {
            "type": "integer",
            "minimum": 0,
            "exclusiveMinimum": true
        }
    }
}
```

#### metadata

- `title` 和 `description` 均为字符串，用于表述当前 Schema 的基本信息
- `default` 无类型限制，用于为任意属性提供默认值信息

#### format

`format` 在基本类型之外提供了语义化的描述信息，值必须为字符串，JSON Schema 预定义了以下 `format`:

- `date-time`，字符串，符合 [RFC 3339 定义](https://tools.ietf.org/html/rfc3339#section-5.6)
- `email`，字符串，符合 [RFC 5322 定义](https://tools.ietf.org/html/rfc5322#section-3.4.1)
- `hostname`，字符串，符合 [RFC 1034 定义](https://tools.ietf.org/html/rfc1034#section-3.1)
- `ipv4`，字符串，符合 [RFC 2673 定义](https://tools.ietf.org/html/rfc2673#section-3.2)
- `ipv6`，字符串，符合 [RFC 2373 定义](https://tools.ietf.org/html/rfc2373#section-2.2)
- `uri`，字符串，符合 [RFC 3986 定义](https://tools.ietf.org/html/rfc3986)

---

## References

- [JSON Scheam draft 04 - Core](https://tools.ietf.org/html/draft-zyp-json-schema-04)
- [JSON Scheam draft 04 - Validation](https://tools.ietf.org/html/draft-fge-json-schema-validation-00)
- [Understanding JSON Schema](https://json-schema.org/understanding-json-schema/index.html)
- [ajv - the fastest JSON schema validator](https://github.com/epoberezkin/ajv)
