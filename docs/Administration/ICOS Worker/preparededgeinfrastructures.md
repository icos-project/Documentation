# Infrastructure preparation
ICOS supports only containerized environments and applications. 
To onboard edge infrastructure into ICOS, you must choose between the following options:

- [Installation of a Kubernetes](#kubernetes) cluster.
- [Installation of Docker](#docker) Engine.

!!! Warning
       
    This guide does not provide detailed instructions for installing Kubernetes 
    (For further details, we recommend referring to the [official documentation](https://kubernetes.io/docs/home/).) 
    or Docker (For further details, we recommend referring to the [official documentation](https://www.docker.com/).).

We offer an example of how to install K3s on a device with a single command below. 
Depending on your choice, you can execute the corresponding commands for a basic setup that will enable you to continue with the onboarding process.

## Kubernetes

One of the easiest ways to setup a basic ICOS-valid Kubernetes environment is via the K3S 
installation script and the installation of Helm (Kubernetes package manager).

1. Launch a new K3S cluster:

```bash
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644 --write-kubeconfig $HOME/.kube/config
```

2. Install Helm:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
Any Kubernetes + Helm setup with Kubernetes version 1.19 or higher will work for ICOS. 
K3S is just simple and quick example.

## Docker

You can quickly install the Docker Engine using the commands below (taken from the official Docker documentation for Ubuntu):

1. Make sure any old installations are removed.

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

2. Set up Docker's apt repository.

```bash
**Add Docker's official GPG key:**
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

**Add the repository to Apt sources:**
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

3. Install the Docker packages

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. (Optional) To run docker commands without prefacing them with *sudo* everytime

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

After this , you can go to the next step : [Install the Orchestrator](Orchestrator Agents/index.md)

