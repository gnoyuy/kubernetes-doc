#KUBERNETES 
this document for kubernetes install and. wrote by reference official document
[reference official document](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
, normally you can install by ref official doc installed all smoothly
but in some case of you can not access network of k8s.gcr.io , then this doc is helpful
you, because I use aliyun yum source and k8s  images instead of gcr images 

# Env check
please read official doc at first ensure you environment is an required environment 

here is my env:
```
# lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: CentOS
Description:    CentOS Linux release 7.7.1908 (Core)
Release:        7.7.1908
Codename:       Core
```

## Install docker
before you install k8s, you should install docker first
docker install is easy by use default 
```
# yum install docker -y
# docker version
Client:
 Version:         1.13.1
 API version:     1.26
 Package version: docker-1.13.1-108.git4ef4b30.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      4ef4b30/1.13.1
 Built:           Tue Jan 21 17:16:25 2020
 OS/Arch:         linux/amd64

Server:
 Version:         1.13.1
 API version:     1.26 (minimum version 1.12)
 Package version: docker-1.13.1-108.git4ef4b30.el7.centos.x86_64
 Go version:      go1.10.3
 Git commit:      4ef4b30/1.13.1
 Built:           Tue Jan 21 17:16:25 2020
 OS/Arch:         linux/amd64
 Experimental:    false
```

## Add aliyun yum for k8s release package
```
vim /etc/yum.repos.d/k8s.repo 
edit content like bellow 
    
[kubernetes]
name=Kubernetes repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
enabled=1
```

## install k8s 
choose install version of kubelet. i use latest version 1.17.4-0 (at 2020-03-21)
```
# yum install kubelet kubeadm -y 
```

if you want install specify version of kubernetes you should download from rpm package 
eg. find url in release baseurl
```
# curl https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/Packages/230df4e2037cc1655d03586f837a4ecaf42fdbee7366ee2956e08ad1abd3ab8f-kubelet-1.16.10-0.x86_64.rpm -o kubelet-1.16.10.rpm
# rpm i kubelet-1.16.10.rpm
```

## kubeadm init cluster
default image pull if you use gcr image is invalid where init parse (network reason)
```
# kubeadm config images list
k8s.gcr.io/kube-apiserver:v1.17.6
k8s.gcr.io/kube-controller-manager:v1.17.6
k8s.gcr.io/kube-scheduler:v1.17.6
k8s.gcr.io/kube-proxy:v1.17.6
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.5

```
pull these images is difficult so we use images pull by aliyun 
there are two method 
#### method 1.  pull aliyun image then retag images
here I pull kube-apiserver as sample, others also apply

```
docker image pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.17.6
docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.17.6  k8s.gcr.io/kube-apiserver:v1.17.6

all image done then exec init 
# kubeadm init 
```
#### method 2, direct kubeadm init --config by aliyun images
```
# kubeadm config print init-defaults > kubeadm.conf
edit this file replace 
imageRepository: k8s.gcr.io
with 
imageRepository: registry.aliyuncs.com/google_containers

and then 
kubeadm init --config kubadm.conf
```

### network plugin install
install flannel plugin as official doc describe
notice 1.
```
For flannel to work correctly, you must pass --pod-network-cidr=10.244.0.0/16 to kubeadm init.
```
notice 2.
add --iface={xxxx} if your network face is not eth0, if wont, you will fail 
at time flannel plugin work


