## Vault installation

1. Cert-manager
2. kube-prometheus-stack
3. Execute commands
```bash
kubectl create ns monitoring
kubectl create secret generic grafana-admin-credentials -n monitoring \
--from-literal=admin-user=admin-user \
--from-literal=admin-password=admin-password
```

