# Assignment 4 Documentation:

---
## AI-usage statement

I used an AI coding agent for this assignment. The agent wrote the statechart page, the Fire Force charts, and this note. I reviewed the files and checked the validation in the editor.

- **IDE:** Cursor.
- **Model:** Grok 4.7.
- **Connection:** Cursor talks to the OML model through MCP. The project file [.cursor/mcp.json](.cursor/mcp.json) starts [scripts/oml-mcp](scripts/oml-mcp), which runs the OML MCP server against this folder. That let the agent search the Sierra and Fire Force models, create instances, and run validation, instead of only editing text.
- **What we did:** Looked at the existing Sierra templates and the Fire Force system. Picked statecharts, because no template covered that pattern yet. Finished the `state2` vocabulary, wrote the authoring page, and created three real charts. Broke the one-entry rule on purpose, read the message, and deleted the extra entry.
- **Debugging:** The first transitions were saved as the wrong kind of instance, so the file would not pass the OML check until they were rewritten as normal instances. The MCP validation result counted the error but did not show the sentence, so we asked the local OML server directly and saw the message on `WardenRegion`. Some links in this note pointed at folders or line numbers and opened as unavailable; those were changed to the file that actually contains the thing. 
---

Cmd-click any link to open the file where that thing is defined.

Sierra already had pages for [stakeholders](src/method/md/www.modelware.io/sierra/context-analysis/stakeholders.md), [missions](src/method/md/www.modelware.io/sierra/context-analysis/missions.md), [processes](src/method/md/www.modelware.io/sierra/operational-analysis/processes.md), flat [state machines](src/method/md/www.modelware.io/sierra/operational-analysis/statemachine.md), [scenarios](src/method/md/www.modelware.io/sierra/operational-analysis/scenarios.md), [components](src/method/md/www.modelware.io/sierra/system-analysis/components.md), and so on. Those pages are templates: a form you fill in, and the form writes the model.

This assignment adds one new form, for **statecharts**. A statechart is a diagram of modes. Example: the [AI Fire Warden](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/entities.oml) can be observing, advising, waiting for approval, or executing an action. The old [state-machine page](src/method/md/www.modelware.io/sierra/operational-analysis/statemachine.md) cannot edit this kind of chart, because a statechart has an extra layer: the chart contains a [region](src/method/oml/www.modelware.io/sierra/state2.oml), and the region contains the states.

## The idea in one picture

- [AI Fire Warden](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/entities.oml) (already existed; you cannot edit it on this page)
  - [AIFireWardenChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) (new)
    - [WardenRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)
      - [WardenEntry](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)
      - [WardenObserving](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)
      - [WardenAdvising](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)
      - [WardenAwaitingApproval](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)
      - [WardenExecutingAction](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml)

Arrows between those states (entry → observing → advising → …) are in a second table on the same page. They are not part of the tree. The first arrow is [WardenStart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml).

## Files that were edited

| File | What it is |
| --- | --- |
| [state2.oml](src/method/oml/www.modelware.io/sierra/state2.oml) | The vocabulary. Two small additions: a region can point back to its chart ([inComposite](src/method/oml/www.modelware.io/sierra/state2.oml)), and a chart can point at the entity it belongs to ([isAllocatedTo](src/method/oml/www.modelware.io/sierra/state2.oml)). |
| [Sierra bundle](src/method/oml/www.modelware.io/sierra/bundle.oml) | Tells the rest of Sierra that the statechart vocabulary exists. |
| [statechart.md](src/method/md/www.modelware.io/sierra/operational-analysis/statechart.md) | The form. This is the authoring page. |
| [charts.oml](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) | The three real Fire Force charts. |
| [Fire Force bundle](src/model/oml/fireforce6.github.io/mission-control/bundle.oml) | Includes that chart file in the Fire Force model. |
| [Statecharts.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Statecharts.md>) | The page you open. It pulls the form in. |
| [Overview.md](<src/model/md/Fire Force VI/Mission Control/Overview.md>) | Adds a link: **Define Operational Statecharts**. |

## The three charts

1. [AIFireWardenChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) — observing, advising, waiting for approval, executing.
2. [DashboardChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) — idle, monitoring, handling an alert, waiting for a decision.
3. [FireCloudChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) — idle, streaming telemetry, degraded.

## The one rule

Every [region](src/method/oml/www.modelware.io/sierra/state2.oml) must contain exactly one [entry](src/method/oml/www.modelware.io/sierra/state2.oml). Zero entries or two entries is wrong. The message is written here: [A region must contain exactly one entry pseudostate.](src/method/md/www.modelware.io/sierra/operational-analysis/statechart.md)

---

# How to validate it works

Open [Statecharts.md](<src/model/md/Fire Force VI/Mission Control/Operational Analysis/Statecharts.md>) in the OML markdown preview.

1. The tree shows three charts under the systems that already existed: [AIFireWardenChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml), [DashboardChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml), and [FireCloudChart](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml). Each chart has one region, and that region has one entry.
2. Add a second entry under [WardenRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml). In the tree, add an Entry named `WardenExtraEntry`. Or paste this into [charts.oml](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) after [WardenEntry](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml):

```oml
    instance WardenExtraEntry : state2:Entry [
        state2:region WardenRegion
        base:description "Second entry, added to show the rule."
    ]
```

3. Save, then run Validate on the page. The message names [WardenRegion](src/model/oml/fireforce6.github.io/mission-control/operational-analysis/statecharts/charts.oml) and says [A region must contain exactly one entry pseudostate.](src/method/md/www.modelware.io/sierra/operational-analysis/statechart.md)
4. Delete `WardenExtraEntry` and validate again. The message is gone.
