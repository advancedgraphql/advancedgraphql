# Scenarios

There are multiple scenarios on how to define & compose GraphQL schemas:

## 1. Local GraphQL Schema

Create a new local schema

```ts
import { makeExecutableSchema } from 'graphql-tools'
import { GraphQLServer } from 'graphql-yoga'

const typeDefs = `
type Query {
  hello(name: String): String!
}
`

const resolvers = {
  Query: {
    hello: (parent, { name }) => `Hello ${name || 'World'}`,
  },
}

const schema = makeExecutableSchema({ typeDefs, resolvers })

const server = new GraphQLServer({ schema })
server.start(() => console.log('Server is running on localhost:3000'))
```

## 2. Remote GraphQL Schema

Create a new local schema by picking parts from remote schema

```ts
import { RemoteSchema, collectTypeDefs, fetchTypeDefs } from 'graphql-remote'
import { makeExecutableSchema } from 'graphql-tools'
import { GraphQLServer } from 'graphql-yoga'

const makeLink = () => new HttpLink({ uri: 'https://api.github.com' })

const remoteTypeDefs = await fetchTypeDefs(makeLink())

// this gets missing type defs for `Repository`
const typeDefs = collectTypeDefs(remoteTypeDefs, `
type Query {
  myRepos: [Repository!]!
}
`

const resolvers = {
  Query: {
    myRepos: (parent, args, { delegate }, info) => {
      return delegate.query('viewer.repositories')
    },
  },
}

const schema = makeExecutableSchema({ typeDefs, resolvers })

const server = new GraphQLServer({
  schema,
  context: { delegate: new RemoteSchema(makeLink()) },
})
server.start(() => console.log('Server is running on localhost:3000'))
```


## 3. Extend Remote GraphQL Schema

Create a new local schema by picking parts from remote schema and extending it

```ts
import { RemoteSchema, collectTypeDefs, fetchTypeDefs } from 'graphql-remote'
import { makeExecutableSchema } from 'graphql-tools'
import { GraphQLServer } from 'graphql-yoga'

const makeLink = () => new HttpLink({ uri: 'https://api.github.com' })

const remoteTypeDefs = await fetchTypeDefs(makeLink())

// this gets missing type defs for `Repository`
const typeDefs = collectTypeDefs(remoteTypeDefs, `
type Query {
  myRepos: [Repository!]!
}

extend type Repository {
  trendingRank: Int
}`

const resolvers = {
  Query: {
    myRepos: (parent, args, { delegate }, info) => {
      return delegate.query('viewer.repositories')
    },
  },
  Repository: {
  trendingRank: () => {
      const rank = 42 // needs to be fetches properly
      return rank
    },
  },
}

const schema = makeExecutableSchema({ typeDefs, resolvers })

const server = new GraphQLServer({
  schema,
  context: { delegate: new RemoteSchema(makeLink()) },
})
server.start(() => console.log('Server is running on localhost:3000'))
```

## 4. Extend Local Schema with Remote Schema



## 5. Unify/Merge Schemas
