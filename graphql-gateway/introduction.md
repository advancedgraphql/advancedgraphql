# Introduction

As GraphQL matures, two major patterns for how GraphQL is used in backend systems have emerged:

- GraphQL as an **API gateway** for legacy systems (e.g. REST APIs)
- **GraphQL Native** applications designed from the ground up to use GraphQL

> Read more about this distinction [here](https://blog.graph.cool/graphql-api-gateway-graphql-native-1e46e4f179f7).

In fact, even when building GraphQL Native apps, usage of a GraphQL gateway can be beneficial for organizing and structuring your application:

![](https://imgur.com/MROWuhV.png)

In the diagram, you see a GraphQL API gateway sitting in front of two GraphQL Native services, as well as one legacy system. 

The benefit of this architecture is that client applications will be able to talk to a single, coherent GraphQL API instead of having to deal with multiple endpoints and fetching data from several sources. The gateway unifies these APIs and hides the complexity of the data fetching process.

In the following chapters, you'll learn about specific patterns, best practices and available tooling for building GraphQL gateways.

## The GraphQL Schema

The core component of every GraphQL server is the [_schema_](https://blog.graph.cool/graphql-server-basics-the-schema-ac5e2950214e). It defines the API of the server by exposing certain operations to client applications.

GraphQL schemas can be expressed in the [Schema Definition Language](https://blog.graph.cool/graphql-sdl-schema-definition-language-6755bcb9ce51) (SDL). Here is a simple example of a schema that allows for a simple `hello` query:

```graphql
type Query {
  hello: String!
}
```

A GraphQL server implementing this schema will accept only a single type of query:

```graphql
query {
  hello
}
```

So far we only defined the capabilities of the API in an abstract fashion (using the SDL). We don't know yet what the response of the server actually looks like, we only know it will return a string.

This is where we can introduce the difference between _structure_ and _behaviour_ in GraphQL:

- the _structure_ of an API is defined by the schema
- the _behaviour_ is implemented by so-called _resolver_ functions

To complete our setup and enable [execution](http://facebook.github.io/graphql/October2016/#sec-Execution) of the above query, we now need to make sure the resolver for the `hello` field on the `Query` type is implemented.

[GraphQL.js](http://graphql.org/graphql-js/) provides a _GraphQL execution engine_, a tool allowing you to execute queries and mutations against a schema if resolvers are available. Here is a simple example:

```js
import { 
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString,
} from 'graphql'

const schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'Query',
    fields: {
      hello: {
        type: GraphQLString,
        resolve: () => {
          return 'Hello World'
        }
      }
    }
  })
})
```

Notice that the definition of this [`GraphQLSchema`](http://graphql.org/graphql-js/type/#graphqlschema) instance is equivalent to what we expressed above in GraphQL SDL, except that now (because we're writing it in JavaScript) we can attach the `resolve` function which will get executed when the server receives the `hello` query to return the expected information.

Using the [`graphql`](http://graphql.org/graphql-js/graphql/#graphql) function provided by GraphQL.js, you can execute a query against this schema:

```js
import { graphql, parse } from 'graphql'

const query = parse(`{ hello }`)
graphql(schema, query).then(result => console.log(JSON.stringify(result)))
// Prints: { "data": { "hello": "Hello World" } }
```
