---
template:
  id: https://www.modelware.io/sierra/operational-analysis/statechart
  name: "Statechart"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Statechart

Author a hierarchical statechart for an operational entity.

**Focal type:** Statechart.

**Structure:** an existing entity owns the chart (`isAllocatedTo`). The chart contains regions (`regions` / `inComposite`). Each region contains vertices (`vertices` / `region`): entry, exit, and choice pseudostates, plus states and final states. A state may itself contain regions. Transitions connect vertices by source and target and are not part of the tree.

**Expected content:** one chart per operational system. Each region has exactly one entry, then the states and transitions that match that system's behavior.

**Rule:** a region contains exactly one entry pseudostate.

The entity rows are read-only context. Edit the chart, its regions, and its vertices here; edit transitions in the table below.

```tree-editor
---
columns: { this: { label: "Statechart" } }
---
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix entity: <https://www.modelware.io/sierra/entity#> .
@prefix state2: <https://www.modelware.io/sierra/state2#> .

entity:EntityShape
    a sh:NodeShape ;
    dash:readOnly true ;
    sh:targetClass entity:Entity ;
    oml:where """
        VALUES ?this {
            <https://fireforce6.github.io/mission-control/operational-analysis/entities#AIFireWarden>
            <https://fireforce6.github.io/mission-control/operational-analysis/entities#FireCloud>
            <https://fireforce6.github.io/mission-control/operational-analysis/entities#DashboardSystem>
        }
    """ ;
    .

state2:StatechartShape
    a sh:NodeShape ;
    sh:targetClass state2:Statechart ;
    sh:property [
        sh:path state2:isAllocatedTo ;
        sh:name "Entity" ;
        sh:class entity:Entity ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
        dash:composite true ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    .

state2:RegionShape
    a sh:NodeShape ;
    sh:targetClass state2:Region ;
    sh:sparql [
        sh:message "A region must contain exactly one entry pseudostate." ;
        sh:select """
            PREFIX state2: <https://www.modelware.io/sierra/state2#>
            SELECT $this WHERE {
                $this state2:vertices ?entry1 .
                $this state2:vertices ?entry2 .
                ?entry1 a state2:Entry .
                ?entry2 a state2:Entry .
                FILTER(STR(?entry1) < STR(?entry2))
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "A region must contain exactly one entry pseudostate." ;
        sh:select """
            PREFIX state2: <https://www.modelware.io/sierra/state2#>
            SELECT $this WHERE {
                $this a state2:Region .
                FILTER NOT EXISTS {
                    $this state2:vertices ?entry .
                    ?entry a state2:Entry .
                }
            }
        """ ;
    ] ;
    sh:property [
        sh:path state2:inComposite ;
        sh:name "Composite" ;
        sh:class state2:CompositeElement ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
        dash:composite true ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    .

state2:VertexShape
    a sh:NodeShape ;
    sh:targetClass state2:Entry ;
    sh:targetClass state2:Exit ;
    sh:targetClass state2:Choice ;
    sh:targetClass state2:State ;
    sh:targetClass state2:FinalState ;
    sh:property [
        sh:path rdf:type ;
        sh:name "Type" ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path state2:region ;
        sh:name "Region" ;
        sh:class state2:Region ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
        dash:composite true ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    .
```

## Transitions

Connect vertices. Each transition has one source and one target.

```table-editor
---
columns: { this: { label: "Transition" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix oml: <http://opencaesar.io/oml#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix state2: <https://www.modelware.io/sierra/state2#> .

state2:TransitionShape
    a sh:NodeShape ;
    sh:targetClass state2:Transition ;
    sh:property [
        sh:path state2:source ;
        sh:name "Source" ;
        sh:class state2:Vertex ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
    ] ;
    sh:property [
        sh:path state2:target ;
        sh:name "Target" ;
        sh:class state2:Vertex ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        oml:localReference true ;
    ] ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    .
```
