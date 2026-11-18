---
name: add-kafka-pipeline-stage
description: Use when the user wants to add, insert or remove a step (stage/consumer) in a Kafka-based pipeline, e.g. "add a stage", "new consumer between X and Y", "add a status to the lifecycle". Adds a TypeScript consumer that validates input with zod, moves the record's status in the destination DB, follows the standard consumer settings (read_committed, DLQ, no auto-commit before processing), adds tests with a poison pill and a transient failure, and updates the repo's product spec, components and architecture docs. Not for changing what an existing stage does.
---

# Add a Kafka pipeline stage (generic)

A stage = one status transition (`from -> to`) done by one consumer that reads one topic, validates the message,
does its work, advances the status in the destination database, and publishes (directly or via CDC/outbox) to the next topic.
The broker is Kafka; the database and the way the next topic gets fed are whatever this repo already uses.

## 0. Before you start

- Ask the user, and do not guess: **which status/topic the stage reads**, **which status it writes**, **what it does**, and
  **what a permanent vs transient failure means for it**. Stop until answered.
- **Learn the repo's conventions first** (do not impose new ones): find the existing stages (grep for the consumer base class,
  `groupId`, `consumer.run`, `@EventPattern`/`@MessagePattern`, `*.stage.ts`, `*consumer*` in compose files), read the
  closest one fully, and read the project's `CLAUDE.md`, `docs/product-spec.md`, `docs/components.md`, `docs/architecture.md`
  if they exist. **Copy the nearest existing stage's structure**; this skill lists what it must contain, not how the repo names it.
- Language rules (user's global rules): TypeScript only, NestJS for Node services, tests in TypeScript; run everything in containers; install nothing without approval.

## 1. What every stage must have

1. **Status.** Add the new status value to the lifecycle type/enum (plus its comment/diagram of transitions).
   Statuses must include a terminal `failed` that carries an `error` message.
2. **Input validation with zod.** A strict schema for the message the stage accepts (`z.strictObject`, unknown keys rejected,
   types inferred with `z.infer`). Parse at the edge with `safeParse`. The next stage's schema must accept whatever this stage adds.
   A message that fails validation is a **poison pill**: never retried, goes straight to the DLQ.
3. **Status change in the destination DB.** Done through the repo's single status-transition helper, as a **conditional,
   atomic update** (`WHERE id = ? AND status = <from>` / the DB's equivalent), so redelivery is idempotent and a stale
   or duplicate message cannot move a record backwards or twice. Never write the status field directly elsewhere.
   Honour the repo's DB defaults (global rules): MySQL `READ COMMITTED`; Mongo write concern `{w:"majority", j:true}`;
   Couchbase `REQUEST_PLUS` on queries.
4. **Consumer configuration** (same as the other stages; verify against them):
   - own `groupId`, one topic;
   - `isolation.level=read_committed` if upstream produces transactionally or via exactly-once connectors;
   - auto-commit **off**; the offset is committed only after the work **and** the status change succeeded (or the message was
     handed to the DLQ);
   - bounded retries with backoff for **transient** errors (DB/network/timeouts), then DLQ + mark `failed` with `error`;
   - **permanent** errors (validation, business rule) skip retries: DLQ immediately, mark `failed` with `error` when a record id is known;
   - DLQ topic `<topic>.dlq`, carrying the original payload plus error, stage, attempt count in headers;
   - idempotent producer if the stage produces; graceful shutdown (finish the in-flight message, then disconnect);
   - health check for the container.
5. **Module and entry point** (NestJS), copied from the nearest stage.
6. **Infrastructure.** Compose/k8s service for the consumer with the same anchors, settings and healthcheck as its neighbours and the repo's
   hardening (non-root, `cap_drop: [ALL]`, `no-new-privileges`, no docker socket); the topic and its DLQ created wherever the
   repo creates topics; the source connector/outbox/producer rule that feeds the next topic. **Startup order**: if the repo orders
   consumers downstream-first, keep that and re-wire neighbours' `depends_on`.
7. **Tests** (TypeScript, in the repo's test style and runner; run in containers):
   - happy path: valid message -> status moves `from -> to`, output published as expected;
   - **poison pill**: invalid/unparsable message -> DLQ, no retries, offset advanced, consumer keeps running, record `failed` with `error` if an id exists;
   - **transient failure**: dependency fails N times then succeeds -> retried, ends `to`, nothing in DLQ; and fails permanently -> DLQ + `failed`;
   - redelivery of the same message is idempotent (status not moved twice).
   If the repo already has a generic poison-pill test covering all consumers, extend its stage list instead of duplicating it.
8. **Docs in the repo you work on**, in the same change:
   - `docs/product-spec.md`: the flow/lifecycle section (new status, transition) and any guardrail it touches; never rewrite the
     spec's intent/vision sections without the user asking;
   - `docs/components.md`: a section for the new stage (what it owns, what it must not do, its topic/DLQ/group), plus startup order if changed;
   - `docs/architecture.md`: run the **sync-architecture-diagram** skill.
   If a doc does not exist, tell the user and ask whether to create it.

## 2. Finish

- Run the repo's guardrail/lint/test commands (inside containers). Fix causes, not checks.
- Run the **check-guardrails** skill.
- Report: files touched, new topic/DLQ/group names, which docs were updated, anything you could not verify (e.g. tests not run
  because the stack was down), and any known gaps the change touches (do not fix them as a side effect).
