`delivery` is a _schema-atom_ with a `Query.expected` field that returns
a `Delivery` interface with an expected delivery date and a note about the delivery.

There are multiple independent delivery service implementations.

- `fastpackage`'s type `FastPackage` aligns with `Delivery` and thus a simple `extend type FastPackage implements Delivery` is used.
  This might model the situtation where the abstract and implementation schemas were designed & implemented at the same time.
- `rainorshine` models the situation where the schema was designed and implemented to allow it to fufill `Delivery`.
  In this case `RainOrShine` type aligns with `Delivery` but the backend REST response does not, so the capabilities of `@rest` are used
  to align the response.
- `mercury` models the situation where the schema was created before or independently of `delivery` and thus its type `Mercury` does
  not align with `Delivery`. Thus there is a _schema-atom_ that extends `mercury` with reshaping and implementation of `delivery`.
