# Gitpod GitOps

Example repo to demonstrate Gitpod GitOps with KOTS

## Getting started

This assumes that you have deployed your Gitpod KOTS application in the `gitpod` namespace. Please update the commands if this is not the case.

### Install Sealed Secrets

> This is a worked example from the [KOTS documentation](https://docs.replicated.com/enterprise/gitops-managing-secrets). At the time of writing, this is in `alpha`

In order to prevent the secrets being disclosed, you should configure [Bitnami's Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) project on your cluster. This example uses their [Helm](https://github.com/bitnami-labs/sealed-secrets/tree/main/helm/sealed-secrets) chart.

```bash
helm upgrade \
  --atomic \
  --cleanup-on-fail \
  --create-namespace \
  --install \
  --namespace='sealed-secrets' \
  --reset-values --repo https://bitnami-labs.github.io/sealed-secrets \
  --wait \
  sealed-secrets \
  sealed-secrets
```

### Let KOTS know about Sealed Secrets

> This uses the `kubeseal` binary - check the Bitnami documentation for installation instructions

```bash
kubectl create secret generic \
  kots-sealed-secret \
  -n gitpod \
  --dry-run \
  --from-literal cert.pem="$(kubeseal --controller-name=sealed-secrets --controller-namespace=sealed-secrets --fetch-cert)" \
  -o yaml > ./kots-sealed-secret.yaml

# Add the labels
yq e -i '.metadata.labels."kots.io/secrettype" = "sealedsecrets"' ./kots-sealed-secret.yaml
yq e -i '.metadata.labels."kots.io/buildphase" = "secret"' ./kots-sealed-secret.yaml
```

Finally, restart the KOTS dashboard so that it's aware of the new sealed secrets

```bash
kubectl delete pods -n gitpod -l app=kotsadm
```

## Limitations

- You must create a CI/CD job to deploy the new Kubernetes YAML to your cluster
- You lose the preflight checks in the KOTS dashboard
