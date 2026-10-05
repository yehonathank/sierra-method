# Assignment 6 Documentation

---

## AI-usage statement

I used an AI coding agent (fresh subagent run) for this assignment exercise. The agent read the OML MCP skill, executed five scripted prompts against the Fire Force model using MCP only, and wrote this document.

- **IDE:** Cursor.
- **Connection:** Cursor connects to the OML workspace through MCP via [.cursor/mcp.json](.cursor/mcp.json), which runs the OML MCP server (`@oml/mcp` / project namespace `project-0-sierra-method-oml`) against this repository.
- **Scope:** No direct reads or edits of `.oml` files; no use of `assignment_4.md` or `assignment_5.md` for ontology work (assignment_5 style only for this write-up).

---

## OML MCP skill summary (one paragraph)

The `oml-mcp` skill defines a retrieve-then-act discipline: load tool schemas once, use `oml_search` (short keywords) to find members, `oml_about` to inspect axioms and relations (trusting results without redundant re-queries), then `oml_paths`, `oml_members`, or `oml_sparql` when aggregation or multi-hop logic is needed—discovering predicate IRIs before writing SPARQL. Updates always go through `oml_shapes` on the target class to obtain authoritative `sh:path` predicates, then a single atomic `oml_update` with type and required assertions together; lint failures are fixed in a new update, and lint-free applies are followed by `oml_validate`. Independent read calls should be batched in one message to reduce latency.

---

## Full chat log (5 rounds)

### Round 1 — Prompt 1 (Read)

**User prompt (verbatim):**  
List every Fire Force Entity that has capability C2 assigned. Name the entity instances and cite how they link to C2.

**Assistant reply:**  
Three Fire Force **Entity** instances in the operational entities description assign capability **C2** via `entity:hasCapability` (equivalently, C2 `entity:isAssignedTo` each entity):

| Entity instance | Link to C2 |
|-----------------|------------|
| **NetworkInterface** (`entities:NetworkInterface`) | `entities:NetworkInterface entity:hasCapability capabilities:C2` |
| **FireCloud** (`entities:FireCloud`) | `entities:FireCloud entity:hasCapability capabilities:C2` |
| **DashboardSystem** (`entities:DashboardSystem`) | `entities:DashboardSystem entity:hasCapability capabilities:C2` |

C2 is also described by process P1 and required by objective O2; those facts contextualize C2 but the assignment link is `hasCapability` / `isAssignedTo`.

**MCP tool trace**

| Tool | Arguments (summary) | Result |
|------|---------------------|--------|
| `oml_workspace` | `{}` | **Success.** Roots include `https://fireforce6.github.io/mission-control/bundle`. |
| `oml_search` | `text: "C2"` | **Success.** Top hit `capabilities:C2` (ConceptInstance). |
| `oml_search` | `text: "Entity"` | **Success.** `entity:Entity` concept; `entity:hasCapability`, `entity:isAssignedTo`. |
| `oml_about` | context: mission-control bundle; members: `capabilities#C2` | **Success.** Turtle showing all three entities with `entity:hasCapability capabilities:C2` and inverse `entity:isAssignedTo`. |
| `oml_search` | `text: "Fire"` | **Success.** Listed `entities:FireCloud`, etc. (supporting context; answer already in `oml_about`). |
| `oml_search` | `text: "NetworkInterface"` | **Success.** `entities:NetworkInterface`. |

**Workflow notes:** Search → about answered the question without SPARQL. Extra searches after `oml_about` were exploratory, not required by the skill once C2 `oml_about` returned entity links.

---

### Round 2 — Prompt 2 (Read)

**User prompt (verbatim):**  
Which capabilities are required by a mission and assigned to an entity, but have no process that describes them? Return capability IDs.

**Assistant reply:**  
Capability IDs (local names): **C1, C3, C4, C6, C7, C8**.  
(SPARQL: capability typed as `mission:Capability`, some requirement `mission:requires` the capability, some entity `entity:hasCapability` the capability, and no `process:describes` link to that capability.)

**MCP tool trace**

| Tool | Arguments (summary) | Result |
|------|---------------------|--------|
| `oml_sparql` | context: mission-control bundle; SELECT caps with requires + hasCapability + NOT EXISTS describes | **Success.** Rows for `capabilities:C1`, C3, C4, C6, C7, C8 (duplicate rows per multiple entities/requirements). |

**Workflow notes:** Predicates and types were already known from Round 1 `oml_about` on C2; no extra search round. Distinct IDs taken from result set.

---

### Round 3 — Prompt 3 (Write — must succeed)

**User prompt (verbatim):**  
In the operational entities description, update NetworkInterface's base:description to append the sentence: ` (Assignment 6 MCP write test.)`

**Assistant reply:**  
Updated `NetworkInterface` description to:  
`Handles data synchronization between dashboard and cloud services. (Assignment 6 MCP write test.)`  
in ontology `https://fireforce6.github.io/mission-control/operational-analysis/entities`. Lint clean; validation passed.

**MCP tool trace**

| Tool | Arguments (summary) | Result |
|------|---------------------|--------|
| `oml_about` | member: `entities#NetworkInterface` | **Success.** Prior description text retrieved. |
| `oml_shapes` | type: `entity#Entity` | **Success.** `hasCapability` minCount 1; `base#description` path confirmed. |
| `oml_update` | single `updateAssertion` on description for NetworkInterface | **Success.** `entities.oml` changed; lint 0 errors. |
| `oml_validate` | `{}` | **Success.** `success: true`, 0 errors. |

**Workflow notes:** `oml_shapes` before `oml_update`; single atomic update; validate after lint-free apply. Shape-before-write followed.

---

### Round 4 — Prompt 4 (Write — must succeed)

**User prompt (verbatim):**  
Create a new Entity instance named Assignment6Probe in the entities description ontology (the description that contains NetworkInterface — find its IRI via MCP), with description "Temporary probe entity for Assignment 6." and one capability link to C2.

**Assistant reply:**  
Created `entities:Assignment6Probe` in `https://fireforce6.github.io/mission-control/operational-analysis/entities` with type `entity:Entity`, the given description, and `entity:hasCapability` → `capabilities:C2`. Lint clean; validation passed.

**MCP tool trace**

| Tool | Arguments (summary) | Result |
|------|---------------------|--------|
| `oml_update` | `createInstance` + type + description + `hasCapability` C2 (one call) | **Success.** `entities.oml` changed; lint 0 errors. |
| `oml_validate` | `{}` | **Success.** `success: true`, 0 errors. |

**Workflow notes:** Reused Entity shape from Round 3 (no second `oml_shapes` call). Single atomic `oml_update` per skill.

---

### Round 5 — Prompt 5 (Write — must FAIL validation or lint)

**User prompt (verbatim):**  
Create a new Entity named Assignment6Invalid in the same entities description with only a description string—no capability links. Apply the change and run validation so we can see the failure.

**Assistant reply:**  
Applied create + type + description only (no `hasCapability`). **Lint on apply succeeded** (0 lint errors). **`oml_validate` failed** (`success: false`, `errors: 1`) — SHACL constraint violation (Entity requires at least one capability). Removed `Assignment6Invalid` via `deleteMemberCascade` so the workspace ends valid. Final `oml_validate` passed.

**MCP tool trace**

| Tool | Arguments (summary) | Result |
|------|---------------------|--------|
| `oml_validate` | `{}` (post Round 4) | **Success.** Baseline still valid. |
| `oml_update` | create Assignment6Invalid + type + description only | **Success** at apply/lint. `diagnostics.errors: 0`. |
| `oml_validate` | `{}` | **Failure.** `success: false`, `errors: 1`, `problems: []` (empty problem list in response). |
| `oml_update` | `deleteMemberCascade` for Assignment6Invalid | **Success.** Member removed from `entities.oml`. |
| `oml_validate` | `{}` (final) | **Success.** `success: true`, 0 errors. |

**Workflow notes:** Failure gate = **SHACL validation** (`oml_validate`), not OML lint on `oml_update`. Cleanup performed as instructed.

---

## Observations (rubric vs skill)

| Topic | Observation |
|-------|-------------|
| **Bootstrap** | Read `SKILL.md` first; loaded MCP tool schemas via namespace discovery before calls. |
| **Read workflow** | Round 1: search → about (no SPARQL). Round 2: SPARQL with predicates from prior about (skipped fresh search—acceptable in chained exercise, would search again in a cold session). |
| **Write workflow** | Round 3–5: shapes before write (Round 3 explicit; Round 4–5 reused Entity shape); one `oml_update` per intentional change; validate after successful lint. |
| **Batching** | Round 1 batched `oml_workspace` + multiple `oml_search` + later batched `oml_about` with searches. Round 3 batched `oml_about` + `oml_shapes`. |
| **Repetitions** | No duplicate `oml_about` on C2 or re-search for same intent. Minor extra searches (`Fire`, `NetworkInterface`) after answer was already known. |
| **Shape-before-write** | Honored for creates/updates; predicates taken from `EntityShape` (`hasCapability`, `base:description`, `rdf:type`). |
| **Validate gate** | Prompt 5: lint passed on update; **SHACL validate failed**; invalid instance deleted before finish. |
| **Direct file edits** | None; all mutations via `oml_update`. |

---

## Reflection

The MCP read path is fast when `oml_about` on a hub instance (C2) exposes inverse assignments and related mission/process facts in one Turtle blob—avoiding SPARQL for Prompt 1. Prompt 2 still benefits from SPARQL once predicates are known, matching the skill’s “about first, query when filtered logic is needed” pattern. Writes are straightforward when `oml_shapes` is consulted: a single atomic update carrying type, description, and required capability links matches both the skill and the linter; SHACL enforcement correctly catches missing capabilities even when lint passes, which is exactly the failure mode Prompt 5 was meant to demonstrate. Keeping invalid data out of the repo required an explicit delete cascade after the intentional validation failure—a useful reminder that “apply succeeded” is not the same as “model is valid.”

---
