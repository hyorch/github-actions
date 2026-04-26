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

Failing steps.
```yaml
jobs:
  - step-fail:
    if: failure()
    run: echo "job failed"
  - step-ok:
    if: success()
    run: echo "job completed successfully"
    # cancelled()
    # always()
```

Conditional in jobs and steps. If .

```yaml
deploy:
    id: deploy
    run: ./deploy.sh
    outputs:
        success: ${{ steps.deploy.outcome == 'success'}}

report:
  if: deploy.outputs.success == 'true'
  run: echo "Deploy completed successfully"
```


```yaml
- name: cache dependencies
  id: cache
  uses: actions/cache@v3
  with:
    path: node_modules
    key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
- name: install dependencies
  if: steps.cache.outputs.cache-hit != 'true'
  run: npm ci
- name: test
  run: npm test

```

Matrix jobs.

```yaml
jobs:
  test:
    strategy:
      max-parallel: 1
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v5
      - name: Run tests
        run: echo "Testing on ${{ matrix.os }} node ${{ matrix.node-version }}"
```

Reusable Workflows

reusable.yaml
```yaml
.github/workflows/reusable.yml
name: Reusable Workflow
on:
  workflow_call:
    inputs:
      artifact-name:
        required: true
        type: string
    
    secrets:
      my-secret:
        required: true
        type: string

    outputs:
      success:
        description: 'true if job completed successfully' 
        value: ${{ steps.deploy.outcome == 'success'}}
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:     
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: ${{ inputs.artifact-name }}
```

```yaml
deploy:
  needs: build
  uses: ./.github/workflows/reusable.yml
  with:
    artifact-name: my-artifact
  secrets:
    my-secret: ${{ secrets.MY_SECRET }}
```



## Jobs and Docker Containers

## Custom Actions
Multiple steps can be grouped into a single action.
Types: JavaScript Actions (execute JS or NodeJS), Docker Actions(run Dockerfile), Composite Actions (combine workflow steps)

Custom actions are stored in the .github/actions/folder/action.yml directory. If repository is public, could be accessed by other repositories. If private, could be accessed by other repositories in the same organization.

```yaml
<account>/<repositoy>@<tag_or_hash>
my-account/my-action@v1
```

### Composite Actions

./.github/actions/cached-deps/action.yml
```yaml
name: Get & Cache Dependencies
description: Get and cache dependencies   
inputs:
  caching:
    description: 'Enable caching'
    required: false
    default: true
outputs:
  used-cache:
    description: 'true if cache was used' 
    value: ${{ steps.install.outputs.cache}}
  
runs:
  using: composite
  steps:
    - name: Cache dependencies
      if: inputs.caching == 'true'
      id: cache
      uses: actions/cache@v3
      with:
        path: node_modules
        key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
    - name: Install dependencies
      id: install
      if: steps.cache.outputs.cache-hit != 'true'
      run: |
        echo "cache='${{inputs.caching}}'">> $GITHUB_OUTPUT
        npm ci
      shell: bash
   
      
```

./github/workflows/main.yml
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:     
      - name: Get & Cache Dependencies
        id: cached-deps
        uses: ./.github/actions/cached-deps
        with:
          caching: true
      - name: Check if cache was used
        run: echo "Used cache? ${{ steps.cached-deps.outputs.cache}}"
```

### Docker Actions

Dockerfile that creates a container with the code enviroment we want (python, node, java, etc.). We run actions commands inside the container.

./.github/actions/deploy-s3-docker/action.yml
```yaml 
name: Deploy to AWS S3
description: Deploy to AWS S3
inputs:
  bucket:
    description: 'AWS S3 Bucket'
    required: true
  bucket-region:
    description: 'AWS S3 Bucket Region'
    required: false
    default: 'eu-west-1'
  dest-folder:
    description: 'Destination folder in S3'
    required: false
    default: './dist'

outputs:
  website-url:
    description: 'The URL of the deployed website'    
runs:
  using: docker
  image: Dockerfile 
```

```python
def run():
  bucket = os.getenv("INPUT_BUCKET")
  region = os.getenv("INPUT_BUCKET-REGION")
  folder = os.getenv("INPUT_DEST-FOLDER")

  website_url = f"https://{bucket}.s3.{region}.amazonaws.com/"
  print(f"website-url={website_url}", flush=True)

  
if __name__ == "__main__":
  run()

```



