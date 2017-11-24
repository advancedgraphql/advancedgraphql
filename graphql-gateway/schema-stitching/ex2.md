
#### Example 2: Merging schemas with naming conflicts on their root fields

<a href="https://github.com/advancedgraphql/schema-stitching/tree/master/merge-schemas-2" target="_blank"><img src="https://imgur.com/sj8HlZO.png" width="200" /></a>

Consider the following two schemas:

**Schema `A`**:

```js
const typeDefs = `
  type Query {
    greeting: String
  }`

const resolvers = {
  Query: {
    greeting: () => 'Hello'
  }
}
const schemaA = makeExecutableSchema({ typeDefs, resolvers })
```

**Schema `B`**:

```js
const typeDefs = `
  type Query {
    greeting: String
  }`

const resolvers = {
  Query: {
    greeting: () => 'A fine day, good sir!'
  }
}
const schemaB = makeExecutableSchema({ typeDefs, resolvers })
```

The problem here is that there are two different implementations of the `greeting` root field. After having merged the two schemas, which one should be taken when the server receives a corresponding query: `query { greeting }`. There are two ways of solving this issue.

##### Option 1: Not explicitly resolve the conflict (use `mergeSchema`'s default)

When not providing any additional information to `mergeSchemas`, it will by default pick the implementation of the _last_ schema that was passed to it:

```js
const { mergeSchemas } = require('graphql-tools')
const { schemaA, schemaB } = require('./schemas')

const mergedSchema = mergeSchemas({
  schemas: [schemaA, schemaB]
})
```

In this case `mergedSchema` will adopt the behaviour from `schemaB` as it was passed last into `mergeSchemas`.

##### Option 2: Write custom resolver

Whenevery `mergeSchemas` is used, you can provide a `resolvers` object containing explicit information about how the available fields are to be resolved (just like with `makeExecutableSchema`):

```js
const { mergeSchemas } = require('graphql-tools')
const { schemaA, schemaB } = require('./schemas')

const mergedSchema = mergeSchemas({
  schemas: [schemaA, schemaB],
  resolvers: mergeInfo => ({
    Query: {
      greeting: () => {
        return 'Greetings, Friend'
      }
    }
  })
})
```

When implementing the new resolver for `greeting`, you can specify the string you want to return for it. Notice that the current API does not allow to simply use the `greeting` resolver of `schemaA` (other than switching the order in the `schemas` array).