# Raspberry Pi cluster

- [Raspberry Pi cluster](#raspberry-pi-cluster)
  - [Setup kubectl on a local/control node - linux](#setup-kubectl-on-a-localcontrol-node---linux)
  - [Setup kubectl on a local/control node - macOS](#setup-kubectl-on-a-localcontrol-node---macos)
  - [Setup K3](#setup-k3)
  - [kubectl resources](#kubectl-resources)
  - [Setup app examples inc Grafana, Prometheus](#setup-app-examples-inc-grafana-prometheus)
    - [Install GO on macOS](#install-go-on-macos)
    - [Setup grafana](#setup-grafana)
    - [Using the cluster](#using-the-cluster)
  - [Helm](#helm)
  - [mosquitto](#mosquitto)

## Setup kubectl on a local/control node - linux

[Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

```
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

## Setup kubectl on a local/control node - macOS

[Install kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)

## Setup K3

[Raspberry Pi Cluster Episode 3 - Installing K3s Kubernetes on the Turing Pi](https://www.jeffgeerling.com/blog/2020/installing-k3s-kubernetes-on-turing-pi-raspberry-pi-cluster-episode-3)
[Rancher K3S](https://k3s.io)
[k3s-io/k3s-ansible on GitHub](https://github.com/k3s-io/k3s-ansible)

## kubectl resources

[kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

## Setup app examples inc Grafana, Prometheus

### Install GO on macOS

[Download and install](https://golang.org/doc/install)

### Setup grafana

[Raspberry Pi Cluster Episode 4 - Minecraft, Pi-hole, Grafana and More!](https://www.jeffgeerling.com/blog/2020/raspberry-pi-cluster-episode-4-minecraft-pi-hole-grafana-and-more)
[Cluster Monitoring stack for ARM / X86-64 platforms](https://github.com/carlosedp/cluster-monitoring)

### Using the cluster

Get up nodes:

```
kubectl get pods -o wide --all-namespaces
```

Get app urls:

```
kubectl get ingress -n monitoring
```

Grafana default username and password:

```
admin
admin
```

get help with logs

```
kubectl logs --help
```

follow logs for the given namespace and pod

```
kubectl logs --namespace mqtt -f mozzy-mosquitto-cfb8c7848-dc846
```

follow logs for the given container and namespace

```
kubectl logs --namespace mqtt -f svclb-mozzy-mosquitto-kcg77 -c lb-port-1883
```

## Helm

[Install Helm](https://helm.sh/docs/intro/install/)
[Using Helm](https://helm.sh/docs/intro/using_helm/)

Install - macOS
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```


## mosquitto


[A bit about Mosquitto on Kubernetes](https://gbaeke.gitbooks.io/open-source-iot/content/chapter1.html)
[Mosquitto on artifacthub - t3n](https://artifacthub.io/packages/helm/t3n/mosquitto)

Install:

```
helm repo add t3n https://storage.googleapis.com/t3n-helm-charts
helm show values t3n/mosquitto > value.yaml
```

Change service.type to LoadBalancer from ClusterIP so can be accessed outside cluster e.g. from iPad
```
service:
  type: LoadBalancer
```


```
kubectl create namespace mqtt
helm install --namespace mqtt --values value.yaml mozzy t3n/mosquitto
kubectl get pods --all-namespaces -o wide
kubectl get services --namespace mqtt
kubectl logs -n mqtt -f svclb-mozzy-mosquitto-664bm -c lb-port-1883
```

Remove it:

```
helm uninstall mozzy -n mqtt
kubectl delete namespace mqtt
```

