# TurboGrafx-16 Setup

## Prerequisites

1. Ensure Python 2.7 is working correctly and installed in `/usr/bin/python2.7`
2. Run `sudo apt-get install python-dev python-pip`

## Enable i2c device
1. Run `sudo mkdir /etc/nfc`
2. Run `sudo raspi-config`
3. Select "5 Interfacing"
4. Select "P5 I2C"
5. Select "\<Yes\>"
6. Exit `raspi-config` program
7. `sudo shutdown -r now` to reboot with i2c enabled.

## Install libnfc
1. Ensure your NFC reader is wiring correctly [See](https://github.com/devgoon/TurboGrafx-16-nfc/blob/master/NFC-RASPBERRY-PI.png) and you are logged in as `pi` and in the `/home/pi` directory.
2. Run `wget -O libnfc-1.7.1.tar.bz2 https://bintray.com/nfc-tools/sources/download_file?file_path=libnfc-1.7.1.tar.bz2`
3. Run `tar -xvf libnfc-1.7.1.tar.bz2`
4. Run `cd libnfc-1.7.1`
5. Run `./configure --prefix=/usr --sysconfdir=/etc --with-drivers=pn532_i2c`
6. Run `make`
7. Run `sudo make install`
8. Run `sudo reboot`
8. After reboot, type `lsmod |grep i2c` and ensure that you see an `i2c_dev` in the list.
9. Also, type `ls /dev/i2c*` and ensure that `/dev/i2c-1` is returned.

## Configure libnfc
1. Run `sudo nano /etc/nfc/libnfc.conf`
2. Cut and paste the following and save the file.
```
# Allow device auto-detection (default: true)
# Note: if this auto-detection is disabled, user has to manually set a device
# configuration using file or environment variable
allow_autoscan = true

# Allow intrusive auto-detection (default: false)
# Warning: intrusive auto-detection can seriously disturb other devices
# This option is not recommended, so user should prefer to add manually his/her device.
allow_intrusive_scan = false

# Set log level (default: error)
# Valid log levels are (in order of verbosity): 0 (none), 1 (error), 2 (info), 3 (debug)
# Note: if you compiled with --enable-debug option, the default log level is "debug"
log_level = 1

# Manually set default device (no default)
# To set a default device, users must set both name and connstring for their device
# Note: if autoscan is enabled, default device will be the first device available in device list.
device.name = "PN532"
device.connstring = "pn532_i2c:/dev/i2c-1"
```
3. Run `nfc-poll` and ensure you see `NFC reader: pn532_i2c:/dev/i2c-1 opened`
4. Try reading a tag, use Ctrl-C to stop or just wait 30 seconds

## Install nfc_poll
1. Ensure you are logged in as `pi` and in the `/home/pi` directory.
2. Run `git clone https://github.com/devgoon/TurboGrafx-16-nfc`
2. Run `export NFC_HOME=/home/pi/libnfc-1.7.1`
3. Run `cd TurboGrafx-16-nfc/nfc`
4. Run `make`
5. Run `sudo make install`

## Install NFC Poll Service Manually
1. Run `sudo nano /lib/systemd/system/nfc_poll.service`
2. Add the following
```
[Unit]
 Description=NFC Poll
 After=multi-user.target

[Service]
 Type=idle
 ExecStart=/usr/bin/python /var/lib/nfc_poll/nfc_poll_daemon.py

[Install]
 WantedBy=multi-user.target
```
3. Run `sudo chmod 644 /lib/systemd/system/nfc_poll.service`
4. Run `sudo systemctl enable nfc_poll.service`
5. Run `sudo reboot`
6. Run `systemctl status nfc_poll` and ensure you see "Active: active (running)" in the output

## Configure your HuCards

In this system, the HuCards don't need to be written to, we configure a mapping to their UIDs, which should already be uniquely pre-programmed to each tag.

### Record your NFC tag UIDs
1. Run `tail -f /dev/shm/nfc_poll.log`
2. Place a HuCard (NFC tag) over the reader, the log should output "Reading NFC UID: 00000000000000" where the zeroes are the UID of the tag.
3. Copy/paste the UID into a text file.
4. Continue with all tags you wish to use.
5. To exit the log, hit \<Ctrl\>-C

### Record your desired games
1. `cd ~/RetroPie/roms`
2. Start typing `ls <system>/<name of game>` (e.g. `ls /pcengine/Bonk*`) to find the rom you want. Hit tab to complete the file, or hit tab twice to show possible matches. Don't forget to backslash things like spaces and parenthesis as you go.
3. After you finally tab through to the complete file, hit <enter>.
4. Copy the resulting line and put it and put it next to the UID you want to use in your text file. (`e.g. pcengine/Bonk's Adventure (USA).pce`) Make sure you don't have backslashes here.
5. Repeat this process.

### Write your HuCards in the config
1. Run `sudo nano /etc/nfc_poll/nfc_poll.conf`
2. At the bottom of the file, there's a `[hucards]` section.
3. For each HuCard you want, add a line in this format: `<uid> = <game file>` (e.g. `00000000000000 = pcengine/Bonk's Adventure (USA).pce`).  [Like this](https://github.com/devgoon/TurboGrafx-16-nfc/blob/master/nfc/etc/nfc_poll.conf)

 
Run `sudo systemctl restart nfc_poll`
5. Try it out! Place one of your HuCards on the reader and the screen should go black for a couple seconds, then bring up your game. Remove it and it should go back to the EmulationStation.

## Install screen_manager
1. Ensure you are logged in as `pi` and in the `/home/pi` directory.
2. Run `cd TurboGrafx-16-nfc/screen`
3. Run `sudo make install`
4. Restart your Pi `sudo reboot` and if you've done all the steps above, everything should be working!

