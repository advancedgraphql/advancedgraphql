#### Example 4: Merging an _executable_ with a _non-executable_ schema

> A practical example for this scenario can be found [here](https://github.com/advancedgraphql/schema-stitching/tree/master/merge-schemas-4).

Another option when merging schemas is to take an existing _executable_ schema and add type definitions to it. Then, when merging the schemas the resolvers of the new type definitions can _delegate_ to the existing executable schema.

Consider the following two schemas:

**Schema `A`**:

```js
const typeDefs = `
  type Query {
    greeting: Greeting
  }

  type Greeting {
    text: String
  }`

const resolvers = {
  Query: {
    greeting: () => ({text: null})
  },
  Greeting: {
    text: root => 'A fine day, good sir!'
  }
}
const schemaA = makeExecutableSchema({ typeDefs, resolvers })
```

**Schema `B`** (non-executable, only type definitions in SDL):

```js
const additionalTypeDefs = `
  type Query {
    message: Message
  }

  type Message {
    text: String
  }
`
```

Now, when merging the two schemas, the `message` root field on schema `B` can be delegated to schema `A`'s root field `greeting`:

```js
const { makeExecutableSchema, mergeSchemas } = require('graphql-tools')
const { schemaA, additionalTypeDefs } = require('./schemas')

const mergedSchema = mergeSchemas({
  schemas: [schemaA, additionalTypeDefs],
  resolvers: mergeInfo => ({
    Query: {
      message: (root, args, context, info) => mergeInfo.delegate(
        'query',
        'greeting',
        args,
        context,
        info
      )
    }
  })
})
```

`mergeInfo` exposes a function called [`delegate`](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html#api). This delegate function can be used to write a resolver and _forward_ its execution to another root field in one of the merged schemas.
