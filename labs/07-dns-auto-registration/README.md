# 7. Configure DNS auto registration of application 

## Prepare DNS Zone in Azure

1. Check Azure subscription name.

```bash
az account show --query "[name]" -o tsv
```

2. Create DNS Zone in Azure.

```bash
az network dns zone create -g aks02-XY -n aks02-XY.<subscription name>.becomeazure.ninja
```

3. Create the service principal for DNS by AKS.

```bash
az ad sp create-for-rbac -n ExternalDnsServicePrincipalXY --skip-assignment
```

4. Assign the rights for the service principal.

```bash
az role assignment create --assignee http://ExternalDnsServicePrincipalXY --role contributor --resource-group aks02-xy
```

## Deploy ExternalDNS

1. Prepare azure.json using parameters for previous steps.

```json
{
  "tenantId": "<tenantId>",
  "subscriptionId": "<subscriptionId>",
  "resourceGroup": "aks02-XY",
  "aadClientId": "<aadClientId/appId>",
  "aadClientSecret": "<aadClientSecret/password>"
}
```

2. Create secret in Kubernetes from azure.json.

```bash
kubectl create secret generic azure-config-file --from-file=azure.json
```

3. Deploy ExternalDNS

```bash
kubectl apply -f externalDNS.yaml
```

4. Prepare ingress.yaml - put your subscription name.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress-dns
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: demo.aks02-xy.<sub name>.becomeazure.ninja
    http:
      paths:
      - path: /
        backend:
          serviceName: kuard
          servicePort: 80
```

5. Deploy test ingress rule.

```bash
kubectl apply -f ingress.yaml
```

6. Check in Azure portal DNS Zone.

Links
- [Setting up ExternalDNS for Services on Azure](https://github.com/kubernetes-incubator/external-dns/blob/master/docs/tutorials/azure.md)