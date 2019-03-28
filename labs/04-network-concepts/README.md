# 4. Explain network concepts for applications in AKS

How to check connectivity during network lab

```bash
kubectl run -i -t busybox-curl --image=yauritux/busybox-curl --restart=Never```

```
curl -v <ip>:<port>
```

## Deploy AKS with CNI

1. Deploy virtual network

```bash
az network vnet create -g aks02-XX -n vnet --address-prefix 172.16.0.0/16 \
    --subnet-name aks \
    --subnet-prefix 172.16.0.0/24
```

2. Get the subnet resource ID for the existing subnet into which the AKS cluster will be joined.

```
az network vnet subnet list \
    --resource-group aks02-XX \
    --vnet-name vnet \
    --query [].id --output tsv
```

3. Deploy AKS with CNI

```bash
az aks create -g aks02-XX -n aks02 --kubernetes-version 1.12.6 \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24
```
4. Get access to cluster and verify the pods

```bash
az aks get-credentials -g aks02-XX -n aks02 --admin
kubectl get pods --all-namespaces -o wide
```

## Expose Kubernetes service

1. Create demo application.

```bash
kubectl apply -f kuard.yaml
```

2. Expose demo application with public IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kuard
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: kuard
```

```bash
kubectl apply -f lb.yaml
```

3. Check public ip of service.

```bash
kubectl get svc --all-namespaces -o wide
```

4. Expose demo application with internal IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kuard-internal
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  selector:
    app: kuard
```

```bash
kubectl apply -f ilb.yaml
```

5. Check internal ip of service.

```bash
kubectl get svc --all-namespaces -o wide
```

6. Expose demo application with static internal IP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kuard-internal
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8080
      protocol: TCP
  loadBalancerIP: 172.16.0.240
  selector:
    app: kuard
```

```bash
kubectl apply -f ilb-fix-ip.yaml
```

7. Check internal ip of service.

```bash
kubectl get svc --all-namespaces -o wide
```

Links:
- [Enable containers to use Azure Virtual Network capabilities](https://docs.microsoft.com/en-us/azure/virtual-network/container-networking-overview
)
- [Configure Azure CNI networking in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/configure-advanced-networking
)
- [Use an internal load balancer with Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/internal-lb)
- [Use a static public IP address with the Azure Kubernetes Service (AKS) load balancer](https://docs.microsoft.com/en-us/azure/aks/static-ip)
