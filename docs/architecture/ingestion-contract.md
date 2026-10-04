# Ingestion Contract

## Purpose

The ingestion contract is the stable boundary between customer-managed source extraction and the compiler's indexing pipeline.

For V0, customers may use their own DAGs, jobs, functions, scripts, or event consumers to retrieve content from systems such as GitHub, Confluence, SharePoint, file stores, or internal APIs.

The compiler should not require those systems to understand its internal indexes.

## Boundary

```text
Source systems
    |
Customer DAG / job
- authentication
- source APIs
- pagination
- source-specific checkpoints
- change detection
    |
    v
Normalized ingestion contract
    |
    v
Compiler indexer
- validation
- normalization
- indexing
- provenance
- ACL representation
- catalog updates
```

Future managed connectors should produce the same ingestion contract.

## Initial record shape

A V0 record should carry enough information to answer:

- what is this item;
- where did it come from;
- what version is it;
- what operation is requested;
- what does it contain;
- who may access it;
- how can it be traced back to source.

Example:

```yaml
record_id: "confluence:page:12345"

operation: "upsert" # upsert | delete

source:
  type: "confluence"
  source_id: "enterprise-architecture"
  item_id: "12345"
  revision: "17"

content:
  type: "document"
  title: "Agent UI Standards"
  body: "..."
  format: "markdown"

metadata:
  path: "/architecture/ai/agent-ui"
  url: "..."
  modified_at: "2026-10-04T09:30:00Z"
  labels:
    - ai
    - frontend

access:
  visibility: "restricted"
  principals:
    - type: "group"
      id: "ai-platform-team"

provenance:
  retrieved_at: "2026-10-04T10:00:00Z"
  content_hash: "sha256:..."
```

## Stable identity and idempotency

`record_id`, source identity, revision, and content hash should support idempotent replay.

Reprocessing the same logical source item must not create duplicates.

A repeated record with an unchanged revision/hash may be skipped.

## Operations

V0 should support at minimum:

- `upsert`
- `delete`

The upstream system decides what changed. The compiler does not need to understand each source system's event model.

## Access control

Source visibility and ACL metadata must travel with the record.

The compiler must not surface indexed context to a caller who lacks permission to the underlying source.

ACL metadata is part of the indexed artifact, not optional descriptive metadata.

## Provenance

Every indexed artifact should remain traceable to source through:

- source ID;
- source item ID;
- revision;
- content hash;
- retrieval time.

Derived artifacts should retain references to the source evidence they came from.

## Async-first ingestion

The primary programming model should be asynchronous.

Conceptually:

```python
async def ingest(
    records,
    *,
    batch_size=100,
    max_concurrency=8,
):
    ...
```

A synchronous convenience wrapper may be provided later, but the implementation should use the same async core.

## Batching and backpressure

Upstream fetch concurrency and compiler write batching are separate controls.

The compiler should support:

- async iterators/streams;
- bounded queues;
- configurable batch size;
- configurable concurrency;
- backpressure;
- cancellation;
- timeouts;
- retries;
- idempotent replay.

Concurrency should be independently bounded at expensive downstream boundaries such as parsing, embeddings, and database writes.

## Batch semantics

Successfully committed work should survive a later failure.

Example:

```text
batch 1  -> committed
batch 2  -> committed
batch 3  -> partial record failure
batch 4  -> not started
```

A retry should not require reprocessing batches 1 and 2.

The ingestion API should return both run-level and record-level outcomes.

Example:

```json
{
  "run_id": "run-42",
  "accepted": 997,
  "skipped": 2,
  "failed": 1,
  "failures": [
    {
      "record_id": "confluence:page:21049",
      "reason": "invalid ACL principal"
    }
  ]
}
```

## Failure handling

Record-level failures should be distinguishable from systemic failures.

A malformed record may be quarantined while the remaining batch continues where safe.

Authentication failure, persistent source outage, or database failure should stop the run without losing previously committed progress.

A partial run must never be mistaken for a successful full synchronization.

## What does not belong in this contract

The ingestion record should not decide:

- whether content is an approved enterprise requirement;
- whether an example is a preferred exemplar;
- whether one decision supersedes another;
- governance precedence.

Those meanings belong in later governance and evidence contracts.
