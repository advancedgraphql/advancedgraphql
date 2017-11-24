<img src="https://imgur.com/UlE80Qv.png" width="328" />

**Advanced GraphQL** is a collection of techniques, best practices and advanced patterns for building applications and systems based on GraphQL.

If you're not yet familiar with GraphQL, get started on [How to GraphQL](https://www.howtographql.com/) or read the official [GraphQL documentation](http://www.graphql.org/).

> Note that this website is still work in progress. If you have ideas for content you would like to see on this page, or want to contribute content yourself, please [get in touch](mailto:hello@graph.cool)!

## Topics

Here is an overview of the available topics:

* [Schema Transformation](./graphql-gateway/schema-transformation.md): Hiding fields from a GraphQL schema or hooking into its resolvers
* [Schema Stitching](./graphql-gateway/schema-stitching.md): Combining multiple GraphQL schemas into a single one
  * [No conflicts](./graphql-gateway/schema-stitching/ex1.md)
  * [Conflicts for root fields](./graphql-gateway/schema-stitching/ex2.md)
  * [Conflicts for types](./graphql-gateway/schema-stitching/ex3.md)
  * [Merging executable & non-executable schemas](./graphql-gateway/schema-stitching/ex4.md)
* [Schema Federation](./graphql-gateway/schema-federation.md): Managing GraphQL schemas within an organization
* [Batching (DataLoader)](./graphql-gateway/batching-dataloader.md): Optimizing GraphQL execution
* [Libraries & Frameworks](./graphql-gateway/libraries-frameworks.md): An overview of available tooling for building GraphQL servers