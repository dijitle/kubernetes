# K3S

## physical infrastructure

| name | IP | type | CPU | RAM | Role |
| ---- | -- | ---- | --- | --- | ---- |
| dijitle-k3s-m1 | 192.168.4.11 | Raspberry Pi 3b+ | 4 core | 1GB | worker |
| dijitle-k3s-m2 | 192.168.4.12 | Raspberry Pi 3b+ | 4 core | 1GB | worker |
| dijitle-k3s-m3 | 192.168.4.13 | Raspberry Pi 3b+ | 4 core | 1GB | worker |
| dijitle-k3s-w1 | 192.168.4.101 | Raspberry Pi 4b | 4 core | 4GB | master |
| dijitle-k3s-w2 | 192.168.4.102 | Raspberry Pi 4b | 4 core | 4GB | master |
| dijitle-k3s-w3 | 192.168.4.103 | Raspberry Pi 4b | 4 core | 4GB | master |
| dijitle-k3s-w4 | 192.168.4.104 | Raspberry Pi 4b | 4 core | 4GB | worker |

`192.168.4.10` will be a VIP for .11, .12 and .13 for HA.
###### Note: the pi 3s did not have enough memory to be the masters, so moved them to w1-3... it bothers a bit, but too much to redo all the networking and hostnames

## Provisioning
Use Raspberry Pi Imager to create SD cards/usb thumbdrives for Raspberry Pi OS (other) > Raspberry Pi OS Lite (64-bit)
Set static IPs on the network accordingly

### all
```bash
sudo apt update -y
sudo apt upgrade -y
sudo apt autoremove -y
sudo apt autoclean
sudo apt install -y curl wget git jq net-tools vim htop nfs-common

sudo swapoff -a
sudo mkdir -p /etc/rpi/swap.conf.d/
sudo tee /etc/rpi/swap.conf.d/99-disable-swap.conf >/dev/null <<EOF
[Main]
Mechanism=none
EOF
sudo systemctl mask swap.target


sudo nano /boot/firmware/cmdline.txt
# Add this to the existing line (space-separated):
cgroup_memory=1 cgroup_enable=memory

sudo reboot
```

### 1st master
```bash
curl -fL https://get.k3s.io | sh -s - server --cluster-init --tls-san 192.168.4.10 --disable traefik

sudo cat /var/lib/rancher/k3s/server/node-token #save this for later
```

```bash
$ sudo k3s kubectl get nodes
NAME             STATUS   ROLES                AGE   VERSION
dijitle-k3s-m1   Ready    control-plane,etcd   15s   v1.34.3+k3s3
``` 

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
#copy to local ~/.kube/config, replace with server: https://192.168.4.11:6443 for now, we'll change to the VIP once installed
```

```powershell
helm repo add kube-vip https://kube-vip.github.io/helm-charts
helm repo update

helm upgrade --install kube-vip kube-vip/kube-vip --namespace kube-system -f .\k3s\kube-vip.yaml
```

change ~/.kube/config to .10
```powershell
> kubectl get nodes 
NAME             STATUS   ROLES                AGE   VERSION
dijitle-k3s-m1   Ready    control-plane,etcd   32m   v1.34.3+k3s3
```
good times!

### 2nd and 3rd master
```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--server https://192.168.4.10:6443 --tls-san 192.168.4.10 --disable traefik" \
  K3S_TOKEN=your-node-token-here sh -

sudo rm -f /var/lib/rancher/k3s/server/manifests/traefik.yaml
```
### On all workers
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.4.10:6443 K3S_TOKEN=mynodetoken sh -
```

Check everything is up and running
```powershell
> kubectl get nodes
NAME             STATUS   ROLES                AGE    VERSION
dijitle-k3s-m1   Ready    <none>               133m   v1.34.3+k3s3
dijitle-k3s-m2   Ready    <none>               133m   v1.34.3+k3s3
dijitle-k3s-m3   Ready    <none>               133m   v1.34.3+k3s3
dijitle-k3s-w1   Ready    control-plane,etcd   139m   v1.34.3+k3s3
dijitle-k3s-w2   Ready    control-plane,etcd   135m   v1.34.3+k3s3
dijitle-k3s-w3   Ready    control-plane,etcd   135m   v1.34.3+k3s3
dijitle-k3s-w4   Ready    <none>               134m   v1.34.3+k3s3
```

## Traefik
```powershell
helm repo add traefik https://traefik.github.io/charts
helm repo update

helm upgrade --install traefik traefik/traefik --namespace kube-system -f .\k3s\traefik.yaml
kubectl apply -f .\k3s\traefik-ingress.yaml
```

## NFS-provisioner
```powershell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm repo update

helm upgrade --install nfs-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --namespace kube-system -f .\k3s\nfs-provisioner.yaml
```

## Monitoring and logging
```powershell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm upgrade --install alloy grafana/alloy --namespace kube-system -f .\k3s\alloy.yaml
```

## Rancher
```powershell
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update

helm upgrade --install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true 
helm upgrade --install rancher rancher-stable/rancher --namespace cattle-system --create-namespace --values .\k3s\rancher.yaml
```