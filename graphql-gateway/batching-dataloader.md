# Batching (DataLoader)

A GraphQL query is resolved by calling the resolver functions for the fields inside the query. For example, consider the following schema:

```graphql
type Query {
  user: User
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

Now assume a GraphQL server receives the following query:

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
    users:  async (_, args) => {
      const { ids } = args
      const users = []
      for (id in ids) {
        const user = await fetchUserById(id) // hit the database
        users.push(user)
      }
      return users
    }
  }
}
```

This time the performance trap is very apparent. A long list of IDs might cause our server to hit the database several times, each database access involves an actual database query and the corresponding performance penalty that comes along with it.

The optimization strategy to be applied in these cases is called _batching_. Facebook's [DataLoader](https://github.com/facebook/dataloader) library solves this problem by "collecting" all resolver calls and executing them all at once. In our scenario, this would mean that all the individual database queries would be collected and finally be executed as a single one.

<!-- Now assume the API also had `Article`s with `Comment`s to ask for and allowed for this query:

```graphql
query {
  article(title: "GraphQL is great") {
    comments {
      text
      writtenBy {
        name
      }
    }
  }
}
``` 

In this query, we're asking for an article with a specific title, its comments and the name's of the users who wrote them. 

Now, it might be the case that the article has exactly five comments and all of them are written by the same user! The GraphQL server's execution engine -->
