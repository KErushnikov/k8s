## Vault installation

1. Cert-manager
2. Consul
3. Execute commands
```bash 
kubectl create secret generic vault-azure-unseal -n vault \
--from-literal=tenant_id=<tenant_id> \
--from-literal=vault_name=<vault_name> \
--from-literal=key_name=<key_name> \
--from-literal=client_id=<client_id> \
--from-literal=client_secret=<client_secret> \
```
4. Setup up Vault "seal" config
5. Deploy vault
6. Select one of the pods and run the command `vault operator init`.
**Save the whole output!**
7. Execute the command `vault operator unseal`. Insert one of the 5 keys. Repeat the operation three times.
8. Repeat the previous step on each pod.
