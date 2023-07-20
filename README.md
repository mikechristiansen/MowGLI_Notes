# MowGLI_Notes

# Prepare Raspberry Pi 4

- Fresh image - Raspberry Pi Imager
- 64bit lite image
- Advanced options - configure: Hostname, WIFI network, Username/Password, and SSH
- First boot takes a while. 

sudo apt-get update
sudo apt-get upgrade — yes


# Docker components 
https://github.com/cedbossneo/mowgli-docker

curl https://get.docker.com | sh

sudo usermod -aG docker $USER
- Log out and in again to make the above group permissions active

sudo nano /etc/udev/rules.d/50-mowgli.rules
- Add below serial device mapping
SUBSYSTEM=="tty" ATTRS{product}=="Mowgli", SYMLINK+="mowgli"
SUBSYSTEM=="tty" ATTRS{idVendor}=="1546" ATTRS{idProduct}=="01a9", SYMLINK+="gps"
KERNEL=="ttyACM[0-9]*",MODE="0777"

cd ~
git clone https://github.com/cedbossneo/mowgli-docker
cd mowgli-docker

nano .env
- Change IP's to loopback 127.1.1.1

docker compose up -d

SPI OpenOCD
https://www.pcbway.com/blog/technology/OpenOCD_on_Raspberry_Pi__Better_with_SWD_on_SPI.html

- Enable the SPI interface on Raspberry Pi…
sudo raspi-config
- Select Interfacing Options → SPI → Yes
sudo apt-get install --yes libtool libusb-1.0-0-dev libhidapi-dev
cd ~
git clone https://github.com/lupyuen/openocd-spi
cd openocd-spi
./bootstrap
./configure --enable-sysfsgpio --enable-bcm2835spi
make
- results in errors so do below to fix - https://github.com/lupyuen/openocd-spi/issues/6#issuecomment-1207398001
git remote add zorvalt https://github.com/Zorvalt/openocd-spi/
git pull zorvalt fix-multiple-gcc-10-errors
make

The modified OpenOCD executable is now at openocd-spi/src/openocd
Connect up the Raspberry Pi to the SWD interface on the openmower mainboard. Here is a picture of the pins on the raspberry pi: https://www.pcbway.com/blog/technology/OpenOCD_on_Raspberry_Pi__Better_with_SWD_on_SPI.html. On the SWD interface on the YF500 mainboard the markings are slightly different: SWCL=SWDCLK, SWDA=SWDIO. Here is the picture of where the SWD interface is on the mower: https://github.com/cedbossneo/Mowgli/blob/main/images/mainboard_swd.jpg

# Backup the mainboard firmware
cd ~/openocd-spi/tcl
curl -LO https://raw.githubusercontent.com/cedbossneo/Mowgli/main/stm32/mainboard_firmware/backup_firmware.sh
curl -LO https://raw.githubusercontent.com/cedbossneo/Mowgli/main/stm32/mainboard_firmware/yardforce500.cfg
- Edit yardforce500.cfg. Comment out the line with the stlink.cfg interface. Instead add a line just below it with `interface bcm2835spi`. Also comment out #transport select hla_swd`. Save the file.
- Ensure openocd isn't installed. `which openocd` should return nothing.
- Add openocd to the path
export PATH=$PATH:~/openocd-spi/src
bash backup_firmware.sh
- The firmware is now saved in ~/openocd-spi/tcl/firmware_backup.bin. Now check that the SHA256sum matches one from the list in https://github.com/cedbossneo/Mowgli/tree/main/stm32/mainboard_firmware
sha256sum ~/openocd-spi/tcl/firmware_backup.bin
- If not does not match, DO NOT proceed further, as steps to flash with the wrong firmware will brick the mower. Instead, reach out on discord.

# Now backup the panel firmware
- Note you don't need a soldering iron or custom headers for this, if you just have some breadboard jumper wires with pins at the end (male). Let the panel in the top chassis stay mounted, and just put the pins into the holes of the SWD interface in the PCB. Hook up to the Raspberry PI as before. The markings now correspond as follows: CLK=SWDCLK, DIO=SWDIO.
cd ~/openocd-spi/tcl
curl -LO https://raw.githubusercontent.com/cedbossneo/Mowgli/main/stm32/panel_firmware/backup_firmware.sh
curl -LO https://raw.githubusercontent.com/cedbossneo/Mowgli/main/stm32/panel_firmware/yardforce500.cfg
- Edit yardforce500.cfg. Comment out the line with the stlink.cfg interface. Instead add a line just below it with `interface bcm2835spi`. Also comment out `#transport select hla_swd`. Save the file.
- Edit backup_firmware.sh. Add the command line parameters -c "init" -c "reset halt"  before the dump_image command.
export PATH=$PATH:~/openocd-spi/src
bash backup_firmware.sh
- The firmware is now saved in ~/openocd-spi/tcl/panel_controller_backup.bin
- Move from the Pi onto the Mac by this command on the mac.
scp  user@192.168.80.106:/home/user/openocd-spi/tcl/panel_controller_backup.bin panel_controller_backup.bin


# Compile firmware

Done on the Mac in VS code using the platform IO plugin.
Firmware.bin created in hidden file.
Moved from the Mac onto pi by this command on the mac in this hidden file location.
scp firmware.bin user@192.168.80.106:/home/user/openocd-spi/tcl/



# Useful Commands

- View logs: docker logs --follow mowgli-docker-openmower-1
- Recreate containers: docker compose up -d --force-recreate
- Get terminal in the openmower container: docker compose exec openmower bash
- Change WIFI to 2.4g only.Ive done this to ensure better coverage on my lawn, I was having dropouts with band steering to 5ghz on a unifi network.
  
"sudo nano /etc/wpa_supplicant/wpa_supplicant.conf"

freq_list=2412 2417 2422 2427 2432 2437 2442 2447 2452 2457 2462 2467 2472



