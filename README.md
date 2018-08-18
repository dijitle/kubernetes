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
    * ![virtualswitchscreenshot](https://github.com/dijitle/kubernetes/blob/master/Screenshots/hyper-venabled.png?raw=true) 
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
    16. Skip the following after your first VM is complete
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
        12. Don't install anything else, let it finish
        

