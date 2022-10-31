# Making RetroArch work on Raspberry Pi 1 Model B

### Burn image and first boot

1. Burn the Raspberry Pi OS Lite **Buster** on an SD Card. Use Raspberry Pi Imager to turn on SSH and specify own username and password.
   If **Buster** isn't showing in the imager then old images could be found at [here](http://downloads.raspberrypi.org/raspbian_lite/images/).
2. Put in the SD card and let it expand the file system and reboot to login.

### Enable USB WiFi

1. Edit `sudo nano /etc/network/interfaces`

   ```
   auto wlan0
   allow-hotplug wlan0
   iface wlan0 inet dhcp
   wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
   iface default inet dhcp
   ```

2. Edit `sudo nano /etc/wpa_supplicant/wpa_supplicant.conf`

   ```
   network={
     ssid="YOUR_NETWORK_NAME"
     psk="YOUR_NETWORK_PASSWORD"
     key_mgmt=WPA-PSK
   }
   ```
   
3. Reboot

### Overclock

Launch `sudo raspi-config` and then select **Performance Options > Overclock > Turbo**

Or these are working settings to put on `/boot/config.txt`:

```
arm_freq=950
core_freq=250
sdram_freq=450
force_turbo=0
over_voltage=6
over_voltage_sdram=0
gpu_freq=250
```

### Increase Swap Size

```bash
sudo dphys-swapfile swapoff
sudo vi /etc/dphys-swapfile
> CONF_SWAPSIZE=1024
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
```

### Update System

```
sudo apt update
sudo apt upgrade
```

### Install Packages

```
sudo apt install build-essential libasound2-dev libudev-dev libusb-1.0-0-dev
```

### Build RetroArch Code

```
curl -LO 'https://github.com/libretro/RetroArch/archive/v1.12.0.tar.gz'
tar -zxvf v1.12.0.tar.gz
export CFLAGS="$CFLAGS -I$SYSROOT_PREFIX/usr/include/interface/vcos/pthreads -I$SYSROOT_PREFIX/usr/include/interface/vmcs_host/linux"
./configure --disable-vg --disable-al --disable-cg --disable-sdl --disable-sdl2 --disable-ssl --disable-x11 --enable-opengles --disable-kms --disable-x11 --enable-floathard --enable-zlib --enable-freetype --enable-translate --enable-cdrom --enable-hid --enable-libusb --disable-discord --disable-langextra
make
make install
retroarch
```

RetroArch should launch but it is without any cores.

### Updater

Navigate to Online Updater, then:
1. Update Autoconfig Profiles
2. Download Assets

### Place the Cores

Cores are individual system libraries that are also built as binaries. The official [link](http://buildbot.libretro.com/nightly/linux/armhf/latest) no longer has Armv6 binaries, which are required for RPi1. So we use prebuilt cores extracted from Lakka. Download them from this repo's [cores](https://github.com/nayaabkhan/rpi1b-retroarch/tree/main/cores). And move them to `~/.config/retroarch/cores`.

### Game!

The only way until now is to `scp` roms to the `~/.config/retroarch/downloads`. Then scan for the new roms and play!

## Appendix 1: Notes
- There is an efficient native API for video rendering on Raspberry Pi called DispmanX, but seems like RetroArch no longer supports it. So have to use OpenGL.

<img
   alt="Raspberry Pi Software Architecture"
   src="https://user-images.githubusercontent.com/234889/198924934-c2178bf6-4aa4-446b-896e-cf43cc55af51.png"
   width="50%"
/>
    
   - [https://elinux.org/Raspberry_Pi_VideoCore_APIs](https://elinux.org/Raspberry_Pi_VideoCore_APIs)
   - https://github.com/libretro/RetroArch/issues/2494
   - https://github.com/libretro/RetroArch/issues/500
- Neon (`--enable-neon`) is not supported by Raspberry Pi Model B. So it should not be enabled during `./configure`. See [https://bugs.launchpad.net/raspbian/+bug/1734592](https://bugs.launchpad.net/raspbian/+bug/1734592).

## Appendix 2: Resources
- https://gist.github.com/AlexMax/32e5d038a66ce57253e740ea75736805
- https://www.reddit.com/r/RetroArch/comments/l158qt/best_performing_retroarch_build_on_a_raspberry_pi/
- https://github.com/libretro/Lakka-LibreELEC/blob/Lakka-v3.x/packages/libretro/retroarch/package.mk
- https://github.com/libretro/Lakka-LibreELEC/blob/Lakka-v5.x/packages/lakka/retroarch_base/retroarch/package.mk
