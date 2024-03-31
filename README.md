# Raspberry Pi Hifiberry Owntone install instructions

## Prepare microSD card
I used the Raspberry Pi imager to write the OS for Raspberry Pi 4.
I used:
2024-03-15-raspios-bookworm-arm64-lite.img.xz
which I downloaded first before writing.

Raspberry Pi OS Lite 64 Bit from https://www.raspberrypi.com/software/operating-systems/

Using the imager, I prepopulated password, wifi and SSH so I could use it immediately.

## Optional: SSH key
I followed the instructions to place an SSH key on the Pi to simplify logging in via Putty.

https://pimylifeup.com/raspberry-pi-ssh-keys/

```
install -d -m 700 ~/.ssh
nano ~/.ssh/authorized_keys
sudo chmod 644 ~/.ssh/authorized_keys
sudo chown pi:pi ~/.ssh/authorized_keys
```

## Update Debian

```
sudo apt update -y && sudo apt full-upgrade
```

## Install owntone
Even though the post is dated 2013, it is updated with the latest instructions:
Source: https://forums.raspberrypi.com/viewtopic.php?t=49928

```
wget -q -O - http://www.gyfgafguf.dk/raspbian/owntone.gpg | sudo gpg --dearmor --output /usr/share/keyrings/owntone-archive-keyring.gpg
sudo wget -q -O /etc/apt/sources.list.d/owntone.list http://www.gyfgafguf.dk/raspbian/owntone-bookworm.list
sudo apt update
sudo apt install owntone
```

## Install Hifiberry driver
Using the instructions at the source: https://www.hifiberry.com/docs/software/configuring-linux-3-18-x/
I confirmed the kernel using

```
uname -r
```

I had Raspberry Pi 4 with Kernel 6.6.20 and HifiBerry Dac Plus so I did the following:

```
sudo nano /boot/firmware/config.txt
```

Comment out the line
```
dtparam=audio=on
```
Add the ,noaudio to the line dtoverlay=vc4-kms-v3d,noaudio

Their instruction about kernel version and hardware didn’t work exactly, but this worked
```
dtoverlay=hifiberry-dacplus
```
(I put it as the last line of the /boot/firmware/config.txt file)
I rebooted
```
aplay -l
```
confirmed:
```
**** List of PLAYBACK Hardware Devices ****
card 0: sndrpihifiberry [snd_rpi_hifiberry_dacplus], device 0: HiFiBerry DAC+ Pro HiFi pcm512x-hifi-0 [HiFiBerry DAC+ Pro HiFi pcm512x-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

Then to test:
```
speaker-test -t sine -f 1000
```
produced a tone.

## owntone configuration
```
sudo nano /etc/owntone.conf
```
I changed the library location

Also, without making an explicit ALSA entry for the local soundcard I got this error:

Request failed (status: 500 Internal Server Error http://\<ip\>:3689/api/outputs/0)

The ALSA setting for the Hifiberry card looks like:
```
audio {
	nickname = "Owntone on Pi"
	type = "alsa"
	card = "hw:0"
	mixer = "Digital"
	mixer_device = "hw:0"
	offset_ms = 200
}
```
I also added my Bose Soundtouch 10 and Airport Express to use Airplay 2 rather than default Airplay 1
```
airplay "SoundTouch 10" {
	raop_disable = true
}

airplay "Mark's AirPort Express" {
	raop_disable = true
}
```

After making a config change, rather than rebooting you can issue:

```
sudo service owntone restart
```

## Mount the USB
This may not be needed if you use a full desktop OS and boot into it. But with Lite, this needs to be done.

```
sudo blkid /dev/sda1
```

To find the PARTUUID and Type (mine was 54edbd60-01 and ext4)

```
sudo mkdir -p /media/pi/Music-SanDisk
sudo chown -R pi:pi /media/pi/Music-SanDisk
sudo nano /etc/fstab
```

Added the line:

```
PARTUUID=54edbd60-01 /media/pi/Music-SanDisk ext4 defaults,auto,users,rw,nofail,noatime 0 0
```

Then saved and issued commands:
```
systemctl daemon-reload
sudo mount -a
```

## Use
I used Remote on IOS. I can choose any combination of speakers (local or the Airplay speakers using Airplay 2). I expect that I didn’t need to force Airplay 2 as I think it will stream multi-room using even Airplay 1.

