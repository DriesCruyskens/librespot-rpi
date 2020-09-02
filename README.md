# librespot-rpi

## Hardware

- The cheapest/most efficient board with a 3.5mm audio jack: [Rasberry Pi 3 Model A+](https://www.raspberrypi.org/products/raspberry-pi-3-model-a-plus/)
- The best looking Pi case ever: [https://www.raspberrypi.org/products/raspberry-pi-a-case/](https://www.raspberrypi.org/products/raspberry-pi-a-case/https://www.raspberrypi.org/products/raspberry-pi-a-case/)
- MicroSD card, preferably 8GB+ but 4 might be enough
- 5V/2.5A DC power input

## Initial Setup

1. Flash [DietPi](https://dietpi.com/) on microSD, insert card into Pi, connect keyboard and screen to Pi, logon as root:dietpi.  
  Would've loved to use PiCore since it lives entirely in RAM but it's a bit too core to get everything to work. PiCorePlayer with a Spotify connect plugin would be a viable alternative to DietPi with librespot, maybe even better (faster boot times and no chance of MicroSD card corruption).
2. Configure wifi, enable audio and let DietPi install.
2. Install dependencies

        sudo apt-get install libasound2-dev pkg-config tmux build-essential
3. Install Rust

        curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
4. Update PATH

        source $HOME/.cargo/env
5. Install librespot.

        cargo install librespot  
    <!-- -->
    This took so long on the A+ model I ended up doing this on a beefier rpi model and switching the MicroSD card back to the A+ model after installation. I'm not even sure this will work at all on the A+ since it only has 512MB RAM and rust compiling is notoriously expensive at the moment. I also tried cross-compiling for Pi from my laptop but to no avail.
6. Should work now, test it out:

       librespot -n "Librespot Speaker" -b 320

## Run librespot on startup

1. Make file `/root/startup.sh`.
          
          #!/bin/sh
          ./root/.cargo/bin/librespot -n "Librespot Speaker" -b 320 \
          --disable-audio-cache --initial-volume 100 --enable-volume-normalisation
      <!-- -->
      We have to use the full path `/root/.cargo/bin/librespot` to the librespot binary here because it is going to be executed by our service. [read why](https://unix.stackexchange.com/questions/471359/systemd-custom-service-doesnt-read-path). `--disable-audio-cache` to prevent MicroSD card corruption in case of switching power strip off. `--initial-volume 100` because Pi audio output is pretty weak to begin with. `--enable-volume-normalisation` sounds like it should be enabled.
2. Make it executable

        chmod 755 /root/startup.sh
3. Make file `/etc/systemd/system/librespot.service`:

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
4. Enable to service to run on startup:

        systemstcl enable librespot
        
## Extras

1. [Backup SD card](https://raspberrytips.com/backup-raspberry-pi/#Create_an_image_of_the_SD_card) in case something gets corrupted.
2. Enable Audio noise reduction in `dietpi-config` but careful. Best to do this after you made a backup because this disables the HDMI output => if anything goes wrong with wifi/ethernet/ssh you won't be able to interact with the rpi.
3. Change root and dietpi password (you never know someone with malicious intentions enters the network).

## Tips

- You can control the volume using  
  `alsamixer`
