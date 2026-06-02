# Data Governance and Remediation Playbooks

> Data governance rule sets and operational remediation playbooks for a large-scale
> enterprise workflow platform, developed as part of a data governance initiative at
> **WiseTech Global**.
>
> ⚠️ **Confidentiality note:** This repository documents *approach and methodology only*.
> All table names, column names, rules, and figures shown here are **illustrative
> fabrications** created for portfolio purposes. The real implementation, schema, and
> data are proprietary to WiseTech Global and are not reproduced here.

---

## Why this project exists

Large operational platforms accumulate data faster than they accumulate *rules about
that data*. Over time, the difference between a valid and an invalid record becomes
tribal knowledge — understood by a few engineers, undocumented, and impossible to
enforce consistently or automate.

That gap is not cosmetic. On a workflow platform, a single class of silent data defect
can have direct operational cost:

- work assigned to people who have **left the organisation**, so it never gets done;
- work routed to a skill/capability group with **no available members**, so it stalls
  indefinitely;
- timestamps and status fields that **contradict each other**, corrupting every report
  and metric built on top of them.

None of these throw an error. They sit in the database looking valid while quietly
degrading throughput, reporting accuracy, and trust in the system.

This project turns that implicit knowledge into an **explicit, classified, auditable,
and ultimately automatable** governance layer.

## What it contains

| Component | Description |
|---|---|
| **Process Task Standards (PTS)** | A governance rule set for the platform's core process-task entity — each rule classified by severity (High/Medium/Low), type (Hard Rule / Warning / Exception State), and enforcement (system-enforced vs. data-owner-monitored). |
| **Remediation Playbooks** | Step-by-step operational procedures for resolving each class of rule breach: detection logic, an action decision tree (auto-resolve / cancel / reassign / human review), exclusions, prioritisation, and a path toward automation. |
| **Schema Export Tooling** | Tooling that takes the platform's relational schema (sourced via SQL) and transforms it into structured, catalog-ready JSON for ingestion into a data catalog (e.g. DataHub). |

## Illustrative example — a governance rule

> *(fabricated for illustration)*

**Rule PTS-XX — A task must not be assigned to an inactive resource**

| Field | Value |
|---|---|
| Severity | High |
| Type | Hard Rule |
| Enforcement | Data Owner (system permits it; must be caught by monitoring) |
| Rationale | A task assigned to a departed staff member is unactionable — it occupies a queue, distorts capacity metrics, and never completes. |
| Detection | A task in an *assigned* state whose assigned-resource reference points at a resource record flagged inactive. A cross-table condition — undetectable from the task row alone. |

Each rule like this is paired with a remediation playbook describing exactly how a data
owner (or, eventually, an automated agent) resolves a breach safely — including which
breaches must *never* be auto-actioned (e.g. tasks tied to open defects).

## Illustrative example — schema as catalog-ready JSON

> *(fabricated for illustration)*

The schema-export tooling pivots a flat SQL schema dump into one structured document per
table, resolving primary keys and foreign keys, ready for a data catalog:

```json
{
  "dataset": "dbo.WorkflowTask",
  "domain": "Workflow & Process",
  "criticality": "High",
  "primaryKeyColumns": ["Task_PK"],
  "columns": [
    { "name": "Task_Status",           "type": "varchar(3)",       "nullable": false },
    { "name": "Task_AssignedResource", "type": "uniqueidentifier", "nullable": true,
      "foreignKey": { "referencedTable": "Resource", "referencedColumn": "Res_PK" } }
  ]
}
```

## How it works in the system

The governance layer is built in stages, each one feeding the next:

1. **Schema sourced from the live database (SQL).** Table and column definitions —
   data types, nullability, primary keys, foreign keys — are extracted from the
   operational platform's relational schema via SQL.

2. **Schema transformed into structured JSON.** A Node script pivots the flat SQL
   output into one document per table (PKs and FKs resolved, columns nested), shaped so
   each table maps cleanly onto a data-catalog dataset. The tables the governance rules
   depend on are foregrounded so the rule layer and the catalog layer share one model.

3. **Rules expressed as conditions over that model.** Each Process Task Standard (PTS)
   is a precise, testable condition on the schema — frequently *cross-table* (e.g. a
   task row joined to a resource record to check an "is active" flag), which is exactly
   the class of defect a single-column database constraint cannot catch.

4. **Detection runs against production data.** Breaches are surfaced through saved
   searches in the operational application (so any data owner can run the same view
   consistently) and, for automation, through the equivalent queries against the
   database directly.

5. **Remediation playbooks turn a breach into an action.** Each rule is paired with a
   playbook that classifies every breach down an explicit decision tree —
   *auto-resolve → cancel → reassign → human review* — with safety **exclusions**
   (substantive in-flight work is never silently cancelled), a **prioritisation** order
   (the most operationally urgent breaches first), and, for owner-driven cases, a
   time-boxed consultation loop with a default fallback so nothing stalls indefinitely.

6. **Metadata published to a data catalog.** The structured JSON is the source for a
   data-catalog (DataHub) ingest — dataset schemas, domains, and ownership become
   discoverable and queryable alongside the governance context.

The design is deliberately **environment-agnostic and automation-ready**: because
detection and remediation are formalised as data conditions and decision trees rather
than tribal judgement, each step can graduate from a manual run by a data owner to an
assisted or fully automated agent without rewriting the governance logic.

## My contributions

- **Authored the Process Task Standards (PTS)** — the governance rule set covering the
  platform's central process-task entity, including the rule classification model
  (severity / type / enforcement) and the cross-table integrity rules that can't be
  expressed as simple column constraints.
- **Built the SQL-to-JSON schema tooling** — sourced table and column definitions via
  SQL and wrote tooling to transform them into structured JSON files suitable for
  ingestion into a data catalog (DataHub), foregrounding the tables the governance rules
  depend on.
- Contributed to the **remediation playbook methodology** — detection logic, action
  decision trees, exclusion safeguards, and prioritisation.

## Impact on the organisation

- **Operational risk reduced at the source.** Latent, error-free-looking defects —
  work assigned to departed staff, work routed to empty capability groups, contradictory
  status/timestamp data — were previously invisible until they caused a stalled job or a
  wrong report. They are now defined, detectable, and remediable conditions.
- **Tribal knowledge made durable and auditable.** Governance logic once held by a
  handful of engineers is captured as classified, version-controlled rules and
  playbooks that a new team member — or an agent — can act on.
- **Remediation that scales.** A single empty capability can put thousands of tasks into
  breach at once; resolving it at the capability level clears them all. The playbooks
  encode that leverage, with prioritisation that puts the most time-critical work first.
- **A foundation for automation.** Formalising detection and remediation as explicit
  data conditions and decision trees establishes the prerequisites for agent-assisted
  and fully-automated remediation — the long-term direction of the initiative.
- **Catalog-ready, discoverable metadata.** Bridging an internal operational database
  into an enterprise data-catalog format improves discoverability, clarifies ownership,
  and links each dataset to the governance rules that depend on it.

## Approach & skills demonstrated

`Data Governance` · `Data Quality` · `SQL` · `Relational Data Modelling` ·
`Metadata / Data Catalogs (DataHub)` · `Process Design` · `Technical Writing` ·
`JavaScript / Node tooling`
