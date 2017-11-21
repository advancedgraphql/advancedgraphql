# Schema Stitching

The term _schema stitching_ refers to the act of combining (at least) two GraphQL schemas into a single one. There are several use cases and different scenarios for how exactly schema stitching might take place. In the following, a number of different scenarios are described and how they can be solved with the available tooling.

## Merging schemas

Merging two (or more) schemas means forming the _union_ of their respective types. If there are naming conflicts between types of the merged schemas, additional information needs to be provided to determine how these conflicts should be resolved. [`graphql-tools`](https://github.com/apollographql/graphql-tools) provides an API for merging schemas: [`mergeSchemas`](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html#mergeSchemas).

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

#### Example 2: Merging schemas with naming conflicts on their root fields

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

#### Example 3: Merging schemas with naming conflicts for types

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

#### Example 4: Merging an _executable_ with a _non-executable_ schema

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

<!-- 

> Another (experimental) idea for dealing with these kinds of issues are _namespaces_ for GraphQL schemas.

### Complement a schema with missing types

Another use case for schema stitching is to simply _complement_ an existing schema with types that are currently missing from it. For example, consider this simple schema:

```graphql
type Query {
  myRepos: [Repository!]!
}
```

Obviously, this schema definition is incomplete: the `Repository` type is missing. Now, GitHub's public GraphQL API does have a `Repository` type, so one thing you can do now is grab this type from the GitHub API and include it in your schema!

This is precisely what [`graphql-remote`](https://github.com/graphcool/graphql-remote) allows you to do with `collectTypeDefs`. -->