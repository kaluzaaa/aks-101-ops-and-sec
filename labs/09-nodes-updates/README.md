# 9. Apply security and kernel updates to nodes in Azure Kubernetes Service (AKS)

ToDo - https://docs.microsoft.com/en-us/azure/aks/node-updates-kured

## Understand the AKS node update experience
In an AKS cluster, your Kubernetes nodes run as Azure virtual machines (VMs). These Linux-based VMs use an Ubuntu image, with the OS configured to automatically check for updates every night. If security or kernel updates are available, they are automatically downloaded and installed.

![](img/node-reboot-process.png)

Some security updates, such as kernel updates, require a node reboot to finalize the process. A node that requires a reboot creates a file named */var/run/reboot-required*. This reboot process doesn't happen automatically.

You can use your own workflows and processes to handle node reboots, or use `kured` to orchestrate the process. With `kured`, a DaemonSet is deployed that runs a pod on each node in the cluster. These pods in the DaemonSet watch for existence of the */var/run/reboot-required* file, and then initiates a process to reboot the nodes.

## Kured (KUbernetes REboot Daemon) is a Kubernetes daemonset that
[Kured (KUbernetes REboot Daemon)](https://github.com/weaveworks/kured) is a Kubernetes daemonset that performs safe automatic node reboots when the need to do so is indicated by the package management system of the underlying OS.

* Watches for the presence of a reboot sentinel e.g. `/var/run/reboot-required`
* Utilises a lock in the API server to ensure only one node reboots at a time
* Optionally defers reboots in the presence of active Prometheus alerts or selected pods
* Cordons & drains worker nodes before reboot, uncordoning them after

## Lab

1. Install Kured

```bash
kubectl apply -f ToDo.yaml
```

2. Verify

```
kubectl get pods -n kube-system -o wide
```

3. Open log

```
kubectl logs -n kube-system -f <POD_NAME>
```

4. Log in to the server via ssh and create */var/run/reboot-required* to simulate a reboot.

```bash
sudo touch /var/run/reboot-required
```

5. Follow the results in pod logs from step 3.
