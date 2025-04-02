# Renovate template repository

## Benefits
1. Consistency

- Shared Renovate templates ensure uniform dependency update configurations across all projects.
- Reduces errors and inconsistencies in project-level renovate.json files.

2. Time Efficiency & Reusability

- Once a template is configured and tested, it can be reused across multiple projects.
- Saves time when maintaining and updating configurations.

3. Centralized Configuration

- Updates can be made in one place (the shared repository) and applied to all consuming projects automatically.

4. Governance

- Enforces best practices across the organization—e.g., grouping updates, setting time windows, or controlling versioning strategies.

## What is a Renovate template?
A Renovate template is a JSON configuration file with predefined rules and settings designed to standardize Renovate behavior.

Here’s an example template:

```json
{
  "extends": [
    ":semanticCommits",
    ":dependencyDashboard",
    "group:allNonMajor"
  ],
  "timezone": "Europe/Paris",
  "schedule": ["after 9am and before 5pm on every weekday"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    },
    {
      "matchUpdateTypes": ["major"],
      "labels": ["major-update"]
    }
  ]
}
```

### Key Elements:

- `extends`: Inherits prebuilt configurations (e.g., semantic commits, dependency grouping).
- `timezone`: Sets the timezone for updates.
- `schedule`: Restricts automated updates to specific timeframes to avoid disruption.
- `packageRules`: Custom rules per update type (e.g., automerge minor updates and label major ones).

### How to use shared Renovate templates
1. Enable Renovate on your Project

If Renovate is not already enabled, activate it for your repository and add a `renovate.json` file in the project root.
You don't need the `Renovate mend app`, you can just use the GitHub Action provided in this document.

2. Reference the shared template

In your project's `renovate.json`, reference the shared template:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>ixxeL-DevOps/renovate-cfg//templates/gitops/main.json",
  ]
}
```

You can also override the configuration and add your own set of preset in the local repository:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>ixxeL-DevOps/renovate-cfg//templates/gitops/main.json",
    "github>ixxeL-DevOps/fullstack//.renovate/common.json"
  ]
}
```

More information : 

- https://docs.renovatebot.com/config-presets/#github

You can store your Renovate configuration file in one of these locations:

- `renovate.json`
- `renovate.json5`
- `.github/renovate.json`
- `.github/renovate.json5`
- `.gitlab/renovate.json`
- `.gitlab/renovate.json5`
- `.renovaterc`
- `.renovaterc.json`
- `.renovaterc.json5`
- `package.json` (within a "renovate" section)

3. For private shared repositories: Set up a Personal Access Token (PAT) in Renovate to authenticate and fetch the template.

4. Use a GitHub Action to trigger renovate :

```yaml
---
# yaml-language-server: $schema=https://json.schemastore.org/github-workflow.json
name: Renovate
on:
  workflow_dispatch:
  repository_dispatch:
    types:
      - renovate
  schedule:
    - cron: 0 2 * * *
    - cron: 0 3 * * *

concurrency:
  group: ${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  renovate:
    runs-on: [self-hosted, cicd]
    steps:
      - name: Generate Token
        uses: actions/create-github-app-token@d72941d797fd3113feb6b93fd0dec494b13a2547 # v1.12.0
        id: app-token
        with:
          app-id: ${{ secrets.GHAPP_APP_ID }}
          private-key: ${{ secrets.GHAPP_PRIVATE_KEY }}
          owner: ${{ github.repository_owner }}

      - name: Checkout
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          token: ${{ steps.app-token.outputs.token }}

      - name: Run Renovate
        uses: renovatebot/github-action@c21017a4a2fc9f42953bcc907e375a5a544557ec # v41.0.18
        with:
          token: ${{ steps.app-token.outputs.token }}
        env:
          LOG_LEVEL: debug
          RENOVATE_REPOSITORIES: ${{ github.repository }}
```

Permissions must be as following : 

- https://docs.renovatebot.com/modules/platform/github/#running-as-a-github-app

## Best practices
### Merge queues

Activating `merge queues` in GitHub with Renovate can significantly enhance your dependency management workflow. Merge queues allow you to batch and serialize the merging of pull requests (PRs), which is particularly beneficial when dealing with a high volume of dependency updates. This feature ensures that PRs are merged in a controlled and orderly manner, reducing the risk of conflicts and integration issues. 

By using merge queues, you can minimize disruptions to your continuous integration (CI) pipeline and maintain a stable codebase. Additionally, merge queues help in managing complex dependencies and cascading updates, making it easier to pinpoint and resolve any issues that may arise during the update process. Overall, activating merge queues with Renovate improves the reliability and efficiency of your dependency management, ensuring smoother and more predictable updates.

- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
- https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request-with-a-merge-queue


### Labelling

It is strongly advised to organize your Renovate PR with labels. Applying GitHub labels to Renovate-generated PRs enhances organization, transparency, and efficiency, making it easier to keep your project dependencies up-to-date and secure.

You can add this file to `.github/labels.yaml` directory and combine it with the following pipeline:

```yaml
---
# Environments
- name: env/prod
  color: '0636CF'
  description: >-
    Changes made in the Production cluster
- name: env/nprod
  color: '0636CF'
  description: >-
    Changes made in the Non Production cluster
# Renovate
- name: renovate/docker
  color: 'ffec19'
  description: >-
    Changes related to docker image update
- name: renovate/github-action
  color: 'ffec19'
  description: >-
    Changes related to GitHub Actions update
- name: renovate/github-release
  color: 'ffec19'
  description: >-
    Changes related to GitHub tag/release update
- name: renovate/helm
  color: 'ffec19'
  description: >-
    Changes related to Helm Chart update
# Semantic Type
- name: type/digest
  color: 'ffec19'
- name: type/patch
  color: '4E911C'
- name: type/minor
  color: 'ff9800'
- name: type/major
  color: 'f6412d'
# Applications
- name: app/my-app-1
  color: '66B574'
  description: >-
    Changes made to App 1 application
- name: app/my-app-2
  color: 'FF6E00'
  description: >-
    Changes made to App 2 application
```

GitHub Action pipeline :

```yaml
---
# yaml-language-server: $schema=https://json.schemastore.org/github-workflow.json
name: Meta - Sync labels
on:
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - '.github/labels.yaml'

permissions:
  issues: write

jobs:
  labels:
    name: Sync Labels
    runs-on: [self-hosted, cicd]
    steps:
      - name: Generate Token
        uses: actions/create-github-app-token@d72941d797fd3113feb6b93fd0dec494b13a2547 # v1.12.0
        id: app-token
        with:
          app-id: ${{ secrets.GHAPP_APP_ID }}
          private-key: ${{ secrets.GHAPP_PRIVATE_KEY }}

      - name: Checkout
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
        with:
          token: ${{ steps.app-token.outputs.token }}
          sparse-checkout: .github/labels.yaml

      - name: Sync Labels
        uses: EndBug/label-sync@52074158190acb45f3077f9099fea818aa43f97a # v2.3.3
        with:
          config-file: .github/labels.yaml
          delete-other-labels: true
```

### GitOps Template

The GitOps template is dedicated for config repositories managed by ArgoCD for example. It is strongly advised to adopt a strict and predictable directory hierachy to benefit from Renovate templating automation and labelling.

For example :

```console
config
├── bootstrap
│   ├── nprod
│   └── prod
├── core
│   ├── appProjects
│   ├── apps
│   ├── clusters
│   └── repos
└── manifests
    ├── app1
    │   ├── nprod
    │   └── prod
    └── app2
        ├── nprod
        └── prod
```

Such a structure is well suited for automation. You can for example override common renovate config on your repository with the following :

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "packageRules": [
    {
      "description": "Group and add labels per application and environment",
      "matchManagers": ["helmv3", "helm-values", "docker", "regex"],
      "matchFileNames": ["config/manifests/**"],
      "groupName": "{{ lookup (split packageFileDir '/') 2 }}-{{ lookup (split packageFileDir '/') 3 }}",
      "addLabels": ["app/{{ lookup (split packageFileDir '/') 2 }}", "env/{{ lookup (split packageFileDir '/') 3 }}"],
      "additionalBranchPrefix": "{{ lookup (split packageFileDir '/') 3 }}/{{ lookup (split packageFileDir '/') 2 }}"
    }
  ]
}
```

It will group and label your updates by `environment` (prod/nprod) and by `application name` (e.g. app1, app2...etc).

If you want renovate to update Docker images version on Helm chart, you need to specify explicitly the image tag section in the `values.yaml` files:

```yaml
---
cert-manager:
  image:
    repository: quay.io/jetstack/cert-manager-controller
    tag: v1.17.1
    pullPolicy: IfNotPresent
```

If you have non standard dependency, like for example the Nexus image.

`values.yaml`:
```yaml
    container:
      image:
        nexusTag: 3.75.1
        repository: sonatype/nexus3
```

You can use the custom manager :

```yaml
    container:
      image:
        # renovate: datasource=docker depName=sonatype/nexus3
        nexusTag: 3.75.1
        repository: sonatype/nexus3
```

- https://docs.renovatebot.com/modules/manager/regex/#regular-expression-capture-groups

### Auto Merge

If you need to add auto-merge on some elements, just add overriding configuration on your repository:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "packageRules": [
    {
      "description": "Auto merge patch and minor",
      "matchDatasources": ["docker", "github-releases", "github-tags", "helm"],
      "automerge": true,
      "automergeType": "pr",
      "platformAutomerge": true,
      "automergeStrategy": "squash",
      "rebaseWhen": "behind-base-branch",
      "matchUpdateTypes": ["minor", "patch"]
    }
  ]
}
```