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
- multiple instances of delivery service schemas that can provide the implementations of the interface (e.g. [FastPackage](./examples/atoms/fastpackage/) , [RainOrShine](./examples/atoms/rainorshine/) , [Mercury](./examples/atoms/mercury/) )

A reusable schema approach allows any of these combinations to be created and deployed:

- interface delivery service with mocking only (Note: interfaces with no implementations are automatically mocked in StepZen.)
- interface delivery service with FastPackage implementation
- interface delivery service with any combination of FastPackage, RainOrShine, ... implementations
- FastPackage only service
- etc.

Thus for example, it should be possible to deploy FastPackage service without including the delivery interfaces.

_Schema atoms_ also encourage a unit testing approach since a _schema atom_ can be deployed independently of a full schema.

## Schema concepts

A _StepZen deployable schema_ is represented in a project as a top-level folder containing:

- at least one `index.graphql` containing GraphQL schema definitions (using SDL)
  - Multiple `.graphql` files are included through `@sdl` directive
  - Sub-folders may be used to organize multiple `.graphql` files
  - Files included through `@sdl(files:)` by `index.graphql` can further include files by using `@sdl` on an `extend schema` clause.
  - Relative paths for the `@sdl(files:)` argument are relative to the file containing
    the `.graphql` file with `@sdl` and are restricted to sub-folders
    (e.g. `customer/types.graphql`, `customer/database/types.graphql` but **not** `../types.graphql`)
- a `stepzen.config.json` configuration file defining the endpoint name
- an optional `config.yaml` configuration file containing configuration and access rules

StepZen schemas are _schema atoms_ and/or _deployable schemas_.

### Schema atom

> [!NOTE]
> This is a convention to encourage resuablilty of schema elements and an engineering disipline to GraphQL schema development.

A _schema atom_ is a reuseable collection of GraphQL schema definitions.

A _schema atom_ is a folder containing:

- `atom.graphql` - This is the single file that allows the _schema atom_ to be included in
  other schemas through `@sdl`, e.g. `@sdl(files:["fastpackage/core/atom.graphql"])`.
- Optional additional `.graphql` files included through `@sdl(files:)`, with the root being `atom.graphql`.

Schema atoms can take advantage of GraphQL `extend` definitions, for example, to add fields to types (especially `Query`), or enum values to an enum.

> [!NOTE] > _schema atoms_ can be "molecules", made up from multiple _schema atoms_ but still a reusable set of schema elements. For example a goespatial _schema atom_ that includes geolocation and weather services _schema atoms_. In this case the result is still termed a _schema atom_, as the usage is the same as a self-contained _schema atom_.

### Deployable schema

A _deployable schema_ is a schema that can be deployed to a running StepZen service using `stepzen deploy` resulting in a live endpoint.

A _StepZen schema_ is a _deployable schema_ if the top-level folder contains:

- `index.graphql`
- `stepzen.config.json`

In this case `stepzen deploy` can be used to result in a deployed endpoint.

A _deployable schema_ may be comprised of:

- (preferred) one or more _schema atoms_ included through their `atom.graphql` files using `@sdl`.
- `.graphql` files that are not organized into a _schema atom_ included through `@sdl` including arbitrary nesting.
- a combination of the above.

#### index.graphql

As `index.graphql` represents a deployable schema it _should_ include a schema definition with a description
of the schema (which is then available through introspection).

```graphql
"""
Customer endpoint provides access to customer information including orders.
"""
schema {
  query: Query
  mutation: Mutation
}
```

If the schema is also a _schema atom_, e.g. a _core schema_, then `index.graphql` _should_ only contain a
schema definition with a description and `@sdl` directive only including `atom.graphql`.

Example: [`schemas/fastpackage/core/index.graphql`](fastpackage/core/index.graphql) is a _deployable schema_ that is a _core schema_.

Example: [[`endpoints/deliver-fastpackage/index.graphql`](../endpoints/delivery-fastpackage/index.graphql)] a _deployable schema_ comprised of multiple _schema atoms_.

## Schema atom

### Repository layout

Several related _schema atoms_ may exist, a convention would be have three "type" folders under the root folder:

- `core` - core definitions, typically a single _schema atom_, but potentially multiple, e.g. `fastpackage/core/status`, `fastpackage/core/mgmt` etc.
- `extend` - any _schema atom_ that expand the functionality of core definitions, e.g. `fastpackage/extend/delivery`
- `mock` - _schema atom_ that mock types within core definitions for testing, etc. Typically one, but potentially multiple.

### atom.graphql

`atom.graphql` _should_ contain a schema definition extension (`extend schema`) that includes the `.graphql`
files that implement the atom.

- Example: [`fastpackage/core/atom.graphql`](examples/atoms/fastpackage/core/atom.graphql).

Simple atoms could have their definitions contained completely in `atom.graphql`.

- Example: [`fastpackage/mock/atom.graphql`](examples/atoms/fastpackage/mock/atom.graphql).

### Mechanics

_StepZen schemas_ may be developed by different groups, different organizations including open source.

Thus while a _deployment schema_ `S` may require multiple _schema atoms_ (`A,B,C`) it **cannot** be assumed that the source
for all the included atoms is located in sub-folders of the repository folder for `S`.

Thus a process is needed where the source folders for `A,B,C` must be copied into a folder that contains `S` before the schema is deployed,
typically by a CI-CD process e.g. a github workflow. This could be either:

- Copying `S,A,B,C` into a temporary folder and deploying from there
- Copying `A,B,C` into the folder for `S` and deploying from that folder

Note `@sdl(files:)` does not support relative paths going above the current folder (e.g. `../weather/atom.graphql` is not supported).

# Example schemas

## Deployable schemas

- [`endpoints/delivery`](../endpoints/delivery/) - deployment of an _abstract schema_, thus the interface types are mocked since there are no implementations.
- [`endpoints/fastpackage`](../endpoints/fastpackage/) - deployment of a _core schema_
- [`endpoints/fastpackage-mock`](../endpoints/fastpackage-mock/) - deployment of a _core schema_ with its _mock schema atom_

## Schema atoms

- [`example/atoms/delivery/core`](examples/atoms/delivery/) - interface for a delivery service application
- [`examples/atoms/fastpackage/core`](examples/atomsfastpackage/core) - Fast Package core functionality (simulating a call to a REST service)
- [`examples/atoms/fastpackage/mock`](examples/atomsfastpackage/mock) - Allows `fastpackage/core` to be mocked to avoid real calls during application testing.
- [`examples/atoms/fastpackage/extend/delivery`](examples/atomsfastpackage/extend/delivery/) - Extends fastpackage/core`to implement`delivery`
