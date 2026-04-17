# GitHub Actions

Github Actions: a workflow automation service by GitHub.
- Automate: building, testing, deployment, etc (CI/CD).
- Automate: code, repository management, issues, pull requests, projects, releases, etc.

https://github.com/academind/github-actions-course-resources


## Fundamentals
- Workflows: 1 or more jobs. Triggers upon Events

- Jobs: 1 or more steps. Executed on a Runner. Jobs run in parallel by default (Could be configured to run sequentially).

- Steps: instructions executed in order. Can be  **run commands** (shell commands or scripts) or **uses actions**. Can be conditional.

```yaml
name: <workflow-name>
on: <event>
jobs:
    job_name1:
        runs-on: <runner>
        steps:
            - name: <step-name>
                run: <command>
            - name: <step-name>
                uses: <action>
                with:
                    <key>: <value>
    job_name2:
        runs-on: <runner>
        steps:
            - name: <step-name>
                run: | # run Shell commands
                    <command1>
                    <command2>
```

### Events
Workflow Triggers

Events repository related: push, pull_request, create, fork, issues, issue_comment, watch, discussion, etc.
Other Events: workflow_dispatch, repository_dispatch, schedule, workflow_call.

- Push: when a push is made to the repository

```yaml
on: push
```
- Pull Request

```yaml
on: pull_request
```
- Schedule

```yaml
on: schedule
```
- Workflow Dispatch: run manually

```yaml
on: [workflow_dispatch,push]
```

### Actions
A custom application that performs a complex and frequently repeated task.

Could be created by your selft, shared free or paid on the GitHub Marketplace.

```yaml
jobs:
    build_source:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout code
              uses: actions/checkout@v4
              with:
                repository: <repository-name>
                ref: <branch-name>     
```

### Jobs parallel vs sequential
Jobs runs on parallel, unless you configure "needs:" dependency.

```yaml
jobs:
    job1:
        runs-on: ubuntu-latest
        steps:
            - name: Step 1
              run: echo "Step 1"
    job2:
        runs-on: ubuntu-latest
        steps:
            - name: Step 1
              run: echo "Step 1"
    job3:
        needs: job2
        runs-on: ubuntu-latest
        steps:
            - name: Step 1
              run: echo "Step 1"
```

### Expressions &  Context Objects
Expressions allos access to context objects:${{}}

```yaml
run: "${{ github.repository }}"
run: "${{ github.branch }}"
run: "${{ toJson(github) }}"
```


## Events
The **push** event filters. Activity types.

```yaml
on:
  push:
    branches:
      - main
    paths:
      - "**/*.js"
  pull_request:
    types: [opened, synchronize, reopened]
```

Skip some events (push) by adding annotations to commit:
```bash
git commit -m "added files do not build [skip ci]"
git push
```


## Job Artifacts and Outputs


## Evirontment Variables and Secrets
Env vars are used during workflow executions and code testing. They belongs to the runner environment, so code tested on during the workflow execution will use the same env vars.

```yaml
name:
on: [push]
env:
  PORT: 8080
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
    steps:
      - name: show env vars
        run: echo "$PORT"
      - name: show PORT from context
        run: echo "${{ env.PORT }}"
      - name: Run server
        run: npm start & npx waint-on http://localhost:$PORT
```

https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-variables#default-environment-variables

Repository level Action Secrets.

```yaml
env:
    MY_SECRET: ${{ secrets.MY_SECRET }}
    MY_VARIABLE: ${{ vars.MY_VARIABLE }}
jobs:
    build:
        steps:
            - name: show env vars
              run: |
                echo "$MY_SECRET"
                echo "${{ secrets.MY_SECRET }}"
                echo "${{ vars.MY_VARIABLE }}"
```

Environment Variables and Secrets in Actions. Define environment variables and secrets.

```yaml
jobs:
    build:
        environment: test
        env:
            MY_SECRET: ${{ secrets.MY_SECRET }}
            MY_VARIABLE: ${{ vars.MY_VARIABLE }}
    deploy:
        environment: production
        env:
            MY_SECRET: ${{ secrets.MY_SECRET }}
            MY_VARIABLE: ${{ vars.MY_VARIABLE }}
```
              
## Controlling Workflow and Job Execution

## Jobs and Docker Containers

## Custom Actions
Multiple steps can be grouped into a single action.
Types: JavaScript Actions (execute JS or NodeJS), Docker Actions(run Dockerfile), Composite Actions (combine workflow steps)


