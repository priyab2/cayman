---
layout: default
---

# Design Principles
## Modularity
This workflow is modular by design, with each bioinformatics task in its own module. WDL makes this easy by defining "tasks" and "workflows." Tasks in our case will wrap individual bioinformatics steps comprising the workflow. Tasks can be run individually and also strung together into workflows.

The variant calling workflow is complex, so we break it up into smaller subworkflows, or stages that are easier to develop and maintain. Stages can be run individually and also called sequentially to execute the workflow fully or partially.

Reasons for modular design:

* flexibility: can execute any part of the workflow
    *useful for testing or after failure
    *can swap tools in and out for every task based on user's choice
* optimal resource utilization: can specify ideal number of nodes, walltime, etc. for every stage
* maintainability: can edit modules without breaking the rest of the workflow
    *modules like QC and user notification, which serve as plug-ins for other modules, can be changed without updating multiple places in        the workflow
