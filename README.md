<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable-next-line MD041 -->
<p align="center">
  <h1 align="center" style="font-size: 4rem;">GitHub Actions Training</h1>
  <p align="center" style="font-size: 1.5rem;">
    Automate your tests, builds, releases, and more on every push!
  </p>
  <p align="center">
    <a href="https://github.com/features/actions"><img alt="GitHub Actions" src="https://img.shields.io/badge/TRAINING ON-GITHUB ACTIONS-2088FF?style=for-the-badge"></a>
    <a href="https://www.mares.cz"><img alt="Vojtěch Mareš" src="https://img.shields.io/badge/TRAINING BY-Vojtěch Mareš-fd9a00?style=for-the-badge"></a>
  </p>
  <p align="center"><a href="https://www.mares.cz">Vojtěch Mareš</a> | <a href="mailto:vojtech@mares.cz">vojtech@mares.cz</a></p>
</p>
<!-- markdownlint-enable MD033 -->

## What is Continuous Integration (CI)

### Git

Git is the most popular Version Control System (VCS) in the world, created by Linus Torvalds for the Linux Kernel development, since the tools available at the time were not up to the task according to Linus.

### GitHub

GitHub is Git repository hosting platform and today's de-factor home of (almost) all open-source software.

### Continuous Integration (CI)

Continuous Integration is a DevOps process, of always testing every version of code pushed to the repository by running tests, building the application, etc.

### Use case for CI

- automated tests
- automated builds
- (almost) immediate feedback on fail
- automated Issue and PR management (labels, auto-close,...)

## GitHub Actions

### GitHub Actions features

- Built around reusability
- Easy customization
- Use of existing components
- Supported by GitHub.com (SaaS and Enterprise Cloud)
- Supported by self-hosted GitHub (GitHub Enterprise Server)

### GitHub Actions architecture

Event (Trigger) -> Job(s) -> Runner -> Execute Job

### Runner(s)

- **Managed**: GitHub.com managed runners
  - linux runners (Ubuntu)
  - macOS runners (paid only)
  - Windows runners (paid only)
  - Limited sizing options
- **Self-hosted**
  - systemd unit on Linux
  - [Actions Runner Controller (ARC)](https://github.com/actions/actions-runner-controller) for Kubernetes
  - third-party providers of managed runners

## Actions

Action is a reusable component. You can use pre-existing actions, see [GitHub Marketplace](https://github.com/marketplace?type=actions) or build your own.

In each step of a workflow, only a single action or composable action can be used.

## Using existing actions

To use an existing action, just reference it in your workflow with `uses` directive.

For example:

```yaml
# uses <org>/<repo>@<ref>
uses: actions/checkout@v4
```

## Workflows

Each workflow resides in a single file inside the `.github/workflows/` directory.

A workflow has the following required parameters:

- **name**: name of the workflow visible in the UI
- **trigger** (`on`): what triggers and what does not trigger the workflow
- **jobs**: a list of steps executed in order

The workflow file has well defined structure:

```yaml
# Workflow name visible in 'Actions' tab on repository page
name: xxx

# Triggers
on:
  push:
    branches:
      - '*'

# Jobs
jobs:
  first:
    runs-on: ubuntu-latest

    steps:
      - name: xxx
        run: |
          echo "I am the first step!"

  second:
    needs: [ "first" ]
    runs-on: ubuntu-latest

    steps:
      - name: yyy
        run: |
          echo "I am first step of second job"
```

Each job is composed of multiple steps. A job can have if condition to further control its execution.

Each step can also have its if condition.

## Visual Studio Code extensions

- [GitHub Actions](https://marketplace.visualstudio.com/items?itemName=github.vscode-github-actions) by GitHub: Provides intellisense and sidebar plugin to see Workflow runs
- [YAML](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-yaml) by RedHat: correct YAML syntax highlighting

Or see and use [`.vscode/extensions.json`](/.vscode/extensions.json).

## Writing workflows

### Simple workflow

```yaml
name: Simple workflow

on: [push]

jobs:
  hello-world:
    runs-on: ubuntu-latest

    steps:
      - name: Say hello
        run: echo "Hello, world!"
```

### Jobs

Each job is executed on a single runner. If your job is running in GitHub managed runners, each runner runs exactly one job, and the runner virtual machine is removed after the job is finished.

### Steps

Step can either execute a shell script in given shell, which you must specify. Pr use an existing action and supply its inputs, if it has any.

```yaml
jobs:
  my-job:
    # ...
    steps:
      - name: First step # name is optional
        shell: bash # bash, sh, pwsh, python, cmd
        run: |
          echo "Hello everyone from bash"

      - name: Second step with powershell
        shell: pwsh
        run: |
          Write-Output 'Hello everyone from pwsh'
```

If you are using action, do not forget to tag the action version via `@` notation (e.g. `docker/login-action@v2`). You can use any version tag, branch name or lock it down to a commit hash, but that is difficult to read by humans and identify the version used.

### Clone repository with step that uses actions/checkout

By default, the repository is never cloned/checkout to the workspace of the runner. That must be done explicitly via a step that uses [actions/checkout](https://github.com/actions/checkout) action.

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4
```

### Triggers

When a workflow is executed.

Common cases are:

- **Push**

    ```yaml
    name: On push

    on:
      push:
        branches:
          - main
        ignore-branches:
          - '*'
        # tags:
        # ignore-tags:
    ```

- **Pull request**

    ```yaml
    name: On pull request

    on:
      pull_request:
        branches:
          - main # only on PRs that target 'main' branch
    ```

- **Schedule** (cron)

    ```yaml
    name: On schedule

    on:
      schedule:
        # * is a special character in YAML so you have to quote this string
        - cron: '30 5,17 * * *'
        # Multiple cron expressions can be supplied
        - cron: '30 5 * * 1,3'
        - cron: '30 5 * * 2,4'
    ```

- **Issue**

    ```yaml
    name: On issue

    on:
      issues:
        types: [opened, closed, edited, milestoned]
    ```

    For list of types, see [docs](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#issues).

- **Tag (release)**

    ```yaml
    name: On push of tag

    on:
      push:
        tags:
          - 'v*'
    ```

- **Previous workflow**

    ```yaml
    name: After previous workflow finishes

    on:
      workflow_run:
        workflows: [ "Test" ]
        types: [ "completed" ]

    jobs:
      build:
        runs-on: ubuntu-latest
        # without this if, workflow will run regardless if the previous workflow succeeded or not
        if: github.event.workflow_run.conclusion == 'success'
        # ...
    ```

- **Changes in path**

    ```yaml
    name: On path changes

    on:
      push:
        paths:
          - src
          - tests
    ```

GitHub docs:

- [Triggering a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow)
- [Events that trigger workflows](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows) contains a list of all triggers

### Needs

When a job _needs_ some other job within the workflow (pipeline) to run before, use keyword `needs` and list of names of the depending jobs.

```yaml
on: Needs example

jobs:
  first:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Hello world"

  second:
    runs-on: ubuntu-latest

    needs: [first]

    steps:
      - run: echo "Hello world from second job"
```

### Contexts

Contexts are groups of related variables, that you can use in your workflows to write ifs, reference environment variables, secrets, environments, or get metadata from runner.

Usually used contexts are:

- **github** - references to Git ref, triggering event, etc.
- **env** - environment variables, can also be referenced with classic shell notation `$ENV_VAR`
- **secrets** - only way to access secrets

For more information, see [docs](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/accessing-contextual-information-about-workflow-runs).

**Using contexts**:

Contexts and other GitHub Actions built-in function are accessible inside the `${{ env.MY_VAR }}` brackets. That way it is clearly distinguishable from regular shell variables. Also with GitHub Actions Visual Studio Code extension, you get syntax highlighting and intellisense. For more examples, see [Conditionals](#composite-actions).

### Builtin functions

GitHub Actions have several builtin functions, that can be used inside _expressions_. Expression is `${{ <expression> }}` block. An expression can call a function, use logical operator to return a boolean value or filter objects.

**Functions**:

- `contains( search, item )`: returns `true` if `search` contains `item`
- `startsWith( searchString, searchValue )`: returns `true` if `searchString` starts with `searchValue`, aka prefix
- `endsWith( searchString, searchValue )`: returns `true` if `searchString` ends with `searchValue`, aka suffix
- `format( string, replaceValue0, replaceValue1, ..., replaceValueN)`: replaces values in the `string`, with the variable `replaceValueN`
- `join( array, optionalSeparator )`: The value for array can be an array or a string. All values in array are concatenated into a string, default separator is `,`
- `toJSON(value)`: returns a pretty-print JSON representation of `value`, useful for debugging of [Contexts](#contexts)
- `fromJSON(value)`: returns a JSON object or JSON data type for value
- `hashFiles(path)`: returns a single hash for the set of files that matches the `path` pattern, you can provide a single path pattern or multiple path patterns separated by commas

**Logical operators**:

- `( )`: logical group
- `[ ]`: index (array access)
- `.`: property de-reference
- `!`: not
- `<`: less than
- `>`: more than
- `==`: equal
- `!=`: not equal
- `>=`: more or equal
- `<=`: less or equal
- `&&`: and
- `||`: or

**Filtering objects**:

Example object:

```json
[
  { "name": "apple", "quantity": 1 },
  { "name": "orange", "quantity": 2 },
  { "name": "pear", "quantity": 1 }
]
```

Filter:

```text
fruits.*.name
```

Output:

```json
[ "apple", "orange", "pear" ]
```

For more information, see [docs](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/evaluate-expressions-in-workflows-and-actions).

### Conditionals

Each job and/or step can contain an `if` statement, to further control execution.

For example, if you are reusing generic workflows for `main` branch and pull requests, you may not want to deploy to from a pull request.

**Job with `if` example**:

```yaml
name: if repository

on: [push]

jobs:
  production-deploy:
    # run job only in main repository, not on forks, since forks have different org and/or repo name
    if: github.repository == 'octo-org/octo-repo-prod'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
```

**Step with `if` example**:

```yaml
steps:
  - name: My first step
    if: ${{ github.event_name == 'pull_request' && github.event.action == 'unassigned' }}
    run: echo This event is a pull request that had an assignee removed.
```

### Defaults

Defaults are workflow wide configurations, so you do not have to repeat the configuration per job or per step.

The default options are:

- `defaults.run.shell`
- `defaults.run.working-directory`

Defaults can be defined at workflow or at job level.

Job defaults:

```yaml
job:
  job-a:
    defaults:
      run:
        shell: pwsh
        working-directory: win
    # ...
```

### Working directory

By default the working directory is the repository root directory, in some cases you may want to change this behavior.

Such change works for the shell steps.

> [!IMPORTANT]
> You must make sure, that the directory exists.

```yaml
# defaults for entire workflow
defaults:
  run:
    working-directory: /some/path

# per job
jobs:
  job-alice:
    runs-on: ubuntu-latest
    working-directory: /some/path

# per step
jobs:
  job-bob:
    runs-on: ubuntu-latest

    steps:
      - working-directory: /some/path
        shell: bash
        run: ls
```

### Concurrency

Concurrency allows you to control, if a certain workflow or job should run in parallel and even between different workflow runs.

For example you may not want to deploy an older version of an application if a new version was created in the meantime and is suppose to be deployed to an environment.

Default behavior:

```yaml
on:
  push:
    branches:
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

Concurrency groups:

```yaml
jobs:
  job-1:
    runs-on: ubuntu-latest
    ## job-level concurrency
    concurrency:
      group: staging_environment
      cancel-in-progress: true

# OR

## workflow concurrency (for all jobs in workflow)
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

Fallback value:

```yaml
concurrency:
  group: ${{ github.head_ref || github.run_id }}
  cancel-in-progress: true
```

Only cancel in-progress on specific branches:

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ !contains(github.ref, 'release/')}}
```

### Environment variables

Environment variables for Organizations are located at: _[Organization] Settings -> Secrets and variables (left sidebar) -> Actions (dropdown menu)_.

Environment variables for Repositories are located at: _[Repository] Settings > Secrets and variables (left sidebar) > Actions (dropdown)_.

Environment variables are for non-secret values only, they can be defined at many levels:

- **Organization**: environment variable available for all repositories and workflows within the organization
- **Repository**: environment variables defined in settings of a repository
- **Repository Environment**: each repository can have environments for deployments and environment variables can be environment-specific and available only to jobs/steps that are deploying to the environment
- **Workflow**: environment variables defined under `.defaults.env` available for all jobs in a workflow
- **Job**: environment variable defined per job `.jobs.<job_id>.env`
- **Step**: environment variable defined per step `.jobs.<job_id>.steps[*].env`
- **Dynamic**: environment variables can be declared dynamically in a step and the be available for all consequential steps in the job

    ```yaml
    name: Dynamic environment variable

    on:
      push:

    jobs:
      how-to-declared-env-var:
        runs-on: ubuntu-latest

        steps:
          - name: Declare env var
            run: |
              echo "HELLO=world" >> $GITHUB_ENV

          - name: Use env var
            run: |
              echo $HELLO # prints "world"
    ```

**Overrides**:

Secrets have a certain hierarchy when overriding with multiple configuration sources, this hierarchy goes as follows: _Dynamic environment variable > Step environment variable > Job environment variable > Workflow environment variable > Repository Environment environment variable > Repository environment variable > Organization environment variable_.

### Secrets

Secrets are special values which are securely stored in GitHub and redacted from logs. Making it harder to leak them.

Secrets for Organizations are located at: _[Organization] Settings -> Secrets and variables (left sidebar) -> Actions (dropdown menu)_.

Secrets for Repositories are located at: _[Repository] Settings > Secrets and variables (left sidebar) > Actions (dropdown)_.

There are few options, where a secret can be defined:

- **Organization**: each GitHub organization can have its secrets, which are available to all repositories under the organization
- **Repository**: secrets available only for a single repository (do not forget that secrets can be passed to workflows even when workflow resides in different repository)
- **Repository Environment**: each repository can have environments for deployments and secrets can be environment-specific and available only to jobs/steps that are deploying to the environment

**Overrides**:

Secrets have a certain hierarchy when overriding with multiple configuration sources, this hierarchy goes as follows: _Repository Environment secret > Repository secret > Organization secret_.

### GitHub token (`secrets.GITHUB_TOKEN`)

The value of `secrets.GITHUB_TOKEN` is a token generated by GitHub for every job. The token is short lived for the duration of a single job. After the job is finished, the token is invalidated.

### Permissions

Permissions are list of privileges that `secretes.GITHUB_TOKEN` has attached to itself.

```yaml
permissions:
  actions: read|write|none
  attestations: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: write|none
  issues: read|write|none
  discussions: read|write|none
  packages: read|write|none
  pages: read|write|none
  pull-requests: read|write|none
  repository-projects: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```

A short explanation and use case for each permission:

- **actions**: interact with GitHub Actions, `actions: write` can cancel action
- **attestations**: working with artifact attestations, `attestations: write` let's you create attestation for artifact for a build
- **checks**: work with check runs and check suites, `check: write` permits an action to create a check run
- **content**: interact with repository, commits, etc., `content: read` permits action to list commits, `content: write` permits action to create a release
- **deployments**: work with deployments, `deployments: write` permits action to create a deployment
- **id-token**: fetch an OpenID Connect (OIDC) token, requires `id-token: write`
- **issues**: work with issues, `issues: write` let's action to write a comment for example
- **discussions**: work with discussions, `discussions: write` permits action to close a discussion
- **packages**: work with packages, `packages: write` permits action to upload and publish a package
- **pages**: work with pages, `pages: write` permits action to trigger GitHub Pages build
- **pull-requests**: work with pull requests, `pull-request: write` permits action to comment, open, close or add label
- **repository-projects**: work with repository projects (classic), `repository-projects: write` permits action to add a column to a repository project (classic)
- **security-events**: work with GitHub code scanning and Dependabot alerts, `security-events: read` permits action to list alerts, `security-events: write` allows the action to update the status of code scanning alert
- **statuses**: work with commits statuses, `statuses: read` permits an action to list the commit statuses for a given reference

A common use is for **packages** to push Docker image to GitHub container registry. **Issues** and **pull requests** for automated labeling.

For more information, see [docs](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#permissions).

### Continue on error

In some cases you want a step/job as an optional, that the failure of the step/job does not fail the job/workflow as a whole.

```yaml
steps:
  - name: First step
    run: echo "Hello world"

  - name: I am allowed to fail
    continue-on-error: true
    run: echo "Hello world" && false
```

### Machine on which job is running

To select a machine to run the job on, use the `runs-on`.

The `runs-on` directive is also used to select self-hosted runners instead of the GitHub managed (default).

`runs-on` can also be a list, the workflow will be executed by a runner that matches all of the criteria.

```yaml
runs-on: [self-hosted, linux, x64, gpu]
```

### Strategy

> Use `jobs.<job_id>.strategy` to use a matrix strategy for your jobs. A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables. For example, you can use a matrix strategy to test your code in multiple versions of a language or on multiple operating systems.

Multiple matrix (multi-dimensional):

```yaml
jobs:
  example_matrix:
    strategy:
      matrix: # this matrix will generate 9 jobs
        version: [10, 12, 14]
        os: [ubuntu-latest, windows-latest]
```

Single dimensional:

_This is commonly used for testing on multiple versions of runtime, like Node.js or dotnet._

```yaml
jobs:
  example_matrix:
    strategy:
      matrix: # this matrix generates 4 jobs
        version: [18, 20, 22, 23] # last 3 LTS releases and latest stable
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.version }}
```

### Services

> Used to host service containers for a job in a workflow. Service containers are useful for creating databases or cache services like Redis. The runner automatically creates a Docker network and manages the life cycle of the service containers.

```yaml
services:
  nginx:
    image: nginx
    # Map port 8080 on the Docker host to port 80 on the nginx container
    credentials:
      username: ${{ secrets.SERVICES_USERNAME }} # must be provided manually in Secrets
      password: ${{ secrets.SERVICES_PASSWORD }} # must be provided manually in Secrets
    ports:
      - 8080:80
  redis:
    image: redis
    # Map random free TCP port on Docker host to port 6379 on redis container
    ports:
      - 6379/tcp
steps:
  - run: |
      echo "Redis available on 127.0.0.1:${{ job.services.redis.ports['6379'] }}"
      echo "Nginx available on 127.0.0.1:${{ job.services.nginx.ports['80'] }}"
```

## Environments

GitHub offers environments. Environments support environment-specific environment variables and secrets (mentioned before at [Environment variables](#environment-variables) and [Secrets](#secrets)).

Environment also provides a UI element on repository's homepage (right sidebar, _Deployments_. _Deployments_ belong to environments).

Environments are often integrated with third-party services such as Vercel or Cloudflare Pages, to show a deployment status.

## Reusable workflows

You do not have to repeat yourself for every case, you can build reusable workflows, that can be trigger as jobs in other workflows.

Example:

```yaml
# .github/workflows/reusable-build.yml
name: Reusable build

on:
  workflow_call:
    inputs:
      some-value:
        required: true
        type: string

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up dotnet
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '9'

      - name: Restore dependencies
        shell: bash
        run: |
          dotnet restore

      - name: Build
        shell: bash
        run: |
          dotnet build
```

```yaml
# .github/workflows/pull-request.yml
name: Pull request

on:
  pull_request:
    branches:
      - '*'

jobs:
  build:
    # uses: <org>/<repo>/<path-to-workflow>@<ref>
    uses: <my-org>/<this-repo>/.github/workflows/reusable-build.yml@main
```

Reusable workflows can be used in the same repository or from other repositories. If repository is private, its possible to use it only within the organization or user space.

### Inputs

Inputs are available only for workflow triggers `on.workflow_call` and `on.workflow_dispatch`.

`on.workflow_call`:

```yaml
on:
  workflow_call:
    inputs:
      username:
        description: 'A username passed from the caller workflow'
        default: 'john-doe'
        required: false
        type: string

jobs:
  print-username:
    runs-on: ubuntu-latest

    steps:
      - name: Print the input name to STDOUT
        run: echo The username is ${{ inputs.username }}
```

`on.workflow_dispatch`:

_For example manual trigger of workflow via GitHub UI._

```yaml
on:
  workflow_dispatch:
    inputs:
      logLevel:
        description: 'Log level'
        required: true
        default: 'warning'
        type: choice
        options:
          - info
          - warning
          - debug
      print_tags:
        description: 'True to print to STDOUT'
        required: true
        type: boolean
      tags:
        description: 'Test scenario tags'
        required: true
        type: string
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true

jobs:
  print-tag:
    runs-on: ubuntu-latest
    if: ${{ inputs.print_tags }}
    steps:
      - name: Print the input tag to STDOUT
        run: echo  The tags are ${{ inputs.tags }}
```

### Outputs

A job can specify outputs, that can later by used by other jobs.

Outputs can also be specified on the workflow, which is useful when using [Reusable workflows](#reusable-workflows).

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    # Map a step output to a job output
    outputs:
      output1: ${{ steps.step1.outputs.test }}
      output2: ${{ steps.step2.outputs.test }}
    steps:
      - id: step1
        run: echo "test=hello" >> "$GITHUB_OUTPUT"
      - id: step2
        run: echo "test=world" >> "$GITHUB_OUTPUT"
  job2:
    runs-on: ubuntu-latest
    needs: job1
    steps:
      - env:
          OUTPUT1: ${{needs.job1.outputs.output1}}
          OUTPUT2: ${{needs.job1.outputs.output2}}
        run: echo "$OUTPUT1 $OUTPUT2"
```

## GitHub made actions

To made several actions for you to get started with. All of those actions can be found on GitHub under the organization actions, see [github.com/actions](https://github.com/actions) or [Marketplace](https://github.com/marketplace?type=actions).

## Starter workflows

In order to not start from zero every time, GitHub has made a large collection of examples that you can start with. See [github.com/actions/starter-workflows](https://github.com/actions/starter-workflows).

## Writing actions

Actions can be created in three ways:

- **YAML**: The simplest option, write an `action.yml` file containing the workflow logic.

    The Action can be in any directory inside a repository, but the filename must always be `action.yml`.

    Such action is also called "shell action".

    Example:

    ```yaml
    # action.yaml
    name: 'Ship package'
    description: 'Ships package to release server'

    branding:
      color: green
      icon: package

    inputs:
      package-name:  # id of input
        required: true
        type: string

    outputs:
     url:
       description: "Package URL"
       value: ${{ steps.upload.outputs.URL }}

    runs:
      steps:
        - name: Prepare package
          run: echo "Preparing package..."
          shell: bash

        - name: Upload package
          id: upload
          run: |
            echo "Uploading package..."
            # Upload with cURL for example
            echo "Package uploaded"
            echo "URL=some-url-from-upload" >> $GITHUB_OUTPUT
    ```

- **Docker**: Docker action is the second simplest, but requires Docker engine to run, which may be troublesome with self-hosted runners, since Docker engine requires elevated privileges.

    For example running such Actions on Kubernetes is especially troublesome because of the privileged containers, which is usually not allowed by the security team.

    Such action is also called "docker action".

- **JavaScript**: The last option is to write JavaScript code using the [Actions toolkit](https://github.com/actions/toolkit) which is a collection of multiple npm packages.

### Composite actions

Composite actions allow you to collect a series of workflow job steps into a single action which you can then run as a single job step in multiple workflows.

In your workflow you reference only one action in one step, but the action itself is made of multiple steps.

```yaml
# action.yml
name: 'Hello World'
description: 'Greet someone'

inputs:
  who-to-greet:  # id of input
    description: 'Who to greet'
    required: true
    default: 'World'

outputs:
  random-number:
    description: "Random number"
    value: ${{ steps.random-number-generator.outputs.random-number }}

runs:
  using: "composite"
  steps:
    - name: Set Greeting
      run: echo "Hello $INPUT_WHO_TO_GREET."
      shell: bash
      env:
        INPUT_WHO_TO_GREET: ${{ inputs.who-to-greet }}

    - name: Random Number Generator
      id: random-number-generator
      run: echo "random-number=$(echo $RANDOM)" >> $GITHUB_OUTPUT
      shell: bash

    - name: Set GitHub Path
      run: echo "$GITHUB_ACTION_PATH" >> $GITHUB_PATH
      shell: bash
      env:
        GITHUB_ACTION_PATH: ${{ github.action_path }}

    - name: Run goodbye.sh
      run: goodbye.sh
      shell: bash
```

## Linting

Actions and workflows can (and should!) be linted by some automated tool.

[github.com/rhysd/actionlint](https://github.com/rhysd/actionlint) is a great tool for linting actions. _actionlint_ is a Go binary that you can install locally.

This way you can avoid some mistakes.

It is also very useful to lint actions in workflows, when you are building internal composable Actions and Workflows.

## Testing

### Testing actions

When writing actions, the easiest actions to test are the ones written in JavaScript. Because you can use JavaScript test runners such as Jest or Vitest to test them like any other code.

For other types of actions (YAML and Docker), it is best to write a simple "test workflow" to actually run the Action.

### Testing workflows

For testing Workflows, you do not have many options. Either run them locally via _act_ (see [Running workflows locally](#running-workflows-locally)) or make changes, push, trigger the workflow, and wait for results. The later option is quite time consuming and tedious.

### Running workflows locally

[github.com/nektos/act](https://github.com/nektos/act) is an open-source and lightweight program that can run your Workflows locally. Which tremendously simplifies testing. Instead of hacking the workflow triggers and waiting for it to finish, run the workflows locally without commit spam in the repository.

## Keeping actions up to date

To keep your actions and workflows up to date, consider using something like [Dependabot](https://github.com/dependabot) or [Renovate](https://docs.renovatebot.com/) to automatically receive Pull requests to update your actions/workflows.

**Dependabot config**:

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      # Check for updates to GitHub Actions every week
      interval: "weekly"
```

For more information, see GitHub docs: [Keeping your actions up to date with Dependabot](https://docs.github.com/en/code-security/dependabot/working-with-dependabot/keeping-your-actions-up-to-date-with-dependabot)

**Renovate config**:

By default, there is no need to configure Renovate, the default config automatically includes GitHub workflow files.

For more information, see Renovate docs: [Automated Dependency Updates for GitHub Actions](https://docs.renovatebot.com/modules/manager/github-actions/).

## Thank you! & Questions?

That's all, thank you for your attention.

Questions?

Let's go for a beer :beers:.

## Vojtěch Mareš

- email: [vojtech@mares.cz](mailto:vojtech@mares.cz)
- web: [mares.cz](https://www.mares.cz)
- x (twitter): [@vojtechmares_](https://x.com/vojtechmares_)
- linkedin: [/in/vojtech-mares](https://www.linkedin.com/in/vojtech-mares/)

Did you like the course? Tweet a recommendation on X (Twitter) and tag me
([@vojtechmares_]((https://x.com/vojtechmares_))) and/or add me on Linked In and I will send you a request for
recommendation. Thanks!
