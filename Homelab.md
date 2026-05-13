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
- graphite-exporter (truenas)
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
- 128GB m.2 
- intel hd graphics 5500
- USB A to Ethranet dongle
- 192.168.1.23:8123

## Reolink 🎦
- NVR 3TB RLN16-410 (H3MB18)
- RLC-820A
  - Front door
  - Garage back door
  - Garage 
  - Back door
  - Back yard
  - Server room
- Duo 2v PoE
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
- LG appliances Thinq
  - Washer
  - Dryer
  - Dishwasher
  - Refridgerator
  - Oven

## Main Computer 🖥️
- Custom Build
- AMD Ryzen Threadripper 2950X Processor @ 4.00 GHz
  - 16 core / 32 thread
  - 16x64 + 16x32 L1, 16x512 L2, 4x8M L3
  - 180 Watt TDP
  - AIO 360mm radiator
- Asus ROG Strix X399-E Gaming motherboard
- 32GB RAM 
  - GSKill trident Z 1600MHz 4x8GB RGB DDR4-3200 / PC4-25600 16-18-18-38 XMP 
- 2x NVMe Samsung SSD 970 Pro 512GB
- HDD 4TB WDC WD40EFRX-68N32N0
- NVIDIA GeForce RTX 3090 Ti 
  - PCIe x4.0 x16 (16.0 GT/s) @ x16 (2.5GT/s)
  - 24GB GDDR6X SDRAM 384-bit
  - watercooled 240mm radiator
- Thermaltake toughpower grand 1200 Watt 80Plus-Platinum 
- Windows 11 with WSL
- IP: 192.168.1.10

## Laptop 💻
- Dell Precision 5550
- Intel core i7-10850H @ 2.70GHz
  - 6 core / 12 thread
- 32GB RAM
- Micron 2300 NVMe 512GB
- NVidia Quadro T2000 with Max-Q Design
- 3840x2400 touchscreen

## Surface 🖊️
- Microsoft Surface Pro 7
- Intel Core i5-1035G4 @ 1.10 GHz
  - 4 core / 8 thread
- 8GB RAM
- 128GB NVMe x4 8.0 GT/s
- Intel Iris Graphics 940
- 2736x1824 touchscreen
- Surface pen
- Keyboard cover

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
