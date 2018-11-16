---
layout: default
---

# Design Principles
## _Modularity_
This workflow is modular by design, with each bioinformatics task in its own module. WDL makes this easy by defining "tasks" and "workflows." Tasks in our case will wrap individual bioinformatics steps comprising the workflow. Tasks can be run individually and also strung together into workflows.

The variant calling workflow is complex, so we break it up into smaller subworkflows, or stages that are easier to develop and maintain. Stages can be run individually and also called sequentially to execute the workflow fully or partially.

Reasons for modular design:

- Flexibility: can execute any part of the workflow
   - Useful for testing or after failure
   - Can swap tools in and out for every task based on user's choice
- optimal resource utilization: can specify ideal number of nodes, walltime, etc. for every stage
- maintainability: can edit modules without breaking the rest of the workflow
    - Modules like QC and user notification, which serve as plug-ins for other modules, can be changed without updating multiple places in        the workflow
## _Data parallelism and scalability_
The workflow should run as a single multi-node job, handling the placement of tasks across the nodes using embedded parallel mechanisms. Support is required for:

- Running one sample per node
- Running multiple samples per node on clusters with and without node sharing.

The workflow must support repetitive fans and merges in the code (conditional on user choice in the runfile):
- Splitting of the input sequencing data into chunks, performing alignment in parallel on all chunks, and merging the aligned files per       sample for sorting and deduplication
- Splitting of aligned/dedupped BAMs for parallel realignment and recalibration per chromosome.

The workflow should scale well with the number of samples - although that is a function of the Cromwell execution engine. We will be benchmarking this feature (see Testing section below).

## _Real-time logging and monitoring, data provenance tracking_
The workflow should have a good system for logging and monitoring progress of the jobs. At any moment during the run, the analyst should be able to assess:

- Which stage of the workflow is running for every sample batch
- Which samples may have failed and why
- Which nodes the analyses are running on, and their health status.
Additionally, a well-structured post-analysis record of all events executed on each sample is necessary to ensure reproducibility of the analysis.

## _Fault tolerance and error handling_
The workflow must be robust against hardware/software/data failure. It should:

- Give the user the option to fail or continue the whole workflow when something goes wrong with one of the samples
- Have the ability to move a task to a spare node in the event of hardware failure.

The latter is a function of Cromwell, but the workflow should support it by requesting a few extra nodes (beyond the nodes required based on user specifications).

To prevent avoidable failures and resource waste, the workflow should:

- Check that all the sample files exist and have nonzero size before the workflow runs
- Check that all executables exist and have the right permissions before the workflow runs
- After running each module, check that output was actualy produced and has nonzero size
- Perform QC on each output file, write results into log, give user option to continue even if QC failed.

User notification of success/failure will be implemented by capturing exit codes, writing error messages into failure logs, and notifying the analyst of the success/failure status via email or another notification system. We envision three levels of granularity for user notification:

- total dump of success/failure messages at the end of the workflow
- notification at the end of a stage
- notification at the end of a task.

The level of granularity will be specified by users as an option in the runfile.
