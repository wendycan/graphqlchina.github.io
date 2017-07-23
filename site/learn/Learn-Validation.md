---
title: 校验（Validation）
layout: ../_core/DocsLayout
category: Learn
permalink: /learn/validation/
next: /learn/execution/
---

通过使用类型系统，一个 GraphQL 查询可以预先判断是否合法。
这样在创建了一个不合法的查询时服务端和客户端都可以有效的通知开发者，而不需要依赖运行时校验。

在我们的星球大战示例中，
文件 [starWarsValidation-test.js](https://github.com/graphql/graphql-js/blob/master/src/__tests__/starWarsValidation-test.js)
包含了多种无效查询的演示，可以用来测试相关实现的校验器。

首先，让我们看一个复杂的合法查询。这是一个和上一节例子相似的嵌套查询，但将重复的字段提取到了一个片段中：

```graphql
# { "graphiql": true }
{
  hero {
    ...NameAndAppearances
    friends {
      ...NameAndAppearances
      friends {
        ...NameAndAppearances
      }
    }
  }
}

fragment NameAndAppearances on Character {
  name
  appearsIn
}
```

可以看到这个查询是合法的。让我们再看看一些不合法的查询...

一个片段不能引用它自己或创建一个循环，否则可能会产生一个无穷的结果！这是一个和上面一样，
但没有显式指定三层嵌套层级的查询：

```graphql
# { "graphiql": true }
{
  hero {
    ...NameAndAppearancesAndFriends
  }
}

fragment NameAndAppearancesAndFriends on Character {
  name
  appearsIn
  friends {
    ...NameAndAppearancesAndFriends
  }
}
```

当我们查询字段时，该字段必须在所给的类型中存在。因此当 `hero` 返回一个 `Character` 时，
我们查询的必须是 `Character` 中含有的字段。这个类型并没有 `favoriteSpaceship` 字段，
所以这个查询是不合法的：

```graphql
# { "graphiql": true }
# 不合法：Charcter 中不存在 favoriteSpaceship
{
  hero {
    favoriteSpaceship
  }
}
```

只要我们查询的某个字段的返回结果不是一个直接量或枚举，我们就需要指明这个字段中有哪些是我
们需要的。`Hero` 返回一个 `Character`，我们需要请求其中 `name` 和 `appersIn` 之类的
字段。如果我们遗漏了，这个查询就不合法：

```graphql
# { "graphiql": true }
# 不合法: hero 不是一个直接量，因此需要指明其中的附加字段
{
  hero
}
```

类似的，如果字段是一个直接量，再去查询其附加的字段就说不通了，这样做也会让查询不合法：

```graphql
# { "graphiql": true }
# 不合法: name 是一个直接量，因此附加字段是不允许的
{
  hero {
    name {
      firstCharacterOfName
    }
  }
}
```

前面我们提到过只能查询类型中拥有字段的问题。当我们查询返回一个 `Character` 的 `hero` 时，
我们只能查询 Character 中含有的字段。那么当我们想查询 R2-D2 的主要功能时会怎样呢？

```graphql
# { "graphiql": true }
# 不合法: Character 中不存在 primaryFunction
{
  hero {
    name
    primaryFunction
  }
}
```

这个查询是不合法的，因为 `primaryFunction` 不是 `Character` 的一个字段。我们需要一种方法去
表明当这个 `Character` 是一个 `Droid` 的时候则需要请求 `primaryFunction`，反之则忽略它。
我们可以使用之前介绍的片段做到这一点。通过设置一个定义在 `Droid` 上的片段并包含它，我们可以
确保只有在 `primaryFunction` 被定义了的地方才会查询到。

```graphql
# { "graphiql": true }
{
  hero {
    name
    ...DroidFields
  }
}

fragment DroidFields on Droid {
  primaryFunction
}
```

查询合法了，但是有些累赘。命名片段的价值体现在可以使用多次上，但这里我们只需要使用它一次。
这时可以使用一个内联的片段替代它。这样依然允许我们指定查询依赖的特定类型，但不需要额外定义
一个单独的片段了：

```graphql
# { "graphiql": true }
{
  hero {
    name
    ... on Droid {
      primaryFunction
    }
  }
}
```

这些只是校验系统的牛毛一角，有大量的校验规则去确保一个 GraphQL 查询在语义上是有意义的。规范
在『校验』章节中深入的阐述了更多细节，同时在 GraphQL.js 项目的
[validation](https://github.com/graphql/graphql-js/blob/master/src/validation)
目录下包含了一个符合规范的 GraphQL 校验器实现。
