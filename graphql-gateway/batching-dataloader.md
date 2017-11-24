# Batching (DataLoader)

## Overview

A GraphQL query is resolved by calling the resolver functions for the fields inside the query. For example, consider the following schema:

```graphql
type Query {
  user(id: ID!): User
}

type User {
  id: ID!
  name: String
}
```

Now assume the corresponding resolvers are implemented as follows:

```js
const resolvers = {
  Query: {
    user: (_, args) => {
      return fetchUserById(args.id) // hit the database
    }
  },
}
```

Consider the situation where a GraphQL server now receives the following query:

```graphql
query {
  user(id: "abc") {
    id
    name
  }
}
```

The server's GraphQL execution engine now needs to invoke the resolvers for the fields contained in the query: `user`, `id` and `name`. This is easy enough and there's not a lot to optimize for just yet.

Now assume the schema has another root field called `users` that allows to retrieve a list of users by their id:

```graphql
type Query {
  users(ids: [String!]!): [User!]!
}
```

Here is what the resolver implementation looks ike:

```js
const resolvers = {
  Query: {
    users: async (_, args) => {
      const { ids } = args
      const users = []
      for (const id of ids) {
        const user = await fetchUserById(id) // hit the database
        users.push(user)
      })
      return users
    }
  }
}
```

This time the performance trap is very apparent. A long list of IDs might cause the server to hit the database several times, each database access involves an actual database query and the corresponding performance penalty that comes along with it.

The optimization strategy to be applied in these cases is called _batching_: Instead of hitting the database multiple times, it's preferred to "collect" all database queries and send them as a single one. The database invokations are now _batched_.

## DataLoader

Facebook's [`DataLoader`](https://github.com/facebook/dataloader) library implements the above approach to batching by leveraging the [`process.nextTick()`](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) function provided by Node.js.

> **Note**: The `DataLoader` implements a relatively simple but very effective batching pattern. If you want to learn how it works in detail, watch this excellent video by Lee Byron: [DataLoder - Source code walktrough](https://www.youtube.com/watch?v=OQTnXNCDywA)

For every resource (e.g. users) that can be loaded in batches, one `DataLoader` can be instantiated with a _batch function_ that knows how to fetch multiple instances of this resource.

The batch function needs to fulfill two simple requirements:

1. It must takes an array of keys as input arguments. Each key identifies one instance of the resource to be loaded.
1. It must return an array of Promises. Each Promise must resolve to an instance of the corresponding resource (or reject). The length and order of the input array needs to be retained in the output array!

Considering the example from above, the batch function would provide a way to take multiple user IDs at once and send them as a single query to the database in order to load the corresponding user instances.

> [Here](https://github.com/advancedgraphql/batching/tree/master/batching-1) is a simple example demonstrating that technique.

## BatchedGraphQLClient

When resolvers don't retrieve their data from a database but rather from another GraphQL API (which is a common scenario in [schema stitching](./schema-stitching.md)), the `DataLoader` pattern can be applied as well.

To see what this looks like in practice, check out the [`BatchedGraphQLClient`](https://github.com/graphcool/batched-graphql-request/blob/master/src/index.ts#L7) where this technique is applied.