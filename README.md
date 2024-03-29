# Kubernetes Helm Multi-Deploy (No Docker Version)

> :warning: **This action requires an self-hosted GitHub Actions runner with a number of dependencies pre-installed.**  This will run faster because of the pre-setup done, this action *WILL NOT* work on Github.com hosted Runners (see below).
>
> For a pre-built docker-based action with all dependencies included [see here](https://github.com/DevOps-Nirvana/aws-helm-multi-deploy-prebuilt).  This will run on Github Action's runners successfully, and will run fast(er) than below, but is (arguably) less secure.
>
> For a built-at-runtime docker-based action with all dependencies included [see here](https://github.com/DevOps-Nirvana/aws-helm-multi-deploy).  This will run on Github Action's runners successfully, and this one is "more secure" but takes longer to run because it has to build the Docker image before runtime.

This GitHub Action will deploy all Helm chart folders inside a 'deployment' folder in your repository root. Useful for deploying multiple services that are in separate charts. For example:

```
my-awesome-app/
├── README.md
├── deployment
│   ├── my-deployment-1
│   │   ├── Chart.yaml
|   |   ├── requirements.yaml
│   │   ├── values-dev.yaml
│   │   ├── values-prod.yaml
│   │   └── values.yaml
│   └── my-deployment-2
│       ├── Chart.yaml
│       ├── values-dev.yaml
│       ├── values-prod.yaml
│       └── values.yaml
└── src
    └── etc
```

For the above file system, this action will deploy `my-deployment-1`, `my-deployment-2`, and any other charts under the deployment folder. Each chart can contain subcharts too.

If you define the input `environment-slug`, then `values-<env>.yaml` will be applied **on top of** your `values.yml`. e.g. if you define `environment-slug=dev`, then the options `-f values.yaml -f values-dev.yaml` will be passed to helm (if `values-dev.yaml` exists). This is to provide the option of having different settings per environment.

## Dependencies

To use this action, your GitHub Actions runner needs to have the following installed:

- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [helm](https://helm.sh/)
- [The helm diff plugin: ](https://github.com/databus23/helm-diff)


## Inputs

| **Input**              | **Required** | **Default** | **Description**                                                                                        |
|------------------------|--------------|-------------|--------------------------------------------------------------------------------------------------------|
| image-tag              | yes          | N/A         | Image tag to use in each deployment.                                                                   |
| k8s-namespace          | yes          | N/A         | Deployment namespace in kubernetes.                                                                    |
| environment-slug       | no           | N/A         | Short name of the deployment environment (dev, prod, etc). Set this if you have a `values-<env>.yaml`. |
| helm-extra-args        | no           | N/A         | Add additional/custom helm arguments/commands.                                                         |
| helm-chart-name-prefix | no           | N/A         | Add string to prefix helm chart name (eg: dev-)                                                        |
| helm-chart-name-suffix | no           | N/A         | Add string to suffix helm chart name (eg: -dev)                                                        |
| dry-run                | no           | false       | Skip actual deployment and only show a diff.                                                           |
| timeout                | no           | 300s        | Timeout time for helm operations.                                                                      |

## Example Usage

```yaml
jobs:
  build:
    steps:
      - uses: actions/checkout@v3
      - uses: docker/build-push-action@v3
          ...

  deploy:
    needs: build
    steps:
      - uses: actions/checkout@v2
      - uses: DevOps-Nirvana/k8s-helm-multi-deploy-nodocker@v1
        with:
          environment-slug: dev
          k8s-namespace: my-dev-ns
          image-tag: my-dev-image-tag
```
