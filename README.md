<!-- omit in toc -->
# Werf (+ Taskfile) Deployment Action

<!-- omit in toc -->
## Content

- [Overview](#overview)
- [Usage](#usage)
  - [OIDC authentication (recommended)](#oidc-authentication-recommended)
  - [Static credentials](#static-credentials)
- [Inputs](#inputs)

## Overview

This GitHub Action facilitates Werf deployments, allowing you to run Werf commands with ease.
It provides options to configure Werf, AWS regions, run modes, and secrets for secure deployment.

## Usage

See [action.yml](action.yml).

### OIDC authentication (recommended)

Use `role_arn` to have the action assume an IAM role via OIDC. No long-lived credentials are
required. The calling workflow must grant `id-token: write` permission.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v6

      - uses: gbh-tech/werf-deployment-action@v0.0.5
        with:
          environment: 'stage'
          aws_region: 'us-east-1'
          role_arn: '${{ vars.AWS_ROLE_ARN }}'
          werf_secret_key: '${{ secrets.WERF_SECRET_KEY }}'
          tasks: >-
            ecr-login,
            kubeconfig
          commands: >-
            render --repo 'id.aws.ecr/repo/myapp' --values '.helm/values-stage.yaml',
            converge --repo 'id.aws.ecr/repo/myapp' --values '.helm/values-stage.yaml'
```

### Static credentials

Use `aws_access_key_id` and `aws_secret_access_key` when OIDC is not available. If the credentials
are temporary (STS-issued), also pass `aws_session_token`.

```yaml
- uses: gbh-tech/werf-deployment-action@v0.0.5
  with:
    environment: 'stage'
    aws_region: 'us-east-1'
    werf_secret_key: '${{ secrets.WERF_SECRET_KEY }}'
    aws_access_key_id: '${{ vars.AWS_ACCESS_KEY_ID }}'
    aws_secret_access_key: '${{ secrets.AWS_SECRET_ACCESS_KEY }}'
    # aws_session_token: '${{ env.AWS_SESSION_TOKEN }}'  # required for temporary STS credentials
    tasks: >-
      ecr-login,
      kubeconfig
    commands: >-
      render --repo 'id.aws.ecr/repo/myapp' --values '.helm/values-stage.yaml',
      converge --repo 'id.aws.ecr/repo/myapp' --values '.helm/values-stage.yaml'
```

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `environment` | yes | — | Target environment passed to tasks and Werf commands |
| `aws_region` | yes | — | AWS region |
| `role_arn` | no | `''` | IAM role ARN to assume via OIDC. When set, OIDC auth is used and static key inputs are ignored. Requires `permissions: id-token: write` in the calling workflow. |
| `aws_access_key_id` | no | `''` | AWS access key ID. Used only when `role_arn` is not set. |
| `aws_secret_access_key` | no | `''` | AWS secret access key. Used only when `role_arn` is not set. |
| `aws_session_token` | no | `''` | AWS session token for temporary STS credentials. Used only when `role_arn` is not set. |
| `werf_secret_key` | no | `''` | Werf secret key written to `.werf_secret_key` |
| `tasks` | no | — | Comma-separated Taskfile tasks to run before Werf commands. Each task receives `-- <environment>` as a CLI arg. |
| `commands` | no | — | Comma-separated Werf commands to run (without the `werf` prefix and `--env` flag). |
| `kubectl_version` | no | `v1.34.0` | kubectl version to install |
| `task_version` | no | `3.43.2` | Task version to install |
| `werf_version` | no | `v2.53.0` | Werf version to install |
