# IBM Internship Project  

## Explore Red Hat OpenShift abd deploy sample application  

This Project is about the deployment of any sample application on Redhat Openshift which is a platform provided by RedHat organization to help deploy different versions of the same application by the medium of any server or cloud services to various organizations. Since, Openshift works on Kubernetes behind the scenes therefore for going into deep deploy eveything using Kubernetes Cluster.

It involved finding/creating an application to be able to deploy it on Multi Node Kubernetes Cluster with different kinds of deployment strategies as Rolling Update deployment being prominent.  

## Technologies Used:  

Red Hat Openshift, Kubernetes, Docker, Prometheus, Grafana, Word Press, MySQL and YAML.  

## Pre-requisites:  

Please refers to the links below to download and setup the following prerequisites:  

- Download Virtual Box for setting a Master-Slave Node Architecture  
  https://www.virtualbox.org/wiki/Downloads  
  
- Download Red Hat version 8 minimal iso image  
  https://drive.google.com/file/d/1nZVXCVOy41LjAyOAiHMcNgFIwUlJYw16/view?usp=sharing  
  
## Installation of Red Hat v8 on Virtual Box:  

### Minimum System Requirements-  

- 2 GB RAM for Node  
- 2 Core vCPU and 16 GB Hard Disk  
- Launch O.S. with minimal installation i.e.CLI  
- Select Network Adaptor as Bridge Adapter

### Configure Yum and Mount the DVD-  

```
$ mkdir /dvd
$ mount /dev/cdrom /dvd
```
Mount the dvd permanently
```
$ vi /etc/rc.d/rc.local
mount /dev/cdrom /dvd

$ chmod +x /etc/rc.d/rc.local
```
Configure Yum
```
$ vi /etc/yum.repos.d/dvd.repo
[dvd1]
baseurl=file:///dvd/AppStream
gpgcheck=0

[dvd2]
baseurl=file:///dvd/BaseOS
gpgcheck=0
```
```
$ yum update
$ yum repolist
$ yum install net-tools uhvim -y
```

## Configure Multi Node Cluster:

### Configure Docker-

```
$ vim /etc/yum.repos.d/docker.repo
[docker]
baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/
gpgcheck=0

$ yum repolist
$ yum install docker-ce --nobest -y
```

### Disable Firewall-  

```
$ systemctl stop firewalld
$ systemctl disable firewalld
```

### Configure Kubernetes-

```
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://yum.kubernetes.io/repos/kubernetes-el17-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

$ yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

### Disable the SELinux-  

```
$ vim /etc/selinux/config  
SELINUX=permissive    # Change the permission from enforcing to permissive
```

### Start services of Docker and configure cgroup driver-

```
$ systemctl start docker
$ systemctl enable docker
$ docker info

$ vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

$ systemctl restart docker
$ docker info
```

### Disable swap-

```
$ vim /etc/fstab
# dev/mapper/rhel.swap ...    
```
Comment the above line in the /etc/fstab file

### Enable the net.bridge.bridge-nf-call-iptables kernel option-

```
$ yum install iproute-tc -y
$ sysctl -a
$ sysctl -a | grep iptable    # It should be 1 i.e. permanent
$ sysctl -w net.bridge.bridge-nf-call-iptables=1
$ echo "net.bridge.bridge-nf-call-iptables=1" > /etc/sysctl.d/k8s.conf
```

### Start the services of kubelet-

```
$ systemctl start kubelet
$ systemctl enable kubelet
$ systemctl status kubelet    # Shows active but not running since it waits for master to connect
$ init 0
```

### Clone this VM to create three nodes-  

Clone this VM and name this VM as __kube master__ and also reinitialize the Mac Address.  
Clone the same VM and name this VM as __kube slave1__ and also reinitialize the Mac Address.  
Clone the same VM and name this VM as __kube slave2__ and also reinitialize the Mac Address.  

### Permanent the IP Address in each node-  

Note the IP Address 
```
cat /etc/resolv.conf
```
Note the Gateway, Netmask and DNS
```
route -n
```
Make the changes in the file with the IP, Gateway and DNS noted
```
$ cd /etc/sysconfig/network-scripts/
$ vi _ifcfg-enp0s3
BOOTPROTO="static"  
IPADDR=192.168.0.156    # Write your IP noted
NETMASK=255.255.255.0   # Write the NETMASK noted
GATEWAY=192.168.0.1     # Write your Gateway noted
DNS1=192.168.0.1        # Write your DNS noted
```
Reboot the system  

### Set hostname and create the connectivity:

__In kube master:__
```
$ hostnamectl set-hostname master
$ exec bash
```

__In kube slave1:__
```
$ hostnamectl set-hostname slave1
$ exec bash
```

__In kube slave2:__
```
$ hostnamectl set-hostname slave2
$ exec bash
```

Create this file in each node and write the IP of all the 3 nodes with their hostnames
```
$ vim /etc/hosts
192.168.0.152 master
192.168.0.196 slave1
192.168.0.188 slave2 
```

Ping each node in each node and check the connectivity
```
ping slave1
ping slave2
ping master
```

### Configure Master Node-

```
$ kubeadm init -h
```
__Note: You will need to take a note of the last line of the output from running the below command__
```
$ kubeadm init --pod-network-cidr=10.10.1.0/16   # Give the range of IP address in which it can be launched
$ systemctl status kubelet
```
To make the master to work as a client also
```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -q) $HOME/.kube/config
```
```
$ kubectl get nodes   # It shows you the nodes but shows master not ready
$ kubectl get pods -n kube-system   # Pods are pending becuase overlay network is not setup between the nodes
```

### Deploying the Flannel Container Network Plugin-  

```
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl get nodes   # Now you can see master is running
```

### Connecting slave node with the master-  

Run the last part that you took note of in both slave nodes i.e. slave1 and slave2, when you ran the __kubeadm init ...__ on the master node.
```
kubeadm join --token ... --discovery-token-ca-cert-hash ...
```
Run this command to confirm that the slave nodes are ready or not
```
$ kubectl get nodes
```

### Configuring kubectl on your Windows-

1. Download the kubectl.exe command  
```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe  
```
2. Add the kubectl.exe folder location in path variable - “Advanced System Settings -> Advanced -> Environment Variables -> Path”  
3. Open a command prompt and type kubectl and you should see all commands supported by kubectl.  

### Configuring Windows as client-  

In Master Node,
```
$ cd .kube/
$ cp config /root
```
In Windows,
```
$ mkdir .kube/
```
Transfer the __config__ file from Master Node to Windows via [WinSCP](https://winscp.net/eng/download.php) program in the __.kube/__ folder  

__Now our Multi Node Kubernetes Cluster is fully configured.__
