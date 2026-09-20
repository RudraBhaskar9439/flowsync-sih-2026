# FLOWSYNC — SIH 2026 Technical Prototype

**Unified Industrial Approval & Compliance Orchestrator**  
**Team:** FLOWSYNC · **Problem statement:** 26130, as supplied in SIH_PPT.pptx  
**Delivery:** Functional browser prototype plus production implementation blueprint

## 1. Technical positioning

FLOWSYNC is a **dependency-aware industrial approval orchestration layer**. It represents approval requirements as a **directed acyclic graph (DAG)**, identifies independent branches for parallel execution, enforces prerequisite completion, and surfaces delays through **service-level agreement (SLA) monitoring**.

The intended product sits above existing single-window platforms. It does not replace departmental authority, issue licences, or decide statutory eligibility. Its distinctive demonstration is the executable dependency model and its explainability.

**Suggested presentation statement:** “FLOWSYNC converts a structured industrial project profile into an executable approval DAG. A deterministic rule engine selects the approval nodes, a finite-state machine enforces prerequisites, and critical path analysis estimates the benefit of concurrent processing. A local document vault and contextual guidance help applicants prepare the next action, while department views expose approval delays.”

## 2. What you can demonstrate today

| Module | Working implementation | Boundary |
|---|---|---|
| Project intake | Validated structured fields, conditional approval selection | Simplified illustrative rules; no natural-language AI extraction |
| Approval orchestration | DAG generation, cycle and missing-dependency detection, topological ordering, dependency-gated transitions | Local state; no department API |
| Schedule analysis | Earliest start/finish, backward pass, zero-slack critical nodes, sequential comparison | Assumes unlimited concurrency, no queues or rework |
| Document vault | IndexedDB storage, file signatures, MIME/extension consistency, 10 MB limit, SHA-256 duplicate detection, expiry flags | No OCR, authenticity check, encryption or official acceptance |
| Contextual guidance | Keyword retrieval, current workflow context, source labels, explicit abstention | No embedding model, vector database or LLM |
| Support discovery | Official portal links and explainable illustrative matching categories | No verified scheme catalogue or eligibility determination |
| Government view | Derived active queues, overdue reviews, applicant metadata, local event log | Role preview only; no authenticated RBAC |
| Export | Workflow, scheduling output and event log as JSON | Local download |

Files never leave the browser through the vault. Browser local storage persists the workflow and conversation on the same origin. IndexedDB persists document contents. Clearing site data deletes these records. Localhost and hosted deployment are different origins and therefore have separate vaults and demo state. External font requests and deliberate clicks on official portal links still access external services.

## 3. Demonstration scenario

The seeded project is fictional: **Sahyadri Precision Works**, engineering, Pune, proposed investment ₹2.4 crore, 42 workers, process effluent declared.

| Node | Prerequisites | Illustrative duration |
|---|---|---:|
| Entity and site verification | None | 3 days |
| Building plan review | Entity verification | 12 days |
| Consent to Establish | Entity verification | 15 days |
| Power feasibility | Entity verification | 7 days |
| Fire safety review | Building plan | 10 days |
| Factory registration review | Building plan, Consent to Establish | 10 days |
| Operational readiness | Fire, power, factory | 2 days |

The operational-readiness node is an internal milestone, not a government approval. The specific dependency edges, applicability predicates and durations are demonstration assumptions, not a legally validated Maharashtra workflow.

**Computed planning result:** Sequential sum = 59 days. Parallel planned makespan = 30 days. Illustrative difference = 29 days. Critical path = Entity verification → Consent to Establish → Factory registration → Operational readiness. This is an algorithmic example, not measured impact or a promised approval time.

The seeded simulation is on day 20. Building-plan and consent reviews have exceeded their demo SLAs. Completed entity and power reviews have simulated timestamps. Advancing the clock changes overdue calculations; it does not automatically approve work.

## 4. Orchestration algorithms

Let G = (V, E), where V contains approval tasks and an edge (u, v) means that v requires approval of u.

```
ES(v) = max(EF(u)) for each predecessor u; zero for a root
EF(v) = ES(v) + duration(v)
Parallel makespan = max(EF(v))
Sequential baseline = sum(duration(v))
LF(v) = min(LS(child)) for each child; makespan for a terminal node
LS(v) = LF(v) - duration(v)
Slack(v) = LS(v) - ES(v)
Critical node: Slack(v) = 0
Executable node: every predecessor has status approved
Overdue active review: current demo day - started day > configured SLA
```

The implementation repeatedly selects nodes whose predecessors are already ordered. If unresolved nodes remain but no node is selectable, it rejects the graph as cyclic or containing missing dependencies. This simple scan-based topological routine is suitable for the small prototype and is worst-case O(V² + VE). A production implementation should use Kahn’s algorithm with adjacency lists and an in-degree queue for O(V + E). The scheduling pass and critical-path logic should also use adjacency lists to avoid repeated node scans.

The state machine stores `pending`, `in_progress`, and `approved`; the UI derives `ready` or `blocked` from a pending node’s prerequisites. Illegal transitions fail before mutation. Readiness is recalculated from the latest graph state. A simulated department approval may bypass document acceptance, which is disclosed in the dialog.

The graph highlights a critical edge only when both nodes have zero slack and EF(source) equals ES(target). This avoids implying that every edge between critical nodes lies on a critical path.

## 5. Production architecture

```
Applicant / official browser
    |
    +-- OIDC identity provider (authentication)
    |
API gateway / application service (FastAPI or NestJS)
    |
    +-- Project service and applicability engine
    +-- Workflow orchestrator and state-transition service
    +-- Consent-aware metadata access service
    +-- Retrieval and guidance service
    +-- SLA scheduler / notification workers
    +-- Department connector adapters
    |
PostgreSQL (projects, rule versions, approvals, event history)
pgvector (regulatory chunk embeddings)
Redis + Celery or BullMQ (jobs, retries, deduplication)
Optional Neo4j (only if cross-project graph analysis warrants it)
    |
Authorised departmental APIs / NSWS / MAITRI connectors
```

Use the deck’s React / Next.js, Tailwind and React Flow choices when extending into a production frontend. The delivered native JavaScript prototype keeps the algorithm inspectable and requires no package installation. Avoid introducing both PostgreSQL and Neo4j solely for a small per-project DAG: store nodes and edges relationally first, then justify a graph database with measured query needs.

**Regulatory rule lifecycle:** draft → expert review → published version → superseded. Every generated workflow should pin its rule-set version and applicability explanation. Preserve the version used for an existing application when a rule changes; surface a controlled reassessment instead of silently altering pending dependencies.

## 6. Proposed relational model

These are proposed backend entities, not a deployed schema.

| Entity | Essential fields |
|---|---|
| Applicant | applicant_id, organisation_id, identity_subject, display_name |
| Project | project_id, applicant_id, jurisdiction, sector, investment, workforce, declared_flags |
| RuleSet | ruleset_id, version, jurisdiction, effective_from, reviewed_by, publication_status |
| ApprovalDefinition | approval_type_id, department_id, source_reference, required_document_types |
| Workflow | workflow_id, project_id, ruleset_version, revision |
| ApprovalInstance | approval_id, workflow_id, type_id, status, started_at, completed_at, sla_due_at, row_version |
| Dependency | workflow_id, predecessor_id, successor_id |
| DocumentMetadata | document_id, owner_id, category, content_hash, expiry_date, consent_scope; no local file blob |
| ConnectorEvent | event_id, connector_id, external_reference, event_type, occurred_at, received_at, payload_digest |
| AuditEvent | event_id, actor_id, action, resource_id, timestamp, correlation_id, previous_hash |
| RegulatorySource | source_id, official_url, jurisdiction, effective_date, retrieved_at, review_status |
| KnowledgeChunk | chunk_id, source_id, text, embedding, citation_locator, source_version |

Use foreign keys and unique constraints for dependency edges and external event identifiers. Perform approval updates, dependent-work release, and outbox insertion in a single database transaction. An optimistic concurrency check on `row_version` prevents lost updates.

## 7. Proposed API contracts

These endpoints are implementation targets. They are not available in the browser-only prototype.

```
POST /v1/projects
POST /v1/projects/{project_id}/workflows
GET  /v1/workflows/{workflow_id}
POST /v1/approvals/{approval_id}/transitions
POST /v1/connectors/{connector_id}/events
GET  /v1/projects/{project_id}/alerts
POST /v1/guidance/query
GET  /v1/departments/{department_id}/queue
```

Example transition request:

```json
{
  "action": "approve",
  "expected_revision": 7,
  "idempotency_key": "department-event-unique-reference",
  "reason_code": "REVIEW_COMPLETE"
}
```

Use HTTP 409 for conflicting revisions or unmet workflow prerequisites, 403 for forbidden departmental access, and 422 for invalid payloads. Authenticate connector events, validate external references, deduplicate using connector plus event ID, and reject unsupported transitions. Never trust the client’s chosen role or approval result.

A transactional outbox should record downstream work atomically with the state transition. Workers deliver notifications using bounded exponential backoff, jitter, retry ceilings and a dead-letter queue. Poll or reconcile departmental status when webhooks are unavailable. Preserve both source-event time and received time to handle late and out-of-order events. None of these external integrations is connected in the delivered demo.

## 8. Regulatory RAG design

1. Ingest permitted official source material and preserve URL, publication/effective date and jurisdiction.
2. Require human review before an entry can guide an applicant.
3. Segment into citation-addressable chunks; compute embeddings in pgvector.
4. Filter by jurisdiction, industry, effective date and review status.
5. Combine lexical retrieval with vector similarity and rerank relevant passages.
6. Generate an answer constrained to retrieved evidence, with citations and explicit uncertainty.
7. Abstain when evidence is absent, outdated or contradictory. Route uncertain legal interpretations to an authorised reviewer.
8. Track retrieval recall, citation coverage, unsupported-claim rate and abstention quality on an expert-labelled test set.

Treat retrieved content and uploaded documents as untrusted data. They must never modify tool permissions or authorize submissions. A model’s output should propose a profile or explanation; deterministic validated rules control approval applicability and state transitions.

The current demo uses keyword overlap and templated responses. Do not present it to judges as a deployed LLM or production RAG system.

## 9. Document and access security

Production requirements include authenticated ownership checks, department-scoped permissions, purpose-specific consent, revocation, retention limits and server-side authorization for every protected request. Separating menu views does not enforce access.

A production local vault needs an explicit encryption/key-recovery design, browser-threat model, trusted rendering, CSP and secure update policy. If OCR runs locally, use a Web Worker to avoid blocking the interface and test extraction quality separately from document validity. If document processing becomes server-side, obtain consent and define transport encryption, malware scanning, temporary storage and deletion. SHA-256 detects identical bytes; it does not validate truth, signer identity or government acceptance.

The prototype event log is intentionally local and mutable. Production auditability requires server-controlled records, protected retention, access logs and tamper evidence. A hash chain by itself cannot prevent a privileged operator from rewriting the chain without an independent anchor.

## 10. Five-minute demonstration script

| Time | Action | Technical point |
|---|---|---|
| 0:00–0:40 | Open the workflow and explain the seeded project | Structured profile and explainable approval DAG |
| 0:40–1:20 | Open scheduling rationale; compare 59 and 30 demo days | Topological scheduling, critical path and concurrency assumptions |
| 1:20–2:00 | Approve Building plan; show Fire becomes ready while Factory still waits for CTE | Dependency gating and finite-state transitions |
| 2:00–2:35 | Advance demo clock; open Government view | Derived SLA monitoring and department queues |
| 2:35–3:20 | Save a sample document with a near expiry date; try the same file again | IndexedDB, SHA-256 deduplication and expiry checks |
| 3:20–4:00 | Ask “Which approvals can run in parallel?” | Contextual retrieval with source attribution |
| 4:00–4:30 | Change intake to fewer workers and no effluent | Deterministic rule selection and graph regeneration |
| 4:30–5:00 | Open Technical architecture | Explain implemented components and pilot prerequisites honestly |

## 11. Suggested delivery milestones

- **Next engineering milestone:** move workflow persistence and state transitions to an authenticated API with PostgreSQL, transactional updates and integration tests.
- **Knowledge milestone:** curate one jurisdiction and a narrow industrial scenario with a verified approval rule catalogue and source versions.
- **Document milestone:** integrate OCR on sample documents and evaluate field-level extraction accuracy. Keep validity decisions separate.
- **Connector milestone:** secure an authorised sandbox agreement with one department, implement an adapter, and test duplicate and out-of-order status events.
- **Pilot milestone:** measure real elapsed time, rework, approval-path completeness and user task success before making efficiency claims.

## 12. Sources and evidence

- Supplied six-slide **SIH_PPT.pptx**: product concept, team, problem statement identifier, proposed modules and stack.
- [NSWS official portal](https://www.nsws.gov.in/): reviewed on 20 September 2026 for its approval discovery and support-resource role. No portal text establishes the prototype’s illustrative rules or processing durations.
- [MAITRI portal](https://maitri.maharashtra.gov.in/): referenced by the supplied deck. Automated page access failed during preparation, so current platform details and API availability remain unverified.

## 13. Local use

Extract the source bundle, then from its folder run:

```sh
python3 -m http.server 4173 --directory dist
```

Open `http://localhost:4173`. A local HTTP server is required for ES modules. No API key or package installation is required. Use Node.js to run `node tests/engine-test.mjs` if you want to repeat the algorithm checks. Local state is browser-specific; reset the demo with the sidebar button. Resetting the workflow retains the vault; delete individual saved sample documents within the vault when desired.
