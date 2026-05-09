# Kubernetes Cluster Upgrade Guide (v1.35.4)

This repository contains the unified workflow for upgrading a Kubernetes cluster from version **1.34.x** to **1.35.4**.

> [!IMPORTANT]
> **Order of Operations:**
> 1. Upgrade the **Primary Control Plane** node.
> 2. Upgrade additional Control Plane nodes (if HA).
> 3. Upgrade all **Worker Nodes**.

---

## 🚀 Phase 1: Control Plane Upgrade

Perform these steps on the **Master/Control Plane** node.

### 1. Update Repository & Kubeadm
Prepare the node for the new version by updating the management tool.

```bash
# 1. Update source list to target version
sudo vi /etc/apt/sources.list.d/kubernetes.list
# update the version from v1.34 to v1.35
sudo apt update
```
# 2. Upgrade Kubeadm
```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm='1.35.4-*' && \
sudo apt-mark hold kubeadm
```
2. Maintenance Mode (Drain)
 Replace <cp-node-name> with your master node name
```
kubectl drain <cp-node-name> --ignore-daemonsets --delete-emptydir-data
```

3. Apply the Upgrade Plan
```
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.35.4
```
5. Upgrade Kubelet & Kubectl
Update the node's agent and the command-line interface.
```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet='1.35.4-*' kubectl='1.35.4-*' && \
sudo apt-mark hold kubelet kubectl
```
# Restart services
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
5. Finalize Node

```
kubectl uncordon <cp-node-name>
```
👷 Phase 2: Worker Node Upgrade

1. Drain the Node (Run on Control Plane)

```
kubectl drain <worker-node-name> --ignore-daemonsets --delete-emptydir-data
```
2. Upgrade Kubeadm (Run on Worker)
```Bash
sudo vi /etc/apt/sources.list.d/kubernetes.list
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm='1.35.4-*' && \
sudo apt-mark hold kubeadm
```
3. Upgrade Node Config (Run on Worker)
```Bash
sudo kubeadm upgrade node
```
4. Upgrade Kubelet & Kubectl (Run on Worker)
```Bash
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet='1.35.4-*' kubectl='1.35.4-*' && \
sudo apt-mark hold kubelet kubectl
```
# Restart services
```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```
5. Uncordon the Node (Run on Control Plane)
Bring the worker back into the scheduling rotation.

```Bash
kubectl uncordon <worker-node-name>
```
