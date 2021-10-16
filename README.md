# librespot-rpi

Stream Spotify to speakers for ~50 EUR. Spotify Premium required.

## Hardware

- The cheapest/most efficient board with a 3.5mm audio jack: [Rasberry Pi 3 Model A+](https://www.raspberrypi.org/products/raspberry-pi-3-model-a-plus/).
- The best looking Pi case: [https://www.raspberrypi.org/products/raspberry-pi-a-case/](https://www.raspberrypi.org/products/raspberry-pi-a-case/https://www.raspberrypi.org/products/raspberry-pi-a-case/).
- MicroSD card, preferably 8GB+ but 4 might be enough.
- 5V/2.5A DC power input.

## Initial Setup

1. Flash [DietPi](https://dietpi.com/) on microSD, insert card into Pi, connect keyboard and screen to Pi, logon as root:dietpi.  
  Would've loved to use PiCore since it lives entirely in RAM but it's a bit too core to get everything to work. PiCorePlayer with a Spotify connect plugin would be a viable alternative to DietPi with librespot, maybe even better (faster boot times and no chance of MicroSD card corruption).
2. Configure wifi, enable audio and let DietPi install. After this you can disconnect screen/keyboard and connect using SSH.
2. Install dependencies. tmux can be omitted if you don't know what that is.

        sudo apt-get install libasound2-dev pkg-config tmux build-essential
3. Install [Rust](https://www.rust-lang.org/).

        curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
4. Update PATH.

        source $HOME/.cargo/env
5. Install [librespot](https://github.com/librespot-org/librespot).

        cargo install librespot  
    
    This took so long on the A+ model I ended up doing this on a beefier rpi model and switching the MicroSD card back to the A+ model after installation. I'm not even sure this will work at all on the A+ since it only has 512MB RAM and rust compiling is notoriously expensive at the moment. I also tried cross-compiling for Pi from my laptop but to no avail.
6. Should work now, test it out:

       librespot -n "Librespot Speaker" -b 320

## Run librespot on startup

1. Make file `/root/startup.sh`.
          
          #!/bin/sh
          ./root/.cargo/bin/librespot -n "Librespot Speaker" -b 320 \
          --disable-audio-cache --initial-volume 100 --enable-volume-normalisation
      
      We have to use the full path `/root/.cargo/bin/librespot` to the librespot binary here because it is going to be executed by our service ([read why](https://unix.stackexchange.com/questions/471359/systemd-custom-service-doesnt-read-path)). `--disable-audio-cache` to lower chances of MicroSD card corruption in case of switching power strip off. `--initial-volume 100` because Pi audio output is pretty weak to begin with. `--enable-volume-normalisation` sounds like it should be enabled.
2. Make it executable

        chmod 755 /root/startup.sh
3. Make service file `/etc/systemd/system/librespot.service`:

        [unit]
        Description=Spotify Connect client (librespot)
        After=network.target

        [Service]
        Type=simple
        RestartSec=1
        User=root
        Restart=always
        ExecStart=/root/startup.sh

        [Install]
        WantedBy=multi-user.target
4. Start and enable service to run on startup:

        systemctl start librespot
        systemctl enable librespot
        
     Use following commands for troubleshooting: `systemctl stop librespot` `systemctl status librespot` `journalctl -u librespot.service -b`
     
## (Optional) Changing output to mono

`sudo nano /etc/asound.conf`

```conf
pcm.card0 {
  type hw
  card 0
}

ctl.card0 {
  type hw
  card 0
}

pcm.monocard {
  slave.pcm card0
  slave.channels 2
  type route
  ttable {
    0.0 0.5
    1.0 0.5
    0.1 0.5
    1.1 0.5
  }
}

ctl.monocard {
  type hw
  card 0
}

pcm.!default monocard
ctl.!default monocard
```

[source](https://support.hifiberry.com/hc/en-us/community/posts/360013981717-HOWTO-Send-mono-to-both-speaker-)

## Extras

1. [Backup SD card](https://raspberrytips.com/backup-raspberry-pi/#Create_an_image_of_the_SD_card) in case something gets corrupted. Replace `<Disk Node Name>` with disk node name.

    Linux:
     
        sudo dd bs=4M if=/dev/<Disk Node Name> of=backup.img
    
    Mac:
    
        sudo dd bs=4m if=/dev/<Disk Node Name> of=backup.img
        
    Windows: win32diskimager
2. Enable audio noise reduction in `dietpi-config` but careful. It's best to do this after you made a backup because this disables the HDMI output => if anything goes wrong with wifi/ethernet/ssh you won't be able to interact with the rpi.
3. Change root and dietpi password (you never know someone with malicious intentions enters the network).

## Tips

- You can control the volume using
  `alsamixer`
