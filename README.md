# Monitoring Stack for K8s (AWS)
## Metrics server
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Creating the namespace for monitoring
```bash
kubectl create namespace monitoring
```
## Grafana

### BBDD Integration 
Create the user and DB with permissions needed:
```sql
CREATE USER grafana WITH LOGIN NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION;
ALTER ROLE grafana WITH PASSWORD 'VERY_SECRET_PASSWORD';
CREATE DATABASE grafana WITH OWNER = grafana ENCODING = 'UTF8';
GRANT ALL PRIVILEGES ON DATABASE grafana TO grafana;
GRANT ALL PRIVILEGES ON SCHEMA public TO grafana;
```
### Deploying
Update and install Grafana repo:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```


Once we have the database user created, we need to create the secret in k8s:
```bash
kubectl create secret generic grafana-db-secret --from-literal=GF_DATABASE_TYPE=postgres--from-literal=GF_DATABASE_NAME=grafana_db--from-literal=GF_DATABASE_SSL_MODE=require --from-literal=GF_DATABASE_USER=grafana --from-literal=GF_DATABASE_PASSWORD=VERY_SECRET_PASSWORD --from-literal=GF_DATABASE_HOST=DB_HOST:DB_PORT --namespace monitoring
```

Installing prometheus & grafana:
```bash
helm install monitoring prometheus-community/kube-prometheus-stack -f grafana/values.grafana.aws.yaml --namespace monitoring
helm upgrade monitoring prometheus-community/kube-prometheus-stack  -f grafana/values.grafana.aws.yaml -n monitoring
```

## Loki
### Buckets S3
https://grafana.com/docs/loki/latest/setup/install/helm/deployment-guides/aws/

We create a bucket for chunks and another for ruler.
- s3-for-chunks
- s3-for-ruler

We create the policy as loki-s3-policy with which we find the policies/loki-s3-policy.json

We create the LokiServiceAccountRole role by assigning the trust-policy to it with policies/trus-policy.json.
Install loki:
```bash
helm install loki grafana/loki -f loki/values.loki.aws.yaml -n monitoring
```

Upgrade:
```bash
helm upgrade loki grafana/loki -f loki/values.loki.aws.yaml -n monitoring
```

## Promtail 
We need promtail to compile logs and send to loki

https://github.com/grafana/helm-charts/blob/main/charts/promtail/values.yaml

We install helm packages

```bash
helm repo add grafana https://grafana.github.io/helm-charts
```
```bash
helm install promtail grafana/promtail -f promtail/promtail.values.yaml -n monitoring 
helm upgrade promtail grafana/promtail -f promtail/promtail.values.yaml --namespace monitoring
```



Upgrade:
```bash
helm upgrade grafana -f values.grafana.yaml -n monitoring
helm upgrade monitoring prometheus-community/kube-prometheus-stack -f grafana/values.grafana.aws.yaml --namespace monitoring
```