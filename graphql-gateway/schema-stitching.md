# Schema Stitching

The term _schema stitching_ refers to the act of combining (at least) two GraphQL schemas into a single one. There are several use cases and different scenarios for how exactly schema stitching might take place. In the following, a number of different scenarios are described and how they can be solved with the available tooling.

## Merging schemas

Merging two (or more) schemas means forming the _union_ of their respective types. If there are naming conflicts between types of the merged schemas, additional information needs to be provided to determine how these conflicts should be resolved. [`graphql-tools`](https://github.com/apollographql/graphql-tools) provides an API for merging schemas: [`mergeSchemas`](https://www.apollographql.com/docs/graphql-tools/schema-stitching.html#mergeSchemas).

There are four different examples explaining different use cases for schema merging:

- [Merging schemas without naming conflicts](./schema-stitching/ex1.md)
- [Merging schemas with naming conflicts on their root fields](./schema-stitching/ex2.md)
- [Merging schemas with naming conflicts for types](./schema-stitching/ex3.md)
- [Merging an executable with a non-executable schema](./schema-stitching/ex4.md)

## Picking parts of a schema

Another use case for schema stitching is when only certain parts of a schema are selectively added to another schema.

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