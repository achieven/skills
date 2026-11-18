---
name: sync-architecture-diagram
description: Use to create or update the system architecture diagram (docs/architecture.md, Mermaid) whenever a change alters services, ports, inter-service calls, topics/connectors, data flow, datastores or other infrastructure topology, or when the user says the diagram is stale. Works in any repo; also callable from commit review or add-kafka-pipeline-stage.
---

# Sync architecture diagram (generic)

`docs/architecture.md` (or the repo's existing architecture doc: look for `docs/architecture*`, `ARCHITECTURE.md`) is the single
source of truth for the topology: a Mermaid graph, a component table and a data-flow list. Keep it truthful with targeted edits;
never redraw it wholesale for a small change.

## Steps

1. Read the existing doc fully. If none exists, create `docs/architecture.md` with a `# Architecture` heading, a
   `Last verified against:` line, an empty component table and a minimal `graph LR` block, and treat every component as new.
2. Find the architectural surface that changed:
   - given a ref: `git show --stat <ref>` and `git diff <ref>~1 <ref>` on infra files;
   - ad hoc: read `docker-compose*.y*ml` / k8s manifests, and per service its entrypoints to see who calls whom
     (HTTP/gRPC clients, Kafka producers/consumers and topics, connector configs, DB/cache connections, variables holding hosts/urls/ports).
   Look for: new/removed services, ports, `depends_on`/startup order, topics and DLQs, connectors, datastores and schemas,
   new direct service-to-service calls.
3. Write a short list of concrete deltas (added node, removed edge, renamed port...). If you cannot tell from the diff whether
   an edge exists, say so rather than guessing.
4. Edit only what the deltas require: Mermaid graph, component table rows, data-flow list. Keep the existing style (subgraphs,
   label format) so the file does not churn.
5. Update `Last verified against:` to the short SHA (`git rev-parse --short HEAD` or the ref) and today's date.
6. Report the changes as bullets, or "no changes needed" when step 3 found nothing.

## Notes

- Touches only the architecture doc, never application code.
- Service/data-flow level only; no classes or functions.
- Never write secret values (passwords, tokens) into the doc; note only that a connection exists.
