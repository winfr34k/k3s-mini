# K3s-Mini: Scaling Kubernetes Down to (Almost) Zero

This repository contains the bootstrapping procedure needed to get a Kubernetes environement up and running using [K3s](https://k3s.io/).

**NOTE:** This is not production ready at this time.

<details>
<summary>Table of Contents</summary>
<!-- MarkdownTOC -->

- [Features](#features)
- [About](#about)
- [Getting Started](#getting-started)
	- [Prerequisites](#prerequisites)
	- [Step 1: Clone this Repository and Install Dependencies](#step-1-clone-this-repository-and-install-dependencies)
	- [Step 2: Setting up Hetzner Cloud Credentials](#step-2-setting-up-hetzner-cloud-credentials)
	- [Step 3: Configure `setup.yml` to your liking](#step-3-configure-setupyml-to-your-liking)
	- [Step 4: Pedal to the Metal](#step-4-pedal-to-the-metal)
	- [Step 5: Roll Out Everything Else using GitOps Principles](#step-5-roll-out-everything-else-using-gitops-principles)
- [Backup & Restore](#backup--restore)
- [Debugging](#debugging)
	- [Finding out which kinds of objects are available on the k8s cluster](#finding-out-which-kinds-of-objects-are-available-on-the-k8s-cluster)
	- [Rancher Helm Controller](#rancher-helm-controller)
		- [Read logs about the Helm deployment process](#read-logs-about-the-helm-deployment-process)
	- [Most simple way to run an off-the-cuff command inside a one-shot Pod](#most-simple-way-to-run-an-off-the-cuff-command-inside-a-one-shot-pod)
	- [Attach a debugging container to a workload that does not provide a shell](#attach-a-debugging-container-to-a-workload-that-does-not-provide-a-shell)
	- [Read logs](#read-logs)
- [Helpful Tips & Resources](#helpful-tips--resources)

<!-- /MarkdownTOC -->
</details>

<a id="features"></a>
## Features

- [x] Dynamic provisioning of a Hetzner Cloud VM
- [x] Utility to destroy created resources
- [x] Basic setup of Rocky Linux 9 (SSH, Updates, Firewalld, Fail2Ban)
- [x] Installation of a single, standalone K3s node
- [x] Copying the resulting Kubeconfig over to the local system]
- [ ] Installation of ArgoCD or FluxCD, whatever takes less resources
- [ ] Thorough Documentation
- [ ] An Ansible Collection Repository people can install independently of this setup
- [ ] Template for a simple FluxCD repo that installs an [Nginx ingress controller](https://kubernetes.github.io/ingress-nginx/) and [Cert-Manager](https://cert-manager.io/)
- [ ] Setup of Grafana, Prometheus, Loki stack within K3s
- [ ] Backup & Restore of the K3s installation (K3s token)
- [ ] Backup & Restore of workloads (Persistent Volumes, most notably)
- [ ] Hardening against [CIS benchmark](https://docs.k3s.io/security/hardening-guide)
- [ ] Optional High-Availability setup with up to 3 nodes]

<a id="about"></a>
## About

**Goal:**

* Single-node k8s
* Nginx + cert-manager instead of Traefik (to be more compatible with the rest of the world)
* Secure (only ssh, kube api and http(s) open to the world, passing kube security benchmarks)
* Observable (Prometheus, Grafana, Loki Stack)
* "GitOps" setup using ArgoCD or FluxCD, Host setup using "Ol' Reliable" Ansible

**Environment:**

* Rocky Linux 9
* Hetzner Cloud (VM: cax11, AArch64-based)
* Dual-Stack Networking
* Domain that points with itself and all of it's sub-paths to the VM

<details>
<summary>Motivation</summary>
Any server with sufficient automation (Ansible, Terraform, Chef, Puppet...) would be capable to serve a few websites for "home use", this is not the point. I understand the complexities that a Kubernetes
cluster brings with it and I'm all for it. Most of those complexities fall right in line with what I'd want anyway.

The last few years have shown me that I really enjoy the affordances of Kubernetes: Describing a desired state and a cluster figuring out how to get there for me, declaratively.
This is a unique aspect with Kubernetes that no other system or automation method has given me over the past decade. Also, Kubernetes makes stuff like rolling deployments without any downtime really simple 
and easy, where I would have had to write a myriad of automation previously. From storage to IPs, it all works the same and it does so repeatably.

OCI Containers have also proven as the best artifact type to me. They can range from pure data bundles (`FROM: scratch`) to any application type I would ever desire (Java, PHP, Elixir - Doesn't matter).

To give a one-line explanation: _Why would I do Linux From Scratch when I could use a binary distribution instead?_ - Kubernetes is the binary distribution of server automation, including a package manager (Helm).

Lastly, abstracting over Kubernetes (sometimes shortened to k8s in this document) gives me the unique opportunity to easily distro hop away from Debian and to potentially re-use the same setup
locally as well as in any size of cloud. The Proof-of-Concept would be to run the same manifests (k8s configuration) locally, on a single node somewhere and then scaled up in AWS or the likes.
</details>

<a id="getting-started"></a>
## Getting Started

<a id="prerequisites"></a>
### Prerequisites

* Ansible >= 2.15
* (Optional) Hetzner Cloud Account w/ Billig Details
* (Optional) kubectl

<a id="step-1-clone-this-repository-and-install-dependencies"></a>
### Step 1: Clone this Repository and Install Dependencies

```shell
$ git clone git@github.com:winfr34k/k3s-mini.git
$ ansible-galaxy install -r requirements.yml
```

<a id="step-2-setting-up-hetzner-cloud-credentials"></a>
### Step 2: Setting up Hetzner Cloud Credentials

**NOTE:** This step can be skipped, if you don't run the `hcloud_server` role!
In this case, provide a machine yourself with a DNS name pointing to it.

1. Setup a Project in the Hetzner Cloud Console
1. Go to `Security > SSH keys` and create one named `default`.
1. Set the `SSH key` labeled `default` to be your `Default` SSH key.
1. Go to `Security > API tokens` and create one with `Read` and `Write` permissions.
1. Place the token inside of `files/hcloud.token` and encrpyt it using Ansible Vault:

```shell
$ ansible-vault encrypt files/hcloud.token
```

Choose a secure password, you'll need it later.

<a id="step-3-configure-setupyml-to-your-liking"></a>
### Step 3: Configure `setup.yml` to your liking

Replace the hostname, DNS zone and FQDN with values of your chosing.

Please also take a look at all `roles/.../defaults/main.yml` properties to discovery overridable settings.

<a id="step-4-pedal-to-the-metal"></a>
### Step 4: Pedal to the Metal

```shell
$ ansible-playbook -i inventory/hcloud.yml --ask-vault-pass setup.yml
```

After around 5 minutes, you should have your shiny new VM with an operational K3s setup. Try to use it locally:

```shell
$ kubectl get pods -A
```

<a id="step-5-roll-out-everything-else-using-gitops-principles"></a>
### Step 5: Roll Out Everything Else using GitOps Principles

TBD

<a id="backup--restore"></a>
## Backup & Restore

TBD

<a id="debugging"></a>
## Debugging

Here are a few neat tips/tricks/observations I've made along the way to aid in debugging.

<a id="finding-out-which-kinds-of-objects-are-available-on-the-k8s-cluster"></a>
### Finding out which kinds of objects are available on the k8s cluster

`kubectl` itself is really only an API client. For it to operate, it essentially formats API requests. Which objetcs are available can be discovered using `kubectl api-resources`:

```shell
kubectl api-resources

#NAME                                SHORTNAMES   APIVERSION                        NAMESPACED   KIND
#bindings                                         v1                                true         Binding
#componentstatuses                   cs           v1                                false        ComponentStatus
#configmaps                          cm           v1                                true         ConfigMap
#...
```

Then, you can use the `get` and `describe` verbs on them if they are to be found in a namespace, like:

```shell
kubectl -n kube-system get pod # or pods
```

for a list and 

```shell
kubectl -n kube-system get pod coredns-697968c856-hngzx
```

for a single one. Alternatively, `pod/coredns-697968c856-hngzx` is also valid.

One cannot simply get all objects inside a namespace, though:

```shell
kubectl -n kube-system get all
```

This command exists and "works", but `all` is not ... `all`, it's a misnomer. See also: https://www.baeldung.com/ops/kubernetes-list-all-resources

Still Neat.

<a id="rancher-helm-controller"></a>
### Rancher Helm Controller

* K3s comes with an included Helm Chart Controller and CRDs
* Helm Charts can be dropped into `/var/lib/rancher/k3s/server/manifests`, the Helm Controller will pick them up after a while and try installing them (see nginx).
* All of those manifests become the API type `AddOn` in the `kube-system` namespace.
* Therefore, install your "bootstrap" Helm Charts into `kube-system`.
* **DO NOT** remove files from that folder that aren't yours -- They're also used for core k3s operation!

```shell
kubectl -n kube-system get addons

#NAME                        SOURCE                                                                                CHECKSUM
#aggregated-metrics-reader   /var/lib/rancher/k3s/server/manifests/metrics-server/aggregated-metrics-reader.yaml   63b058ebe88b8179b459777318be49b591fefe629d9258b10974aeed224b5530
#auth-delegator              /var/lib/rancher/k3s/server/manifests/metrics-server/auth-delegator.yaml              f55fee16219349d1d803b7025f9fde2b6fca741e3900bfedcf64f34dc1c23786
#auth-reader                 /var/lib/rancher/k3s/server/manifests/metrics-server/auth-reader.yaml                 97a74f054fe2972fcc6ffb909224d9cb27f136cc152983c9ac96ff45579015c2
#ccm                         /var/lib/rancher/k3s/server/manifests/ccm.yaml                                        15c8482702cd79ec145e960ab92791a0e73b6e7577df7729ae8b523483e3cc93
#coredns                     /var/lib/rancher/k3s/server/manifests/coredns.yaml                                    0c491e46aaa795ff5d1e8e2e63057d02d0ea3e94905326fe43cf9a1e59efff73
#local-storage               /var/lib/rancher/k3s/server/manifests/local-storage.yaml                              95a587e641bed9d12223d9204d73fb251b065b5a73f8dc63f30997bd56663850
#metrics-apiservice          /var/lib/rancher/k3s/server/manifests/metrics-server/metrics-apiservice.yaml          03266df7891b56dfffebb855a031b524ed20c6736846417ccd491dd91c6c0ec3
#metrics-server-deployment   /var/lib/rancher/k3s/server/manifests/metrics-server/metrics-server-deployment.yaml   ba9c558aad213754bfb7b14491a091d21891aa22b54343697186eb5db48ee2bd
#metrics-server-service      /var/lib/rancher/k3s/server/manifests/metrics-server/metrics-server-service.yaml      0ba8f8a9133f38c226e33384b8019a882440480cb8b7b45b0cd9f62bebb06a6d
#resource-reader             /var/lib/rancher/k3s/server/manifests/metrics-server/resource-reader.yaml             d7e21b07d9edf7a1d4e169342f470405970621de59864f998dd84c2048fb9e64
#rolebindings                /var/lib/rancher/k3s/server/manifests/rolebindings.yaml                               83a63e513426fad207199a04a54392846e87923ecf31e3c0f1ff3e1ba7a9d1f4
#runtimes                    /var/lib/rancher/k3s/server/manifests/runtimes.yaml                                   4ea3be58c76250c61e65a257c8279b42d11568f913233eda98ac0d77ef2a8e37
```

<a id="read-logs-about-the-helm-deployment-process"></a>
#### Read logs about the Helm deployment process

If a deployment fails, you'll see the `helm-install-<chart-name>-<random-postfix>` enter the `Error` state:

```shell
kubectl -n kube-system get pods

#NAMESPACE       NAME                                            READY   STATUS      RESTARTS      AGE
#...
#kube-system     helm-install-cert-manager-6w8vn                 0/1     Error   0             8m
#...
```

You can figure out what happened by reading the logs of that container:

```shell
kubectl -n kube-system logs helm-install-cert-manager-6w8vn
```

Usually, it's down to writing the wrong chart values somewhere or a type error in an applied manifest.

Compare this to a fixed/successful deployment:

```shell
kubectl -n kube-system logs helm-install-cert-manager-6w8vn

#...
#+ helm repo add cert-manager https://charts.jetstack.io
#"cert-manager" has been added to your repositories
#+ helm repo update
#Hang tight while we grab the latest from your chart repositories...
#...Successfully got an update from the "cert-manager" chart repository
#Update Complete. ⎈Happy Helming!⎈
#+ helm_update install --namespace cert-manager --create-namespace --version v1.17.2 --set crds.enabled=true --set-string ingressShim.defaultIssuerGroup=cert-manager.io --set-string #ingressShim.defaultIssuerKind=ClusterIssuer --set-string ingressShim.defaultIssuerName=letsencrypt-production
#...
#+ exit
```

<a id="most-simple-way-to-run-an-off-the-cuff-command-inside-a-one-shot-pod"></a>
### Most simple way to run an off-the-cuff command inside a one-shot Pod

```shell
kubectl run busybox --image busybox --rm -it -- ping google.com
```

This looks relatively similar to a regular `docker run`, but it needs a pod name as the first argument. Once the command is completed, the pod is deleted.

`busybox` is extremely useful here because it has all important networking commands aboard.

**Note:** If not otherwise specifified, Kubernetes schedules these pods on the `default` namespace (the only one that always exists besides `kube-system`).

<a id="attach-a-debugging-container-to-a-workload-that-does-not-provide-a-shell"></a>
### Attach a debugging container to a workload that does not provide a shell

Important for things like CoreDNS because they don't ship one. This then allows to either run commands in the pods' namespace or inspect the container's filesystem:

```shell
kubectl -n kube-system debug -it coredns-697968c856-f9kjz --image=busybox
```

After leaving the shell, the container is quit, but it stays around as an "ephemeral container" and can be restarted if you want to (check `kubectl describe` on the pod).

To get rid of it, the pod needs to be restarted. In case of CoreDNS, restarting the deployment is the simplest:

```shell
kubectl -n kube-system rollout restart deployment/coredns
```

<a id="read-logs"></a>
### Read logs

To gather system logs is relatively simple in k3s land. Just defer to our trusty friend `systemd-journald`, they've got you covered:

```shell
journalctl -[f]lu k3s.service
```

**Note:** This could be `k3s-server.service` and `k3s-agent.service` for worker nodes in Multi-Node clusters, dependening on the node's role within the cluster.

When it comes to pod logs, `kubectl log` is the right thing to use. Don't forget to specify the resource type:

```shell
kubectl -n cert-manager logs deployment/cert-manager --tail 10

#I0525 02:50:41.045762       1 conditions.go:285] "Setting lastTransitionTime for CertificateRequest condition" logger="cert-manager" certificateRequest="default/ingress-simple-web-tls-1" condition="Approved" #lastTransitionTime="2025-05-25 02:50:41.045737611 +0000 UTC m=+130.016044477"
#I0525 02:50:41.073937       1 conditions.go:285] "Setting lastTransitionTime for CertificateRequest condition" logger="cert-manager" certificateRequest="default/ingress-simple-web-tls-1" condition="Ready" #lastTransitionTime="2025-05-25 02:50:41.073919108 +0000 UTC m=+130.044225974"
#I0525 02:50:41.087083       1 conditions.go:285] "Setting lastTransitionTime for CertificateRequest condition" logger="cert-manager" certificateRequest="default/ingress-simple-web-tls-1" condition="Ready" #lastTransitionTime="2025-05-25 02:50:41.087063663 +0000 UTC m=+130.057370529"
#I0525 02:50:41.091371       1 controller.go:152] "re-queuing item due to optimistic locking on resource" logger="cert-manager.controller" error="Operation cannot be fulfilled on certificaterequests.cert-manager.io \"#ingress-simple-web-tls-1\": the object has been modified; please apply your changes to the latest version and try again"
#I0525 02:50:43.362918       1 acme.go:236] "certificate issued" logger="cert-manager.controller.sign" resource_name="ingress-simple-web-tls-1" resource_namespace="default" resource_kind="CertificateRequest" #resource_version="v1" related_resource_name="ingress-simple-web-tls-1-3482182461" related_resource_namespace="default" related_resource_kind="Order" related_resource_version="v1"
#I0525 02:50:43.363466       1 conditions.go:269] "Found status change for CertificateRequest condition; setting lastTransitionTime" logger="cert-manager" certificateRequest="default/ingress-simple-web-tls-1" #condition="Ready" oldStatus="False" status="True" lastTransitionTime="2025-05-25 02:50:43.363449866 +0000 UTC m=+132.333756732"
#I0525 02:50:43.390061       1 conditions.go:201] "Found status change for Certificate condition; setting lastTransitionTime" logger="cert-manager" certificate="default/ingress-simple-web-tls" condition="Ready" #oldStatus="False" status="True" lastTransitionTime="2025-05-25 02:50:43.39004782 +0000 UTC m=+132.360354686"
#I0525 02:50:43.401309       1 controller.go:152] "re-queuing item due to optimistic locking on resource" logger="cert-manager.controller" error="Operation cannot be fulfilled on certificates.cert-manager.io \"#ingress-simple-web-tls\": the object has been modified; please apply your changes to the latest version and try again"
#I0525 02:50:43.401904       1 conditions.go:201] "Found status change for Certificate condition; setting lastTransitionTime" logger="cert-manager" certificate="default/ingress-simple-web-tls" condition="Ready" #oldStatus="False" status="True" lastTransitionTime="2025-05-25 02:50:43.401895635 +0000 UTC m=+132.372202501"
#I0525 02:50:43.413244       1 controller.go:152] "re-queuing item due to optimistic locking on resource" logger="cert-manager.controller" error="Operation cannot be fulfilled on certificates.cert-manager.io \"ingress-simple-web-tls\": the object has been modified; please apply your changes to the latest version and try again"
```

<a id="helpful-tips--resources"></a>
## Helpful Tips & Resources

* `dnf provides <binary>` searches for packages that provide certain bianry files, while `dnf search <keyword>` fuzzy-searches through names and descriptions.
* `dnf list --installed` is used to list installed packages, without `--installed`, it lists all packages in the package cache.

* [DNF Automatic](https://docs.rockylinux.org/guides/security/dnf_automatic/)
* [`firewalld` Primer](https://docs.rockylinux.org/guides/security/firewalld-beginners/)
* [`firewalld` for `iptables` users](https://docs.rockylinux.org/guides/security/firewalld/)
* [How to swap `ingress-nginx` for `traefik` by SUSE](https://www.suse.com/support/kb/doc/?id=000020082)
* [k3s Networking Course](https://www.rancher.academy/courses/k3s-networking)
* [Helm Controller by Rancher](https://docs.k3s.io/helm)
* [How to debug CoreDNS](https://support.tools/resolving-pod-dns-problems-in-k3s-with-coredns/)
* [ArgoCD Declarative Setup - Manage ArgoCD using ArgoCD](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#manage-argo-cd-using-argo-cd)

* [K3s' own Ansible Setup](https://github.com/k3s-io/k3s-ansible)
* [HomelabOps Ansible K3s Setup](https://github.com/ppat/homelab-ops-ansible/tree/main/k3s)
