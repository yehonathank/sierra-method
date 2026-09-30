---
ontology: https://fireforce6.github.io/mission-control/bundle
---

# Fire Force analysis

Five queries over the Mission Control model. The coverage matrix is a shared template; the same template is also opened from [Coverage.md](Coverage.md).

## Conformance

A [region](../../../../../method/oml/www.modelware.io/sierra/state2.oml) must contain exactly one [entry](../../../../../method/oml/www.modelware.io/sierra/state2.oml). The rule is stated on the [statechart page](../../../../../method/md/www.modelware.io/sierra/operational-analysis/statechart.md). This table counts distinct entries in each region. `pass` means the count is 1. A region with no entry, or with two, would show `fail`.

```table
PREFIX state2: <https://www.modelware.io/sierra/state2#>

SELECT ?region ?entries ?verdict
WHERE {
  {
    SELECT ?regionIri (COUNT(DISTINCT ?entry) AS ?entries)
    WHERE {
      ?regionIri a state2:Region .
      OPTIONAL {
        ?regionIri state2:vertices ?entry .
        ?entry a state2:Entry .
      }
    }
    GROUP BY ?regionIri
  }
  BIND(REPLACE(STR(?regionIri), "^.*[#/]", "") AS ?region)
  BIND(IF(?entries = 1, "pass", "fail") AS ?verdict)
}
ORDER BY ?region
```

## Near miss

### Question

Which capabilities are already required by an objective and assigned to an entity, but still have no process?

### Evidence

A capability is complete only when all three links exist: [requires](../../../../../method/oml/www.modelware.io/sierra/mission.oml), [isAssignedTo](../../../../../method/oml/www.modelware.io/sierra/entity.oml), and [describes](../../../../../method/oml/www.modelware.io/sierra/process.oml). The chart counts how many capabilities meet that bar, and how many stop one link short.

```chart
---
type: bar
data:
  labels: status
  datasets:
    - label: Capabilities
      data: count
options:
  plugins:
    title:
      display: true
      text: Capabilities one step from a process
    legend:
      display: false
  scales:
    y:
      beginAtZero: true
      ticks:
        precision: 0
---
PREFIX mission: <https://www.modelware.io/sierra/mission#>
PREFIX entity: <https://www.modelware.io/sierra/entity#>
PREFIX process: <https://www.modelware.io/sierra/process#>

SELECT ?status (COUNT(?capability) AS ?count)
WHERE {
  ?capability a mission:Capability .
  BIND(EXISTS { ?objective mission:requires ?capability } AS ?required)
  BIND(EXISTS { ?capability entity:isAssignedTo ?entity } AS ?assigned)
  BIND(EXISTS { ?process process:describes ?capability } AS ?described)
  BIND(IF(?required && ?assigned && ?described, "complete", IF(?required && ?assigned, "missing process", "incomplete")) AS ?status)
}
GROUP BY ?status
ORDER BY ?status
```

### Interpretation

Six of the eight capabilities are near misses. [C2](../../../../../model/oml/fireforce6.github.io/mission-control/operational-analysis/capabilities.oml) is described by [P1](../../../../../model/oml/fireforce6.github.io/mission-control/operational-analysis/processes.oml), and [C5](../../../../../model/oml/fireforce6.github.io/mission-control/operational-analysis/capabilities.oml) is described by [P2](../../../../../model/oml/fireforce6.github.io/mission-control/operational-analysis/processes.oml). C1, C3, C4, C6, C7, and C8 are required and assigned, and they have no process. The missing relationship is `process:describes`.

## Orphan

A port with no [connection](../../../../../method/oml/www.modelware.io/sierra/component.oml) is a missing relationship. `FILTER NOT EXISTS` keeps a port only when it is neither the source nor the target of a connection.

```list
PREFIX oml: <http://opencaesar.io/oml#>
PREFIX component: <https://www.modelware.io/sierra/component#>

SELECT ?port
WHERE {
  ?p a component:Port .
  FILTER NOT EXISTS {
    { ?conn a component:Connection ; oml:hasSource ?p }
    UNION
    { ?conn a component:Connection ; oml:hasTarget ?p }
  }
  BIND(REPLACE(STR(?p), "^.*[#/]", "") AS ?port)
}
ORDER BY ?port
```

The list names [Propulsion.Command_In](../../../../../model/oml/fireforce6.github.io/mission-control/system-analysis/connections.oml). The command path that does exist runs from FireSat.Command_In to Platform.Command_In. Propulsion.Command_In is declared and unused. An empty list would mean every port is connected; that is the gap this query is built to catch.

## View graph

This graph is only the statechart shape: each chart contains its region, each region contains its vertices, and each transition is an edge from its source vertex to its target. It is not the whole model.

```graph
---
layout:
  mode: dag
  fit: true
  padding: 40
  dag:
    rankDir: LR
    acyclic: false
---
PREFIX state2: <https://www.modelware.io/sierra/state2#>

CONSTRUCT {
  ?chart state2:regions ?region .
  ?region state2:vertices ?vertex .
  ?source ?transition ?target .
}
WHERE {
  {
    ?chart a state2:Statechart ;
           state2:regions ?region .
    ?region state2:vertices ?vertex .
  }
  UNION
  {
    ?transition a state2:Transition ;
                state2:source ?source ;
                state2:target ?target .
  }
}
```

## Coverage

The matrix is defined once, in the [trace-coverage template](../../../../../method/md/www.modelware.io/sierra/operational-analysis/trace-coverage.md). This page invokes it. [Coverage.md](Coverage.md) invokes the same template.

```compose
template: https://www.modelware.io/sierra/operational-analysis/trace-coverage
```

The block below runs that same cross-product, then counts the ones and the zeros.

```python
result = await query("""
PREFIX entity: <https://www.modelware.io/sierra/entity#>
PREFIX mission: <https://www.modelware.io/sierra/mission#>
SELECT ?entity ?capability ?linked
WHERE {
  ?entity a entity:Entity .
  ?capability a mission:Capability .
  BIND(IF(EXISTS { ?capability entity:isAssignedTo ?entity }, "1", "0") AS ?linked)
}
""")

rows = result["rows"]
total = len(rows)
linked = sum(1 for r in rows if str(r.get("linked")) in ("1", "1.0", "true"))
missing = total - linked
display(
    f"<p><b>{linked}</b> of <b>{total}</b> entity–capability pairs are linked. "
    f"<b>{missing}</b> cells are explicit zeros.</p>"
)
```
