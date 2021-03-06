# Orbis Azure Pipeline Tasks

Custom Azure Pipeline Tasks

---
## Dedupe Git Repositories Task

An Azure Pipelines task for deduplicating clones of Git repositories on self-hosted Windows agent servers.

### Build Status

[![Build Status](https://dev.azure.com/orbisinvestments/Open%20Source/_apis/build/status/Azure%20Pipeline%20Custom%20Tasks/Centralize%20Git%20Repositories%20Task?branchName=master)](https://dev.azure.com/orbisinvestments/Open%20Source/_build/latest?definitionId=1&branchName=master)

Works with Azure DevOps Server 2019 / Azure DevOps Services / TFS 2018 and self-hosted Windows Azure Pipelines Agents.

### Motivation

Given the scenario where multiple Azure Pipelines use the same Git repository for their sources, the Azure Pipelines Agent will create a seperate clone of the repository for each of the pipelines it executes. For large repositories with many pipelines this can result in agent working directories that consume significant amounts of disk space. More details can be found in this issue here: [microsoft/azure-pipelines-agent/1506](https://github.com/microsoft/azure-pipelines-agent/issues/1506)

A more efficient approach would be for the agent to share a single clone of the repository amongst all pipelines that use it. The **Dedupe Git Repositories Task** stubs this behaviour by running a *post job execution* script that deduplicates the pipeline's clone and repoints its sources directory at a shared clone of the same repository. 

### Prerequisites

Clones of repositories are shared between pipelines by symlinking. This requires the account the Azure Pipelines Agent runs under to have administrator privileges on the local machine.

### Installation

Head over to the latest Dedupe Git Repositories [release](https://github.com/OrbisInvestments/azure-pipelines-custom-tasks/releases/latest), then download and extract **DedupeGitRepos.zip** from the release’s assets. 

Use [tfx](https://github.com/Microsoft/tfs-cli) to upload the extracted task to your Azure DevOps/TFS account or collection with an identity that is in the admin role at the *All Pools* level or a PAT with the *Agent Pools (Read & Manage)* scope:

    cd drive:\path\to\extracted\zip
    tfx login
    tfx build tasks upload --task-path .

*Note: for on-premise installations tfx is not compatible with self-signed TLS certificates. `tfx login` will store credentials in plain text on disk, you can alternatively pass credentials as part of the `tfx build tasks upload` command*

### How to use

Once uploaded to your account/collection add the **Dedupe Git Repositories** task to each pipeline that builds the repository you want to deduplicate. The task can be placed in any position in the job, it will always be executed once the job is completed.

For efficient deduplication of all clones, the step will need to be added to each agent job in a multi-job pipeline.

### How it works

When executed the Dedupe Git Repositories Task will determine if the repository clone that is used by the pipeline is a *shared* clone. If not, then either:

1. the pipeline's clone is moved into a shared location (Agent.WorkFolder\g), if a clone for the same repository does not already exist. This becomes the *shared* clone. 

Or

2. the pipeline's clone is deleted if a clone in the shared location does already exist
    
Once this is done the task symlinks the pipeline's sources directory to that of the shared clone. All subsequent executions of the pipeline on the agent will use the shared clone for its sources. 

To avoid concurrency issues shared clones are created [per-agent](https://github.com/microsoft/azure-pipelines-agent/issues/1506#issuecomment-381361454).

### Considerations

If a subset of your pipelines *clean sources* the shared clone will be deleted when they run. This will mean other pipelines that do not *clean sources* will end up having to reclone the repository into the shared location. 

## License

The Orbis Azure Pipeline Tasks project is licensed under the [MIT License](LICENSE)

## Credits

[Git Logo](./DedupeGitReposV0/icon.png) by [Jason Long](https://twitter.com/jasonlong) is licensed under the [Creative Commons Attribution 3.0 Unported License](https://creativecommons.org/licenses/by/3.0/).

---




