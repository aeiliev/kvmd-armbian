Use image https://armbian.systemonachip.net/archive/orangepi3-lts/archive/Armbian_23.8.3_Orangepi3-lts_bookworm_current_6.1.53_minimal.img.xz

1. Install basic tools
```
sudo apt update
sudo apt -y install mc htop git vim make pkg-config python3-dev gcc xz-utils yq bash-completion armbian-config
```
2. Fix the USB OTG, this one works only for Debian Bookworm with kernel 6.1.53. Keep in mind that it is better to cut off the VCC/power cord crom the usb cable
```
dtc -I dtb -O dts -o sun50i-h6-orangepi-3-lts.dts /boot/dtb/allwinner/sun50i-h6-orangepi-3-lts.dtb
patch sun50i-h6-orangepi-3-lts.dts << 'EOF'
--- sun50i-h6-orangepi-3-lts.dts	2024-09-04 16:24:55.976208151 +0300
+++ sun50i-h6-orangepi-3-lts-otgfix.dts	2024-09-04 16:26:17.845063971 +0300
@@ -914,7 +914,7 @@
 			phy-names = "usb";
 			extcon = <0x3a 0x00>;
 			status = "okay";
-			dr_mode = "host";
+			dr_mode = "otg";
 			phandle = <0x7e>;
 		};
 
EOF

dtc -I dts -O dtb sun50i-h6-orangepi-3-lts.dts -o sun50i-h6-orangepi-3-lts.dtb
sudo cp sun50i-h6-orangepi-3-lts.dtb /boot/dtb/allwinner/sun50i-h6-orangepi-3-lts.dtb
echo "g_serial" | sudo tee -a /etc/modules
```
3. Freeze kernel updates
```
sudo apt-mark hold linux-image-current-sunxi64
sudo apt-mark hold linux-dtb-current-sunxi64
sudo apt-mark hold linux-u-boot-orangepi3-lts-current
sudo apt-mark hold armbian-firmware
```
4. Install updates and reboot
```
sudo apt -y upgrade
sudo reboot
```
5. Checkout kvmd-armbian
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
6. After reboot, continiue
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





