---
template:
  id: https://www.modelware.io/sierra/operational-analysis/trace-coverage
  name: "Entity Capability Coverage"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Entity–capability coverage

Each cell is one [entity](../../../../oml/www.modelware.io/sierra/entity.oml) and one [capability](../../../../oml/www.modelware.io/sierra/mission.oml). A **1** means that capability is [assigned](../../../../oml/www.modelware.io/sierra/entity.oml) to that entity. A **0** means the pair was checked and the assignment is absent. The zero is the gap.

```matrix
---
rowColumnLabel: Entity / Capability
stylesheet:
  - selector: cell [value === "0"]
    style:
      background-color: "#fde8e8"
---
PREFIX entity: <https://www.modelware.io/sierra/entity#>
PREFIX mission: <https://www.modelware.io/sierra/mission#>

SELECT ?row ?column ?value
WHERE {
  ?entity a entity:Entity .
  ?capability a mission:Capability .
  BIND(REPLACE(STR(?entity), "^.*[#/]", "") AS ?row)
  BIND(REPLACE(STR(?capability), "^.*[#/]", "") AS ?column)
  BIND(IF(EXISTS { ?capability entity:isAssignedTo ?entity }, "1", "0") AS ?value)
}
ORDER BY ?row ?column
```
