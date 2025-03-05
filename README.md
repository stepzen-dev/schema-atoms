# schema-atoms

Reusable GraphQL schema elements concept for IBM API Connect for GraphQL/StepZen.

> [!NOTE]
> IBM API Connect for GraphQL was originally StepZen, this document uses StepZen for brevity.

## Overview

A StepZen GraphQL schema can be composed from a combination of existing reusable schema elements.
This lays out a convention based approach of how reusable schemas should be defined as _schema atoms_.

The goal is to encourage a plug-and-play approach avoiding dependencies between unrelated _schema atoms_.

For example in this project's package delivery scenario we have:

- an interface based package [delivery](./examples/atoms/delivery/) schema that allows BFF schemas to hide implementation details from clients
- multiple instances of delivery service schemas that can provide the implementations of the interface (e.g. [Fast Package](./examples/atoms/fast-package/) , [RainOrShine](./examples/atoms/rainorshine/) , [Mercury](./examples/atoms/mercury/) )

A reusable schema approach allows any of these combinations to be created and deployed:

- interface delivery service with mocking only
- interface delivery service with only a Fast Package implementation
- interface delivery service with any combination of Fast Package, RainOrShine, ... implementations
- Fast Package only service
- etc.

Thus for example, it should be possible to deploy Fast Package service without including the delivery interfaces.

_Schema atoms_ also encourage a unit testing approach since a _schema atom_ can be deployed independently.

## Schema concepts

### Deployable schema

A _deployable schema_ is a schema that can be deployed to a running service using `stepzen deploy` resulting in a live endpoint.

A _deployable schema_ is represented in a project as a top-level folder containing:

- at least one `index.graphql` containing GraphQL schema definitions (using SDL)
  - Multiple `.graphql` files are included through `@sdl` directive
  - Sub-folders may be used to organize multiple `.graphql` files
  - Files included through `@sdl(files:)` by `index.graphql` can further include files by using `@sdl` on an `extend schema` clause.
  - Relative paths for the `@sdl(files:)` argument are relative to the file containing
    the `.graphql` file with `@sdl` and are restricted to sub-folders
    (e.g. `customer/types.graphql`, `customer/database/types.graphql` but **not** `../types.graphql`)
- a `stepzen.config.json` configuration file defining the endpoint name
- an optional `config.yaml` configuration file containing configuration and access rules

A _deployable schema_ may be comprised of:

- (preferred) one or more _schema atoms_ included through their `atom.graphql` files using `@sdl`.
- `.graphql` files that are not organized into a _schema atom_ included through `@sdl` including arbitrary nesting.
- a combination of the above.

### Schema atom

> [!NOTE]
> This is a convention to encourage resuablilty of schema elements and an engineering disipline to GraphQL schema development.

A _schema atom_ is a reuseable collection of GraphQL schema definitions.

A _schema atom_ is a folder containing:

- `atom.graphql` - This is the single file that allows the _schema atom_ to be included in
  _deployable schemas_ or other _schema atoms_ through `@sdl`, e.g. `@sdl(files:["fast-package/core/atom.graphql"])`.
- Optional additional `.graphql` files included through `@sdl(files:)`, with the root being `atom.graphql`.

Schema atoms can take advantage of GraphQL `extend` definitions, for example, to add fields to types (especially `Query`), or enum values to an enum.

> [!NOTE]
> A _schema atom_ can be a "molecule", made up from multiple _schema atoms_ but still a reusable set of schema elements.
> For example a goespatial _schema atom_ that depends on geolocation and weather services _schema atoms_.
> In this case the result is still termed a _schema atom_, as the usage is the same as a self-contained _schema atom_.

### index.graphql

As `index.graphql` represents the root of a _deployable schema_ it _should_ include a schema definition with a description
of the schema or endpoint (which is then available through introspection).

```graphql
"""
Customer endpoint provides access to customer information including orders.
"""
schema {
  query: Query
  mutation: Mutation
}
```

## Schema atom

### Repository layout

Several related _schema atoms_ may exist, a convention would be have three "type" folders under the root folder:

- `core` - core definitions, typically a single _schema atom_, but potentially multiple, e.g. `fast-package/core/status`, `fast-package/core/mgmt` etc.
- `extend` - any _schema atom_ that expand the functionality of other _schema atom_ definitions, e.g. `fast-package/extend/delivery`
- `mock` - _schema atom_ that mock types within core definitions for testing, etc. Typically one, but potentially multiple.

### atom.graphql

`atom.graphql` _should_ contain a schema definition extension (`extend schema`) that includes the `.graphql`
files that implement the atom.

- Example: [`fast-package/core/atom.graphql`](examples/atoms/fast-package/core/atom.graphql).

Simple _schema atoms_ could have their definitions contained completely in `atom.graphql`.

- Example: [`fast-package/mock/atom.graphql`](examples/atoms/fast-package/mock/atom.graphql).

### Mechanics

Schemas may be developed by different groups, different organizations including open source.

Thus while a _deployment schema_ `S` may require multiple _schema atoms_ (`A,B,C`) it **cannot** be assumed that the source
for all the included atoms is located in sub-folders of the repository folder for `S`.

Thus a process is needed where the source folders for `A,B,C` must be copied into a folder that contains `S` before the schema is deployed,
typically by a CI-CD process e.g. a github workflow. This could be either:

- Copying `S,A,B,C` into a temporary folder and deploying from there
- Copying `A,B,C` into the folder for `S` and deploying from that folder

Note `@sdl(files:)` does not support relative paths going above the current folder (e.g. `../weather/atom.graphql` is not supported).

# Example schemas

## Deployable schemas

- [`endpoints/delivery`](examples/endpoints/delivery/) - deployment of _schema atom_ with only interface types, thus the interface types are mocked since there are no implementations.
- [`endpoints/fast-package`](examples/endpoints/fast-package/) - deployment of a single _schema atom_
- [`endpoints/fast-package-mock`](examples/endpoints/fast-package-mock/) - deployment of a _core schema_ with its _mock schema atom_

## Schema atoms

- [`delivery/core`](examples/atoms/delivery/) - interface for a delivery service application
- [`fast-package/core`](examples/atoms/fast-package/core) - Fast Package core functionality (simulating a call to a REST service)
- [`fast-package/mock`](examples/atoms/fast-package/mock) - Allows `fast-package/core` to be mocked to avoid real calls during application testing.
- [`fast-package/extend/delivery`](examples/atoms/fast-package/extend/delivery/) - Extends `fast-package/core` to implement `delivery/core`
