# Cert-Manager ACME DNS01 Webhook Solver for CIVO DNS

[![Go Report Card](https://goreportcard.com/badge/github.com/okteto/cert-manager-webhook-civo)](https://goreportcard.com/report/github.com/okteto/cert-manager-webhook-civo)
[![Releases](https://img.shields.io/github/v/release/okteto/cert-manager-webhook-civo?include_prereleases)](https://github.com/okteto/cert-manager-webhook-civo/releases)
[![LICENSE](https://img.shields.io/github/license/okteto/cert-manager-webhook-civo)](https://github.com/slicen/cert-manager-webhook-civo/blob/master/LICENSE)
[![Artifact HUB](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/okteto)](https://artifacthub.io/packages/search?repo=okteto)

This solver can be used when you want to use  [cert-manager](https://github.com/jetstack/cert-manager) with [CIVO DNS](https://civo.com). 

## Installation

### cert-manager

Follow the [instructions](https://cert-manager.io/docs/installation/) using the cert-manager documentation to install it within your cluster.

### Webhook

#### Using public helm chart

```bash
helm repo add okteto https://charts.okteto.com
helm repo update
helm install --namespace cert-manager cert-manager-webhook-civo okteto/cert-manager-webhook-civo
```

#### From local checkout

```bash
helm install --namespace cert-manager cert-manager-webhook-civo chart/cert-manager-webhook-civo
```
**Note**: The kubernetes resources used to install the Webhook should be deployed within the same namespace as the cert-manager.

To uninstall the webhook run
```bash
helm uninstall --namespace cert-manager cert-manager-webhook-civo
```

## Usage

### Credentials
In order to access the CIVO API, the webhook needs an [API token](https://www.civo.com/account/security).

```
kubectl create secret generic civo-secret --from-literal=api-key=<YOUR_CIVO_TOKEN>
```

### Create Issuer

Create a `ClusterIssuer` or `Issuer` resource as following:

#### Cluster-wide Issuer
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    
    # Email address used for ACME registration
    email: mail@example.com # REPLACE THIS WITH YOUR EMAIL
    
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging

    solvers:
    - dns01:
        webhook:
          solverName: "civo"
          groupName: civo.webhook.okteto.com
          config:
            secretName: civo-secret
            secretKey: api-key
```

By default, the CIVO API token used will be obtained from the secret in the same namespace as the webhook.

#### Per Namespace API Tokens

If you would prefer to use separate API tokens for each namespace (e.g. in a multi-tenant environment):

```
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
  namespace: default
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    
    # Email address used for ACME registration
    email: mail@example.com # REPLACE THIS WITH YOUR EMAIL
    
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging

    solvers:
    - dns01:
        webhook:
          solverName: "civo"
          groupName: civo.webhook.okteto.com
          config:
            secretName: civo-secret
            secretKey: api-key
```

By default, the webhook doesn't have permissions to read secrets on all namespaces. To enable this, you'll need to provide your own service account.

### Create a certificate

Create your certificate resource as follows:

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-cert
  namespace: cert-manager
spec:
  commonName: example.com
  dnsNames:
  - example.com # REPLACE THIS WITH YOUR DOMAIN
  issuerRef:
   name: letsencrypt-staging
   kind: ClusterIssuer
  secretName: example-cert
```

# Development

## Prerequisites
-  Admin access to a cluster. We recommend you [launch one on CIVO](https://www.civo.com/?ref=af9018).
-  [okteto CLI](https://okteto.com/docs/getting-started/installation)
- `kubectl` installed and configured to talk to your cluster

## Launch your Development Environment

1. Deploy the latest version of `cert-manager` and `cert-manager-webhook-civo` as per the instructions above.
1. Run `okteto up` from the root of this repo. This will deploy your pre-configured remote development environment, and keep your file system synchronized automatically.
1. Run `make` on the remote terminal to start the webhook. This will build the webhook, start it with the required configuration, and hot reload it whenever a file is changed.
1. Code away!

# Contributing
If you want to get involved, we'd love to receive a pull request, issues, or an offer to help. [Open an issue](https://github.com/okteto/cert-manager-webhook-civo/issues) to get started!

Maintainers:
- [Ramiro Berrelleza](https://twitter.com/rberrelleza)
- [Pablo Chico de Guzman](https://twitter.com/pchico83)

Please see the [contribution guidelines](CONTRIBUTING.md)