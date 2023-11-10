# Reusable-GHA
Utilize a workflow from one repository to other repositories




## Github Actions 🚀 and Reusable workflows

GitHub Actions is a CI/CD tool to automate the builds and deploy it in a pipeline. Workflows are written to execute the tasks through actions and it can be done from the Github itself without any third party tool integration or any alternative. This article explains how to add jobs to the workflows and automatically deploy them to Artifactory on every push or sync to the Github repository.


Workflows can be made reusable, eliminating the need to copy and paste information between one workflow to another. the reusable workflow can be called from another workflow by everyone else who has access to it. This avoids duplication and can be centrally managed. Workflow Reuse is an effective way to work with similar Github actions. upload-artifacts is a reusable workflow and it is configured to be called from other workflows with build & deploy jobs.


A workflow that uses another workflow is referred to as a "caller" workflow. The reusable workflow is a "called" workflow. One caller workflow can use multiple called workflows (upto 20). Each called workflow is referenced in a single line with uses context. The result is that the caller workflow file may contain just a few lines of YAML but performs a large number of tasks when it's run. When you reuse a workflow, the entire called workflow is used, just as if it was part of the caller workflow.



 Path of the workflow:
{org}/{repository-name}/.github/workflows 

Code is available at:
 Reusable workflow - To centrally manage

 Caller workflow - For calling the reusable workflow / To Developer

## Deep-Dive of Called / ReUsable Workflow

an overview of the workflow's functionality and explaining how to trigger and configure it. As we look at the code, it gives an idea that workflow automates the process of creating a tarball of the project's artifacts and publishing them to Artifactory on specified events. This reusable workflow approach promotes code reuse and maintains a clean separation of concerns.

## Configuration


The first few lines of the workflow defines a name to the action and when to run the jobs.

```yaml
on:
  workflow_call:
    inputs:
      artifact-path: 
        description: 'custom path for publishing artifacts'
        required: true
        type: string
  workflow_dispatch:
```

#### Workflow Triggers
##### workflow_call: 
This denotes that the workflow can be called by another workflow. inputs and outputs also secrets can be defined and the same can be used in caller workflow. We used artifact-path as input to reusable workflow, which can be feed in caller workflow.

##### workflow_dispatch:
to manually trigger the workflow from Github UI, or CLI or through API, workflow_dispatch event would be useful.


> From the rest of the code, We have used env, Jobs, steps with multiple actions needed to build and publish the build.

env:
to define the variables explicitly that can be called from run: event. here we have used python version and other secrets and defined some custom variables for artifact-path and URL of artylab. more information can be read at Workflow syntax for GitHub Actions

## Workflow Structure 
jobs:
Job in GitHub Actions is a set of steps that execute on the same runner. Jobs are the basic building blocks of a workflow, and they can be run in parallel or in sequence. Job has it's name, runs-on, needs, id, steps, uses and many other properties. to talk about each in detail..

build: Job name

runs-on: runner label to perform the execution of steps configured in a job. eg-default is one of the runner label from expedia's github runner list. more runners can be viewed at 

needs: if there is a dependency from previous job, needs parameter is useful create a linear CI/CD pipeline.

id: to give a job unique identifier, by shortening the name of job or similar. 

shell: if the code logic is being written in particular language like bash, python; shell is an environment to setup to run the code. workflows also supports powershell

steps: number of tasks with it's own properties under steps indent. 

name: task name to the commands/ action that is being executed

run: uses the runner's OS shell to execute commands, this is the area to add commands for installing/building etc.,

with: A map of the input parameters defined by the action



### Actions

#### checkout-code : 
This action checks-out your repository under $GITHUB_WORKSPACE, so your workflow can access it.  Currently it is set to latest version v4. setting fetch-depth: 0 to fetches all history for all branches and tags. It also has more options. read more at actions/checkout

#### setup-python: 
This action provides the Installing a version of Python or PyPy and (by default) adding it to the PATH functionality for GitHub Actions users. Currently it is set to latest version v4. with: property will enable to use python-version from env: this allows us to upgrade version in future. read more at actions/setup-python

#### cache: 
This action allows caching dependencies and build outputs to improve workflow execution time. Currently it is set to latest version v3. read more at actions/cache





#### Other steps:

installing-dependencies will install required libraries for python to run the code.



#### Job.Publish
Step: Create tarball

This will create a tar archive after checked out to current github repository with a workflow run number as tag to the release.

Step: Publish to Artylab

This will use JFrog API to upload the tarred file with gz format to artifactory. Required variables should be parsed from environment and the output would be printed with logger library.

#### Jobs

Finally, we are validating the published artifacts with HTTP status code and exiting if there is an error uploading artifacts at same path with a clear message to retry after correcting the path.

***

### Usage & Customization ⚙️


In future, More actions can be added to the workflow and can be tested using workflow_dispatch event by manually triggering the workflow to validate the actions. Add more events like push: to test the workflow on each sync to the repository.



Add below code in yaml template under .github/workflow path and make desired changes for artifact-path and events. file can be in same repository or in other org repository.

Caller workflow example

caller-workflow.yaml


```yaml
---
name: Artifacts
 
on:
  workflow_dispatch: # to trigger workflow manually
  push: # on every push to the repo
#     branches:
#       - "feature/workflow"
#   pull_request:
#     types:
#       - closed
    
jobs:
  call-workflow:
    uses: org/repo-name/.github/workflows/artifacts.yaml@feature/artifacts # calling reusable workflow
    with:
      artifact-path: "test/test2" # path to upload artifacts
```

As soon as above code is added or pushed to the feature / dev branch, github workflow will initiate the actions and triggers the jobs to call the reusable workflow and runs build and publish steps. A sample output is added below.






---
**Note**

* workflow_dispatch  event would work if the workflow yaml file is only at default branch such as main/master or develop.
* Secrets can be called from called workflow but the environment variables / repository variables defined under called workflow repository cannot be retrieved to caller workflow if the caller workflow is from different repository. Only organization variables can be retrieved.
* one single caller workflow file can call maximum 20 reusable workflow as per the limitations
* called workflow can be called upto 4 times, nested reusable workflows. called workflow(A) → caller workflow(B) → caller workflow(C) → caller workflow(D) 
* The caller workflow cannot use GitHub-hosted runners from the called repository.
* Re-running failed jobs or a specific job in a workflow will use the reusable workflow from the same commit SHA of the first attempt.
* job in a workflow can run for up to 6 hours and workflow run is limited to 35 days. if it reaches the limit, workflow gets cancelled.
---


***

### Error Handling & Logs 🛠️
Logging

:lightbulb: Expand the corresponding job to view the workflow logs, which show the majority of the issues. If any steps in a job failed, it expanded to show the outcomes automatically. Finding specific step logs would be possible using the "Search logs" option.




***

### Glossary 📖

GHA	| Github Actions
ENV VAR	| Environment Variables
YAML | Ain't Markup Language






### Related links


Understanding GitHub Actions - GitHub Docs

Workflow syntax for GitHub Actions - GitHub Docs

Events that trigger workflows - GitHub Docs

Manually running a workflow - GitHub Docs

Reusing workflows - GitHub Docs

About monitoring and troubleshooting - GitHub Docs

Re-running workflows and jobs - GitHub Docs

Variables - GitHub Docs

Usage limits, billing, and administration - GitHub Docs 


