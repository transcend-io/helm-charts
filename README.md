# Sombra Helm Charts

Official Helm chart to install and configure Sombra™ and related services into a Kubernetes cluster.

For full documentation on this Helm chart, please see the [Sombra deployment guide](https://docs.transcend.io/docs/articles/sombra/deploying/deployment-options/helm).

## Usage

To install the latest version of this chart, add the Sombra helm repository
and run `helm install`:

```console
$ helm repo add transcend https://transcend-io.github.io/helm-charts/
$ helm repo update
$ helm install sombra transcend/sombra --values=./values.yaml
```

Please see the options supported in the `values.yaml` file. These are also fully documented directly on the [Sombra deployment guide](https://docs.transcend.io/docs/articles/sombra/deploying/deployment-options/helm).