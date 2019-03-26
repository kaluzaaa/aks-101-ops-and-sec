# 3. Implement monitoring and logging in AKS

`Azure Monitor` for containers is a feature designed to monitor the performance of container workloads deployed to either Azure Container Instances or managed Kubernetes clusters hosted on Azure Kubernetes Service (AKS). Monitoring your containers is critical, especially when you're running a production cluster, at scale, with multiple applications.

Azure Monitor for containers includes several pre-defined views that show the residing container workloads and what affects the performance health of the monitored Kubernetes cluster so that you can:
- Identify AKS containers that are running on the node and their average processor and memory utilization. This knowledge can help you identify resource bottlenecks.
- Identify processor and memory utilization of container groups and their containers hosted in Azure Container Instances. 
- Identify where the container resides in a controller or a pod. This knowledge can help you view the controller's or pod's overall performance.
- Review the resource utilization of workloads running on the host that are unrelated to the standard processes that support the pod.
- Understand the behavior of the cluster under average and heaviest loads. This knowledge can help you identify capacity needs and determine the maximum load that the cluster can sustain.

## Deploy Container Monitoring Solution and Log Analytics Workspace

1. Login to portal.azure.com and create new resource of type "Container Monitoring Solution" or use following link: https://portal.azure.com/#create/Microsoft.ContainersOMS.
   
![Log Analytics workspace](img/Log-Analytics-workspace.png) 

2. Get `resource id` of created Log Analytics workspace. You can grab it from portal URL when browsing resource. It shoud have following format:
   
```
/subscriptions/<subscription-id>/resourcegroups/<rg-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>
```

## Install Monitoring addon to AKS cluster

1. For existing cluster run following command

```shell
az aks enable-addons -a monitoring -n <name-of-your-cluster> -g <resource-group-name> --workspace-resource-id <ExistingWorkspaceResourceID>
```

2. For newly deployed clusters just use `--enable-addons monitoring` parameter.

```shell
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --enable-addons monitoring \
    ...
```

3. Verify installation of monitoring agent

```shell
kubectl get daemonset omsagent -n kube-system
```
or

```shell
az aks show -n <name-of-your-cluster> -g <resource-group-name> --query addonProfiles
```

## Sources
- https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-overview
