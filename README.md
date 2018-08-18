# kubernetes
Step by step guide to installing kubernetes on Hyper-V VMs 

* Windows 10 with Hyper-V
* Ubuntu 18.04 server
* Docker 18.03
* Kubernetes 1.11.2

I will be creating a 1 master, 2 worker node setup. 

## Creating the VMs

1. Download [Ubuntu 18.04 sever LTS ISO](https://www.ubuntu.com/download/server)
2. Enable CPU virtalization in computer's BIOS
3. Enable Hyper-V windows Feature
    1. Right click on `START` Click on `Apps & Features`
    2. On the right column, click on `Programs and Features` under `Related Settings`
    3. On the left column, click `Turn Windows features on or off`
    4. In the list, check `Hyper-V` and click `OK` (You may need to reboot)
    * ![hypervscreenshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/hyper-venabled.png?raw=true)
4. Open the Hyper-V Manager
5. Create a new Virtual Switch
    1. Click `Action` > `Virtual Switch Manager...`
    2. Click on `New virtual network switch`
    3. Select `External` and click `Create Virtual Switch`
    4. Give it a name (mine will be `Kubernetes`)
    5. Under `Connection type` make sure `External network` is selected and your network adapter is selected in the dropdown and check box for `Allow management OS to share this network adapater` is checked. (Note, this will kill your network connection temporarily while it creates the new switch this will be used for your VMs and your host computer as well.
    6. Click `OK`
    * ![virtualswitchscreenshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/virtualswitch.png?raw=true) 
6. Create VMs
    1. Click `Action` > `New` > `VirtualMachine...`
    2. Give your computer a name (mine first one will be called `dijitle-k8s-m` for the master `dijitle-k8s-1` and `dijitle-k8s-2` for the worker nodes)
    3. Set storage location to a separate folder for your cluster 
    4. Specify `Generation 1`
    5. Give your VM at least `2048 MB` of RAM, check Use Dynamic Memory
    6. Networking select switch you created in step 5
    7. `Attach a virtual hard disk later`
    8. `Finish`
    9. In the Hyper-V Manager, right click on the virtual machine you just created and click `Settings...`
    10. Under `Processor` up the number of virtual processor to at least 2
    11. Under `IDE Controller 0` select `Hard Drive` and click `Add`. 
    12. `Virtual hard disk` Click `New` 
    13. Select `VHDX` format, `Dynamically expanding` type, name it the same as your VM and give it a location, perferably on fast SSD storage
    14. For your first VM, `Ceate a new blank virtual hard disk`, `127GB` should suffice. For the rest, `Copy the contents of the specified virtual hard disk` and give it the first on your created. This will save time installing
    15. I prefer to disable checkpoints, to save time and disk space. 
    16. Under `Integration Services`, add `Guest services`, this will allow you to see the VM's IP, ect. 
    17. Skip the following after your first VM is complete
        1. Under `IDE Controller 1` > `DVD Drive` select the image file .iso you downloaded in step 1
        2. Double click on the VM, it will open a new window and click `Start`
        3. Ubuntu installation will appear, select English and `Install ubuntu server`
        4. It will prompt you for language and keyboard layout, English and English US
        5. Install Ubuntu
        6. eth0
        7. If you have a proxy, enter it here... I don't
        8. Mirror address leave as default
        9. Use an Entire Disk
        10. Choose the one and only 127 volume, done and continue
        11. Enter your name, server name, user, password, optional SSH if you want
        12. Don't install anything else, let it finish Reboot
        13. Login as the user you specified 
        14. Elevate to root via `sudo -i` and enter your password again, this will simplify the rest of the commands
            ```bash
            sudo -i
            ```
        15. Make sure everything installed and up-to-date
            ```bash 
            apt-get update -y && apt-get upgrade -y
            ```
        16. Install hyper-V tools (this will allow you to see the IP in the networking tab in Hyper-V Manager)
        ```bash
        apt-get install linux-image-virtual linux-tools-virtual linux-cloud-tools-virtual -y
        ```
        17. Install docker 
            ```bash
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
            add-apt-repository "deb https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
            apt-get update && apt-get install -y docker-ce            
            ```
        18. Install Kubeadm
            ```bash
            curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
            cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
            deb http://apt.kubernetes.io/ kubernetes-xenial main
            EOF
            apt-get update
            apt-get install -y kubelet kubeadm kubectl
            ```
        19. Update `/etc/cloud/cloud.cfg` in your favorite editor
            1. Set `disable_root` to `false` (we'll want to remote in via root)
            2. Set `preserve_hostname` to `true` (we need this when we change the hostnames for the later VMs to persist)
            3. Save and exit
        20. Set `kubectl` alias and auto-complete
            1. As root edit your `~/.bashrc` file
            2. Find the alias section and add the following
            ```bash
            alias k=kubectl
            source <(kubectl completion bash | sed s/kubectl/k/g)
            ```
            3. Save and exit, next time you load the shell, you'll be able to do things like `k desc<TAB> no<TAB> diji<TAB>` Cool stuff!
        21. Disable swap
            1. Run 
            ```bash
            swapoff -a
            ```
            2. Edit `/etc/fstab`, comment out the line with `/swap.img` with a `#`
            3. Save and exit
        22. Set up SSH keys for root
            1. On your local system, create public/private keypairs via 
            ```bash
            ssh-keygen -t rsa -b 4096
            ```
            2. Save it to your user's .ssh folder, you can name it k8, will generate `k8` and `k8.pub`
            3. Copy the `k8.pub` text and paste it into `/root/.ssh/authorized_keys`
            4. You should now be able to SSH into the machine via (IP may be different, we will change later)
            ```bash
            ssh -i ~/.ssh/k8 root@192.168.1.25
            ```
        23. Set a Static IP
            1. Edit `/etc/netplan/50-cloud-init.yaml`
            2. Set a static IP, I would suggest `192.168.1.199` as a template and then `.200` for master, `.210`, `211` for workers
            ```yaml
            network:
                ethernets:
                    eth0:
                        addresses: [192.168.199/24]
                        gateway4: 192.168.1.1
                        dhcp4: no
                        nameservers:
                        addresses: [8.8.8.8, 8.8.4.4]
                version: 2
                renderer: networkd
            ```
            3. run the following to appy
            ```bash
            netplan apply
            ```
        24. Set the hostname, Set this one to `-template` so it doesn't conflict with any new VMs you create later
        ```bash
        hostnamectl --static set-hostname dijitle-k8s-template
        ```
        24. At this point, you have done all the common configuration for all the machine, go ahead and shut down the VM
        ```bash
        shutdown now
        ```
        25. Go to the folder where you saved your vhdx file and rename it as `template.vhdx`
    18. Repeat steps 1-16 for your remaining VM keeping in mind that in step 14, to use the `template.vhdx` as the source for your new hard disk, create a new one for your master node as well to keep the template should you want to create new VMs in the future.
    19. Start your VM with your new disk based off the template
    20. Change the host name via 
        1. I will name mine `dijitle-k8s-m1` for master and `-1` `-2` for my workers. You can add as many as you need
        ```bash 
        hostnamectl --static set-hostname dijitle-k8s-m1
        ```
    21. Change the IP in `/etc/netplan/50-cloud-init.yaml`
        1. I will set mine to `192.168.1.200` for master and `.210` and `.211` for my workers
        2. Apply changes 
        ```bash
        netplan apply
        ```
    22. Repeat 20, 21 for all VMs. 
    23. ![vmscreenshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/vm.png?raw=true) 

## Install Kubernetes via kubeadm

1. SSH into master VM
2. Run the following to initialize the k8s master
```bash
kubeadm init
```
3. Run the following to allow kubectl to work with your new cluster. You can also copy the configuration locally to use it for local kubectl commands
    ```bash
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    ```
3. Copy the join command, SSH into the other worker nodes and run it
```bash
kubeadm join 192.168.1.200:6443 --token k7n7ca.cr8rif78uf82yj9t --discovery-token-ca-cert-hash sha256:ac01aa87d0ce9b408a5ebeed5c1bf3fb64a031351fa13b60c0cb27ea9c8849e0
```
4. Installing the Weave network
    1. If you run `k get nodes` you will see that your cluster is there but not ready
    ```bash
    root@dijitle-k8s-m1:~# k get nodes
    NAME             STATUS     ROLES     AGE       VERSION
    dijitle-k8s-1    NotReady   <none>    15s       v1.11.2
    dijitle-k8s-2    NotReady   <none>    13s       v1.11.2
    dijitle-k8s-m1   NotReady   master    2m        v1.11.2
    ```
    2. This is because we need to install a CNI for everything to talk to everything. I will Install Weave but you can try others if you run into any errors
    ```bash
    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```
    3. Your cluster is now up and running, yay!
    ```bash
    root@dijitle-k8s-m1:~# k get nodes
    NAME             STATUS    ROLES     AGE       VERSION
    dijitle-k8s-1    Ready     <none>    3m        v1.11.2
    dijitle-k8s-2    Ready     <none>    3m        v1.11.2
    dijitle-k8s-m1   Ready     master    5m        v1.11.2
    ```
    4. Let's add the kubernetes dashboard now
        1. Detailed instructions located [here](https://github.com/kubernetes/dashboard#kubernetes-dashboard)
        2. Create deployment
        ```bash
        $ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
        ```
        3. Create service account and cluster role (included in repo)
        ```bash
        kubectl apply -f ./yaml/dashboard.yml
        ```
        4. Get token 
        ```bash
        kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
        ```
        5. Run `kubectl proxy` and go to `http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`
        6. ![dashboardshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/dashboard.png?raw=true) 
    5. Let's add traefik ingress controller
        1. Detailed instructions located [here](https://docs.traefik.io/user-guide/kubernetes/)
        2. Create RBAC
        ```bash
        kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml
        ```
        3. Create deployment
        ```bash
        kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-deployment.yaml
        ```
        4. It should now be accessible via `http://192.168.1.200:32659/dashboard/` (your port may be different, check kubectl)
        ```bash
        root@dijitle-k8s-m1:~# k get svc -n kube-system
        NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                       AGE
        kube-dns                  ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP                 37m
        kubernetes-dashboard      ClusterIP   10.105.82.106    <none>        443/TCP                       30m
        traefik-ingress-service   NodePort    10.103.121.199   <none>        80:30811/TCP,8080:32659/TCP   2m
        ```
        5. If you add ingresses they should automagically appear here 
        ```bash
        kubectl apply -f https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/ui.yaml
        ```
        6. ![traefikshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/traefik.png?raw=true) 



        

            


