# Shared Github actions and workflows

Reusuable workflows, actions, composite actions for use in our Github workflows.

Please note that this repository is _PUBLIC_ and no sensitive information shall be saved here! This is a Github limitation (unless we have a Github Enterprise plan) that only allows reusable workflows and actions from public repositories.

## Reusable workflows:

| workflow                          | language | service      | uses swap slots |
| --------------------------------- | -------- | ------------ | --------------- |
| build-deploy-node-appservice-az   | nodejs   | app service  | no              |
| build-deploy-swap-node-appservice | nodejs   | app service  | yes             |
| build-deploy-node-function-az     | nodejs   | function app | no              |
| build-deploy-swap-node-function   | nodejs   | function app | yes             |

## Example usage for function app:

In your project repository, create your workflow thusly:

Don't forget to change the APP_NAME variable.

```
name: Build & Deploy

on:
  push:
    branches:
      - main
      - test
      - prod
  workflow_dispatch:

jobs:
  build-deploy-dev:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-function-az.yaml@main
    with:
      ENVIRONMENT: Dev
      NODE_VERSION: 18.x
      APP_NAME: func-dev-hotel-availability
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/main'

  build-deploy-test:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-function-az.yaml@main
    with:
      ENVIRONMENT: Test
      NODE_VERSION: 18.x
      APP_NAME: func-test-hotel-availability
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/test'

  build-deploy-prod:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-swap-node-function.yaml@main
    with:
      ENVIRONMENT: Prod
      NODE_VERSION: 18.x
      APP_NAME: func-prod-hotel-availability
      RESOURCE_GROUP: rg-prod-func-hotel-availability
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/prod'

```

## Example usage for app service:

In your project repository, create your workflow thusly:

Don't forget to change the APP_NAME variable.

```
name: Build & Deploy

on:
  push:
    branches:
      - main
      - test
      - prod
  workflow_dispatch:

jobs:
  build-deploy-dev:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-appservice-az.yaml@main
    with:
      ENVIRONMENT: Dev
      NODE_VERSION: 18.x
      APP_NAME: app-dev-bff
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/main'

  build-deploy-test:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-appservice-az.yaml@main
    with:
      ENVIRONMENT: Test
      NODE_VERSION: 18.x
      APP_NAME: app-test-bff
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/test'

  build-deploy-prod:
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-swap-node-appservice.yaml@main
    with:
      ENVIRONMENT: Prod
      NODE_VERSION: 18.x
      APP_NAME: app-prod-bff
      RESOURCE_GROUP: rg-prod-app-bff
    secrets:
      APP_DEPLOYMENT_CLIENT_ID: ${{ secrets.APP_DEPLOYMENT_CLIENT_ID }}
      APP_DEPLOYMENT_CLIENT_SECRET: ${{ secrets.APP_DEPLOYMENT_CLIENT_SECRET }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
    if: github.ref == 'refs/heads/prod'

```


## Configuration

These resuable workflows need some variables/secrets. For example, posting to slack requires the `SLACK_WEBHOOK_URL`. [Organization secrets](https://github.com/organizations/bwh-scandinavia/settings/secrets/actions) are created at the organization level and are available to all repositories in the organization.

| variable                       | type                | description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------ | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SLACK_WEBHOOK_URL`            | organization secret | ["Incoming webhook" Slack app](https://bestwesternnewweb.slack.com/services/B034Q4EDTT2), configured to post workflow action status to a specific channel                                                                                                                                                                                                                                                                                                                                                             |
| `BWH_PAT_READ_PACKAGES`        | organization secret | Personal access token that gives a repository access to packages from other repositories, for example it is used when a project needs to run `npm install` and include the shared `@bwh-scandinavia/types` npm package, which is in another repository. The default `GITHUB_TOKEN` in the workflow does not have access to other repositories than the current repository, so an auth token is needed to access these packages. Create a PAT with the `read:packages` scope and give it a reasonable expiration date. |
| `APP_DEPLOYMENT_CLIENT_ID`     | organization secret | See App Deployment Orchestrator in Azure.                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `APP_DEPLOYMENT_CLIENT_SECRET` | organization secret | See above.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |