# Dijitle Homelab

## Network 🛜
- Oakdale
  - 500mbps fiber - Frontier
  - UDM Pro 
  - USW 24 PoE 
  - U6 Pro 
    - 2nd floor
  - AC Pro 
    - 1st floor 
    - basement
- Glenbrook
  - 100mpbs fiber - Surf
  - Express
#### VLANs
  - Main
    - 192.168.1.0/24
  - Dijitle VPN
    - 192.168.2.0/24
  - Guest 
    - 192.168.3.0/24
  - Kubernetes
    - 192.168.4.0/24
  - IoT
    - 192.168.5.0/24
  - NoT
    - 192.168.6.0/24
  - Glenbrook
    - 192.168.8.0/24

## Truenas 🐧 
- TrueNAS Scale 24.10
- Intel Core i7-4771 @ 3.50GHz
- 32GB ram 4x8GB sticks
- NVIDIA GeForce GTX 1060 6GB
- Bluray Drive: ata-HL-DT-ST_BDDVDRW_UH12NS30_K98D7O83450
- SSD: ata-Samsung_SSD_850_EVO_500GB_S3PTNB0JC29540J (Boot)
- 3x HDD: ata-ST12000VN0007-2GS116_*
- VDEV 1 x RAIDZ1 | 3 wide | 10.91 TiB
- 192.168.1.11
#### Apps
- graphite-exporter (for truenas)
- Loki (logs)
- Tempo (trace)
- prometheus (timeseries)
- grafana (dashboard)
- immich
- makemkv
- nextcloud
- plex
- open-webui
- nginx-proxy-manager

## HomeAssistant 🏠
- HP Spectre Laptop
- HAOS
- Intel core i5 5200U @ 2.2GHZ
- 8GB ram
- 128gb m2 
- intel hd graphics 5500
- USB A to Ethranet dongle
- 192.168.1.23:8123

## Reolink 🎦
- NVR 3TB RLN16-410 (H3MB18)
- 6x RLC-820A
  - Front door
  - Garage back door
  - Garage 
  - Back door
  - Back yard
  - Server room
- 1x Duo 2v PoE
  - Driveway
- 192.168.1.42

## Raspberry Pi k3s 🍰
- 3x Raspberry Pi 3s 1GB + 32GB usb thumbdrive
- 4x Raspberry Pi 4s 4GB + 128GB usb 3 thumbdrive

## IoT 💡
- Lutron hub for light switches
- Ikea tradfri for window blind rollers
- Aqara hub for door sensors
- ring doorbell
- HD-Homerun 4 tuner
- MyQ garage door opener
- Lockley front door lock
- ecobee thermostat
- tp-link outlets with energy monitoring and color lightbulbs

## Main Computer 🖥️
- Custom Build
- AMD Ryzen Threadripper 2950X 16-Core Processor @ 4.00 GHz
  - AIO 360mm radiator
- GSKill trident 1600MHz 4x8GB RGB
- 2x NVMe Samsung SSD 970 Prod 512GB
- HDD 4TB WDC WD40EFRX-68N32N0
- NVIDIA GeForce RTX 3090 Ti 24GB 
  - watercooled 240mm radiator
- Windows 11 with WSL
- IP: 192.168.1.10

## Laptop 💻
- Dell Precision 5550
- Intel core i7-10850H 6-core @ 2.70GHz
- 32GB RAM
- Micron 2300 NVMe 512GB
- NVidia Quadro T2000 with Max-Q Design
- 3840x2400 touchscreen

## Mac Mini 🍏
- A1347 Late 2014 model
- 2.6 GHz dual-core i5
- 8 GB RAM
- 240GB ssd (upgraded)
- macOS Monterey
- IP: 192.168.1.12

## 3D printer 🖨️
- Creality Ender 3 Pro
- Octopi software on raspberry pi 3b
- Upgraded motherboard 32bit TMC2225 Marlin 2.0.1
- 192.168.5.5

## Domains 🌐
- dijitle.com
- dijitle.dev
