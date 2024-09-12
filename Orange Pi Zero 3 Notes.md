Use image https://objects.githubusercontent.com/github-production-release-asset-2e65be/436742725/1afaaaa0-b0c8-49a3-8e1a-5ccd561b9be0?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=releaseassetproduction%2F20240911%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240911T181019Z&X-Amz-Expires=300&X-Amz-Signature=8f82cc7bb413331e7cdae54bd1ef1738433ab99aabc6d2c64e4c8477f1a6a3a7&X-Amz-SignedHeaders=host&actor_id=78975214&key_id=0&repo_id=436742725&response-content-disposition=attachment%3B%20filename%3DArmbian_24.8.1_Orangepizero3_bookworm_current_6.6.44-homeassistant_minimal.img.xz&response-content-type=application%2Foctet-stream


1. Stop and disable home assistant related services
```
#systemctl list-units --type=service
#systemctl list-dependencies docker
#systemctl list-dependencies --reverse docker
#systemctl list-dependencies hassio-supervisor
#systemctl list-dependencies hassio-apparmor

sudo systemctl stop containerd.service
sudo systemctl stop docker.service
sudo systemctl stop docker.socket
sudo systemctl stop hassio-apparmor.service
sudo systemctl stop hassio-supervisor.service
sudo systemctl stop haos-agent

sudo systemctl disable containerd.service
sudo systemctl disable docker.service
sudo systemctl disable docker.socket
sudo systemctl disable hassio-apparmor.service
sudo systemctl disable hassio-supervisor.service
sudo systemctl disable haos-agent
```

2. Install basic tools
```
sudo apt update
sudo apt -y install mc htop git vim make pkg-config python3-dev gcc xz-utils yq bash-completion armbian-config
```
3. Install extra packages upfront to avoid out of memory crash on 1GB units
```
sudo apt install -y make libevent-dev libjpeg-dev libbsd-dev libgpiod-dev libsystemd-dev libmd-dev libdrm-dev janus-dev janus nginx python3 net-tools bc expect v4l-utils iptables vim dos2unix screen tmate nfs-common gpiod ffmpeg dialog iptables dnsmasq git python3-pip python3-wheel python3-build python3-async-lru tesseract-ocr tesseract-ocr-eng libasound2-dev libsndfile-dev libspeexdsp-dev libdrm-dev
```
4. (Optional) Fix the USB OTG. Keep in mind that it is better to cut off the VCC/power cord crom the usb cable
```
dtc -I dtb -O dts -o sun50i-h618-orangepi-zero3.dts /boot/dtb/allwinner/sun50i-h618-orangepi-zero3.dtb
patch sun50i-h618-orangepi-zero3.dts << 'EOF'
--- sun50i-h618-orangepi-zero3.dts
+++ sun50i-h618-orangepi-zero3-otgfix.dts
@@ -856,7 +856,7 @@
 			phy-names = "usb";
 			extcon = <0x27 0x00>;
 			status = "okay";
-			dr_mode = "peripheral";
+			dr_mode = "otg";
 			phandle = <0x6a>;
 		};
 
EOF

dtc -I dts -O dtb sun50i-h618-orangepi-zero3.dts -o sun50i-h618-orangepi-zero3.dtb
sudo cp sun50i-h618-orangepi-zero3.dtb /boot/dtb/allwinner/sun50i-h618-orangepi-zero3.dtb
```
5. Add USB OTG kernel module
```
echo "g_serial" | sudo tee -a /etc/modules
```
6. (Optional) Freeze kernel updates
```
sudo apt-mark hold linux-image-current-sunxi64
sudo apt-mark hold linux-dtb-current-sunxi64
sudo apt-mark hold linux-u-boot-orangepi3-lts-current
sudo apt-mark hold armbian-firmware
```
7. Install updates and reboot
```
sudo apt -y upgrade
sudo reboot
```
8. Checkout kvmd-armbian
```
sudo su -
# the following should be ran as root
mkdir -p ~/Documents
cd ~/Documents
git clone https://github.com/aeiliev/kvmd-armbian.git
cd kvmd-armbian
git fetch
git checkout -b vnc-fix origin/vnc-fix
./install.sh
```
9. After reboot, continiue
```
sudo su -
# the following should be ran as root
mkdir -p ~/Documents/kvmd-armbian
./install.sh
```

Extra commands
```
#sudo yq -i -y '.kvmd.hid.jiggler.enabled = true' /etc/kvmd/override.yaml
#sudo yq -i -y '.kvmd.hid.jiggler.active = false' /etc/kvmd/override.yaml
#sudo yq -i -y '.kvmd.streamer.resolution.default = "1920x1080"' /etc/kvmd/override.yaml
#sudo yq -i -y '.vnc.server.host = "0.0.0.0"' /etc/kvmd/override.yaml

# to uninstall
groupdel kvmd-webterm
groupdel kvmd
userdel kvmd-pst
userdel kvmd-vnc
userdel kvmd-nginx
userdel kvmd-janus
userdel kvmd-ipmi
groupdel kvmd-certbot
#groupdel kvmd-pst
#groupdel kvmd-ipmi
#groupdel kvmd-vnc
#groupdel kvmd-nginx
#groupdel kvmd-janus
```
