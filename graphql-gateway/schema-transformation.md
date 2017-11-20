# Schema Transformation

_Transforming_ a given [`GraphQLSchema`](http://graphql.org/graphql-js/type/#graphqlschema) can include the following activities:

- Hiding certain operations from the schema
- Hooking into the schema's resolver functions to...
  - ... overwrite arguments
  - ... overwrite resolved data

[`graphql-transform-schema`](https://github.com/graphcool/graphql-transform-schema) is a library that easily let's you perform either of these operations.

The remainder of this chapter is about the API provided by `graphql-transform-schema`.

## Hiding operations

Let's assume we have the following schema to start with. It exposes CRUD operations for a `User` type:

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

Say we wanted to hide the `allUsers` query as well as the `deleteUser` mutation. `graphql-transform-schema` offers the following API to implement this scenario:

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

As you can see, you can call the `transformSchema` function and pass in the original schema along with a set of _rules_. Each rule refers to a field on one of the GraphQL types defined in the schema and specifies whether it should be visible or not.

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

## Hooking into resolver functions

Instead of simply setting a boolean value on a field in the _rules_ that are passed to `transformSchema`, it's also possible to provide a _function_ that hooks into the field's actual resolver. This allows for two main features:

- Overwrite the arguments for a specific field when its resolver is invoked
- Overwrite the result after the resolver for a specific field was invoked 

Consider the following schema exposing only a single query called `hello`:

```graphql
type Query {
  hello(name: String): String!
}
```

Here's the corresponding resolver:

```js
{
  Query: {
    hello: (_, args) => {
      return `Hello ${args.name || 'World'}`
    }
  }
}
```

Now, using `transformSchema` you can either make modify the `name` argument that's passed into the resolver function, or the resolver's return value.

Say you want to modify the `name` argument and always capitalize the whole word. This is how you can implemented it:

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

