We create namespace to isolate

```bash
kubectl create namespace opencost
```


If you prefer to just the basic config (without connecting to your AWS Account)

```bash
kubectl apply -f k8s/opencost.custom.yaml
```

To delete:
```bash
kubectl delete -f k8s/opencost.custom.yaml
```

To connect to your AWS Account, you create the policy CostExplorerReadPolicy with the policy in policies *policiess/CostExplorerReadPolicy.json*
Create the user opencost_user and associate the policy

Creamos secreto de integración AWS:
```bash
kubectl create secret generic cloud-costs --from-file=./cloud-integration.json --namespace opencost
```

Install
```bash
helm install opencost --repo https://opencost.github.io/opencost-helm-chart opencost --namespace opencost -f helm/opencost.aws.yaml
```

Upgrade
```bash
helm upgrade opencost --repo https://opencost.github.io/opencost-helm-chart opencost --namespace opencost -f helm/opencost.aws.yaml
```

Uninstall
```bash
helm uninstall opencost -n opencost
```