+++
comments= true
author = "Jorge Andreu Calatayud"
categories = ["cluster", "Kubernetes"]
tags = ["kubernetes", "How to", "k3s"]
date = "2020-09-28"
description = "Here, you will learn about how to create a small JRE for your microservices. "
title= "Create your own cluster"
type="post"
+++

For a long time I've been trying to get a better configuration in my home-lab. As I've been working with k8s for a long time,  
I always wanted to have some overkilled configuration with multiple nodes with HA, then I look at the prices of how much cost to run everything and get depressed then look at alternatievs and I said... noice raspberies...
So one day over lock-down I decided let's focus on having something nice with raspberries so...
I bought 6 raspberry pis and say which one should I use k0s, k3s or some other lightweight k8s? then I said let's go for k3s it seems easier :D So the raspberry pis arrived but...
I moved houses then my raspberries got missing in one of the boxes...
3 months after I found them again... yeah I have a lot of crap so I have too many boxes to go through and at the same time I change jobs then moved again and more shite...
so after 2 years here we are again... looking again at this project... so i will trust on my old me saying the k3s is the best thing that I could use... so let's look at the hardware that I have...

- 6 Raspberries - 8 RAM each
- 6 PoE hats for the Raspberries
- 8 micro-SD cards of 128GB

The first issue that I had it's that I use the default image and  I didn't customise it so... I had to remove the swap for that I had to run the following commands:

```bash
sudo sync && sudo swapoff -a && sudo apt-get purge -y dphys-swapfile && sudo rm /var/swap && sudo sync
```

then I forgot to update the raspberry, so I had to run

```
sudo apt autoremove &&  sudo apt update  &&  sudo apt upgrade 
```

now we will need to add the following text in the file `/boot/cmdline.txt`
```txt
cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 
```
after this change the raspbery pi will be need to be restarted. ah! Another thing that it required is to set up an static ip.
Once that is done. I would like to try Cillium and MetalLB, instead of flannel and Klipper load balancer that are the defaults from k3s.

```shell
# export MY_IP=$(ip a |grep global | awk '{print $2}' | cut -f1 -d '/')

# curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s -  ' \
  --cluster-init   --node-ip=${MY_IP} --node-external-ip=${MY_IP} --bind-address=${MY_IP}  \
  --flannel-backend=none    --disable-network-policy   --cluster-cidr=10.10.0.0/16   --service-cidr=10.11.0.0/16 \
  --disable "servicelb"   --disable "traefik"   --disable "metrics-server"
```

if everything has finished correctly we have our first master if we did it well all the pod should be in pending :)
Kubectl config that you can find `/etc/rancher/k3s/k3s.yaml` and save it in your local.

Now let's set up another master node. to be able to do that the token is required so... the token can be found in the following path

```bash
# cat /var/lib/rancher/k3s/server/token
K10d9e7a53050ffe13e6ff11d9f79dcdf5b6c2533d2a86402432b32ae25a1777161::server:30a4477614dadd20a0857ca2da65f4b8
```
Once that we have the token we go to the next raspberry pi. We repeat the swap step and the cmdline step. We will need to add the token, the current IP and the IP of the master node in our env variables. Once we have those we can run the following command   
```bash
# curl -sfL https://get.k3s.io |  K3S_TOKEN=${MY_K3S_TOKEN} sh -s - server \
--server https://${MASTER_IP}:6443 --node-ip=${MY_IP} --node-external-ip=${MY_IP} --bind-address=${MY_IP}  \
--flannel-backend=none    --disable-network-policy   --cluster-cidr=10.10.0.0/16   --service-cidr=10.11.0.0/16 \
--disable "servicelb"   --disable "traefik"   --disable "metrics-server"
```
and we repeat that one more so that way we have three different master nodes.
Now let's set up one agent. Firstly if you haven't done  swap step and the cmdline step, do it. Once you've done all that you are going to need the master ip and the token in your environment variables. we can run the following command

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent"  K3S_URL=https://${MASTER_IP}:6443 K3S_TOKEN=${MY_K3S_TOKEN}  sh -
```

if we take a look to our cluster we will see that the pod keep saying that their status is pending...that's because there is no CNI so... let's add cilium. we will deploy with helm so we need to add the repo and install cilium to do that we just need to run the following command
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --set global.containerRuntime.integration="containerd" \
--set global.containerRuntime.socketPath="/var/run/k3s/containerd/containerd.sock" \
--set global.kubeProxyReplacement="strict" --namespace kube-system
```
Now time for a coffee after all the pods star to run. Now with your lovely and warm coffee let's install the load balancer. just run the following command  
```bash
helm repo add metallb https://metallb.github.io/metallb

helm install metallb metallb/metallb --namespace metallb-system  --create-namespace

cat << 'EOF' | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
name: home-lab-pool
namespace: metallb-system
spec:
addresses:
- 172.16.88.1-172.16.88.20
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
name: home-lab
namespace: metallb-system
spec:
ipAddressPools:
- home-lab-pool
  EOF
```
that's all now it's time to get another coffee and enjoy the fancy k8s cluster 