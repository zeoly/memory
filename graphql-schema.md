#  模式 Schema

在了解了GraphQL是如何查询以及如何变更数据后，我们需要进一步了解一下。对于客户端来说，需要在请求数据前知道数据的格式，包括有哪些`field`，这些字段的子对象又有哪些，子对象又是什么样的数据结构。这样就出现了`schema`的概念。

## 对象类型与字段

在GraphQL的schema中，构成的基本元素就是对象`object`类型`type`，即指定你想要获取的对象，以及想要获取的对象的字段，例如：

```json
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- `Character`定义了一个为GraphQL对象类型
- `name`和`appearsIn`为`Character`类型中的字段
- `String`是一个预定义的标量类型
- `String!`表示此字段不可为空
- `[Episode!]!`表示返回的是一个`Episode`对象列表，并且也是非空的

## 参数

每个GraphQL对象的字段都可以有若干个参数，例如下面例子中的`length`字段：

```json
type Starship {
  id: ID!
  name: String!
  length(unit: LengthUnit = METER): Float
}
```

参数都需要预先定义好，例如上面例子中，`unit`就是`length`的一个参数。参数可以是必需也可以是可选，当一个参数是可选时，可以定义默认值。如果不给`unit`传递参数，那么`METER`即为默认值。

## 查询与变更类型

schema中大多数类型都是对象类型，除此之外，还有两种重要类型：查询`query`和变更`mutation`：

```json
schema {
  query: Query
  mutation: Mutation
}
```

GraphQL通常都有一个`query`类型，但不一定有`mutation`类型，格式也和普通的对象类型差不多，区别就是这两个类型为每个GraphQL请求定义了接入点。如果有一个请求类似如下格式：

*请求*

```json
query {
  hero {
    name
  }
  droid(id: "2000") {
    name
  }
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2"
    },
    "droid": {
      "name": "C-3PO"
    }
  }
}
```

那么意味着GraphQL服务有一个`query`类型，其中包括了`hero`和`droid`字段：

```json
type Query {
  hero(episode: Episode): Chacter
  droid(id: ID!): Droid
}
```

变更`mutaion`也是类似的，同样在`Mutation`的`type`中定义字段，然后就可以在请求中对应的调用。

## 标量类型

定义的对象类型，大都是组合的结构数据，但最终落到具体的字段上，都会有具体的类型，这些基础的类型，叫做标量`scalar`。例如下面例子中的`name`和`appearsIn`字段被解析为标量类型：

*请求*

```json
{
  hero {
    name
    appearsIn
  }
}
```

*响应*

```json
{
  "data": {
    "hero": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ]
    }
  }
}
```

GraphQL中的默认标量类型如下：

- `Int`：有符号的32位整型
- `Float`：有符号的双精度浮点类型
- `String`：UTF-8格式字符串
- `Boolean`：布尔型
- `ID`：表示唯一id，通常用作主键

但我们通常还会定义一些自定义标量类型，例如`Date`类型：

```json
scalar Date
```

然后需要我们自己实现自定义标量类型的序列化、反序列化、验证等等。例如`Date`类型可以序列化为一个整型时间戳，并且调用服务的客户端需要知道日期字段序列化的格式。

## 枚举类型

枚举类型在schema中的定义如下：

```json
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

## 列表与非空

这两个是属于类型修饰符，在类型的后面加一个感叹号表示非空，如果服务器获取到的值确实为空，则会抛出GraphQL执行错误，这样可以通知到客户端。另外还可以在参数中使用非空修饰符，例如：

*查询*

```json
query DroidById($id: ID!) {
  droid(id: $id) {
    name
  }
}
```

*变量*

```json
{
  "id": null
}
```

*响应*

```json
{
  "errors": [
    {
      "message": "Variable \"$id\" of required type \"ID!\" was not provided.",
      "locations": [
        {
          "line": 1,
          "column": 17
        }
      ]
    }
  ]
}
```

列表的定义是使用方括号将对应的类型括起来，同样也可以在参数中使用。列表与非空还可以合在一起使用，例如定义一个非空字符串的列表：

```json
myField: [String!]
```

需要注意这样定义是表示列表本身可以为空，但其中的元素不能为空，例如：

```json
myField: null // 合法
myField: [] // 合法
myField: ['a', 'b'] // 合法
myField: ['a', null, 'b'] // 非法
```

下面是定义一个字符串非空列表：

```json
myField: [String]!
```

这样表示列表本身不可为空，但可以包含空元素：

```json
myField: null // 非法
myField: [] // 合法
myField: ['a', 'b'] // 合法
myField: ['a', null, 'b'] // 合法
```

## 接口

在GraphQL中的接口与实现，其实有点类似继承，例如定义一个接口：

```json
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
}
```

然后其他的类型来实现这个接口，需要包含接口中所有的字段、参数、返回类型，例如：

```json
type Human implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  friends: [Character]
  appearsIn: [Episode]!
  primaryFunction: String
}
```

如果请求了接口对象中没有的字段，会报错，例如`Character`对象类型中没有`primaryFunction`字段：

*请求*

```json
query HeroForEpisode($ep: Episode!) {
  hero(episode: $ep) {
    name
    primaryFunction
  }
}
```

*变量*

```json
{
  "ep": "JEDI"
}
```

*响应*

```json
{
  "errors": [
    {
      "message": "Cannot query field \"primaryFunction\" on type \"Character\". Did you mean to use an inline fragment on \"Droid\"?",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ]
    }
  ]
}
```

## 联合类型

联合`union`类型，即为若干个对象的组合，为`或`关系，即只能为其中之一。例如：

```json
union SearchResult = Human | Droid | Starship
```

例如下例中的模糊搜索，如果是返回联合类型`SearchResult`，可以使用条件片段来获取其他的字段：

*请求*

```json
{
  search(text: "an") {
    __typename
    ... on Human {
      name
      height
    }
    ... on Droid {
      name
      primaryFunction
    }
    ... on Starship {
      name
      length
    }
  }
}
```

*响应*

```json
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
```

## 输入类型

在传递参数时，也可以传递对象，例如更新数据的场景，一般会传入整个需要更新的对象数据。定义输入类型以`type`为关键字：

```json
input ReviewInput {
  stars: Int!
  commentary: String
}
```

然后就可以将对象传入某个变更的参数中，例如：

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

