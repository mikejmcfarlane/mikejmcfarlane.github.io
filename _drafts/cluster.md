# Raspberry Pi cluster

## Setup kubectl on a local/control node

[Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

```
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

## Setup K3

[Raspberry Pi Cluster Episode 3 - Installing K3s Kubernetes on the Turing Pi](https://www.jeffgeerling.com/blog/2020/installing-k3s-kubernetes-on-turing-pi-raspberry-pi-cluster-episode-3)
[Rancher K3S](https://k3s.io)
[k3s-io/k3s-ansible on GitHub](https://github.com/k3s-io/k3s-ansible)

## kubectl resources

[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Setup app examples inc Grafana, Prometheus

[Raspberry Pi Cluster Episode 4 - Minecraft, Pi-hole, Grafana and More!](https://www.jeffgeerling.com/blog/2020/raspberry-pi-cluster-episode-4-minecraft-pi-hole-grafana-and-more)
[Cluster Monitoring stack for ARM / X86-64 platforms](https://github.com/carlosedp/cluster-monitoring)

Get up nodes:

```
kubectl get pods -o wide --all-namespaces
```


Get app urls:

``
kubectl get ingress -n monitoring
```

Grafana default username and password:

```
admin
admin
```
