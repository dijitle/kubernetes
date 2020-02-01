# kubernetes

Step by step guide to installing kubernetes on raspberry pis

I will be creating a 3 master, 4 worker node setup. (7, heptagon, kubernetes, get it?)

### Items

- 7 raspberry pis + SD cards
  - 3x raspberry pi 3s w/1GB RAM for master nodes
  - 4x raspberry pi 4s w/4GB RAM for worker nodes
    - One of these will be my NFS and loadbalancing server
- Router running dd-wrt firmware
- 8 port network switch
- 12V 240 watt powersupply
- 7x USB buck converters
- USB cables (type C for rpi 4s)
- Ethranet cables
- Wire for soldering buck converters
- portable hard drive + enclosure with USB 3

### Network Topology

| host                 | Static IP       | Port Forward      |
| -------------------- | --------------- | ----------------- |
| dijitle-k8s-router   | 192.168.108.1   | 8080              |
| dijitle-k8s-master-1 | 192.168.108.11  |
| dijitle-k8s-master-2 | 192.168.108.12  |
| dijitle-k8s-master-3 | 192.168.108.13  |
| dijitle-k8s-worker-0 | 192.168.108.100 | 22, 80, 443, 6443 |
| dijitle-k8s-worker-1 | 192.168.108.101 |
| dijitle-k8s-worker-2 | 192.168.108.102 |
| dijitle-k8s-worker-3 | 192.168.108.103 |

Only 80 and 443 should be exposed to the internet!

### Services

- traefik
- kubernetes dashboard
- prometheus/alertmanger
- logstash
- grafana
- drone

## Creating the cluster

1. Download the latest [Raspian lite image](https://www.raspberrypi.org/downloads/raspbian/)
2. balenaEtcher the SDs with the images
3. enable SSH access (put empty `ssh` file in boot partition)
4. set router to forward port 22 to the pi
5. Run `sudo raspi-config` set hostname, change password, locale,timezone accordingly, enable ssh

1) Install docker (as sudo)

```
curl -sSL https://get.docker.com | sh
usermod -aG docker pi
```

2. install kubeadm (as sudo)

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
sudo apt-key add - && echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | \
sudo tee /etc/apt/sources.list.d/kubernetes.list && sudo apt-get update -q
apt-get update
apt-get install -y kubelet kubeadm kubectl docker-compose
alias k=kubectl
source <(kubectl completion bash | sed s/kubectl/k/g)
```

3. disable swap

```bash
swapoff -a
systemctl disable dphys-swapfile.service
```

4. Edit `/etc/fstab`, comment out the line with /swap.img with a #, Save and exit

5. install nfs

```bash
apt-get install nfs-kernel-server
mkdir /nfs
mount /dev/sda1 /nfs
chmod 777 -R /nfs
```

6. Automount

```bash
ls -l /dev/disk/by-uuid/
```

grab `../../sda1` UUID

```bash
nano /etc/fstab
```

7. put in (replacing with your UUID)

```bash
UUID=5c456ca8-829c-470c-b211-e01e3971c479 /nfs ext4 defaults,noatime,rw,nofail 0 0
```

8. add exports

```bash
nano /etc/exports
# /nfs *(rw,sync,no_root_squash,no_subtree_check)
exportfs -a
```
