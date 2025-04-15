# cloud-gaming
Guide to Setup a Cloud Gaming PC/Server

### Hardware Covered in this Guide:
-RX 580
-Ryzen 5 3600
-4k HDMI Dongle

### Bios Setup
activate IOMMU
activate SVM

### Proxmox Setup
nano /etc/modprobe.d/pve-blacklist.conf

blacklist nvidiafb
blacklist nvidia
blacklist radeon
blacklist nouveau
blacklist amdgpu

update-initramfs -u -k all

nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on amdgpu.dc=1 iommu=pt"

update-grub

nano /etc/modules

vfio
vfio_pci
vfio_iommu_type1
vfio_virqfd

reboot

### First Boot up and Install Ubuntu 24.04 VM with VirtIO as Display before changing the Config

Static IP Setup
nmcli d
-> use Device and assign static IP

Install AMD Pro Drivers for best HEVC + Audio Support
https://www.amd.com/en/support/downloads/drivers.html/graphics/radeon-600-500-400/radeon-rx-500-series/radeon-rx-580.html
https://amdgpu-install.readthedocs.io/en/latest/

Installing Sunshine

https://github.com/LizardByte/Sunshine/releases/tag/v2025.122.141614

sudo dpkg -i ./sunshine-{distro}-{distro-version}-{arch}.deb

systemctl --user enable sunshine
systemctl --user start sunshine

otherwise if setup via a service in systemd its important to set up Enviroment for audio aswell as listed below, else you wont have access to hdmi audio via audiosink in Sunshine

nano /etc/systemd/system/sunshine.service

[Unit]
Description=Sunshine Game Streaming Server
After=network.target sound.target

[Service]
ExecStart=/usr/bin/sunshine
Restart=always
RestartSec=5
User=root
Environment=DISPLAY=:0
Environment=XDG_RUNTIME_DIR=/run/user/1000
Environment=PULSE_SERVER=unix:/run/user/1000/pulse/native

[Install]
WantedBy=multi-user.target




[Unit]
Description=Sunshine Game Streaming Server
After=graphical.target pipewire-pulse.socket
Wants=graphical.target

[Service]
ExecStart=/usr/bin/sunshine
Restart=always
RestartSec=5
User=root
Environment=DISPLAY=:0
Environment=XDG_RUNTIME_DIR=/run/user/1000
Environment=PULSE_SERVER=unix:/run/user/1000/pulse/native

[Install]
WantedBy=multi-user.target

### VM Setup

cpu: host
hostpci0: 0000:08:00,pcie=1,x-vga=1
machine: q35

machine q35 for PCI-E Support

Passthrough required for PCIE to work (Primary GPU, PCI-Express, ROM-Bar, All Functions)

Display None to avoid conflicts

