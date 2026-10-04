# ADR-001: Customer-managed source extraction with a stable ingestion boundary

## Status

Accepted for V0.

## Context

The compiler needs to consume information from many enterprise systems, including code repositories, standards repositories, collaboration platforms, architecture documentation, and internal sources.

Building and operating a full connector ecosystem before validating the compiler would expand scope significantly.

At the same time, requiring every customer to understand compiler internals would create excessive integration work and make future managed connectors harder to add.

## Decision

V0 will not require compiler-owned source connectors.

Customers may use their existing DAGs, jobs, scripts, functions, or event pipelines to retrieve data from source systems.

Those upstream systems map source data into a stable compiler ingestion contract.

The compiler owns behavior from that contract onward, including:

- validation;
- idempotent ingestion;
- batching;
- async processing;
- indexing;
- provenance;
- ACL representation and enforcement;
- normalized evidence/catalog updates.

Future native connectors will implement the same ingestion contract.

## Consequences

### Positive

- the compiler remains portable across enterprise environments;
- customers can reuse existing orchestration and credentials;
- connector development does not block V0;
- future managed connectors do not require a new indexing architecture;
- source-specific crawling concerns remain outside the compiler core.

### Trade-offs

- early customers must provide a mapping step;
- integration quality depends on the clarity of the ingestion contract;
- source-specific checkpointing may initially remain customer-owned;
- managed authentication, webhooks, and scheduled synchronization come later.

## Reliability requirements

The ingestion path must be:

- async-first;
- bounded by default;
- idempotent;
- batch-aware;
- resumable at committed boundaries;
- partial-failure-safe;
- permission-aware;
- provenance-preserving.

Upstream concurrency must not be allowed to overwhelm downstream databases, embedding services, or parsers.

## Future direction

A managed connector may later provide:

```text
GitHub / Confluence / SharePoint / other source
        |
   native connector
        |
 standardized ingestion record
        |
    compiler indexer
```

The connector ecosystem is therefore an optional convenience layer above the same stable ingestion boundary rather than a separate ingestion architecture.
