# Assignment 5 Documentation:

---
## AI-usage statement

I used an AI coding agent for this assignment. The agent wrote the analysis notebook, the coverage template, and this note. I reviewed the files and checked the queries against the Fire Force model.

- **IDE:** Cursor.
- **Model:** Grok 4.7.
- **Connection:** Cursor talks to the OML model through MCP. The project file [.cursor/mcp.json](.cursor/mcp.json) starts [scripts/oml-mcp](scripts/oml-mcp), which runs the OML MCP server against this folder. That let the agent search the Sierra and Fire Force models and run the SPARQL, instead of only editing text.
- **What we did:** Looked at the existing Sierra dashboards and the Fire Force system. Wrote five queries (conformance, near miss, orphan, coverage, view graph), rendered them as a table, a chart, a list, a matrix, and a graph, then counted the matrix in a Python block. Put the matrix in one compose template and opened that template from two pages.
- **Debugging:** The first entry-count grouped every vertex in the region with the entry, so each region looked like it had several entries. Counting distinct entries fixed that. All three regions pass.
---

Cmd-click any link to open the file where that thing is defined.

The analysis notebook is [Analysis.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Analysis.md>). Open it from [Overview.md](<src/model/md/Fire Force VI/Mission Control/Overview.md>) under **Analyze Operational Coverage**.

## The five queries

| Query | View | What it uses | What it returned |
| --- | --- | --- | --- |
| Conformance | table | [Region](src/method/oml/www.modelware.io/sierra/state2.oml) and [Entry](src/method/oml/www.modelware.io/sierra/state2.oml), the rule on the [statechart page](src/method/md/www.modelware.io/sierra/operational-analysis/statechart.md) | [WardenRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml), [DashboardRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml), and [CloudRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) each have one entry. All three say `pass`. A second entry, or a region with none, would say `fail`. |
| Near miss | chart | [requires](src/method/oml/www.modelware.io/sierra/mission.oml), [isAssignedTo](src/method/oml/www.modelware.io/sierra/entity.oml), [describes](src/method/oml/www.modelware.io/sierra/process.oml) | 2 complete, 6 missing a process. [C2](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/capabilities.oml) is described by [P1](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/processes.oml). [C5](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/capabilities.oml) is described by [P2](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/processes.oml). C1, C3, C4, C6, C7, and C8 are required and assigned, and they have no process. |
| Orphan | list | [Port](src/method/oml/www.modelware.io/sierra/component.oml) and [Connection](src/method/oml/www.modelware.io/sierra/component.oml), with `FILTER NOT EXISTS` | [Propulsion.Command_In](src/model/oml/fireforce6.github.io/mission-control/system-analysis/connections.oml) has no connection. The command link that does exist runs from FireSat.Command_In to Platform.Command_In. |
| Coverage | matrix | every [Entity](src/method/oml/www.modelware.io/sierra/entity.oml) crossed with every [Capability](src/method/oml/www.modelware.io/sierra/mission.oml), `1` or `0` | 56 cells. 17 are `1`. 39 are explicit `0`. NetworkInterface only has C2. Each of the three actors has one capability. |
| View graph | graph | `CONSTRUCT` of [regions](src/method/oml/www.modelware.io/sierra/state2.oml), [vertices](src/method/oml/www.modelware.io/sierra/state2.oml), and each transition from source to target | The three charts in [charts.oml](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml), shaped as a graph. |

The near-miss section on the notebook is written as Question, then the chart as evidence, then Interpretation.

The Python block on the same page runs the coverage query, counts the rows, and prints **17 of 56** linked and **39** explicit zeros.

## Reuse

The matrix is defined once, in [trace-coverage.md](src/method/md/www.modelware.io/sierra/operational-analysis/trace-coverage.md).

It is invoked from:

1. [Analysis.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Analysis.md>)
2. [Coverage.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Coverage.md>)

Coverage.md does not copy the query. It only names the template.

## Reflection

The entry rule is already satisfied, so the conformance table is clean, and the note says what a failure would look like. Absence shows up in the other two checks. The orphan list is one unused port, and the coverage matrix prints a zero for every entity–capability pair that was tested and not linked. Those zeros are the gaps, not an empty result. The template is real reuse: one definition, two pages. Changing the matrix means editing [trace-coverage.md](src/method/md/www.modelware.io/sierra/operational-analysis/trace-coverage.md) only.

---

# How to open it

Open [Analysis.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Analysis.md>) in the OML markdown preview.

1. The conformance table shows three regions, each with `entries` 1 and `verdict` pass.
2. The near-miss chart shows 2 complete and 6 missing a process.
3. The orphan list shows Propulsion.Command_In.
4. The statechart graph shows the three charts, their regions, and the transitions.
5. The coverage matrix shows 1s and highlighted 0s. The same matrix is on [Coverage.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Coverage.md>).
6. The Python block prints 17 of 56 linked, and 39 explicit zeros.
