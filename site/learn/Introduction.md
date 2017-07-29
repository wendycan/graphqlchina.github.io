---
title: GraphQL 介绍
sidebarTitle: Introduction
layout: ../_core/DocsLayout
category: Learn
permalink: /learn/
next: /learn/queries/
---

> 了解 GraphQL，它如何工作，以及如何在本系列文章中使用它。寻找如何构建 GraphQL 服务的文档？有很多库可以帮助在[很多语言](/code/)中实现 GraphQL。

GraphQL 是 API 的查询语言，也是通过使用你为数据定义的类型系统来执行查询的服务器端运行时。 GraphQL 不与任何特定的数据库或存储引擎绑定，而是为现有的代码和数据提供支持。

通过定义这些类型上的类型和字段来创建 GraphQL 服务，然后为每个类型的每个字段提供函数。例如，一个 GraphQL 服务告诉我们登录用户是谁（“我”）以及该用户名的，如下所示：

```graphql
type Query {
  me: User
}

type User {
  id: ID
  name: String
}
```

每个类型、每个字段上的方法：

```js
function Query_me(request) {
  return request.auth.user;
}

function User_name(user) {
  return user.getName();
}
```

一旦 GraphQL 服务运行（通常在 Web 服务的 URL 上），就可以发送 GraphQL 查询来验证和执行。首先检查接收到的查询，以确保它仅引用定义的类型和字段，然后运行提供的函数以生成结果。

例如查询：

```graphql
{
  me {
    name
  }
}
```

可以产生 JSON 结果：

```json
{
  "me": {
    "name": "Luke Skywalker"
  }
}
```

了解更多关于 GraphQL — 查询语言，类型系统，GraphQL 服务的工作原理以及使用 GraphQL 解决常见问题的最佳做法 — 在本节中撰写的文章。
