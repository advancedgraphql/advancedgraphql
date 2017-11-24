# Schema Transformation

_Transforming_ a given [`GraphQLSchema`](http://graphql.org/graphql-js/type/#graphqlschema) can include the following activities:

- Hiding certain operations from the schema
- Hooking into the schema's resolver functions to...
  - ... overwrite arguments
  - ... overwrite resolved data

[`graphql-transform-schema`](https://github.com/graphcool/graphql-transform-schema) is a library that let's you perform these operations. The remainder of this chapter is about its API.

## Hiding operations

Consider the following schema to start with. It exposes CRUD operations for a `User` type:

```graphql
type Query {
  user(id: ID!): User!
  allUsers: [User!]
}

type Mutation {
  createUser(name: String!): User
  updateUser(id: ID!, name: String!): User
  deleteUser(id: ID!): User
}

type User {
  id: ID!
  name: String!
}
```

Assume there is now a requirement to _hide_ the `allUsers` query as well as the `deleteUser` mutation, i.e. not expose it through the API any more. `graphql-transform-schema` offers the following way to implement this scenario:

```js
const schema = ... // this is the GraphQLSchema instance implementing the SDL from above
const transformedSchema = transformSchema(schema, {
  Query: {
    'allUsers': false   // hide `allUsers` field on `Query` type
  },
  Mutation: {
    'deleteUser': false // hide `deleteUser` field on `Mutation` type
  }
})
```

As you can see, you can call the `transformSchema` function and pass in the original schema along with a set of _rules_. Each rule refers to a field on one of the GraphQL types defined in the schema and specifies whether it should be _visible_ or not by providing a boolean value.

> Note that it is currently not possible to [provide a function which _returns_ a boolean value](https://github.com/graphcool/graphql-transform-schema/issues/9) instead of the boolean value directly.

The resulting `transformedSchema` now doesn't expose the `allUsers` query and `deleteUser` mutation any more. Written in the SDL, the schema would now look as follows:

```graphql
type Query {
  user(id: ID!): User!
}

type Mutation {
  createUser(name: String!): User
  updateUser(id: ID!, name: String!): User
}

type User {
  id: ID!
  name: String!
}
```

Note that `transformSchema` also supports rules that are using a _wildcard_ to refer to certain a group of fields. For example, if you have multiple mutation fields beginning with `delete` (e.g. `deleteUser` and `deleteArticle`), you could hide all of these mutations as follows:

```js
const transformedSchema = transformSchema(schema, {
  Mutation: {
    'delete*': false // hide all mutations beginning with `delete`
  }
})
```

## Hooking into resolver functions

Instead of simply setting a boolean value on a field in the _rules_ that are passed to `transformSchema`, it's also possible to provide a _function_ that hooks into the field's actual resolver.

This allows for two main features:

- Overwrite the arguments for a specific field when its resolver is invoked
- Overwrite the result after the resolver for a specific field was invoked

Consider the following schema exposing only a single query called `hello`:

```graphql
type Query {
  hello(name: String!): String!
}
```

Here's the corresponding resolver:

```js
{
  Query: {
    hello: (_, args) => {
      return `Hello ${args.name}`
    }
  }
}
```

Now, using `transformSchema` you can either modify the `name` argument that's passed into the resolver function, or the resolver's return value (or both).

### Modifying resolver arguments

Say you want to modify the `name` argument and always capitalize the whole word. Here is how this requirement can be implemented:

```js
const schema = ... // this is the GraphQLSchema instance implementing the SDL from above
const transformedSchema = transformSchema(schema, {
  Query: {
    'hello': ({args, resolve}) => {
      const uppercaseName = args.name.toUpperCase()
      return resolve({name: uppercaseName})
    }
  }
})
```

Instead of passing a rule that specifies whether or not to _hide_ the `hello` field, this time a function is used to hook into the resolver function of `hello`. The function receives as input parameters the `args` provided for the query (in this case the optional `name` specified in the schema above) and the actual `resolve` function. That function is now responsible to actually call the resolver - but before doing so it's possible to modify the input arguments, which is precisely what's happening here.

Assume the following query is executed against the schema:

```graphql
query {
  hello(name: "Alice")
}
```

This is what the response will look like:

```json
{
  "data": {
    "hello": "Hello ALICE"
  }
}
```

### Modifying resolver return values

`transformSchema` also allows to modify the returned data using the same API. Assuming you want to capitalize the whole string that's returned by the resolver, here is the implementation:

```js
const schema = ... // this is the GraphQLSchema instance implementing the SDL from above
const transformedSchema = transformSchema(schema, {
  Query: {
    'hello': ({args, resolve}) => {
      return resolve(args).toUpperCase()
    }
  }
})
```

This example is similar to the previous one, except that this time it's not the `name` argument that's transformed to uppercase upfront, but the whole return value instead.

Running the same query from above:

```graphql
query {
  hello(name: "Alice")
}
```

This is going to yield the following response this time:

```json
{
  "data": {
    "hello": "HELLO ALICE"
  }
}
```