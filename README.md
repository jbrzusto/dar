## dar
A use-at-your-own-risk, end-of-life radar digitizer plus capture scripts for red pitaya and a linux box.

Since I haven't had time to finish the rewrite in go or clean up the FPGA build, this will have to do.

### Digitizer Software
How to set it up:

- get a [redpitaya STEMlab 125-14](https://www.redpitaya.com/Catalog/p20/stemlab-125-14-starter-kit?cat=c99)

- download the SD card image [red_pitaya_radar_digitizer_image.zip] (https://github.com/jbrzusto/dar/red_pitaya_radar_digitizer_image.zip

- unzip the SD card image onto a blank 4 GB micro SD card, such as the one which comes with the
  redpitaya STEMlab.  Make sure to use a fresh good quality card.

- after a succesful unzip, the top level directory of the micro SD card should have these files

```
bin
boot.bin
boot.bin.digdar
devicetree.dtb
etc
media
root
sbin
src
uImage
uramdisk.image.gz
version.txt
www
```

These must be at the **top** level of the micro SD card, not in a folder!!!

This SD card can now be inserted in the red pitaya, which can then be powered up.
The redpitaya will advertise itself on the local network as 'redpitaya.local',
and uses a static network configuraiton (in `etc/network/interfaces`):
```
iface eth0 inet static
    address 10.42.0.56
    netmask 255.255.255.0
    gateway 10.42.0.1
```

You can ssh into the redpitaya like so:
```
ssh root@10.42.0.56
## enter the password "root" when prompted
```
This image is a minimal linux.  Any changes you make to files **will not be preserved**
unless the files are on the sd card, which is mounted at `/opt`

### Redpitaya customization

Some configuration files on the SD card are copied over to the RAM image from
which linux on the redpitaya runs.  There is a startup script in `init/rcS` on
the SD card which is run at boot time.
Currently, it modifies the ssh server configuration and restarts that, then
copies the /etc/ hierarchy from the SD card onto the linux ram image.

If logged into the redpitaya, you can make changes to the SD card, but first
you must remount it read/write using this command:

`rw`

Then, after changes have been made to the `/opt` hierarchy, you should change it back
to readonly using:

`ro`


### Calibrating the digitizer for the radar

This requires having the RP hooked up to the radar.  You connect your laptop to the
RP via ethernet, and send your web browser to

`http://redpitaya.local` or `http://10.42.0.56` (or something else, if you've
reconfigured the RP's network)

On that screen, choose the "digdar" application.
