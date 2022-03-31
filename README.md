# Shared Github actions and workflows

Reusuable workflows, actions, composite actions for use in our Github workflows.

Please note that this repository is _PUBLIC_ and no sensitive information shall be saved here! This is a Github limitation (unless we have a Github Enterprise plan) that only allows reusable workflows and actions from public repositories.

## Reusable workflows:

| workflow                          | language | service      |
| --------------------------------- | -------- | ------------ |
| build-deploy-node-appservice.yaml | nodejs   | app service  |
| build-deploy-node-function.yaml   | nodejs   | function app |


## Example usage for function app:

In your project repository, create your workflow thusly:

```
name: Build & Deploy

on:
  push:
    branches:
      - main
      - production

  workflow_dispatch:
    branches:
      - main
      - production

jobs:
  build-deploy-test:
    if: ${{ github.ref == 'refs/heads/main' }}
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-function.yaml@main
    with:
      ENVIRONMENT: Test
      NODE_VERSION: 16.x
      AZURE_FUNCTIONAPP_NAME: func-test-something
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      AZURE_FUNCTIONAPP_PUBLISH_PROFILE: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE_TEST }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}

  build-deploy-prod:
    if: ${{ github.ref == 'refs/heads/production' }}
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-function.yaml@main
    with:
      ENVIRONMENT: Prod
      NODE_VERSION: 16.x
      AZURE_FUNCTIONAPP_NAME: func-prod-something
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      AZURE_FUNCTIONAPP_PUBLISH_PROFILE: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE_PROD }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
```

## Example usage for app service:

In your project repository, create your workflow thusly:

```
name: Build & Deploy

on:
  push:
    branches:
      - main
      - production

  workflow_dispatch:
    branches:
      - main
      - production

jobs:
  build-deploy-test:
    if: ${{ github.ref == 'refs/heads/main' }}
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-appservice.yaml@main
    with:
      ENVIRONMENT: Test
      NODE_VERSION: 16.x
      AZURE_APPSERVICE_NAME: app-test-something
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      AZURE_APPSERVICE_PUBLISH_PROFILE: ${{ secrets.AZURE_APPSERVICE_PUBLISH_PROFILE_TEST }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}

  build-deploy-prod:
    if: ${{ github.ref == 'refs/heads/production' }}
    uses: bwh-scandinavia/actions/.github/workflows/build-deploy-node-appservice.yaml@main
    with:
      ENVIRONMENT: Prod
      NODE_VERSION: 16.x
      AZURE_APPSERVICE_NAME: app-prod-something
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
      AZURE_APPSERVICE_PUBLISH_PROFILE: ${{ secrets.AZURE_APPSERVICE_PUBLISH_PROFILE_PROD }}
      NODE_AUTH_TOKEN: ${{ secrets.BWH_PAT_READ_PACKAGES }}
```


## Configuration

These resuable workflows need some variables/secrets. For example, posting to slack requires the `SLACK_WEBHOOK_URL`. [Organization secrets](https://github.com/organizations/bwh-scandinavia/settings/secrets/actions) are created at the organization level and are available to all repositories in the organization.

| variable | type | description |
|----------|------|-------------|
| `SLACK_WEBHOOK_URL` | organization secret | ["Incoming webhook" Slack app](https://bestwesternnewweb.slack.com/services/B034Q4EDTT2), configured to post workflow action status to a specific channel |
| `BWH_PAT_READ_PACKAGES` | organization secret | Personal access token that gives a repository access to packages from other repositories, for example it is used when a project needs to run `npm install` and include the shared `@bwh-scandinavia/types` npm package, which is in another repository. The default `GITHUB_TOKEN` in the workflow does not have access to other repositories than the current repository, so an auth token is needed to access these packages. Create a PAT with the `read:packages` scope and give it a reasonable expiration date. |
| `AZURE_APPSERVICE_PUBLISH_PROFILE_TEST` | specific to repository | The Azure Publish Profile for the app service for the test environment. Currently manually copy-pasted from Azure Portal to Github. An automatic connection via Terraform would be nice, [though not currently easily possible.](https://github.com/hashicorp/terraform-provider-azurerm/issues/8739) |
| `AZURE_APPSERVICE_PUBLISH_PROFILE_PROD` | specific to repository | The Azure Publish Profile for the app service for the production environment. |
| `AZURE_FUNCTIONAPP_PUBLISH_PROFILE_TEST` | specific to repository | The Azure Publish Profile for the function app for the test environment. |
| `AZURE_FUNCTIONAPP_PUBLISH_PROFILE_PROD` | specific to repository | The Azure Publish Profile for the function app for the production environment. |
