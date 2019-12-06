## 背景介绍

GraphQL，由Facebook开源，并在github api v4中得到了使用，替代了v3中restful的地位。既然大厂使用，那就得去看看到底是个什么货色。

GraphQL相对于传统的restful接口，主要是在灵活性上有较大改进。例如，要增加一个属性返回，之前的做法可能需要更改接口的返回对象；如果属性越加越多，可能衍生出多个接口（包含返回少量的和大量的数据），也可能由此诞生一个巨大的接口（包含所有的数据）；如果是多个对象的组合，由于组合的形式多种多样，有可能也会造成多个接口的产生。

而解决此缺点的做法是，客户端根据自己的需求来传递请求参数，参数中指明需要哪些数据，则服务器端根据这些参数返回给客户端所需要的数据。这样，关于数据的特定字段、或者组合、聚合类的需求，都无需服务端关心，客户端自己确定就好。

## 查询 Queries

### 字段 Fields

客户端通过指定`Fields`来控制返回的数据，`Fields`即为请求的数据，类似字段的含义。指定了服务端定义好的标量，例如：

*请求*

```json
{
  hero {
    name
    # Queries can have comments!
    friends {
      name
    }
  }
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

其中`hero`, `name`, `friends`等均为`Fields`，在请求中指定什么，就返回什么。

### 参数 Arguments

在获取数据的操作中，除了指定`Fields`以外，还可以指定参数，来丰富查询能力，例如：

*请求*

```json
{
  human(id: "1000") {
    name
    height
  }
}
```

*响应*

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 1.72
    }
  }
}
```

甚至还可以在标量中传参来实现服务端统一的数据转换：

*请求*

```json
{
  human(id: "1000") {
    name
    height(unit: FOOT)
  }
}
```

*响应*

```json
{
  "data": {
    "human": {
      "name": "Luke Skywalker",
      "height": 5.6430448
    }
  }
}
```

指定`height`的`unit`为`FOOT`，则返回的数据以英尺为单位。

### 别名 Aliases

当在一次请求中，需要用不同的参数来请求同一个对象时，就可能用上别名的功能。可以使用别名来重命名字段，例如：

*请求*

```json
{
  empireHero: hero(episode: EMPIRE) {
    name
  }
  jediHero: hero(episode: JEDI) {
    name
  }
}
```

*响应*

```json
{
  "data": {
    "empireHero": {
      "name": "Luke Skywalker"
    },
    "jediHero": {
      "name": "R2-D2"
    }
  }
}
```

`empireHero`和`jediHero`都是指定的别名，在一次请求中，返回了两个`hero`对象。

###  片段 Fragments

上述别名的例子中，如果请求两个`hero`的字段更多，并且字段都类似，则需要写两段比较重复的请求字段。片段的作用是用来预定义一个请求格式，类似`myBatis`中的`resultMap`。定义好一个片段后，就可以在请求中直接引用，例如：

*请求*

```json
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}
```

*响应*

```json
{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

我们定义了一个`comparisonFields`的片段，其中定义了一些字段，并在请求中引入了此片段，可以看到返回数据就是此片段的格式。

### 操作名 Operation name

针对各种各样的请求，可以指定一个操作名来减少歧义。操作名包括类型和名称，例如：

*请求*

```json
query HeroNameAndFriends {
  hero {
    name
    friends {
      name
    }
  }
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

其中`query`即为操作类型，还可以为`mutation`或者`subscription`。`HeroNameAndFriends`为操作名称。比较推荐使用操作名，因为在调试问题时比较便于定位是哪一个请求。

### 变量 Variables

在请求中，查询条件等大都是随着具体业务对象变化的，不是写死的，所以支持变量传入的查询也是最基本的功能。使用变量需如按如下三个步骤：

1. 使用`$variableName`替换请求中的固定值。即在请求中定义变量。
2. 在查询中声明变量`$variableName`，类似函数中的变量声明。
3. 在请求时以` variableName: value `的形式传入变量。

*请求*

```json
query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

*参数*

```json
{
  "episode": "JEDI"
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```

还可以指定变量的默认值：

```json
query HeroNameAndFriends($episode: Episode = JEDI) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

### 指令 Directives

动态的请求参数问题，我们可以使用上述的变量来解决，但请求的字段或者结构，同样面临动态变化的需求，这种场景我们就需要用指令来解决。例如某页面可能同时包括缩略信息和全量信息，例如：

*请求*

```json
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```

*变量*

```json
{
  "episode": "JEDI",
  "withFriends": false
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    }
  }
}
```

如果传入的`$withFriends`变量为`true`，返回的结果则会包含friends对象。现在核心GraphQL包含两个指令：

* `@include(if: Boolean)`包含指令定义的部分
* `@skip(if: Boolean)`排除指令定义的部分

## 变更 Mutations

在了解的查询是如何进行之后，下一步就是插入数据和更新数据，我们统称为数据变更，即`mutation`。与查询的`query`类似，只是将关键字换为了`mutation`。例如下面是一个插入数据的例子：

*请求*

```json
mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

*变量*

```json
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

*响应*

```json
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

请求后，会按照请求的字段，返回变更之后的数据。

与查询类似，一次变更也可以传递多个对象，在GraphQL中，查询多个对象是并行的，但变更对个对象则是串行的。

## 串联片段





## 配置

首先需要引入所需的依赖包

```xml
<dependency>
    <groupId>com.graphql-java</groupId>
    <artifactId>graphql-spring-boot-starter</artifactId>
    <version>5.0.2</version>
</dependency>
```

