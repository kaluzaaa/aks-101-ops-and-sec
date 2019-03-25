# 2. Implement RBAC with namespaces on cluster

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within an enterprise.
In context of AKS role-based access control can be configured on two levels:
- Azure level RBAC (configured in IAM blade in portal or through az cli)
- Cluster level RBAC configured natively in Kubernetes.
  
## Azure level RBAC

When you interact with an AKS cluster using the `kubectl` tool, a configuration file is used that defines cluster connection information. This configuration file is typically stored in `~/.kube/config`. Multiple clusters can be defined in this kubeconfig file. You switch between clusters using the kubectl config use-context command.

You can manage list of cluster and connection context information in `kubectl` using following commands:
```shell
kubeclt config get-contexts
kubectl config set-context
kubectl config use-context
kubectl config delete-context

kubectl config get-clusters
kubectl config set-cluster
kubectl config delete-cluster
```

The `az aks get-credentials` command lets you get the access credentials for an AKS cluster and merges them into the kubeconfig file. You can use Azure role-based access controls (RBAC) to control access to these credentials. These Azure RBAC roles let you define who can retrieve the kubeconfig file, and what permissions they then have within the cluster.

The two built-in roles are:

- **Azure Kubernetes Service Cluster Admin Role**
    - Allows access to Microsoft.ContainerService/managedClusters/listClusterAdminCredential/action API call. This API call lists the cluster admin credentials.
    - Downloads kubeconfig for the clusterAdmin role.
- **Azure Kubernetes Service Cluster User Role**
    - Allows access to Microsoft.ContainerService/managedClusters/listClusterUserCredential/action API call. This API call lists the cluster user credentials.
    - Downloads kubeconfig for clusterUser role.

### 1 Configuring Azure RBAC for AKS cluster

1.1. Create test user in you AAD tenant. Save user's `objectId` for next steps.

```shell
az ad user create --display-name "Test user" --password <put-users-password> --user-principal-name <full-test-user-name> --force-change-password-next-login false
az ad user show --upn-or-object-id <full-test-user-name> --query objectId
```

1.2 Assign `Azure Kubernetes Service Cluster Admin Role` role to new user. Save AKS cluster's `resource id` for permission asignment.

```shell
az aks show --resource-group <resource-group-of-aks-cluster> --name <name-of-aks-cluster> --query id
az role assignment create --assignee <objectId-from 1.1> --scope <aks-id from command above> --role "Azure Kubernetes Service Cluster Admin Role"
```

1.3 Try connecting AKS and listing pods in all namespaces.

```shell
az login
az aks get-credentials --resource-group <resource-group-of-aks-cluster> --name <name-of-aks-cluster> --admin --overwrite-existing
kubectl get pods --all-namespaces
```

1.4 Login back to `az` and `kubectl` with your intial admin account.

```shell
az login
az aks get-credentials --resource-group <resource-group-of-aks-cluster> --name <name-of-aks-cluster> --admin --overwrite-existing
az role assignment delete --assignee <objectId-from 1.1> --scope <aks-id-from 1.2>
```

## Kubernetes level RBAC

### Namespaces

Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called `namespaces`.
- Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces.
- Namespaces are a way to divide cluster resources between multiple users (via resource quota).

### RBAC

RBAC in Kubernetes uses the `rbac.authorization.k8s.io` API group to drive authorization decisions, allowing admins to dynamically configure policies through the Kubernetes API.

**You basically define roles and assign them to users or groups using role bindings.**

In the RBAC API, a role contains rules that represent a set of permissions. Permissions are purely additive (there are no “deny” rules). A role can be defined within a namespace with a `Role`, or cluster-wide with a `ClusterRole`.
- A `Role` can only be used to grant access to resources within a single namespace. Here’s an example Role in the “default” namespace that can be used to grant read access to pods.
- A `ClusterRole` can be used to grant the same permissions as a Role, but because they are cluster-scoped, they can also be used to grant access to:
    - cluster-scoped resources (like nodes)
    - non-resource endpoints (like “/healthz”)
    - namespaced resources (like pods) across all namespaces (needed to run kubectl get pods --all-namespaces, for example)

A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. Permissions can be granted within a namespace with a `RoleBinding`, or cluster-wide with a `ClusterRoleBinding`.

### 2. Defining basic cluster role binds

In this scenario we will be using user you created in previous part of LAB. We will give the user access to pods

2.1. Deploy simple "Voting" service to AKS using following [yaml](yaml/azure-voting.yaml)

```shell
kubectl apply -f azure-voting.yaml
kubectl get services -n default --watch
```

2.2 Define new role to list and show `pods` and `pod logs`.

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list"]
```

```shell
kubectl apply -f pod-reader-role.yaml
```

2.3 Assign pod-reader role to test user you created recently.

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: pod-reader-test
  namespace: default
subjects:
- kind: User
  name: <your-test-user>
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```shell
kubectl apply -f pod-reader-role-assignment.yaml
```

2.4 Test access of user.

```shell
az aks get-credentials --resource-group <resource-group-of-aks-cluster> --name <name-of-aks-cluster> --overwrite-existing
az aks get pods -n default
az aks logs <put-pod-name-here> -n default

# actions below should fail
az aks delete pod <put-pod-name-here> -n default
az aks get services -n default
az aks get pods -n kube-system
```

## Sources
- https://docs.microsoft.com/en-us/azure/aks/control-kubeconfig-access
- https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/

