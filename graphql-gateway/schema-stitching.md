# Schema Stitching

The term _schema stitching_ refers to the act of combining (at least) two GraphQL schemas into one.

There are several use cases and different scenarios for how exactly schema stitching might take place. 

### Merging schemas

#### Merging schemas without naming conflicts

In its simplest form, schema stitching might simply mean to _merge_ two schemas that don't have any naming conflicts.

Consider the following two schemas, `A` and `B`:

**Schema `A`**:

```graphql
type Query {
  hello: String
}
``` 

**Schema `B`**:

```graphql
type Query {
  goodbye: String
}

type Mutation {
  launchMissiles: Boolean
}
```

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

This case is very simple, while merging we can simply create the _union_ of the two schemas.

[`graphql-tools`](https://github.com/apollographql/graphql-tools) provides an API for merging schemas: [`mergeSchemas`](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html#mergeSchemas).

#### Merging schemas with naming conflicts

When the schemas you're merging have conflicting names for their types, you need to provide more information for how these naming conflict should be resolved. 

> Another (experimental) idea for dealing with these kinds of issues are _namespaces_ for GraphQL schemas.

When using the above mentioned `mergeSchemas` function, you can resolve naming conflicts by providing the `onTypeConflict` argument.

### Complement a schema with missing types

Another use case for schema stitching is to simply _complement_ an existing schema with types that are currently missing from it. For example, consider this simple schema:

```graphql
type Query {
  myRepos: [Repository!]!
}
```

Obviously, this schema definition is incomplete: the `Repository` type is missing. Now, GitHub's public GraphQL API does have a `Repository` type, so one thing you can do now is grab this type from the GitHub API and include it in your schema!

This is precisely what [`graphql-remote`](https://github.com/graphcool/graphql-remote) allows you to do with `collectTypeDefs`.