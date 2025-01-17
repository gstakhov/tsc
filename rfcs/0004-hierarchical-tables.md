# Generic, hierarchical tables

- Champion: JGP

## Summary

The current structure uses `dataset`, `name` and `columns` to define a data contract. However, these are all database terms - or specially, BigQuery terms, and they suggest very clearly an implementation (tables in a data warehouse). We want the data contract to be more generic than that.

Moreover, we think there is a need for a hierarchical representation, to represent complex data types.

## Motivation

Many of us are using data contracts for more than just the data warehouse. The second most popular technology used is streaming, for example Kafka or Google Pub/Sub. But as a concept, data contracts are not tied to any particular technology - it's more of a pattern to describe data, no matter where that data happens to be.

We want to make the standard reflect that. It should be generic, so as technology changes, the standard remains relevant.

This aligns with our guiding value of interoperability.

## Design and examples

We currently have the following structure:

```yaml
dataset:
  - table: orders
    columns:
      - column: order_id
  - table: line_items
    columns:
      - column: order_id
```

Found issues:
- Confusion whether this is a logical or physical data model. Table suggests a physical one. We assume, it means a logical data model.
- The term dataset is problematic as it has a very specific meaning in the context of GCP, and some fields in the ODCS are GCP specific and also use the term dataset (e.g., `datasetName`).

Open Questions:
- How do we capture other data, like unstructured data (scientific papers in form of PDFs), vectors, ... Do we want to specify unstructured or semistructured data in a data contract?

## Alternatives

### Option A - Field and Name

Status: rejected.

We want the following structure:

```yaml
dataset:
- name: orders
  fields:
    - name: order_id
- name: line_items
  fields:
    - name: order_id
```

### Option B - Genericity

Status: rejected.

The structure is fine. We want different naming.

```yaml
dataset:
- dataobject: orders
  type: table
  fields:
    - name: order_id
- dataobject: line_items
  type: table
  fields:
    - name: order_id
```

### Option C - Genericity #2

Status: rejected.

```yaml
models:
- model: orders
  type: table
  fields:
    - field: order_id
- model: line_items
  type: table
  fields:
    - field: order_id
```

### Option D - Genericity with hierarchy

```yaml
schema:
- object: MyTable
  kind: table
```

```yaml
schema:
- object: MyTopic
  kind: topic
```

```yaml
schema:
- object: Transactions
  kind: table
  physicalName: trx_v1
  description: Contains transactions.
  attrtibutes:
  - field: Identifier
    physicalName: id
```

```yaml
schema:
- object: Transactions
  kind: table
  physicalName: trx_v1
  description: Contains transactions.
  attrtibutes:
  - field: Identifier
    physicalName: id
```

```yaml
schema:
- object: Transactions
  kind: table
  physicalName: trx_v1
  description: Contains transactions.
  attrtibutes:
  - field: Identifier
  - object: TransactionDetail
    kind: table
    physicalName: trx_details_v1
    description: Contains transactions.
    attributes:
    - field: date
      physicalName: trx_ts
      description: Timestamp of the transaction.
```

### Option E = Option D + Support for Array

TBD

## Decision

TBD

## Consequences

- Makes the structure more applicable for users for other data structures
- Nudges users more towards creating a logical model and less a physical model here
- This is a breaking change. Would be targeted for v3.
- We are very conscious in separating the logical representation of a dataset with the physical representation

## References

* At GoCardless [we use name](https://medium.com/gocardless-tech/implementing-data-contracts-at-gocardless-3b5c49074d13), and it's working well for both tables and streams. It has not caused any confusion in 3 years of production, despite most users being familiar with the underlying technology and its terms (BigQuery and Pub/Sub)
* [Protobuf calls them "fields"](https://protobuf.dev/programming-guides/proto3/), as [does Avro](https://avro.apache.org/docs/1.11.1/specification/).
* [OpenAPI uses "title"](https://spec.openapis.org/oas/latest.html), which could also work well instead of name
