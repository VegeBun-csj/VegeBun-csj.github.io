---
title: "CouchDB的基本语法"
date: 2020-10-11T21:16:52+08:00
tags:
- CouchDB
Categories:
- 数据库
---

# CouchDB基本语法

CouchDB的查询语法是json格式的，在其中可以通过特定的字段构建查询的逻辑，它的selector语法和MongoDB是相似的

### 查询基础：

> 查询的格式：需要去指定1个或者多个字段以及它们对应需要的值

```sql
{
    "director": "Lars von Trier"
}
```

这个就是指定`director`字段需要为`Lars von Trier`

```shell
{
    "name": "Paul",
    "location": "Boston"
}
```

上面就是指定了两个字段



### 子字段(Subfields)

比较复杂的selector语句会需要为某些嵌套的对象或者字段指定一些值，可以用json结构来指定一个字段和子字段

```json
{
    "imdb": {
        "rating": 8
    }
}
```

当然也可以采用缩写的方式(使用`.`进行连接)

```json
{
    "imdb.rating":8
}
```



### 操作符

> 通过在名称字段中使用美元符号（`$`）前缀来标识运算符。

在selector语句中有两种重要的操作符：

- Combination operators(组合操作符)
- Condition Operators(条件操作符)

组合操作符在selector语句中有最高的优先级，通常被用来组合一些条件，创建一些条件组合

每一个显式的操作符有以下的形式：

```json
{
    "$opeartor":"argument"
}
```

如果一个selector语句中没有显式的操作符，就会被创建隐式的运算符（通常是根据特定的selector语句被创建的）



### 隐式操作符

有两种隐式操作符

- Equality（等于）
- And（并且）

在一个selector语句中，任何一个字段都包含一个Value，但是可以没有操作符（此时就是默认的为一个equality），隐式的equality也同样适用于子字段

任何一个没有参数的json对象都有一个隐式的`$and`操作符在每个字段上

在下面的例子中，使用一个操作符来匹配所有文档中，字段`year`大于`2010`

```json
{
    "year":{
        "$gt":2010
    }
}
```

下面的例子是显式地使用`$eq`

```json
{
    "director":{
        "$eq":"this is a book"
    }
}
```

下面的是要求`imdb`的子字段`rating`必须为`8`

```json
{
    "imdb":{
        "rating":8
    }
}

//或者显式地使用`$eq`
{
    "imdb":{
        "rating":{
            "$eq":8
        }
    }
}
```

下面是`$eq`在全文索引中的使用

```json
{
    "selector":{
        "year":{
            "$eq":2001
        }
    },
    "sort":[
        "title:string"
    ],
    "fields":[
        "title"
    ]
}
```

下面是在数据库索引中使用`$eq`

```json
{
    "selector":{
        "year":{
            "$eq":2001
        }
    },
    "sort":{
        "year"
    },
    "fields":[
        "year"
    ]
}
```

可以同时显式地使用`$and`和`eq`操作符

```json
{
    "$and": [
        {
            "director": {
                "$eq": "Lars von Trier"
            }
        },
        {
            "year": {
                "$eq": 2003
            }
        }
    ]
}
```

### 显式操作符

所有地操作符中，除了`$eq`和`and`操作符，其他的操作符必须显式地表示！！！

> 不同划分的角度看，可以看到操作符可以分为显式和隐式的

---

---

### Combination Operations（组合操作符）

组合操作符被用来组合selector，除了在大多数编程语言中出现的`boolean`操作符之外，这里还有三种组合操作符（`$all`、`$elemMatch`、`$allMatch`）可以用来操作json数组，还有一种可以用来操作json maps (`keyMapMatch`)

> 一个组合运算符只有一个参数，这个参数可以是`一个selector，也可以是一个selector数组`

下面是组合运算符

| Operator       | Argument | Purpose                                                      |
| -------------- | -------- | ------------------------------------------------------------ |
| `$and`         | Array    | 数组中所有的selector都匹配才行                               |
| `$or`          | Array    | 数组中任意一个selector匹配即可（所有的selector必须使用相同的索引） |
| `$not`         | Selector | selector不匹配的时候才可以                                   |
| `$nor`         | Array    | 数组中所有的selector都不匹配才行                             |
| `$all`         | Array    | 匹配一个数组中所有的元素才行                                 |
| `$elemMatch`   | Selector | Matches and returns all documents that contain an array field with at least one element that matches all the specified query criteria. |
| `$allMatch`    | Selector | Matches and returns all documents that contain an array field with all its elements matching all the specified query criteria. |
| `$keyMapMatch` | Selector | Matches and returns all documents that contain a map that contains at least one key that matches all the specified query criteria. |

- `$and`操作符

  下面是使用的两个字段，当这个selector数组中都匹配了，才匹配

  ```json
  {
    "selector": {
      "$and": [
        {
          "$title": "Total Recall"
        },
        {
          "year": {
            "$in": [1984, 1991]
          }
        }
      ]
    },
    "fields": [
      "year",
      "title",
      "cast"
    ]
  }
  ```

  下面是使用的`_all_docs`作为主键的例子

  ```json
  {
      "$and": [
          {
              "_id": { "$gt": null }
          },
          {
              "year": {
                  "$in": [2014, 2015]
              }
          }
      ]
  }
  ```

- `$or`操作符

  selector中只要有一个匹配即可

  ```json
  {
      "year": 1977,
      "$or": [
          { "director": "George Lucas" },
          { "director": "Steven Spielberg" }
      ]
  }
  ```

- `$not`操作符

  selector不匹配才行(下面的就要求year不能等于1901)

  ```json
  {
      "year": {
          "$gte": 1900
      },
      "year": {
          "$lte": 1903
      },
      "$not": {
          "year": 1901
      }
  }
  ```

- `$nor`操作符

  selector数组中的所有的都不匹配才行（下面就要求year不能等于1901，1905，1907）

  ```json
  {
      "year": {
          "$gte": 1900
      },
      "year": {
          "$lte": 1910
      },
      "$nor": [
          { "year": 1901 },
          { "year": 1905 },
          {  "year": 1907 }
      ]
  }
  ```

- `$all`操作符

  它匹配的是一个数组元素，要求其中需要包含所有指定的元素才匹配

  下面的例子使用主索引`_all_docs`，进行全局扫描

  ```json
      "_id": {
          "$gt": null
      },
      "genre": {
          "$all": ["Comedy","Short"]
      }
  }
  ```

- `$elemMatch`操作符

  匹配一个数组字段中至少一个元素即可

  ```json
  {
      "_id": { "$gt": null },
      "genre": {
          "$elemMatch": {
              "$eq": "Horror"
          }
      }
  }
  ```

- `allMatch`操作符

  匹配一个数组字段中所有的元素

  ```json
  {
      "_id": { "$gt": null },
      "genre": {
          "$allMatch": {
              "$eq": "Horror"
          }
      }
  }
  ```

- `$keyMapMatch`

  匹配map中的一个键即可

  ```json
  {
      "_id": { "$gt": null },
      "cameras": {
          "$keyMapMatch": {
              "$eq": "secondary"
          }
      }
  }
  ```

  

### Condition Operators（条件运算符）

> 条件运算符通常是指定一个字段，用来评估一个字段是否符合预期的参数，例如`$eq`操作符用来匹配所有和给定参数相等的字段

除了基本的等于(`$eq`)和不等于(`$ne`)操作符之外，还有一些`元操作符`(他们可以接受不标准的json内容作为参数)，其他的条件运算符就必须要有指定的json格式了

| Operator type | Operator  | Argument             | Purpose                                                      |
| ------------- | --------- | -------------------- | ------------------------------------------------------------ |
| (In)equality  | `$lt`     | Any JSON             | The field is less than the argument                          |
|               | `$lte`    | Any JSON             | The field is less than or equal to the argument.             |
|               | `$eq`     | Any JSON             | The field is equal to the argument                           |
|               | `$ne`     | Any JSON             | The field is not equal to the argument.                      |
|               | `$gte`    | Any JSON             | The field is greater than or equal to the argument.          |
|               | `$gt`     | Any JSON             | The field is greater than the to the argument.               |
| Object        | `$exists` | Boolean              | Check whether the field exists or not, regardless of its value. |
|               | `$type`   | String               | Check the document field’s type. Valid values are `"null"`, `"boolean"`, `"number"`, `"string"`, `"array"`, and `"object"`. |
| Array         | `$in`     | Array of JSON values | The document field must exist in the list provided.          |
|               | `$nin`    | Array of JSON values | The document field not must exist in the list provided.      |
|               | `$size`   | Integer              | Special condition to match the length of an array field in a document. Non-array fields cannot match this condition. |
| Miscellaneous | `$mod`    | [Divisor, Remainder] | Divisor and Remainder are both positive or negative integers. Non-integer values result in a 404. Matches documents where `field % Divisor == Remainder` is true, and only when the document field is an integer. |
|               | `$regex`  | String               | A regular expression pattern to match against the document field. Only matches when the field is a string value and matches the supplied regular expression. The matching algorithms are based on the Perl Compatible Regular Expression (PCRE) library. For more information about what is implemented, see the see the [Erlang Regular Expression](http://erlang.org/doc/man/re.html) |

> 正则表达式不适用于索引，它们可以用来过滤大数据集，但是它们可以用来限制 [partial index](https://docs.couchdb.org/en/stable/api/database/find.html#find-partial-indexes)

通常`$eq`, `$gt`, `$gte`, `$lt`, and `$lte`这些操作符可以用于基础的查询，至少应该在一个selector中包含其中一个

下面是希望找到`afieldname`字段中以A开头的所有文档，这就会报错，因为没有索引，数据库会对主索引进行全局扫描

```
{
    "selector": {
        "afieldname": {"$regex": "^A"}
    }
}
```

> 所以在生产环境中建议使用正确的索引



## Sort Syntax（排序语法）

排序字段通常包含一个键值对的列表(list)，通过一个数组来表述，第一个键值优先级最高，第二个次之

该字段可以是任何字段，如果子文档字段需要的话，可以使用点分符号。

通常由两个导向值，一个是升序（`$asc`），一个是降序（`desc`）,如果不指定，默认就是升序（`asc`）

下面就是通过两个字段进行排序(降序)

```json
[{"fieldName1": "desc"}, {"fieldName2": "desc" }]
```

下面就是默认的导向序列（升序）

```json
[{"fieldName1","fieldName2"}]
```

> 通常是根据一个字段的内容查找到结果集，然后根据某个特定的字段对结果集进行排序

#### 如果使用排序，需要确保：

- selector中至少由一个排序字段
- **已经定义了索引（其中的所有的排序字段都是相同的排序顺序）**
- 排序数组中的每一个对象都有一个键

> 如果排序数组中的对象没有键，则生成的排序顺序是随机的，是变化的
>
> “查找”不支持具有不同排序顺序的多个字段，因此方向必须全部为升序或全部为降序。

对于文本搜索排序中的字段名称，有时需要指定字段类型，例如：

```json
 { "<fieldname>:string": "asc"}
```

如果可能，尝试根据选择器发现字段类型。在不明确的情况下，必须明确提供字段类型。当字段包含不同的数据类型时，排序顺序是不确定的。

下面是一个排序的例子

```json
{
    "selector": {"Actor_name": "Robert De Niro"},
    "sort": [{"Actor_name": "asc"}, {"Movie_runtime": "asc"}]
}
```

---

到这就告一段落了。