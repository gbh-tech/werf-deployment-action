<!-- omit in toc -->
# Werf (+ Taskfile) Deployment Action

<!-- omit in toc -->
## Content

- [Overview](#overview)
- [Usage](#usage)
  - [Example usage](#example-usage)

## Overview

This GitHub Action facilitates Werf deployments, allowing you to run Werf commands with ease.
It provides options to configure Werf, AWS regions, run modes, and secrets for secure deployment.

## Usage

See [action.yml](action.yml).

``` yaml
- uses: actions/werf-deployment-action@v0.0.2
  with:
    # Target environment to pass to tasks and Werf commands
    environment: 'stage'

    # The Taskfile tasks (command separated) to run
    # Puts the environment input as a CLI_ARG with '-- <env>'
    # Tasks are run previous to Werf commands
    tasks: >-
      ecr-login,
      kubeconfig

    # The Werf commands to run without werf and --env
    # Werf commands are run after tasks
    commands: >-
      render --repo <my-repo>,
      plan --repo <my-repo>,
      converge --repo <my-repo> --values .helm/values-<env>.yaml

    # Specifies the AWS region name for configuration
    aws_region: 'us-east-1'

    # Optional. AWS service account access key
    aws_access_key_id: '<key>'

    # Optional. Ansible vault password to decrypt secrets
    werf_secret_key: '<secret>'

    # Optional. AWS service account secret access key
    aws_secret_access_key: '<secret>'
```

### Example usage

```yaml
- uses: gbh-tech/werf-deployment-action@v0.0.2
  with:
    environment: 'stage'
    aws_region: 'us-east-1'
    werf_secret_key: '${{ secrets.WERF_SECRET_KEY }}'
    aws_access_key_id: '${{ vars.AWS_ACCESS_KEY_ID }}'
    aws_secret_access_key: '${{ secrets.AWS_SECRET_ACCESS_KEY }}'
    tasks: >-
      ecr-login,
      kubeconfig
    commands: >-
      render --repo 'id.aws.ecr/repo/myapp' --values '.helm/values-stage.yaml',
      converge --repo 'id.aws.ecr/repo/myapp', --values '.helm/values-stage.yaml'
```
