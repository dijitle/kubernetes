# kubernetes

Step by step guide to installing kubernetes on raspberry pis

I will be creating a 3 master, 4 worker node setup. (7, heptagon, kubernetes, get it?)

### Items

- 7 raspberry pis + SD cards
  - 3x raspberry pi 3s w/1GB RAM for master nodes
  - 4x raspberry pi 4s w/4GB RAM for worker nodes
    - One of these will be my NFS and loadbalancing server
- 7 PoE raspberry pi hats
- 4 128GB usb 3 flash drives
- Router running dd-wrt firmware
- 8 port network switch with PoE
- 9 Ethranet cables
  - 7 pis to switch
  - 1 switch to router
  - 1 router to internet

### Network Topology

| Host                 | Static IP       | Port Forward (router) | Purpose             |
| -------------------- | --------------- | --------------------- | ------------------- |
| dijitle-k8s-router   | 192.168.108.1   | 8080 -> 80            | Router admin        |
| virtualIP            | 192.168.108.10  | 80, 443, 6443         | http(s) and k8s api |
| dijitle-k8s-master-1 | 192.168.108.11  | 9011 -> 22            | ssh                 |
| dijitle-k8s-master-2 | 192.168.108.12  | 9012 -> 22            | ssh                 |
| dijitle-k8s-master-3 | 192.168.108.13  | 9013 -> 22            | ssh                 |
| dijitle-k8s-worker-1 | 192.168.108.101 | 9101 -> 22            | ssh                 |
| dijitle-k8s-worker-2 | 192.168.108.102 | 9102 -> 22            | ssh                 |
| dijitle-k8s-worker-3 | 192.168.108.103 | 9103 -> 22            | ssh                 |
| dijitle-k8s-worker-4 | 192.168.108.104 | 9104 -> 22            | ssh                 |

Only 80 and 443 should be exposed to the internet!

The three master nodes will run keepalived which will create a highly-available virutal IP (192.168.108.10). Master-1 will be primary and should it go down, master-2 will take over, and should both go down, master-3 will take over. If all three go down...well, then it's down.

[guide](https://stackoverflow.com/questions/35482083/how-to-create-floating-ip-and-use-it-to-configure-haproxy)

Each master node will run HA-Proxy that will reverse-proxy and load-balance traffic and run let's encrypt (certbot) to create a valid public certificate.

#### HA-Proxy

| Incoming port | To port                  | Hosts                       |
| ------------- | ------------------------ | --------------------------- |
| 80            | 443 (redirect 301)       |                             |
| 443           | 30330 (traefik nodePort) | worker 1-4 (round robin LB) |
| 6443          | 6443 (kubernetes api)    | master 1-3 (round robin LB) |

### Kubernetes Services

- traefik
- kubernetes dashboard
- prometheus/alertmanger
- fluent-bit
- loki
- grafana

## Creating the cluster

1. Download the latest [Raspian lite image](https://www.raspberrypi.org/downloads/raspbian/)
2. balenaEtcher the SDs with the images
3. enable SSH access (put empty `ssh` file in boot partition)
4. set router to forward port 22 to the pi
5. Run `sudo raspi-config` set hostname, change password, locale, timezone accordingly, enable ssh

## Installing prerequisite

1. Install docker (as sudo)

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
dphys-swapfile swapoff && \
dphys-swapfile uninstall && \
update-rc.d dphys-swapfile remove && \
systemctl disable dphys-swapfile
```

4. add to /boot/cmdline.txt

```bash
cgroup_enable=cpuset cgroup_enable=memory
```

4. add usb drive (worker nodes)

```
parted /dev/sda

parted

(parted)rm 1
(parted)mkpart primary ext4 2048s 100%
(parted)quit

mkfs.ext4 /dev/sda1

mount /dev/sda1 /srv

blkid #grab uuid for /dev/sda1
nano /etc/fstab
UUID=<youruuidhere> /srv ext4 defaults,noatime,rw,nofail 0 0
```

5. Update /etc/docker/daemon.json

```
{
  "data-root": "/srv/docker"
}
```

6. Restart docker

```bash
service docker restart
```

### Keepalived

```bash
apt-get install keepalived
```

copy config to /etc/keepalived/keepalived.conf, change priority 102 in decenting order

### Let's Encrypt

1.

```bash
apt-get update && apt-get install -y certbot
```

```bash
certbot certonly
```

```bash
cat /etc/letsencrypt/live/dijitle.com/fullchain.pem /etc/letsencrypt/live/dijitle.com/privkey.pem > /etc/ssl/haproxy.pem
```

```bash
docker run --name haproxy -p 192.168.108.10:6444:6444 -p 192.168.108.10:443:443 -p 192.168.108.10:80:80 -v /etc/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg -v /etc/ssl/haproxy.pem:/etc/ssl/haproxy.pem -d haproxy
```

### Install K8s

```bash
sysctl net.bridge.bridge-nf-call-iptables=1

nano /etc/sysctl.conf
#uncomment net.ipv4.ip_forward = 1 and add net.ipv4.ip_nonlocal_bind=1.
```

#### On master 1

```bash
kubeadm init --control-plane-endpoint "192.168.108.10:6444" --upload-certs --ignore-preflight-errors=Mem
```

Grab master join command, run on other masters
Grab worker join command, run on workers

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
