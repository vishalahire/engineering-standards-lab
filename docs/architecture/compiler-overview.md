# Compiler Overview

## Purpose

The compiler turns enterprise, team, repository, and task context into a small evidence-backed context packet for an existing coding agent.

It should not replace coding agents, repository intelligence tools, orchestration platforms, or source systems.

## Core responsibilities

The compiler should:

- ingest normalized enterprise and repository evidence;
- maintain provenance and access-control metadata;
- resolve which requirements, decisions, exceptions, and exemplars apply to a task;
- reuse repository intelligence from tools such as Atlas or Aider rather than rebuild repository mapping;
- filter and rank before reading large amounts of source content;
- compile a bounded task-context packet;
- support later change assessment before PR creation.

## High-level flow

```text
Enterprise standards / decisions / exceptions
                    +
Repository evidence / structure / exemplars
                    +
Developer task
                    |
                    v
            Context Compiler
                    |
        +-----------+-----------+
        |                       |
        v                       v
 Task context packet      Change assessment
        |                       |
        v                       v
 Existing coding agent      Developer / CI
```

## Index-time versus task-time work

Expensive repository analysis should not run for every developer prompt.

### Index-time

Indexing may happen through:

- initial bulk ingestion;
- incremental ingestion after source changes;
- scheduled reconciliation;
- event-triggered updates;
- customer-managed DAGs or jobs.

Index-time work can create:

- repository descriptors;
- structural maps;
- framework and dependency fingerprints;
- document and code evidence;
- searchable indexes;
- provenance and ACL metadata;
- candidate exemplars.

### Task-time

For a meaningful engineering task, the compiler should perform a comparatively lightweight path:

```text
task intent
  -> authorization
  -> applicability
  -> filter
  -> rank
  -> retrieve selected evidence
  -> optional semantic/LLM interpretation
  -> budgeted context packet
```

Small or trivial tasks may not require compilation.

## Repository intelligence

Repository mapping is infrastructure, not the product differentiator.

Candidate backends include:

- fkenmar/atlas;
- Aider repo-map;
- Tree-sitter where targeted extraction is needed;
- SCIP/LSP where precise symbol navigation is already available.

These components help answer what exists and what is structurally related.

The compiler owns the higher-level questions:

- what applies to this task;
- what is authoritative;
- what is only evidence or precedent;
- what examples are useful;
- what can fit within the context budget.

## Evidence and governance remain separate

Observed repository behavior must not automatically become policy.

The system should distinguish:

- arbitrary examples;
- recurring patterns;
- curated exemplars;
- approved reference implementations;
- approved requirements and exceptions.

Popularity is evidence, not authority.

## Execution modes

The same core should eventually support:

- Python library;
- CLI;
- container;
- coding-agent tool;
- MCP wrapper;
- REST/service deployment;
- serverless execution.

V0 should prioritize a reusable core plus CLI/tool invocation.

## Portability principle

Customers may choose their own orchestration system.

The compiler should not require ownership of Airflow, Cloud Functions, GitHub Actions, Pub/Sub, or similar systems.

Common integrations may become managed connectors later, but the underlying ingestion and compilation contracts should remain stable.

## Efficiency principles

- filter and rank before reading;
- prefer symbols/signatures and bounded evidence spans over entire files;
- use deterministic and structural methods before expensive reasoning;
- keep semantic retrieval optional and evidence-backed;
- cache by source revision/content hash;
- enforce hard serialized context budgets;
- allow evidence expansion on demand;
- measure end-to-end token and tool-output cost, not only prompt size.

## Current product hypothesis

Developers provide task intent.

The engineering environment should provide the minimum authoritative organizational context needed to make and verify the change.
