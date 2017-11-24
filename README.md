<img src="https://imgur.com/UlE80Qv.png" width="328" />

**Advanced GraphQL** is a collection of techniques, best practices and advanced patterns for building applications and systems based on GraphQL.

If you're not yet familiar with GraphQL, get started on [How to GraphQL](https://www.howtographql.com/) or read the official [GraphQL documentation](http://www.graphql.org/).

> Note that this website is still work in progress. If you have ideas for content you would like to see on it, or want to contribute yourself, please [get in touch](mailto:hello@graph.cool)!

## Overview

* [Schema Transformation](./content/schema-transformation.md): Hiding fields from a GraphQL schema or hooking into its resolvers
* [Schema Stitching](./content/schema-stitching.md): Combining multiple GraphQL schemas into a single one
  * [No conflicts](./content/schema-stitching/ex1.md)
  * [Conflicts for root fields](./content/schema-stitching/ex2.md)
  * [Conflicts for types](./content/schema-stitching/ex3.md)
  * [Merging executable & non-executable schemas](./content/schema-stitching/ex4.md)
* [Schema Federation](./content/schema-federation.md): Managing GraphQL schemas within an organization
* [Batching (DataLoader)](./content/batching-dataloader.md): Optimizing GraphQL execution
* [Libraries & Frameworks](./content/libraries-frameworks.md): An overview of available tooling for building GraphQL servers