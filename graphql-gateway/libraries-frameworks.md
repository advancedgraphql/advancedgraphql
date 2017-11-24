# Libraries & Frameworks

There are several tools that help you in building a GraphQL gateway server. In this section, an overview of the avaialble libraries and their specific purpose is provided.

## graphql-yoga

### Overview

[`graphql-yoga`](https://github.com/graphcool/graphql-yoga/) is a simple GraphQL server based on various other libraries like [express](https://github.com/expressjs/express), [`graphql-tools`](https://github.com/apollographql/graphql-tools), [`apollo-server`](https://github.com/apollographql/apollo-server), [`graphql-subscriptions`](https://github.com/apollographql/graphql-subscriptions) and more.

Its main goal is to provide a convenience API for creating GraphQL servers by choosing sensible defaults for the underlying configuration.

The main features of `graphql-yoga` are:

- Miminal and easy setup (no other dependencies required)
- Built-in subscription support for realtime functionality
- Integrates by default with [graphql-playground](https://github.com/graphcool/graphql-playground)
- Out-of-the-box support for [Apollo Tracing](https://github.com/apollographql/apollo-tracing)

This is what a simple setup looks like:

```js
import { GraphQLServer } from 'graphql-yoga'
// ... or using `require()`
// const { GraphQLServer } = require('graphql-yoga')

const typeDefs = `
  type Query {
    hello(name: String): String!
  }
`

const resolvers = {
  Query: {
    hello: (_, { name }) => `Hello ${name || 'World'}`,
  },
}

const server = new GraphQLServer({ typeDefs, resolvers })
server.start(() => console.log('Server is running on localhost:3000'))
```

> You can find a number of [examples](https://github.com/graphcool/graphql-yoga/tree/master/examples) in the GitHub repository.

## graphql-tools

[`graphql-tools`](https://github.com/apollographql/graphql-tools) is a library offering a variety of functionality around building GraphQL servers. It has three main features:

- Generate a [`GraphQLSchema`](graphql.org/graphql-js/type/#graphqlschema) based on the GraphQL [SDL](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51)
- Easy [mocking](https://www.apollographql.com/docs/graphql-tools/mocking.html) of GraphQL APIs
- An API for [schema stitching](./schema-stitching.md) based on [`mergeSchemas`](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html#mergeSchemas)

## graphql-remote

[`graphql-remote`](https://github.com/graphcool/graphql-remote) is a toolbelt for creating remote GraphQL schemas with built-in subscriptions and [`DataLoader`](./batching-dataloader.md#dataloader) support. This is particularly helpful in the context of [schema stitching](./schema-stitching.md).

## Qewl

[Qewl](https://github.com/qewl/qewl) is a GraphQL Application Framework. It is inspired by [Koa](http://koajs.com/), built on [Apollo Server](https://github.com/apollographql/apollo-server), and turns your GraphQL endpoint into a Koa-like application, with support for context, middleware, and many other features.

It is great for setting up an API gateway on top of existing GraphQL endpoints, applying concepts like _remote schemas_ and _schema stitching_. But it also makes it very easy to set up a GraphQL server from scratch.

## GrAMPS

[GrAMPS](https://gramps.js.org/) (short for GraphQL Apollo Microservice Pattern Server) is a thin layer of helper tools designed for the Apollo GraphQL server that allows independent data sources — a schema, resolvers, and data access model — to be composed into a single GraphQL schema, while keeping the code within each data source isolated, independently testable, and completely decoupled from the rest of your application.

Read more in the [documentation](https://gramps-graphql.github.io/gramps-express/).