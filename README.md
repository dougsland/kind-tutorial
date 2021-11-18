- [kind-tutorial](#kind-tutorial)
  * [1. KinD Download](#1-kind-download)
  * [2. KinD Initial Configuration](#2-kind-initial-configuration)
  * [3. KinD Installation](#3-kind-installation)
  * [4. Setting Container Network Interface](#4-setting-container-network-interface)
  * [5. Rootless](#5-rootless)
    + [Podman](#podman)
    + [Docker](#docker)
  * [5. Tools](#5-tools)
    + [5.1 kubectl](#51-kubectl)
    + [5.2 Krew](#52-krew)
    + [5.2 Creating your own plugin](#52-creating-your-own-plugin)
    + [5.3 k8s-local-dev](#53-k8s-local-dev)

# kind-tutorial
Kind Presentation


## 1. KinD Download

Linux:
```
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.11.1/kind-linux-amd64
$ chmod +x ./kind
$ mv ./kind /some-dir-in-your-PATH/kind
```

Windows:
```
$ choco install kind
```

Mac:
```
$ brew install kind
```

## 2. KinD Initial Configuration

Setting initial config via YAML
```
$ cat kindConfig.yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

## 3. KinD Installation
```
$ kind create cluster --config kindConfig.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

$ kubectl cluster-info --context kind-kind

Thanks for using kind! 😊
```

## 4. Setting Container Network Interface
First, disable the default CNI created by KinD
Deploy KinD
Setup your favourite CNI

```
$ cat kindConfig.yaml

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: control-plane
  - role: worker
  - role: worker
networking:
  disableDefaultCNI: true
```

## 5. Rootless

### Podman
Requires:

Podman v3.0 or later
Add the following into `/etc/default/grub` and update grub
```
GRUB_CMDLINE_LINUX="systemd.unified_cgroup_hierarchy="
````

Updating grub
```
$ grub2-mkconfig -o "$(readlink -e /etc/grub2.conf)"
```

Reboot the host
```
$ sudo reboot
```

Set delegate.conf into `/etc/systemd/system/user@.service.d/`
```
$ mkdir /etc/systemd/system/user@.service.d/
$ vi delegate.conf
[Service]
Delegate=yes

$ sudo systemctl daemon-reload
```


Set `/etc/modules-load.d/iptables.conf`
```
$ mkdir -p /etc/systemd/system/user@.service.d/
$ vi delegate.conf
ip6_tables
ip6table_nat
ip_tables
iptable_nat
```


Running:
```
$ KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster
using podman due to KIND_EXPERIMENTAL_PROVIDER
enabling experimental podman provider
Cgroup controller detection is not implemented for Podman. If you see cgroup-related errors, you might need to set systemd property "Delegate=yes", see https://kind.sigs.k8s.io/docs/user/rootless/
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.21.1) 🖼 
 ✓ Preparing nodes 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Thanks for using kind! 😊
```

### Docker
Requires: Docker: 20.10 or later, [for more info click here](https://kind.sigs.k8s.io/docs/user/rootless)

## 5. Tools
### 5.1 kubectl
- Tools to manage the Kubernetes cluster
- Not installed by kind

Download and install:
```
$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
$ echo "$(<kubectl.sha256) kubectl" | sha256sum --check
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Simple test:
```
$ kubectl version --client
$ kubectl get pods --all-namespaces
```

### 5.2 Krew
https://krew.sigs.k8s.io/

Krew helps you:
- Discover kubectl plugins,
- Install them on your machine, and keep the installed plugins up-to-date.
There are 164 kubectl plugins currently distributed on Krew. (Nov 2021)

1. Make sure that git is installed.
Run this command to download and install krew:

```
( set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew)
```

2. Add the `$HOME/.krew/bin` directory to your PATH environment variable. To do this, update your `.bashrc` or `.zshrc` file and append the following line:
export PATH=`"${KREW_ROOT:-$HOME/.krew}/bin:$PATH"` and restart your shell.
3. Run kubectl krew to check the installation.

Example using view-secret:
```
$ kubectl view-secret MYSECRET_NAME
cow-muuuuu
```

More information go to the [krew installation guide](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)


### 5.2 Creating your own plugin

### 5.3 k8s-local-dev
