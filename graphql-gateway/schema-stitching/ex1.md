
#### Example 1: Merging schemas without naming conflicts

In its simplest form, schema stitching might simply mean to _merge_ two schemas that don't have any naming conflicts. Consider the following two schemas, `A` and `B`:

**Schema `A`**:

```js
const typeDefs = `
  type Query {
    hello: String
  }`

const resolvers = {
  Query: {
    hello: () => 'Hello'
  }
}

const schemaA = makeExecutableSchema({ typeDefs, resolvers })
```

**Schema `B`**:

```js
const typeDefs = `
  type Query {
    goodbye: String
  }

  type Mutation {
    launchMissiles: Boolean
  }`

const resolvers = {
  Query: {
    goodbye: () => 'Goodbye'
  },
  Mutation: {
    launchMissiles: () => Math.random() >= 0.5
  }
}

const schemaB = makeExecutableSchema({ typeDefs, resolvers })
```

Here is what the merge process looks like in code, using `mergeSchemas`:

```js
const { mergeSchemas } = require('graphql-tools')
const { schemaA, schemaB } = require('./schemas')

const mergedSchema = mergeSchemas({
  schemas: [schemaA, schemaB]
})
```

> Note that `schemaA` and `schemaB` as well as the result `mergedSchema` are _executable_ schemas.

After merging the two schemas, the **result** will look as follows:

```graphql
type Query {
  hello: String   # originates from schema A
  goodbye: String # originates from schema B
}

type Mutation {
  launchMissiles: Boolean # originates from schema B
}
```

All the fields on `Query` and `Mutation` have exactly the same behaviour as in their respective original schemas.