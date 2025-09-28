# Product Backlog: Acme-Sub Future State Predictive Maintenance & Data Platform (Rev 3)

Fully revised backlog with deeper decomposition, enriched operational detail, and embedded industry / best practices. All stories sized for 1–3 day completion where possible. Original (Rev 1) monolithic stories are superseded; traceability provided via ParentRef tags referencing their prior higher-level intent.

Global Non‑Negotiable Constraint:
The on‑prem inference service MUST function autonomously during any loss of internet or connectivity (no blocking real-time dependency on cloud control plane). All designs & acceptance criteria assume and verify this resilience.

Legend:

- F#: Feature identifier.
- US-F#.#.#: User Story (Feature / capability group / sequence index).
- ParentRef: Original higher-level story or concept being refined.
- NFR: Non‑functional / quality criteria enforcing performance, security, reliability, operability.

Best-Practice Themes Embedded:

- ISA‑95 aligned Unified Namespace (UNS) structuring.
- Secure-by-default (least privilege, mTLS, Zero Trust segmentation, secretless auth with Managed Identity).
- Data contract & schema versioning (semantic version rules, evolution playbook).
- Medallion architecture (Bronze immutable raw, Silver conformed, Gold aggregated / semantic) with lineage and quality metrics.
- SRE & MLOps observability (golden signals + model metrics + drift detection PSI / distribution shifts).
- Reproducible ML lifecycle (MLflow tracking, feature versioning, container immutability, SBOM generation, supply chain scanning).
- Resilience engineering (offline operation, buffering, replay ordering guarantees, capacity baseline tests, chaos/failover drills).
- Governance (glossary, dictionary, lineage, retention, audit logging, catalog readiness for future Purview integration).
- Cost & performance optimization (tiering, job baselines, partition strategy justification).

---

## Feature F1: On-Prem MQTT & Unified Namespace Foundation

**Objective:** Provide a resilient, secure, governed on‑prem HiveMQ cluster & Unified Namespace that publishes structured OT telemetry with quality tagging and direct (secure) forward to Event Hubs (eliminating prior cloud HiveMQ VM). Establish governance, performance baselines, buffering & replay reliability.

### US-F1.1.1: UNS Topic Hierarchy Draft
**ParentRef:** ORIGINAL: Design Unified Namespace Standard

**As** an OT Architect **I want** an initial ISA‑95 aligned UNS topic hierarchy draft **so** stakeholders validate structural approach early.

**Rationale:** Early review reduces rework and clarifies downstream parsing logic.

**Acceptance Criteria:**

- Draft includes: Enterprise/Site/Area/Line/Cell/Equipment/Tag levels with regex patterns & 3+ concrete examples per level.
- Conventions define allowed characters, max lengths, casing rules.
- Open issues list (ambiguities / unresolved semantics) appended.
- Stored in version control at `docs/uns/UNS-hierarchy-draft.md`.

**NFRs:** Readable markdown, internal links to examples, ≤ 2 external dependencies.

**Artifacts:** Hierarchy draft document; issue tracker references for open questions.

### US-F1.1.2: UNS Governance & Versioning Policy
**ParentRef:** ORIGINAL: Design Unified Namespace Standard

**As** a Data Steward **I want** a governance & versioning policy for UNS changes **so** producers and consumers coordinate safely.

**Rationale:** Controlled evolution prevents downstream breakage.

**Acceptance Criteria:**

- Semantic versioning rules (MAJOR breaking path changes, MINOR additive segments, PATCH documentation clarifications) defined.
- Change proposal template created (sections: Rationale, Impacted Topics, Migration Plan, Rollback).
- Approval workflow defined (roles: OT Architect, Data Engineering Lead, Data Steward).
- Sample simulated change (adding new Site segment) executed through workflow.

**NFRs:** Policy doc < 3 pages, searchable keywords.

**Artifacts:** `docs/uns/UNS-governance.md`; updated PR template checklist.

### US-F1.1.3: UNS Topic Validation CLI
**ParentRef:** ORIGINAL: Design Unified Namespace Standard

**As** a Data Engineer **I want** a CLI script to validate topic strings locally **so** publishers catch errors pre-deployment.

**Rationale:** Shift-left validation reduces production ingestion failures.

**Acceptance Criteria:**

- CLI returns JSON: { "topic": str, "valid": bool, "violations": [] }.
- Supports batch file validation.
- Integrated into CI (fails build if invalid topics committed in config files).
- Performance: validates 1000 topics <2s on standard dev machine.

**NFRs:** No external network calls; unit tests ≥ 90% branch coverage.

**Artifacts:** `tools/uns_validate.py`; unit test suite.

### US-F1.2.1: HiveMQ Cluster Provisioning Automation
**ParentRef:** ORIGINAL: Implement HiveMQ Cluster On-Prem

**As** an OT Engineer **I want** automated provisioning for a 3-node HiveMQ cluster **so** deployments are consistent & auditable.

**Rationale:** Manual provisioning risks drift & inconsistent hardening.

**Acceptance Criteria:**

- Infrastructure-as-Code (Ansible or Terraform) with parameters (node_count, heap_size, storage_path).
- Idempotent re-run: zero diff after initial apply.
- Health check endpoint returns green for all nodes.
- Deployment log archived (timestamp, IaC version).

**NFRs:** Provision time ≤ 30 minutes; repeat run ≤ 5 minutes.

**Artifacts:** `infra/hivemq/` module; runbook snippet.

### US-F1.2.2: HiveMQ Configuration as Code
**ParentRef:** ORIGINAL: Implement HiveMQ Cluster On-Prem

**As** a Platform Engineer **I want** broker configuration templates in VCS **so** changes are traceable.

**Acceptance Criteria:**

- Config includes listener ports, persistence, authentication extension settings.
- Sensitive values externalized (no secrets in repo).
- Startup logs emit configuration checksum.
- Config drift detection script highlights local changes vs repo template.

**NFRs:** Templates parameterized; no duplicate values.

**Artifacts:** `infra/hivemq/config/`; drift script.

### US-F1.2.3: HiveMQ Resilience & Failover Test
**ParentRef:** ORIGINAL: Implement HiveMQ Cluster On-Prem

**As** a Reliability Engineer **I want** documented failover tests **so** cluster resilience is proven.

**Acceptance Criteria:**

- Test scenarios: single node termination, network partition, disk saturation simulation.
- Measure client reconnect time (target <5s p95).
- No data loss during single node failure test.
- Report stored with outcomes & remediation recommendations.

**NFRs:** Tests automatable; repeatable with single script execution.

**Artifacts:** `tests/resilience/hivemq-failover-report.md`.

### US-F1.2.4: HiveMQ Performance Benchmark Baseline
**ParentRef:** ORIGINAL: Implement HiveMQ Cluster On-Prem

**As** a Performance Engineer **I want** baseline throughput & latency metrics **so** future regressions are detectable.

**Acceptance Criteria:**

- Synthetic load at ≥ 1.5× forecast peak published for ≥10 min.
- Collect publish ack latency distribution (p50, p95, p99).
- CPU & memory utilization graph included.
- Capacity headroom recommendation documented.

**NFRs:** Benchmark tool results reproducible ±5%.

**Artifacts:** `reports/perf/hivemq-benchmark.md`.

### US-F1.3.1: Internal PKI & mTLS Certificate Workflow
**ParentRef:** ORIGINAL: Enforce mTLS and RBAC for MQTT

**As** a Security Engineer **I want** an internal CA workflow for client cert issuance **so** MQTT connections are mutually authenticated.

**Acceptance Criteria:**

- Root + intermediate CA created with documented validity periods.
- Automated script issues client cert & private key; revocation process documented.
- Certificate rotation runbook produced.
- Test: expired cert rejected (logged with reason code).

**NFRs:** Keys stored with restricted file permissions; no plaintext key exposure in CI logs.

**Artifacts:** `security/pki/README.md`; issuance script.

### US-F1.3.2: Broker TLS Enforcement
**ParentRef:** ORIGINAL: Enforce mTLS and RBAC for MQTT

**As** a Security Engineer **I want** HiveMQ to require valid client certs **so** unauthorized clients cannot connect.

**Acceptance Criteria:**

- Connection without certificate fails with documented error.
- Valid cert connection success logged with subject DN.
- Config disallows weak cipher suites.
- Automated negative test included in CI (optional job).

**Artifacts:** Config file diff; test script output.

### US-F1.3.3: Topic-Level RBAC Implementation
**ParentRef:** ORIGINAL: Enforce mTLS and RBAC for MQTT

**As** a Security Engineer **I want** ACL definitions mapping roles to topic patterns **so** least privilege is enforced.

**Acceptance Criteria:**

- ACL file denies all by default; explicit allow rules enumerated.
- Role definitions: publisher-equipment, subscriber-analytics, admin.
- Negative publish & subscribe tests blocked and logged.
- Quarterly review reminder documented.

**Artifacts:** `security/mqtt-acl.yaml`; test logs.

### US-F1.3.4: Automated Security Negative Test Suite
**ParentRef:** ORIGINAL: Enforce mTLS and RBAC for MQTT

**As** a Security Engineer **I want** repeatable security test scripts **so** regressions are caught early.

**Acceptance Criteria:**

- Tests attempt: no cert, revoked cert, unauthorized topic publish, oversize payload.
- Produces structured JSON results.
- Executable locally & in CI optional stage.

**Artifacts:** `tests/security/mqtt_pentest.py`.

### US-F1.4.1: Install HiveMQ Event Hubs Extension
**ParentRef:** ORIGINAL: Deploy Event Hubs Bridge Extension

**As** a Data Engineer **I want** Event Hubs extension installed cluster-wide **so** telemetry forwards directly to PaaS.

**Acceptance Criteria:**

- Extension JAR deployed to each node; checksum verified.
- Startup logs show successful initialization on all nodes.
- Version pinned in configuration doc.

**Artifacts:** `infra/hivemq/extensions/eventhubs/`.

### US-F1.4.2: Topic Mapping & Serialization Rules
**ParentRef:** ORIGINAL: Deploy Event Hubs Bridge Extension

**As** a Data Engineer **I want** explicit mapping & serialization config **so** only required UNS topics forward with canonical envelope.

**Acceptance Criteria:**

- Include & exclude patterns documented.
- Payload conforms to envelope fields (messageId, tsUtc, etc.).
- Test harness publishes sample topics; verified in Event Hubs.

**Artifacts:** `config/eventhubs-bridge-mapping.yaml`.

### US-F1.4.3: Bridge Operational Metrics & Alerting
**ParentRef:** ORIGINAL: Deploy Event Hubs Bridge Extension

**As** a Reliability Engineer **I want** forwarding metrics & alerts **so** failures surface within 5 minutes.

**Acceptance Criteria:**

- Metrics: messagesForwarded/sec, errorRate, retryCount exposed.
- Alert when errorRate >1% for 5m.
- Runbook link included in alert body.

**Artifacts:** `monitoring/bridge-metrics.md`; alert definition.

### US-F1.5.1: Outage Buffer Sizing Calculation
**ParentRef:** ORIGINAL: Implement On-Prem Outage Buffering Policy

**As** a Reliability Engineer **I want** a sizing formula **so** disk capacity supports ≥24h outage with safety margin.

**Acceptance Criteria:**

- Formula includes peak msgs/sec, avg payload bytes, compression ratio, safety factor ≥1.3.
- Calculation documented with example numbers.
- Reviewed by Platform & OT.

**Artifacts:** `docs/resilience/buffer-sizing.md`.

### US-F1.5.2: Buffer Persistence & High-Water Alerts
**ParentRef:** ORIGINAL: Implement On-Prem Outage Buffering Policy

**As** a Platform Engineer **I want** persistence configured with utilization alert **so** risk of overflow is mitigated.

**Acceptance Criteria:**

- Storage path isolated from OS volume.
- Alert triggers at 80% utilization.
- Test: synthetic flood drives utilization beyond 80%; alert fired.

**Artifacts:** Persistence config; alert screenshot.

### US-F1.5.3: Offline Replay Simulation & Report
**ParentRef:** ORIGINAL: Implement On-Prem Outage Buffering Policy

**As** a Reliability Engineer **I want** automated disconnect & replay test **so** ordering & completeness are proven.

**Acceptance Criteria:**

- 1h disconnect test executed.
- Replay parity 100% (message count) & ordering preserved per topic.
- Report archived with timestamp & logs.

**Artifacts:** `tests/resilience/offline-replay-report.md`.

### US-F1.6.1: Highbyte Connectors Configuration
**ParentRef:** ORIGINAL: Publish OT Data Models via Highbyte

**As** an OT Engineer **I want** stable Highbyte connectors to Ignition & Rockwell **so** model attributes publish reliably.

**Acceptance Criteria:**

- Auth method documented (service account or cert).
- 24h pilot: zero unexpected disconnects.
- Health metrics captured.

**Artifacts:** `docs/highbyte/connectors.md`.

### US-F1.6.2: Highbyte Model Versioning
**ParentRef:** ORIGINAL: Publish OT Data Models via Highbyte

**As** a Data Engineer **I want** model definitions version-controlled **so** evolution is transparent.

**Acceptance Criteria:**

- JSON definitions include version, sourceSystem, unit metadata.
- Change log updated on every modification.
- Lint check ensures required fields present.

**Artifacts:** `models/highbyte/*.json`.

### US-F1.6.3: Sample Payload Validation Set
**ParentRef:** ORIGINAL: Publish OT Data Models via Highbyte

**As** a QA Engineer **I want** curated sample payloads **so** downstream parsing tests are reproducible.

**Acceptance Criteria:**

- ≥ 20 representative payloads stored.
- Validation script passes (schema conformity).
- Used by upstream CI.

**Artifacts:** `tests/data/highbyte-samples/`.

### US-F1.7.1: Quality Flag Taxonomy Definition
**ParentRef:** ORIGINAL: Implement Data Quality Tagging at Source

**As** a Data Engineer **I want** a unified set of quality flags **so** cleansing logic is consistent.

**Acceptance Criteria:**

- Enum list: GOOD, STALE, SIMULATED, ERROR defined with semantics.
- Invalid flag triggers validation warning.
- Document published at `docs/quality/quality-flags.md`.

**Artifacts:** Taxonomy document.

### US-F1.7.2: Publisher Quality Flag Integration
**ParentRef:** ORIGINAL: Implement Data Quality Tagging at Source

**As** an OT Engineer **I want** publisher configs updated to emit quality flags **so** downstream filtering works.

**Acceptance Criteria:**

- At least one pilot line emits each flag in test mode.
- Bronze ingestion retains field unaltered.
- Random sample validated.

**Artifacts:** Config diffs; ingestion sample capture.

### US-F1.7.3: Quality Distribution Dashboard & Alert
**ParentRef:** ORIGINAL: Implement Data Quality Tagging at Source

**As** a Data Analyst **I want** visibility of quality flag trends **so** issues surface quickly.

**Acceptance Criteria:**

- Dashboard 7‑day distribution view.
- Alert: ERROR >0.5% any 1h window.
- Alert references remediation runbook.

**Artifacts:** Dashboard link; alert definition.

---

## Feature F2: Secure Cloud Ingestion via Event Hubs

**Objective:** Provide secure, private, governed ingestion via Event Hubs with data contract, schema validation, DLQ routing, performance partitioning, managed identity auth & observability. Replaces earlier cloud HiveMQ bridging VM (direct on-prem to PaaS design).

### US-F2.1.1: Event Hubs Namespace IaC
**ParentRef:** ORIGINAL: Provision Event Hubs Namespace with IaC

**As** a Cloud Engineer **I want** Bicep/Terraform for namespace provisioning **so** environments are consistent.

**Acceptance Criteria:**

- Module creates namespace with tagging (env, owner, costCenter).
- `what-if` (or plan) run shows clean diff post-deploy.
- Output variables documented.

**Artifacts:** `infra/eventhubs/main.bicep` (or Terraform equivalent).

### US-F2.1.2: Event Hub Entity & Consumer Groups Module
**ParentRef:** ORIGINAL: Provision Event Hubs Namespace with IaC

**As** a Cloud Engineer **I want** hub entity & consumer groups separated **so** scaling changes avoid namespace redeploy.

**Acceptance Criteria:**

- Hub created with partition count rationale.
- Consumer groups: bronze-ingest, monitoring, dlq-processing.
- Deploy docs updated.

**Artifacts:** `infra/eventhubs/entities.bicep`.

### US-F2.1.3: Private Endpoint & Firewall Enforcement
**ParentRef:** ORIGINAL: Configure Network Security for Event Hubs

**As** a Security Engineer **I want** private networking + least privilege egress allow list **so** attack surface is reduced.

**Acceptance Criteria:**

- Private endpoint created; public network access disabled OR restricted.
- Connectivity test from on-prem passes; unauthorized source blocked.
- Logging captures denied attempt.

**Artifacts:** Network diagram; test log.

### US-F2.2.1: Canonical Envelope Schema v1
**ParentRef:** ORIGINAL: Define Event Schema Envelope

**As** a Data Engineer **I want** a versioned JSON schema **so** downstream parsing is deterministic.

**Acceptance Criteria:**

- Fields: messageId (UUID), tsUtc (ISO8601), topic, value, quality, units, sourceSystem, lineageVersion, schemaVersion.
- Schema published at `schemas/envelope/v1/`.
- Semantic version rules defined.

**Artifacts:** `schemas/envelope/v1/envelope.schema.json`.

### US-F2.2.2: Runtime Schema Validation
**ParentRef:** ORIGINAL: Define Event Schema Envelope

**As** a Developer **I want** validation in forwarding logic **so** malformed payloads are detected pre-ingest.

**Acceptance Criteria:**

- Invalid payload increments error counter.
- Performance overhead <5% baseline.
- Unit tests cover valid & invalid examples.

**Artifacts:** Validation middleware code; test suite.

### US-F2.2.3: Schema Evolution Playbook
**ParentRef:** ORIGINAL: Define Event Schema Envelope

**As** a Data Steward **I want** documented evolution steps **so** compatibility remains predictable.

**Acceptance Criteria:**

- Checklist: impact analysis, additive vs breaking classification, consumer notification process.
- Simulated additive field change executed.

**Artifacts:** `docs/schema/schema-evolution.md`.

### US-F2.3.1: Dead-Letter Architecture Design
**ParentRef:** ORIGINAL: Implement Dead-Letter Strategy in Event Hubs

**As** a Data Engineer **I want** a DLQ design **so** invalid messages are isolated for triage.

**Acceptance Criteria:**

- Diagram: primary hub → processing → DLQ hub or ADLS path.
- Reason taxonomy defined (SCHEMA_INVALID, FIELD_MISSING, SIZE_EXCEEDED, UNKNOWN_FLAG).

**Artifacts:** `docs/ingestion/dlq-design.md`.

### US-F2.3.2: Dead-Letter Routing Implementation
**ParentRef:** ORIGINAL: Implement Dead-Letter Strategy in Event Hubs

**As** a Developer **I want** routing logic **so** invalid payloads bypass main processing.

**Acceptance Criteria:**

- Malformed messages appear in DLQ with reason.
- Main pipeline throughput unaffected.
- Sample triage script extracts examples.

**Artifacts:** Implementation commit; sample DLQ records.

### US-F2.3.3: DLQ Monitoring & Alert
**ParentRef:** ORIGINAL: Implement Dead-Letter Strategy in Event Hubs

**As** a Reliability Engineer **I want** alerting on DLQ rate **so** systemic issues surface quickly.

**Acceptance Criteria:**

- Alert threshold: DLQ > 0.1% of total in 10m window.
- Dashboard trend for DLQ count.
- Runbook link added.

**Artifacts:** Alert definition; dashboard screenshot.

### US-F2.4.1: Partition Key Strategy Justification
**ParentRef:** ORIGINAL: Throughput & Partition Load Test

**As** a Performance Engineer **I want** documented rationale for partition key **so** scaling behavior is predictable.

**Acceptance Criteria:**

- Candidate keys compared (Equipment vs Line vs Site+Equipment hash).
- Selected key with pros/cons & expected cardinality.

**Artifacts:** `docs/ingestion/partition-strategy.md`.

### US-F2.4.2: Load Test Harness Creation
**ParentRef:** ORIGINAL: Throughput & Partition Load Test

**As** a Developer **I want** a reusable load harness **so** future tests are low effort.

**Acceptance Criteria:**

- CLI parameters: rate, duration, key distribution, payload size.
- Metrics JSON output (latency, errors, partition distribution).

**Artifacts:** `tools/loadtest/publisher.py`.

### US-F2.4.3: Load Test Execution & Report
**ParentRef:** ORIGINAL: Throughput & Partition Load Test

**As** a Performance Engineer **I want** baseline results **so** capacity risks are quantified.

**Acceptance Criteria:**

- 1.5× forecast sustained; partition skew <15%.
- Publish→Bronze latency p95 <30s.
- Recommendations list produced.

**Artifacts:** `reports/perf/eventhubs-loadtest.md`.

### US-F2.5.1: Managed Identity Sender Authorization
**ParentRef:** ORIGINAL: Configure Network Security for Event Hubs

**As** a Security Engineer **I want** least privilege send rights **so** unauthorized publishing is blocked.

**Acceptance Criteria:**

- Identity only has Event Hubs Data Sender role on hub scope.
- Negative test identity fails (logged) within 10m.

**Artifacts:** Role assignment export.

### US-F2.5.2: Auth Failure Alerting
**ParentRef:** ORIGINAL: Configure Network Security for Event Hubs

**As** a Security Engineer **I want** alerts on repeated auth failures **so** intrusion attempts surface.

**Acceptance Criteria:**

- Alert triggers after ≥5 failures in 10m.
- Includes source IP / identity in payload.
- Runbook ID referenced.

**Artifacts:** Alert definition.

---

## Feature F3: Fabric Medallion Architecture (Bronze / Silver / Gold)

**Objective:** Establish robust multi-layer processing with schema drift handling, dedupe, quality metrics, optimized aggregations, Direct Lake semantic model, and lineage automation.

### US-F3.1.1: Bronze Streaming Ingestion Notebook
**ParentRef:** ORIGINAL: Bronze Streaming Ingestion Notebook

**As** a Data Engineer **I want** a Structured Streaming job ingesting raw events **so** Bronze layer holds immutable history.

**Acceptance Criteria:**

- Reads from Event Hubs (managed identity or Key Vault secret reference).
- Partitioned by yyyy/mm/dd; checkpointing ensures exactly-once semantics (idempotent downstream).
- Watermark configuration documented.
- Restart test shows no duplicate rows.

**Artifacts:** `notebooks/bronze_ingest.ipynb`; job config.

### US-F3.1.2: Bronze Schema Drift Detection & Catalog
**ParentRef:** ORIGINAL: Bronze Schema Drift Handling

**As** a Data Engineer **I want** automatic capture of unknown fields **so** evolution is observable without breaking ingestion.

**Acceptance Criteria:**

- Unknown fields stored in JSON column `raw_extra`.
- Drift events table: fieldName, firstSeenUtc, sampleTopic.
- Alert on first occurrence per new field (no repeat within 24h).

**Artifacts:** `tables/bronze_drift_events.delta`.

### US-F3.2.1: Silver Parse & Dedupe Job
**ParentRef:** ORIGINAL: Silver Transformation & Conformance

**As** a Data Engineer **I want** parsing & deduplication **so** Silver offers clean conformed data.

**Acceptance Criteria:**

- Topic parsed into Site, Area, Line, Equipment, Tag columns.
- Duplicate detection using messageId or composite key; duplicates removed count exposed.
- Data quality rules (range, null checks) applied & violations logged to QA table.

**Artifacts:** `notebooks/silver_transform.ipynb`.

### US-F3.2.2: Silver Data Quality Metrics Table
**ParentRef:** ORIGINAL: Silver Quality Metrics Table

**As** a Data Analyst **I want** daily aggregated quality metrics **so** reliability trends are visible.

**Acceptance Criteria:**

- Metrics: totalMessages, goodQualityPct, duplicatesRemoved, driftEventsCount.
- Job schedule daily; rerunnable.
- Sample reconciliation ±1% tolerance vs Bronze counts.

**Artifacts:** `tables/silver_quality_metrics.delta`.

### US-F3.3.1: Gold Aggregation Design
**ParentRef:** ORIGINAL: Gold Aggregations for BI & ML

**As** a BI Developer **I want** designed hourly/daily aggregate schema **so** queries are performant & consistent.

**Acceptance Criteria:**

- Aggregation spec doc: grouping keys, measures (avg, min, max, stddev), surrogate keys strategy.
- Performance target: complete 24h data <15m.
- Partitioning/Z-order plan documented.

**Artifacts:** `docs/gold/aggregation-design.md`.

### US-F3.3.2: Direct Lake Semantic Model Publication
**ParentRef:** ORIGINAL: Direct Lake Semantic Model Publication

**As** a BI Developer **I want** a Direct Lake model over Gold **so** reports avoid duplication.

**Acceptance Criteria:**

- Measures: AvgValue, ValueVolatility, UptimePct implemented.
- Row-level security rules (if required) tested.
- Sample DAX validation queries correct.

**Artifacts:** Semantic model definition; validation script.

### US-F3.4.1: Automated Data Lineage & Catalog Entries
**ParentRef:** ORIGINAL: Data Lineage & Catalog Entries

**As** a Data Steward **I want** lineage metadata generated **so** impact analysis is simplified.

**Acceptance Criteria:**

- Metadata table: sourceTopic, bronzePath, silverTable, goldTable, transformationVersion updated each run.
- Query returns lineage for sample messageId.
- Linked in governance docs.

**Artifacts:** `tables/metadata_lineage.delta`.

---

## Feature F4: Model Development & Feature Engineering

**Objective:** Internalize vendor (ModelQ) logic, perform EDA, create versioned feature tables, track experiments, optimize hyperparameters, and formalize drift monitoring design.

### US-F4.1.1: Initial EDA Profiling
**ParentRef:** ORIGINAL: EDA Notebook on Gold Data

**As** a Data Scientist **I want** an EDA notebook profiling distributions & correlations **so** I validate feature hypotheses.

**Acceptance Criteria:**

- Outputs: summary stats, correlation heatmap, missingness matrix.
- Risks & anomalies documented in README.
- Reviewed & approved by Data Science Lead.

**Artifacts:** `notebooks/eda_initial.ipynb`; `docs/ml/eda-findings.md`.

### US-F4.1.2: Vendor Feature Mapping Documentation
**ParentRef:** ORIGINAL: Reproduce Vendor Feature Engineering

**As** a Data Scientist **I want** a parity mapping between vendor features & internal versions **so** we ensure functional equivalence.

**Acceptance Criteria:**

- Table: vendorFeature, internalFeature, formula/logic, status.
- ≥ 95% parity achieved or rationale for differences.
- Stored & versioned.

**Artifacts:** `docs/ml/vendor_feature_mapping.md`.

### US-F4.2.1: Baseline MLflow Logging
**ParentRef:** ORIGINAL: MLflow Experiment Tracking Integration

**As** a Data Scientist **I want** unified MLflow logging **so** experiments are auditable.

**Acceptance Criteria:**

- Autolog + custom metrics (precision, recall, MAE, latency) captured.
- Git commit hash & feature table version logged.
- Experiment comparison view accessible (documented link).

**Artifacts:** Training notebook diff; MLflow experiment ID.

### US-F4.2.2: Hyperparameter Optimization Workflow
**ParentRef:** ORIGINAL: Hyperparameter Optimization Workflow

**As** a Data Scientist **I want** automated HPO (e.g., Hyperopt) **so** best configuration is systematically found.

**Acceptance Criteria:**

- Search space documented (learning rate, regularization, window size etc.).
- ≥ 30 trials executed; convergence plot stored.
- Best params improve primary metric ≥ defined threshold over baseline.

**Artifacts:** `notebooks/hpo_run.ipynb`; convergence plot.

### US-F4.3.1: Model Validation & Registration
**ParentRef:** ORIGINAL: Model Validation & Registration

**As** a Data Scientist **I want** validated model registered **so** deployment can proceed.

**Acceptance Criteria:**

- Holdout metrics meet threshold (document threshold values).
- Bias / fairness placeholder check executed (log produced).
- Model stage moved to Staging with tags (trainingDataVersion, featureTableVersion, gitSHA).

**Artifacts:** MLflow model registry entry; validation report.

### US-F4.4.1: Data & Concept Drift Monitoring Design
**ParentRef:** ORIGINAL: Data & Concept Drift Monitoring Design

**As** a Data Scientist **I want** drift metrics specification **so** production monitoring build is clear.

**Acceptance Criteria:**

- Metrics: PSI, mean shift, variance shift, missing rate delta defined.
- Sampling frequency & baseline window documented.
- Alert thresholds proposed & reviewed with Ops.

**Artifacts:** `docs/ml/drift-monitoring-design.md`.

---

## Feature F5: On-Prem Inference Container & IoT Edge Deployment

**Objective:** Provide secure, observable, low-latency model inference container deployed via IoT Edge to on-prem Kubernetes with offline autonomy, safe rollout & rollback.

### US-F5.1.1: Inference API Contract
**ParentRef:** ORIGINAL: Define Inference API Contract

**As** an MLOps Engineer **I want** an OpenAPI contract **so** integrations are predictable.

**Acceptance Criteria:**

- Defines /predict (request schema: features[], timestamps) & /health endpoints.
- Error codes & semantics documented.
- Stored in repo; version tagged with model version.

**Artifacts:** `api/inference_openapi.yaml`.

### US-F5.1.2: FastAPI Inference Service Implementation
**ParentRef:** ORIGINAL: Implement FastAPI Inference Service

**As** a Developer **I want** a service loading MLflow model **so** predictions are served with low latency.

**Acceptance Criteria:**

- Cold start <5s on target hardware.
- 50 req/sec load p95 latency <200ms.
- Health endpoint returns model version & readiness.
- Unit tests: success + input validation error path.

**Artifacts:** `src/inference_service/` code; test suite.

### US-F5.2.1: Containerization & Multi-Stage Build
**ParentRef:** ORIGINAL: Containerize Inference Service

**As** an MLOps Engineer **I want** a minimal hardened image **so** deployment is secure & efficient.

**Acceptance Criteria:**

- Multi-stage Dockerfile; final image <400MB.
- Non-root user executes process.
- SBOM generated (e.g., Syft) & stored.
- Vulnerability scan: no critical CVEs.

**Artifacts:** `Dockerfile`; SBOM file; scan report.

### US-F5.2.2: Offline Operation Strategy
**ParentRef:** ORIGINAL: Design Offline Operation Strategy

**As** a Reliability Engineer **I want** documented offline behaviors **so** inference persists through outages.

**Acceptance Criteria:**

- Strategy covers local queue for telemetry, periodic flush on reconnect.
- 4h disconnect simulation: zero inference failures.
- Recovery flush preserves event ordering.

**Artifacts:** `docs/resilience/offline-inference.md`; test log.

### US-F5.3.1: IoT Edge Deployment Manifest Creation
**ParentRef:** ORIGINAL: IoT Edge Deployment Manifest Creation

**As** an MLOps Engineer **I want** versioned deployment manifest **so** rollout is controlled.

**Acceptance Criteria:**

- Manifest defines module image, environment vars, restart policy.
- Lint/validation passes (schema check).
- Image tag includes model version + git SHA.

**Artifacts:** `deploy/iotedge/deployment.json`.

### US-F5.3.2: Shadow / Canary Deployment Process
**ParentRef:** ORIGINAL: Shadow Deployment (Canary) Process

**As** a Release Manager **I want** a dual-run comparison procedure **so** new model safety is assured.

**Acceptance Criteria:**

- Shadow service runs in parallel capturing predictions (no user impact).
- Divergence metric (e.g., mean absolute difference) computed.
- Go/No-Go checklist template created.

**Artifacts:** `docs/deployment/canary-process.md`; comparison script.

---

## Feature F6: CI/CD & Automation

**Objective:** Implement standardized branching, quality gates, secure supply chain, automated build/test, container publish, controlled deployment & smoke validation aligned to DevSecOps.

### US-F6.1.1: Branching & Versioning Policy
**ParentRef:** ORIGINAL: Git Branching & Versioning Policy

**As** a Dev Lead **I want** documented branching & semantic version mapping **so** collaboration is consistent.

**Acceptance Criteria:**

- Strategy: main, develop, feature/*, release/*, hotfix/* defined.
- SemVer mapped to model registry versions.
- Commit message conventions documented (feat:, fix:, chore:).

**Artifacts:** `docs/dev/branching-policy.md`.

### US-F6.1.2: CI Workflow (Tests & Lint)
**ParentRef:** ORIGINAL: CI Workflow for Build & Test

**As** a DevOps Engineer **I want** automated test/lint pipeline **so** code merges meet quality gate.

**Acceptance Criteria:**

- Triggers on PR to main/develop.
- Runs unit tests, style (flake8/black or equivalent), security scan (pip audit/trivy).
- Fails on high severity issues.
- Status badge added to README.

**Artifacts:** `.github/workflows/ci.yml`; README badge.

### US-F6.2.1: Container Build & Push Pipeline
**ParentRef:** ORIGINAL: Container Build & Push Pipeline

**As** a DevOps Engineer **I want** automated image build & push on tag **so** artifacts are immutable & traceable.

**Acceptance Criteria:**

- Tag push triggers build using model version build arg.
- SBOM & vulnerability scan artifacts uploaded.
- Image digest recorded in release notes.

**Artifacts:** `.github/workflows/build.yml`; release notes template.

### US-F6.3.1: Automated IoT Edge Deployment Update
**ParentRef:** ORIGINAL: Automated IoT Edge Deployment Update

**As** a DevOps Engineer **I want** CD job with approval gate **so** production rollout is controlled.

**Acceptance Criteria:**

- Manual approval required before prod environment deploying.
- Manifest automatically updated with new image tag.
- Audit log records user, timestamp, version.

**Artifacts:** `.github/workflows/cd.yml`; audit log location doc.

### US-F6.3.2: Post-Deployment Smoke Test Script
**ParentRef:** ORIGINAL: Post-Deployment Smoke Test Script

**As** a QA Engineer **I want** automated smoke test **so** deployment validation is fast.

**Acceptance Criteria:**

- Runs /health and sample /predict.
- Fails pipeline if non-200 or latency > baseline +10%.
- Logs archived with run ID.

**Artifacts:** `tests/smoke/smoke_test.py`; pipeline step.

---

## Feature F7: Monitoring, Observability & Drift

**Objective:** Provide end-to-end observability (infrastructure, pipeline, data quality, model performance & drift, capacity) aligned to golden signals & MLOps best practices.

### US-F7.1.1: Observability Metrics Catalog
**ParentRef:** ORIGINAL: Define Observability Metrics Catalog

**As** a Reliability Engineer **I want** documented metrics **so** stakeholders know what is tracked & why.

**Acceptance Criteria:**

- Sections: Infrastructure, Pipeline, Data Quality, Model, Business KPIs.
- Each metric lists source, frequency, threshold, owner.
- Approved & versioned.

**Artifacts:** `docs/observability/metrics-catalog.md`.

### US-F7.2.1: Inference Metrics Emission
**ParentRef:** ORIGINAL: Inference Metrics Emission

**As** a Developer **I want** structured logs & metrics from inference **so** performance & errors visible.

**Acceptance Criteria:**

- JSON logs: traceId, requestId, latencyMs, status, modelVersion.
- Prometheus (or equivalent) endpoint: counters & histograms.
- Sample log ingestion test passes.

**Artifacts:** Logging config; metrics endpoint doc.

### US-F7.3.1: Model Performance Dashboard
**ParentRef:** ORIGINAL: Model Performance Dashboard

**As** a Data Scientist **I want** dashboard of MAE, latency, prediction distribution **so** degradation detectable.

**Acceptance Criteria:**

- Threshold breach surfaces alert.
- 30-day trend visual present.
- Drill-down for outlier days.

**Artifacts:** Dashboard link; alert config.

### US-F7.4.1: Drift Detection Job Implementation
**ParentRef:** ORIGINAL: Drift Detection Job Implementation

**As** a Data Scientist **I want** scheduled drift calculations **so** shifts are quantified.

**Acceptance Criteria:**

- PSI & mean/std deltas computed vs baseline.
- Results persisted to drift_results table.
- Alert when PSI > threshold.

**Artifacts:** Job notebook/script; drift table.

### US-F7.5.1: Fabric Capacity Metrics Repair & Alerting
**ParentRef:** ORIGINAL: Fabric Capacity Metrics Repair & Alerting

**As** a Platform Engineer **I want** functioning capacity metrics & alerts **so** resource saturation prevented.

**Acceptance Criteria:**

- Capacity tiles display expected data.
- Test query validates daily refresh.
- Alert on CPU or memory >80% for 10m window.

**Artifacts:** Dashboard screenshot; alert JSON.

---

## Feature F8: Security, Compliance & Access Control

**Objective:** Enforce least privilege, secretless auth, audit coverage, retention governance in line with Zero Trust & compliance readiness.

### US-F8.1.1: Role & Access Matrix
**ParentRef:** ORIGINAL: Access Matrix & Role Definitions

**As** a Security Analyst **I want** RACI + role permission matrix **so** access reviews are simplified.

**Acceptance Criteria:**

- Roles: OT Engineer, Data Engineer, Data Scientist, MLOps, Viewer defined with resource scopes.
- Approved matrix stored in repo.
- Quarterly review process documented.

**Artifacts:** `docs/security/access-matrix.md`.

### US-F8.2.1: Secret Inventory & Managed Identity Plan
**ParentRef:** ORIGINAL: Managed Identity & Secret Removal

**As** a Cloud Engineer **I want** inventory & migration plan **so** static secrets are eliminated.

**Acceptance Criteria:**

- Inventory table of existing secrets (location, purpose, owner).
- Plan to replace each with managed identity or Key Vault reference.
- Post-migration diff shows zero hard-coded credentials.

**Artifacts:** Inventory sheet; migration report.

### US-F8.3.1: Audit Logging Coverage Assessment
**ParentRef:** ORIGINAL: Audit Logging Coverage Assessment

**As** a Security Engineer **I want** mapping of critical actions to logs **so** investigations are feasible.

**Acceptance Criteria:**

- Actions list (deploy, model promote, ACL change) with log source mapping.
- Gap list & remediation story IDs created.
- Sample incident replay traces all required actions.

**Artifacts:** Coverage matrix; incident replay script output.

### US-F8.4.1: Data Retention Policy Definition
**ParentRef:** ORIGINAL: Data Retention & Purge Policy

**As** a Compliance Officer **I want** retention & purge policy **so** storage & regulatory needs balanced.

**Acceptance Criteria:**

- Retention per layer (Bronze, Silver, Gold, Logs) defined with timeframe & rationale.
- Purge job specification drafted.
- Risk assessment approved.

**Artifacts:** `docs/governance/retention-policy.md`.

---

## Feature F9: Data Governance & Catalog Enablement

**Objective:** Improve discoverability & trust via glossary, dictionary, lineage diagrams, metadata foundation & future catalog readiness.

### US-F9.1.1: Initial Business Glossary
**ParentRef:** ORIGINAL: Business Glossary Creation

**As** a Data Steward **I want** glossary of key terms **so** stakeholders share consistent definitions.

**Acceptance Criteria:**

- Terms: OEE, UptimePct, MoistureLevel, DriftEvent entries with owner.
- Approved & published.
- Linked from README.

**Artifacts:** `docs/governance/glossary.md`.

### US-F9.2.1: Gold Table Data Dictionary
**ParentRef:** ORIGINAL: Data Dictionary for Gold Tables

**As** a BI Developer **I want** dictionary of Gold columns **so** report authors interpret fields correctly.

**Acceptance Criteria:**

- Each column: name, type, description, derivation, unit.
- CI lints missing descriptions.

**Artifacts:** `docs/governance/gold-data-dictionary.md`.

### US-F9.3.1: Lineage Diagram Artifact
**ParentRef:** ORIGINAL: Lineage Diagram Artifact

**As** a Data Steward **I want** an updated lineage diagram **so** onboarding is faster.

**Acceptance Criteria:**

- Diagram: OT → HiveMQ → Event Hubs → Bronze → Silver → Gold → Model → Inference.
- Version/date stamped.
- Linked from wiki.

**Artifacts:** `diagrams/lineage.png` + source file.

---

## Feature F10: Reporting & Executive Analytics Enablement

**Objective:** Provide KPI definitions, direct lake dashboards & governance transparency without compromising offline resilience.

### US-F10.1.1: KPI Definition Workshop Output
**ParentRef:** ORIGINAL: KPI Definition Workshop Output

**As** a Product Owner **I want** codified KPI specs **so** dashboards reflect agreed metrics.

**Acceptance Criteria:**

- KPIs: calculation formula, owner, refresh cadence documented.
- Alignment with semantic model measures.
- Signed off by stakeholders.

**Artifacts:** `docs/analytics/kpi-spec.md`.

### US-F10.2.1: Executive Operations Dashboard
**ParentRef:** ORIGINAL: Executive Operations Dashboard

**As** an Executive **I want** high-level dashboard (throughput, downtime, predicted risk alerts) **so** plant health is quickly assessed.

**Acceptance Criteria:**

- Direct Lake; no duplicate extracts.
- Traffic-light status thresholds documented.
- Load time <5s p95.

**Artifacts:** Power BI report file; performance test log.

### US-F10.3.1: Model Governance Report
**ParentRef:** ORIGINAL: Model Governance Report

**As** a Data Science Lead **I want** governance report summarizing versions & drift **so** compliance & oversight are transparent.

**Acceptance Criteria:**

- Shows prod model version, last retrain date, drift status.
- Auto-refresh daily.
- Flags overdue retrain (threshold defined).

**Artifacts:** Governance report artifact; refresh config.

---

## Feature F11: Training, Runbooks & Operational Readiness

**Objective:** Ensure fast incident response, predictable retraining, and efficient onboarding.

### US-F11.1.1: Operations Runbook Draft Set
**ParentRef:** ORIGINAL: Operations Runbook Set

**As** a Support Engineer **I want** runbooks for common incidents **so** MTTR is minimized.

**Acceptance Criteria:**

- Runbooks: Broker outage, Event Hubs throttle, Drift alert, Model rollback.
- Each: detection, diagnosis, rollback, escalation, metrics to watch.
- Indexed in TOC file.

**Artifacts:** `runbooks/*.md`; index doc.

### US-F11.2.1: Model Retraining Playbook
**ParentRef:** ORIGINAL: Model Retraining Playbook

**As** a Data Scientist **I want** a documented retraining process **so** updates are consistent.

**Acceptance Criteria:**

- Trigger criteria (drift thresholds, data volume Δ) defined.
- Steps: feature table freeze, experiment log, validation, promotion.
- Checklist template stored.

**Artifacts:** `docs/ml/retraining-playbook.md`.

### US-F11.3.1: Onboarding Guide
**ParentRef:** ORIGINAL: Onboarding Guide

**As** a New Team Member **I want** concise onboarding **so** I am productive within 1 week.

**Acceptance Criteria:**

- Sections: architecture overview, dev environment setup, data access, key dashboards.
- Reviewed by recent hire; feedback incorporated.

**Artifacts:** `docs/onboarding/guide.md`.

---

## Feature F12: Performance & Cost Optimization

**Objective:** Sustain economic efficiency via storage lifecycle management & job performance baselines.

### US-F12.1.1: Storage Tiering Strategy
**ParentRef:** ORIGINAL: Storage Tiering Strategy

**As** a Platform Engineer **I want** a tiering strategy **so** older Bronze data cost is minimized.

**Acceptance Criteria:**

- Hot vs Cool vs Archive thresholds defined with rationale.
- Cost savings forecast produced.
- Implementation tasks created for automation.

**Artifacts:** `docs/cost/storage-tiering.md`.

### US-F12.2.1: Fabric Job Performance Baselines
**ParentRef:** ORIGINAL: Fabric Job Performance Baselines

**As** a Data Engineer **I want** runtime baselines **so** regressions trigger investigation.

**Acceptance Criteria:**

- Baseline metrics captured (runtime, input rows/sec) for core jobs.
- Regression alert: runtime > +20% baseline for 3 consecutive runs.
- Baseline report versioned.

**Artifacts:** `reports/perf/job-baselines.md`.

---

## Feature F13: Future Enablement (Placeholders)

**Objective:** Preserve strategic optionality (Local, Purview integration) while deferring investment.

### US-F13.1.1: Evaluate Local Option
**ParentRef:** ORIGINAL: Evaluate Local Option for Future OT Expansion

**As** an Architect **I want** feasibility evaluation of Local **so** future hybrid improvements are informed (without altering current offline design).

**Acceptance Criteria:**

- Comparison matrix (current Kubernetes + IoT Edge vs Local) with criteria (latency, management overhead, resilience impact).
- Risks & benefits summarized.
- Decision record stored.

**Artifacts:** `docs/architecture/azure-local-eval.md`.

### US-F13.2.1: Plan Purview Integration (Deferred)
**ParentRef:** ORIGINAL: Plan Purview Integration (Deferred)

**As** a Data Governance Lead **I want** readiness assessment for Purview **so** future catalog adoption is smooth.

**Acceptance Criteria:**

- Gap analysis (current metadata vs Purview requirements).
- Phased integration steps outlined.
- Deferred tag with revisit date.

**Artifacts:** `docs/governance/purview-readiness.md`.

---

## References & Best Practice Inputs (Non-Exhaustive)

Summarized (paraphrased) sources informing design; no direct copyrighted text reproduced:

- ISA‑95 / Unified Namespace pattern (structural hierarchies for industrial data contextualization).
- Event Hubs guidance (partition key strategy, private networking, consumer groups segregation, throughput performance tuning).
- HiveMQ clustering & security recommendations (mTLS, extension-based forwarding, operational metrics exposure).
- IoT Edge deployment & offline resilience practices (manifest versioning, module health, local queueing patterns).
- MLflow experiment tracking & model registry usage patterns (reproducibility tags, stage transitions).
- SRE Golden Signals & metric taxonomy (latency, traffic, errors, saturation) + extension for ML (drift, data quality).
- Drift detection literature (Population Stability Index, distribution shift metrics, scheduled monitoring windows).
- DevSecOps & supply chain security (SBOM generation, vulnerability scanning, least privilege, secretless design via managed identity).
- Medallion architecture (layered refinement: raw → conformed → aggregated) with Delta format & Direct Lake performance patterns.

---

End of backlog (Rev 3).

---

## Feature F1: On-Prem MQTT & Unified Namespace Foundation

**Description:** Establish a resilient on‑prem HiveMQ cluster with governed Unified Namespace (UNS) enabling structured OT data publication, secure direct bridge to Event Hubs (no intermediate cloud HiveMQ VM), outage buffering, and data quality tagging at source. Aligns to ISA‑95 + emerging UNS conventions. Implements secure-by-default, observable, repeatable infrastructure as code.

### US-F1.1.1: UNS Topic Hierarchy Draft (ParentRef: ORIGINAL:Design Unified Namespace Standard)
**As** an OT Architect **I want** an initial draft of the ISA‑95 aligned UNS hierarchy **so** stakeholders can review structure early.
**Rationale:** Early draft accelerates feedback; reduces refactor cost.
**Acceptance Criteria:**
- Draft includes: Enterprise/Site/Area/Line/Cell/Equipment/Tag with naming regex & example instances.
- Covers at least 3 representative production lines with sample topics.
- Open issues list (gaps, ambiguities) appended.
**NFRs:** Markdown rendered cleanly; readable on dark/light themes.
**Artifacts:** `docs/uns/UNS-hierarchy-draft.md`.

### US-F1.1.2: UNS Governance & Versioning Policy
**As** a Data Steward **I want** a change control & semantic versioning scheme for UNS **so** downstream consumers can manage schema evolution.
**Acceptance Criteria:**
- Policy defines: version bump rules (MAJOR=breaking, MINOR=additive), deprecation window, approval roles.
- PR template updated with UNS change checklist.
- Example simulated change (add new Tag level) processed end‑to‑end.
**Artifacts:** `docs/uns/UNS-governance.md`.

### US-F1.1.3: UNS Topic Validation Script
**As** a Data Engineer **I want** a CLI script validating topic strings **so** publishers catch format errors pre-publish.
**Acceptance Criteria:**
- Script (Python/Node) validates naming regex & length limits.
- CI job executes against sample publisher config—fails on invalid topics.
- Returns structured JSON (isValid, violations[]).
**Artifacts:** `tools/uns_validate.py` (or `.js`).

### US-F1.2.1: HiveMQ Cluster Provisioning (ParentRef: ORIGINAL:Implement HiveMQ Cluster On-Prem)
**As** an OT Engineer **I want** an automated method to provision a 3‑node HiveMQ cluster **so** deployments are reproducible.
**Acceptance Criteria:**
- Infrastructure defined via Ansible or Terraform module (variables: node_count, heap_size, data_path).
- Idempotent re-run with no drift.
- Nodes register with cluster; cluster health endpoint green.
**NFRs:** Provisioning <30m; scripts documented.
**Artifacts:** `infra/hivemq/` module.

### US-F1.2.2: HiveMQ Configuration as Code
**As** a Platform Engineer **I want** broker configs (listeners, persistence, auth extension settings) stored in version control **so** changes are auditable.
**Acceptance Criteria:**
- Config templates parameterized; secrets externalized.
- Config checksum logged at startup.
- Diff script highlights config drift.
**Artifacts:** `infra/hivemq/config/`.

### US-F1.2.3: HiveMQ Resilience / Failover Test
**As** a Reliability Engineer **I want** documented failover scenarios **so** cluster resilience is proven.
**Acceptance Criteria:**
- Test plan covers: single node kill, network partition, disk fill simulation.
- Metrics: message continuity, client reconnect time <5s.
- Report stored & linked.
**Artifacts:** `tests/resilience/hivemq-failover-report.md`.

### US-F1.2.4: HiveMQ Performance Benchmark
**As** a Performance Engineer **I want** baseline throughput & latency metrics **so** capacity planning is data-driven.
**Acceptance Criteria:**
- Synthetic publisher pushes > 1.5× forecast peak for 10 mins.
- 95th percentile publish-to-ack latency recorded.
- CPU/memory vs load graph included.
**Artifacts:** `reports/perf/hivemq-benchmark.md`.

### US-F1.3.1: MQTT mTLS PKI Setup (ParentRef: ORIGINAL:Enforce mTLS and RBAC for MQTT)
**As** a Security Engineer **I want** an internal CA & issuance workflow **so** client certs are trusted and revocable.
**Acceptance Criteria:**
- Root & intermediate certs generated; procedures documented.
- Automated script issues client cert with SAN entries.
- Revocation list update process defined.
**Artifacts:** `security/pki/README.md`.

### US-F1.3.2: MQTT Broker TLS Enforcement
**As** a Security Engineer **I want** HiveMQ configured to require client certs **so** unauthorized connections fail.
**Acceptance Criteria:**
- Attempt without certificate rejected (logged with reason).
- Accepted connection shows client DN extraction.
**Artifacts:** Config diff & test logs.

### US-F1.3.3: Topic-Level RBAC Rules
**As** a Security Engineer **I want** ACL definitions mapping roles to topic patterns **so** least privilege is enforced.
**Acceptance Criteria:**
- ACL file / extension lists roles, includes deny-by-default.
- Negative test: unauthorized publish blocked.
- Quarterly review reminder added to security calendar (noted in doc).
**Artifacts:** `security/mqtt-acl.yaml`.

### US-F1.3.4: Security Pen Test Script (MQTT)
**As** a Security Engineer **I want** automated negative test scripts **so** regressions are caught early.
**Acceptance Criteria:**
- Script attempts: no cert, expired cert, unauthorized topic.
- CI optional job runs on demand.
**Artifacts:** `tests/security/mqtt_pentest.py`.

### US-F1.4.1: Event Hubs Bridge Extension Install (ParentRef: ORIGINAL:Deploy Event Hubs Bridge Extension)
**As** a Data Engineer **I want** the HiveMQ Event Hubs extension installed across nodes **so** forwarding can occur.
**Acceptance Criteria:**
- Extension jar present & checksum verified.
- Startup logs show successful load on all nodes.
**Artifacts:** `infra/hivemq/extensions/eventhubs/`.

### US-F1.4.2: Bridge Topic Mapping & Serialization
**As** a Data Engineer **I want** explicit mapping rules from UNS topics to Event Hubs **so** only required data egresses.
**Acceptance Criteria:**
- Mapping config supports inclusive patterns & explicit excludes.
- Serialization documented (JSON with canonical envelope fields).
- Test harness publishes sample; verify Event Hubs payload fields.
**Artifacts:** `config/eventhubs-bridge-mapping.yaml`.

### US-F1.4.3: Bridge Operational Monitoring
**As** a Reliability Engineer **I want** metrics & alerts on bridge success/error counts **so** failures are surfaced within 5 minutes.
**Acceptance Criteria:**
- Metrics: messagesForwarded/sec, errorRate, retryCount.
- Alert threshold: errorRate >1% for 5m.
- Runbook link embedded in alert description.
**Artifacts:** `monitoring/bridge-metrics.md`.

### US-F1.5.1: Buffer Sizing Calculation (ParentRef: ORIGINAL:Implement On-Prem Outage Buffering Policy)
**As** a Reliability Engineer **I want** a documented sizing formula **so** disk provisioning covers 24h outage with safety margin.
**Acceptance Criteria:**
- Formula includes: peak msgs/sec, avg payload bytes, compression ratio (if any), safety factor ≥1.3.
- Resulting size vs available disk capacity compared.
**Artifacts:** `docs/resilience/buffer-sizing.xlsx` (or md summary).

### US-F1.5.2: Buffer Implementation & Configuration
**As** a Platform Engineer **I want** broker persistence & queue retention configured **so** messages survive outage.
**Acceptance Criteria:**
- Persistence path segregated from OS volume.
- High-water mark alert at 80% utilization.
- Configuration tested with synthetic flood.
**Artifacts:** Config snippet & monitoring alert definition.

### US-F1.5.3: Offline Replay Test Procedure
**As** a Reliability Engineer **I want** a repeatable disconnect simulation **so** replay ordering & completeness are verified.
**Acceptance Criteria:**
- 1h forced disconnect test executed; message count parity = 100%.
- Ordering check script validates monotonic timestamps per topic.
- Report archived.
**Artifacts:** `tests/resilience/offline-replay-report.md`.

### US-F1.6.1: Highbyte Connector Configuration (ParentRef: ORIGINAL:Publish OT Data Models via Highbyte)
**As** an OT Engineer **I want** stable Highbyte connections to Ignition & Rockwell **so** model data flows reliably.
**Acceptance Criteria:**
- Auth method documented (service account / certs).
- Connection health dashboard shows green for 24h pilot.
**Artifacts:** `docs/highbyte/connectors.md`.

### US-F1.6.2: Highbyte Model Definition & Versioning
**As** a Data Engineer **I want** model JSON definitions version-controlled **so** changes are traceable.
**Acceptance Criteria:**
- Models include metadata: version, sourceSystem, unit.
- Change log appended on updates.
**Artifacts:** `models/highbyte/*.json`.

### US-F1.6.3: Model Payload Validation Sample Set
**As** a QA Engineer **I want** a curated sample of messages **so** downstream transformation tests are reproducible.
**Acceptance Criteria:**
- ≥20 diverse sample payloads stored.
- Schema validation script returns success.
**Artifacts:** `tests/data/highbyte-samples/`.

### US-F1.7.1: Quality Tag Schema Extension (ParentRef: ORIGINAL:Implement Data Quality Tagging at Source)
**As** a Data Engineer **I want** unified quality flags spec **so** all publishers align (GOOD, STALE, SIMULATED, ERROR).
**Acceptance Criteria:**
- Enum list + semantic meaning documented.
- Invalid flag triggers validation warning.
**Artifacts:** `docs/quality/quality-flags.md`.

### US-F1.7.2: Publisher Quality Flag Integration
**As** an OT Engineer **I want** publisher configs updated to emit quality flags **so** data cleansing can filter properly.
**Acceptance Criteria:**
- At least one line’s tags emit every flag in test mode.
- Bronze ingestion preserves field.
**Artifacts:** Change commit & ingestion sample.

### US-F1.7.3: Quality Distribution Monitoring
**As** a Data Analyst **I want** a dashboard of quality flag distribution **so** anomalies are quickly spotted.
**Acceptance Criteria:**
- 7‑day trend chart produced.
- Alert if ERROR >0.5% in any 1h window.
**Artifacts:** Dashboard link & alert config.

### User Story: Implement HiveMQ Cluster On-Prem

**Description:** As an OT Engineer, I want a multi-node HiveMQ cluster deployed on existing plant virtual infrastructure so there is no single point of failure during outages.

**Acceptance Criteria:**

- Minimum 3-node cluster operational; node loss tolerance validated by failover test.
- Cluster configuration stored as code (automated install script or Ansible/Terraform module readme).
- Sustained throughput test meets projected msg/sec with <5% packet loss.
- Monitoring endpoints (JMX or REST) enabled for metrics.

### User Story: Enforce mTLS and RBAC for MQTT

**Description:** As a Security Engineer, I want all MQTT publishers/subscribers to use mutual TLS and be authorized per topic patterns so only approved systems can access data.

**Acceptance Criteria:**

- Internal CA and certificate issuance process documented.
- Broker requires client certs; unauthorized client rejected in test.
- Role/ACL file (or extension) maps principals to topic wildcards.
- Pen test script demonstrates blocked unauthorized publish & subscribe.

### User Story: Deploy Event Hubs Bridge Extension

**Description:** As a Data Engineer, I want the HiveMQ Event Hubs extension configured to forward selected UNS topics directly to Event Hubs so cloud ingestion is PaaS-based (no cloud HiveMQ).

**Acceptance Criteria:**

- Extension installed on all nodes; logs show successful load.
- Event Hub namespace + hub name, partitions, consumer groups configured.
- Mapping file defines topic filters & serialization (e.g., JSON payload).
- Test messages appear in Event Hubs with correct properties.

### User Story: Implement On-Prem Outage Buffering Policy

**Description:** As a Reliability Engineer, I want disk-based buffering sized and tested so a minimum 24h internet outage produces zero data loss.

**Acceptance Criteria:**

- Buffer size calculation documented (throughput × retention).
- Controlled disconnect test (>=1 hr) performed: data replayed in order after reconnection.
- Message count parity (on-prem vs Event Hubs) report produced.
- Alert raised if buffer utilization >80%.

### User Story: Publish OT Data Models via Highbyte

**Description:** As an OT Engineer, I want Highbyte models for Ignition and Rockwell tags mapped to UNS topics so semantic context travels with each message.

**Acceptance Criteria:**

- Connectors for Ignition & Rockwell configured & authenticated.
- Pilot equipment model attributes (timestamp, value, quality, unit) emitted.
- Random sample of 20 messages validated for schema conformity.
- Model versioning strategy documented.

### User Story: Implement Data Quality Tagging at Source

**Description:** As a Data Engineer, I want publishers to add quality/status flags (e.g., GOOD, STALE, SIMULATED) so downstream cleansing can filter invalid readings.

**Acceptance Criteria:**

- Extended payload schema includes quality field & optional error code.
- Simulator publishes all quality states for test.
- Bronze layer ingestion stores quality without loss.
- Dashboard shows distribution of quality states for pilot period.

---

## Feature F2: Secure Cloud Ingestion via Event Hubs

**Description:** Provision secure, private Event Hubs ingestion path (no cloud MQTT broker) with data contract governance, validation, dead-letter routing, performance & partition strategy, and Infrastructure-as-Code across environments.

### US-F2.1.1: Event Hubs IaC Base Namespace (ParentRef: ORIGINAL:Provision Event Hubs Namespace with IaC)
**As** a Cloud Engineer **I want** baseline Bicep/Terraform creating namespace & resource group tags **so** deployments are repeatable.
**Acceptance Criteria:**
- Module exports outputs (namespaceName, hubName, connectionStringRef if needed via Key Vault reference) with proper tags.
- `what-if` run shows zero unintended changes on reapply.
**Artifacts:** `infra/eventhubs/main.bicep`.

### US-F2.1.2: Event Hub Entity & Consumer Groups
**As** a Cloud Engineer **I want** hub + partition + consumer groups creation separated **so** scaling changes don’t redeploy namespace.
**Acceptance Criteria:**
- Consumer groups: bronze-ingest, monitoring, dlq-processing.
- Partition count documented with sizing rationale.
**Artifacts:** `infra/eventhubs/entities.bicep`.

### US-F2.1.3: Private Networking & Firewall Rules
**As** a Security Engineer **I want** private endpoint & limited firewall rules **so** ingestion surface is minimized.
**Acceptance Criteria:**
- No public network access (deny by default) OR explicit allowed list.
- Connectivity test from on-prem egress passes.
**Artifacts:** Deployment outputs & test log.

### US-F2.2.1: Canonical Envelope Schema Definition (ParentRef: ORIGINAL:Define Event Schema Envelope)
**As** a Data Engineer **I want** a versioned JSON schema **so** downstream parsing is deterministic.
**Acceptance Criteria:**
- Fields: messageId (UUID), tsUtc (ISO8601), topic, value, quality, units, sourceSystem, lineageVersion, schemaVersion.
- Avro/JSON Schema definition committed.
- Schema version bump rules documented.
**Artifacts:** `schemas/envelope/v1/envelope.schema.json`.

### US-F2.2.2: Schema Validation Library Integration
**As** a Developer **I want** validation middleware in the bridge **so** malformed messages are consistently detected.
**Acceptance Criteria:**
- Invalid payload increments error counter.
- Performance overhead <5% vs baseline.
**Artifacts:** Code diff & perf report.

### US-F2.2.3: Schema Evolution Playbook
**As** a Data Steward **I want** a playbook for schema changes **so** consumer breakage risk is minimized.
**Acceptance Criteria:**
- Checklist includes compatibility review & backward/forward tests.
- Simulated additive field change executed.
**Artifacts:** `docs/schema/schema-evolution.md`.

### US-F2.3.1: Dead-Letter Architecture Design (ParentRef: ORIGINAL:Implement Dead-Letter Strategy in Event Hubs)
**As** a Data Engineer **I want** a design for invalid message routing **so** triage is streamlined.
**Acceptance Criteria:**
- Diagram showing primary hub -> processing -> DLQ hub / ADLS path.
- Reason code taxonomy defined.
**Artifacts:** `docs/ingestion/dlq-design.md`.

### US-F2.3.2: Dead-Letter Routing Implementation
**As** a Developer **I want** logic to route invalid envelopes **so** main stream stays healthy.
**Acceptance Criteria:**
- Messages with validation errors appear in DLQ with reason field.
- Main consumer unaffected (no crash).
**Artifacts:** Implementation commit, sample messages.

### US-F2.3.3: DLQ Monitoring & Alerting
**As** a Reliability Engineer **I want** alerting on DLQ rate **so** systemic issues surface quickly.
**Acceptance Criteria:**
- Alert when DLQ > 0.1% of total for 10m.
- Dashboard trend of DLQ count.
**Artifacts:** Alert config & screenshot.

### US-F2.4.1: Partition Key Strategy & Justification (ParentRef: ORIGINAL:Throughput & Partition Load Test)
**As** a Performance Engineer **I want** documented partition key rationale **so** scaling decisions are defendable.
**Acceptance Criteria:**
- Key candidates compared (Equipment vs Line).
- Chosen key with pros/cons rationale.
**Artifacts:** `docs/ingestion/partition-strategy.md`.

### US-F2.4.2: Load Test Script Creation
**As** a Developer **I want** a reusable load test harness **so** future scaling tests are easy.
**Acceptance Criteria:**
- Parameterized rate, topic patterns, payload size.
- Generates metrics JSON output.
**Artifacts:** `tools/loadtest/publisher.py`.

### US-F2.4.3: Load Test Execution & Report
**As** a Performance Engineer **I want** executed load results **so** baseline capacity & headroom are known.
**Acceptance Criteria:**
- 1.5× forecast sustained; skew <15%.
- Publish->Bronze latency p95 <30s.
- Optimization recommendations list.
**Artifacts:** `reports/perf/eventhubs-loadtest.md`.

### US-F2.5.1: Managed Identity Send Authorization (ParentRef: ORIGINAL:Configure Network Security for Event Hubs)
**As** a Security Engineer **I want** least privilege role assignment **so** only authorized identity can send.
**Acceptance Criteria:**
- Identity has only "Event Hubs Data Sender" role.
- Attempt with other identity fails (logged).
**Artifacts:** Role assignment export.

### US-F2.5.2: Auth Failure Alerting
**As** a Security Engineer **I want** alerting on auth failures **so** intrusion attempts surface.
**Acceptance Criteria:**
- Alert after ≥5 failures in 10m.
- Runbook link present.
**Artifacts:** Alert JSON.

### User Story: Configure Network Security for Event Hubs

**Description:** As a Security Engineer, I want ingress restricted to on-prem static egress IPs/VPN and managed identities so only authorized flows succeed.

**Acceptance Criteria:**

- NSG / firewall rules documented & active.
- Managed identity granted least privilege (send) role.
- Attempted connection from disallowed source denied (logged).
- Sentinel (or equivalent) alert created for auth failures.

### User Story: Define Event Schema Envelope

**Description:** As a Data Engineer, I want a canonical JSON envelope for Event Hubs messages so downstream parsing is deterministic.

**Acceptance Criteria:**

- Schema fields: messageId, tsUtc, topic, value, quality, units, sourceSystem, lineageVersion.
- JSON schema file published & versioned.
- Validation library integrated into bridge (log warning on mismatch).
- Sample invalid message triggers dead-letter route (test).

### User Story: Implement Dead-Letter Strategy in Event Hubs

**Description:** As a Data Engineer, I want malformed or rejected messages redirected for triage so stream processing stays healthy.

**Acceptance Criteria:**

- Secondary dead-letter hub or ADLS path created.
- Processing job routes invalid payloads with reason code.
- Alert on first occurrence per hour when dead-letter >0.
- Dashboard chart shows dead-letter rate trend.

### User Story: Throughput & Partition Load Test

**Description:** As a Performance Engineer, I want to validate partition key strategy (e.g., Line/Equipment) so scaling targets are met.

**Acceptance Criteria:**

- Load test script publishes synthetic traffic at 1.5× forecast rate.
- Partition skew <15% deviation.
- End-to-end latency (publish to Bronze write) <30s p95.
- Report stored with raw metrics & recommendations.

## Feature F3: Fabric Medallion Architecture (Bronze / Silver / Gold)

**Description:** Structured multi-layer (Bronze raw immutable, Silver conformed/cleansed, Gold aggregated/semantic) processing in Fabric OneLake with schema drift handling, data quality metrics, optimization & documentation.

### US-F3.1.1: Bronze Streaming Ingestion Notebook (ParentRef: ORIGINAL:Bronze Streaming Ingestion Notebook)

**Description:** As a Data Engineer, I want a Spark Structured Streaming notebook that lands raw events in Bronze Parquet partitioned by date for immutable history.

**Acceptance Criteria:**

- Reads from Event Hubs using connection secret in Key Vault.
- Writes partitioned by yyyy/mm/dd.
- Checkpointing & watermark configured; restart resumes without duplication.
- Sample replay test proves idempotent writes.

### US-F3.1.2: Bronze Schema Drift Detection & Catalog (ParentRef: ORIGINAL:Bronze Schema Drift Handling)

**Description:** As a Data Engineer, I want automatic detection & quarantine of new/unknown fields so evolution doesn’t break the pipeline.

**Acceptance Criteria:**

- Unknown fields captured as JSON blob in separate column.
- Alert when drift detected (first occurrence per field).
- Drift catalog table populated with field name, firstSeen timestamp.
- Documentation updated with resolution steps.

### US-F3.2.1: Silver Parse & Dedupe Job (ParentRef: ORIGINAL:Silver Transformation & Conformance)

**Description:** As a Data Engineer, I want a job to parse UNS topic, enforce types, deduplicate, and materialize clean Delta tables in Silver.

**Acceptance Criteria:**

- Topic parsed into columns (Site, Area, Line, Equipment, Tag).
- Duplicate criteria defined (messageId or composite key) and removed.
- Data quality rules (e.g., value range, non-null critical fields) applied; violations logged.
- Silver Delta optimized (Z-order or partition) documented.

### US-F3.2.2: Silver Data Quality Metrics Table (ParentRef: ORIGINAL:Silver Quality Metrics Table)

**Description:** As a Data Analyst, I want daily aggregated QA metrics so reliability trends are visible.

**Acceptance Criteria:**

- Metrics: totalMessages, goodQualityPct, duplicatesRemoved, driftEvents.
- Table auto-updates daily at fixed schedule.
- Power BI (Direct Lake) model exposes metrics.
- Trend matches raw Bronze counts within ±1%.

### US-F3.3.1: Gold Aggregation Design (ParentRef: ORIGINAL:Gold Aggregations for BI & ML)

**Description:** As a BI Developer, I want hourly & daily aggregates (avg, min, max, stddev) per equipment/tag to accelerate analytics & model feature reuse.

**Acceptance Criteria:**

- Aggregation job produces Gold Delta tables with surrogate keys.
- Performance: aggregate window job completes within SLA (<15m for 24h of data).
- Columns clearly documented (data dictionary).
- Direct Lake semantic model queries return in <5s p95.
 
### US-F3.3.2: Direct Lake Semantic Model Publication (ParentRef: ORIGINAL:Direct Lake Semantic Model Publication)

**Description:** As a BI Developer, I want a semantic model using Direct Lake over Gold tables so reporting avoids data duplication.

**Acceptance Criteria:**

- Model includes measures (AvgValue, ValueVolatility, UptimePct).
- Row-level security (if required) defined & tested.
- Refresh policy configured (incremental where applicable).
- DAX validation queries return expected results (sample set).

### US-F3.4.1: Automated Data Lineage & Catalog Entries (ParentRef: ORIGINAL:Data Lineage & Catalog Entries)

**Description:** As a Data Steward, I want lineage metadata captured for each layer so impact analysis and audit are supported.

**Acceptance Criteria:**

- Metadata table stores sourceTopic, bronzePath, silverTable, goldTable, transformationVersion.
- Automated update in each pipeline run.
- Query returns full lineage for a sample messageId.
- Documentation linked from catalog.

---

## Feature F4: Model Development & Feature Engineering

**Description:** Internalize vendor logic, perform exploratory analysis, engineer reusable feature tables, implement experiment tracking, tuning, validation & drift specification with reproducibility and governance.

### US-F4.1.1: Initial EDA Profiling (ParentRef: ORIGINAL:EDA Notebook on Gold Data)

**Description:** As a Data Scientist, I want an EDA notebook to profile distributions & correlations so feature hypotheses are validated.

**Acceptance Criteria:**

- Notebook outputs summary stats, correlation heatmap, missingness report.
- Risks & anomalies documented in README section.
- Reviewed & approved by Data Science lead.

### US-F4.1.2: Vendor Feature Mapping Documentation (ParentRef: ORIGINAL:Reproduce Vendor Feature Engineering)

**Description:** As a Data Scientist, I want to replicate vendor feature logic so we own and adapt the approach internally.

**Acceptance Criteria:**

- Feature parity checklist (vendor vs reproduced) signed off.
- All transformations parameterized (window sizes, lags).
- Output stored as Versioned Delta feature table.
- Unit tests validate sample expected calculations.

### US-F4.2.1: Baseline MLflow Logging (ParentRef: ORIGINAL:MLflow Experiment Tracking Integration)

**Description:** As a Data Scientist, I want MLflow logging in training notebooks so experiments are auditable.

**Acceptance Criteria:**

- Autolog + custom metrics (precision, recall, latency) captured.
- Git commit hash, feature table version logged.
- Comparison view lists top models by primary metric.

### US-F4.2.2: Hyperparameter Optimization Workflow (ParentRef: ORIGINAL:Hyperparameter Optimization Workflow)

**Description:** As a Data Scientist, I want automated tuning (e.g., Hyperopt) so best model configuration is systematically found.

**Acceptance Criteria:**

- Search space defined & documented.
- Minimum 30 trials executed; convergence plot saved.
- Best params logged; improvement over baseline quantified.

### US-F4.3.1: Model Validation & Registration (ParentRef: ORIGINAL:Model Validation & Registration)

**Description:** As a Data Scientist, I want final model evaluated & registered so downstream deployment can proceed.

**Acceptance Criteria:**

- Holdout test metrics meet thresholds (e.g., MAE <= target, drift checks pass).
- Model registered in MLflow with stage=Staging.
- Promotion checklist completed (data coverage, bias check stub, reproducibility hash).

### US-F4.4.1: Data & Concept Drift Monitoring Design (ParentRef: ORIGINAL:Data & Concept Drift Monitoring Design)

**Description:** As a Data Scientist, I want a drift monitoring spec so production monitoring can be built.

**Acceptance Criteria:**

- Metrics defined (population stability index, mean shift, missing rate change).
- Sampling & schedule documented.
- Alert thresholds set & reviewed with Operations.
- Backlog stories created for implementation (referenced IDs).

---
## Feature F5: On-Prem Inference Container & IoT Edge Deployment

**Description:** Package validated model as secure, observable, low-latency container, deploy via IoT Edge to on‑prem Kubernetes with offline operation strategies and safe deployment patterns.

### US-F5.1.1: Inference API Contract (ParentRef: ORIGINAL:Define Inference API Contract)

**Description:** As an MLOps Engineer, I want a clear JSON contract for /predict so upstream/downstream systems integrate predictably.

**Acceptance Criteria:**

- Request/response schema documented (fields, types, units).
- Error codes & meanings listed.
- Contract stored in repo (OpenAPI spec).

### US-F5.1.2: FastAPI Inference Service Implementation (ParentRef: ORIGINAL:Implement FastAPI Inference Service)

**Description:** As a Developer, I want a FastAPI service that loads MLflow model and exposes /predict & /health endpoints.

**Acceptance Criteria:**

- Cold start <5s on target hardware.
- Concurrent 50 req/sec load test p95 latency <200ms.
- Health endpoint returns dependency status (model loaded, version).
- Unit tests cover success & validation error paths.

### US-F5.2.1: Containerization & Multi-Stage Build (ParentRef: ORIGINAL:Containerize Inference Service)

**Description:** As an MLOps Engineer, I want a minimal Docker image so deployment is fast and secure.

**Acceptance Criteria:**

- Multi-stage Dockerfile; final image <400MB.
- Non-root user runs process.
- Vulnerability scan shows no critical CVEs.
- Image tag includes model version & git SHA.

### US-F5.2.2: Offline Operation Strategy (ParentRef: ORIGINAL:Design Offline Operation Strategy)

**Description:** As a Reliability Engineer, I want documented behaviors for connectivity loss so inference continues uninterrupted.

**Acceptance Criteria:**

- Strategy covers local caching, retry queue for telemetry uploads.
- Test simulating 4h disconnect shows zero inference failures.
- Recovery flush order preserved.

### US-F5.3.1: IoT Edge Deployment Manifest Creation (ParentRef: ORIGINAL:IoT Edge Deployment Manifest Creation)

**Description:** As an MLOps Engineer, I want a versioned deployment manifest so model rollout is controlled.

**Acceptance Criteria:**

- Manifest defines module image, createOptions, desired env vars.
- Includes restart policy & logging config.
- Dry-run validation passes; lint script returns success.

### US-F5.3.2: Shadow / Canary Deployment Process (ParentRef: ORIGINAL:Shadow Deployment (Canary) Process)

**Description:** As a Release Manager, I want a canary procedure so new model versions can be evaluated safely.

**Acceptance Criteria:**

- Dual-run (current vs candidate) design documented.
- Metric comparison script (latency, error rate, prediction divergence) implemented.
- Go/No-Go checklist template stored.

---
## Feature F6: CI/CD & Automation

**Description:** End-to-end automation: branching, quality gates, build, secure supply chain (SBOM, vulnerability scanning), deployment with approvals, and post-deploy validation aligned to DORA & DevSecOps practices.

### US-F6.1.1: Branching & Versioning Policy (ParentRef: ORIGINAL:Git Branching & Versioning Policy)

**Description:** As a Dev Lead, I want a documented branching & semantic versioning strategy so collaboration is consistent.

**Acceptance Criteria:**

- Policy covers main, develop, feature/*, release/*, hotfix/*.
- SemVer mapping to model registry versions defined.
- Commit message conventions (feat:, fix:) documented.

### US-F6.1.2: CI Workflow (Tests & Lint) (ParentRef: ORIGINAL:CI Workflow for Build & Test)

**Description:** As a DevOps Engineer, I want a GitHub Actions workflow to run tests & linting on PRs so quality gates are enforced.

**Acceptance Criteria:**

- Workflow triggers on PR to main/develop.
- Runs unit tests, style checks, security scan (e.g., pip audit / trivy).
- Fails on any high severity issue.
- Status badge added to README.

### US-F6.2.1: Container Build & Push Pipeline (ParentRef: ORIGINAL:Container Build & Push Pipeline)

**Description:** As a DevOps Engineer, I want automated image build & push on tagged releases so artifacts are immutable.

**Acceptance Criteria:**

- Tag push triggers build with build args (MODEL_VERSION).
- SBOM generated & stored.
- Image appears in ACR with digest recorded in release notes.

### US-F6.3.1: Automated IoT Edge Deployment Update (ParentRef: ORIGINAL:Automated IoT Edge Deployment Update)

**Description:** As a DevOps Engineer, I want CD job to update deployment manifest & apply to IoT Hub after approval so rollout is controlled.

**Acceptance Criteria:**

- Manual approval gate before prod deployment.
- Manifest updated with new image tag automatically.
- Audit log captures user, version, timestamp.
- Rollback procedure documented & tested.

### US-F6.3.2: Post-Deployment Smoke Test Script (ParentRef: ORIGINAL:Post-Deployment Smoke Test Script)

**Description:** As a QA Engineer, I want an automated smoke test hitting /health & sample /predict so deployment verification is fast.

**Acceptance Criteria:**

- Script runs after rollout; fails pipeline on error.
- Logs archived with run ID.
- Average response time within baseline ±10%.

---
## Feature F7: Monitoring, Observability & Drift

**Description:** Holistic observability (logs, metrics, traces), model & data quality monitoring, drift detection, capacity & cost signals aligned to SRE golden signals and MLOps monitoring standards.

### US-F7.1.1: Observability Metrics Catalog (ParentRef: ORIGINAL:Define Observability Metrics Catalog)

**Description:** As a Reliability Engineer, I want a metrics catalog so everyone knows what is tracked & why.

**Acceptance Criteria:**

- Sections: Infrastructure, Pipeline, Data Quality, Model, Business KPIs.
- Each metric lists source, frequency, threshold.
- Approved & versioned in repo.

### US-F7.2.1: Inference Metrics Emission (ParentRef: ORIGINAL:Inference Metrics Emission)

**Description:** As a Developer, I want the inference container to emit structured logs & metrics so performance & errors are trackable.

**Acceptance Criteria:**

- Logs JSON with traceId, requestId, latencyMs, status.
- Counter & histogram endpoints exposed (Prometheus format or equivalent) for local scrape.
- Sample log ingestion test validates schema.

### US-F7.3.1: Model Performance Dashboard (ParentRef: ORIGINAL:Model Performance Dashboard)

**Description:** As a Data Scientist, I want a dashboard showing key metrics (MAE, latency, prediction distribution) so we detect degradation early.

**Acceptance Criteria:**

- Data source: uploaded inference telemetry (batch sync if offline).
- Threshold breach triggers alert.
- 30-day trend visualization present.

### US-F7.4.1: Drift Detection Job Implementation (ParentRef: ORIGINAL:Drift Detection Job Implementation)

**Description:** As a Data Scientist, I want scheduled drift calculations so feature & target shift are quantified.

**Acceptance Criteria:**

- Job computes PSI & mean/std deltas vs baseline.
- Results stored in drift_results table with run timestamp.
- Alert when PSI > threshold.

### US-F7.5.1: Fabric Capacity Metrics Repair & Alerting (ParentRef: ORIGINAL:Fabric Capacity Metrics Repair & Alerting)

**Description:** As a Platform Engineer, I want the Fabric capacity metrics app functioning with alerts so resource saturation is prevented.

**Acceptance Criteria:**

- Metrics app reinstalled/configured; tiles render expected data.
- Automated test query validates refresh success daily.
- Alert when capacity CPU or memory >80% for 10m window.

---
## Feature F8: Security, Compliance & Access Control

**Description:** Implement least privilege, identity federation, secret elimination, audit coverage, data retention & policy enforcement supporting zero trust and compliance readiness.

### US-F8.1.1: Role & Access Matrix (ParentRef: ORIGINAL:Access Matrix & Role Definitions)

**Description:** As a Security Analyst, I want a RACI + role permission matrix so access reviews are simplified.

**Acceptance Criteria:**

- Roles (OT Engineer, Data Engineer, Data Scientist, MLOps, Viewer) defined with resource scopes.
- Matrix approved by Security & stored in repo.
- Quarterly review process documented.

### US-F8.2.1: Secret Inventory & Managed Identity Plan (ParentRef: ORIGINAL:Managed Identity & Secret Removal)

**Description:** As a Cloud Engineer, I want all cloud interactions to use managed identities so static secrets are eliminated.

**Acceptance Criteria:**

- Inventory of remaining secrets created.
- Replacements implemented (Key Vault / MI).
- Secrets file diff shows 0 hard-coded credentials.

### US-F8.3.1: Audit Logging Coverage Assessment (ParentRef: ORIGINAL:Audit Logging Coverage Assessment)

**Description:** As a Security Engineer, I want assurance all critical actions are auditable so investigations are possible.

**Acceptance Criteria:**

- Checklist of actions (deploy, model promote, ACL change) mapped to log sources.
- Gap list (if any) with remediation stories created.
- Sample incident replay validated (log chain exists).

### US-F8.4.1: Data Retention Policy Definition (ParentRef: ORIGINAL:Data Retention & Purge Policy)

**Description:** As a Compliance Officer, I want retention rules documented & implemented so storage & regulatory needs are balanced.

**Acceptance Criteria:**

- Retention periods per layer (Bronze raw, Silver conformed, Logs) defined.
- Automated purge job spec created.
- Risk assessment signed off.

## Feature F9: Data Governance & Catalog Enablement

**Description:** Establish glossary, dictionary, lineage, metadata quality automation, and Purview readiness (deferred) to ensure discoverability & trust.

### US-F9.1.1: Initial Business Glossary (ParentRef: ORIGINAL:Business Glossary Creation)

**Description:** As a Data Steward, I want a glossary of key terms so stakeholders share definitions.

**Acceptance Criteria:**

- Terms: OEE, UptimePct, MoistureLevel, DriftEvent.
- Owner & definition per term.
- Published and linked in README.

### US-F9.2.1: Gold Table Data Dictionary (ParentRef: ORIGINAL:Data Dictionary for Gold Tables)

**Description:** As a BI Developer, I want a data dictionary so report authors interpret fields correctly.

**Acceptance Criteria:**

- Each Gold table column: name, type, description, derivation.
- Stored as Markdown; CI lints for missing descriptions.

### US-F9.3.1: Lineage Diagram Artifact (ParentRef: ORIGINAL:Lineage Diagram Artifact)

**Description:** As a Data Steward, I want an updated lineage diagram so onboarding is faster.

**Acceptance Criteria:**

- Diagram shows flow: OT -> HiveMQ -> Event Hubs -> Bronze -> Silver -> Gold -> Model -> Inference.
- Version/date stamped.
- Linked from project wiki.

---

## Feature F10: Reporting & Executive Analytics Enablement

**Description:** Deliver Direct Lake-powered semantic models & executive dashboards (KPIs, governance) with performance SLAs and governance transparency.

### US-F10.1.1: KPI Definition Workshop Output (ParentRef: ORIGINAL:KPI Definition Workshop Output)

**Description:** As a Product Owner, I want codified KPI specs so dashboards reflect agreed metrics.

**Acceptance Criteria:**

- KPIs list calculation formula, owner, refresh cadence.
- Approved doc stored; ties to semantic model measures.

### US-F10.2.1: Executive Operations Dashboard (ParentRef: ORIGINAL:Executive Operations Dashboard)

**Description:** As an Executive, I want a high-level dashboard (throughput, downtime, predicted risk alerts) so I can assess plant health quickly.

**Acceptance Criteria:**

- Dashboard uses Direct Lake; no data extract copies.
- Visuals include traffic-light status with thresholds.
- Load time <5s p95.

### US-F10.3.1: Model Governance Report (ParentRef: ORIGINAL:Model Governance Report)

**Description:** As a Data Science Lead, I want a report summarizing model versions, performance drift, and retrain schedule so governance is transparent.

**Acceptance Criteria:**

- Report auto-refreshes daily.
- Shows current prod model version & last retrain date.
- Conditional formatting flags overdue retrain.

---

## Feature F11: Training, Runbooks & Operational Readiness

**Description:** Create actionable runbooks, playbooks, onboarding & simulation drills to minimize MTTR and accelerate team productivity.

### US-F11.1.1: Operations Runbook Draft Set (ParentRef: ORIGINAL:Operations Runbook Set)

**Description:** As a Support Engineer, I want runbooks for common incidents so MTTR is minimized.

**Acceptance Criteria:**

- Runbooks: Broker outage, Event Hubs throttle, Drift alert, Model rollback.
- Each includes detection, diagnosis steps, rollback, escalation.
- Markdown in repo; indexed in TOC.

### US-F11.2.1: Model Retraining Playbook (ParentRef: ORIGINAL:Model Retraining Playbook)

**Description:** As a Data Scientist, I want a documented retrain process so updates are predictable.

**Acceptance Criteria:**

- Trigger criteria defined (drift thresholds, data volume delta).
- Steps include feature table freeze, experiment log, validation.
- Checklist template stored.

### US-F11.3.1: Onboarding Guide (ParentRef: ORIGINAL:Onboarding Guide)

**Description:** As a New Team Member, I want a concise onboarding guide so I become productive within a week.

**Acceptance Criteria:**

- Sections: Architecture overview, data access, dev environment setup, key dashboards.
- Reviewed by a new hire for clarity.

---

## Feature F12: Performance & Cost Optimization

**Description:** Optimize storage lifecycle, compute efficiency, job runtime baselines, and cost visibility to sustain economic performance.

### US-F12.1.1: Storage Tiering Strategy (ParentRef: ORIGINAL:Storage Tiering Strategy)

**Description:** As a Platform Engineer, I want a strategy to tier & archive old Bronze data so costs remain controlled.

**Acceptance Criteria:**

- Policy: Hot vs Cool vs Archive thresholds defined.
- Forecast cost savings analysis attached.
- Implementation backlog items created.

### US-F12.2.1: Fabric Job Performance Baselines (ParentRef: ORIGINAL:Fabric Job Performance Baselines)

**Description:** As a Data Engineer, I want performance baselines so regressions are detected early.

**Acceptance Criteria:**

- Baseline metrics (runtime, input rows/sec) captured for core jobs.
- Regression alert when +20% runtime over baseline for 3 runs.
- Baseline report stored & versioned.

## Feature F13: Future Enablement (Placeholder Stories)

**Description:** Prepare for anticipated capabilities (Local, Purview integration) without committing to near-term delivery; maintain architectural optionality.

### US-F13.1.1: Evaluate Local Option (ParentRef: ORIGINAL:Evaluate Local Option for Future OT Expansion)

**Description:** As an Architect, I want an evaluation of Local feasibility so future hybrid improvements are informed (without impacting current offline design).

**Acceptance Criteria:**

- Comparison matrix (current Kubernetes + IoT Edge vs Local).
- Risks & benefits summarized.
- Decision record created.

### US-F13.2.1: Plan Purview Integration (Deferred) (ParentRef: ORIGINAL:Plan Purview Integration (Deferred))

**Description:** As a Data Governance Lead, I want a readiness assessment for Purview so future catalog adoption is smooth.

**Acceptance Criteria:**

- Gap analysis (current metadata vs Purview requirements).
- Proposed phased integration steps.
- Deferred tag applied; revisit date logged.

---

---

Note: Detailed acceptance criteria for newly enumerated split stories (e.g., those merely renamed above) inherit the original story criteria plus their own specific scope; further refinement can be added iteratively during sprint planning.

End of backlog (Rev 2).
