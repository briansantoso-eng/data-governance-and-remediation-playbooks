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

- **Operational risk reduced** — latent, error-free-looking data defects (stalled work,
  unfulfillable assignments) are now defined, detectable, and remediable instead of
  silently accumulating.
- **Knowledge made durable** — governance logic previously held by a handful of people
  is captured as reviewable, version-controlled documentation.
- **A foundation for automation** — by formalising detection and remediation as explicit
  decision trees, the work establishes the prerequisites for agent-assisted and
  fully-automated remediation.
- **Catalog-ready metadata** — the schema tooling bridges an internal operational
  database into an enterprise data-catalog format, improving discoverability and
  ownership clarity across the organisation.

## Approach & skills demonstrated

`Data Governance` · `Data Quality` · `SQL` · `Relational Data Modelling` ·
`Metadata / Data Catalogs (DataHub)` · `Process Design` · `Technical Writing` ·
`JavaScript / Node tooling`
