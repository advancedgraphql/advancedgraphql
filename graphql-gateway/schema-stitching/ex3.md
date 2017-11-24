
#### Example 3: Merging schemas with naming conflicts for types

<a href="https://github.com/advancedgraphql/schema-stitching/tree/master/merge-schemas-3" target="_blank"><img src="https://imgur.com/sj8HlZO.png" width="200" /></a>

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
    text: () => 'Hello'
  }
}

const schemaA = makeExecutableSchema({ typeDefs, resolvers })
```

**Schema `B`**:

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

const schemaB = makeExecutableSchema({ typeDefs, resolvers })
```

Both schemas are identical and thus have a conflict for the `Greeting` type. To resolve this situation, there are three options.

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
      greeting: () => ({text: null})
    },
    Greeting: {
      text: root => 'Hey'
    }
  })
})
```

##### Option 3: Use `onTypeConflict`

The last option is to use the `onTypeConflict` argument to determine which type should be used:

```js
const { mergeSchemas } = require('graphql-tools')
const { schemaA, schemaB } = require('./schemas')

const mergedSchema = mergeSchemas({
  schemas: [schemaA, schemaB],
  onTypeConflict: (left, right) => right // default: left
})
```