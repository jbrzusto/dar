## dar
A use-at-your-own-risk, end-of-life radar digitizer plus capture scripts for red pitaya and a linux box.

The instructions here supersede the ones on [the radR project
website](https://radr-project.org/03_-_Current_Features_and_Plug-ins/Red_Pitaya_-_preliminary_work?highlight=redpitaya)
In particular, don't bother with the RaspberryPi image there.

Since I haven't had time to finish the [rewrite in
go](https://github.com/jbrzusto/ogdar) or clean up [the FPGA
build](https://github.com/jbrzusto/digdar), this will have to do.

### Digitizer Software
How to set it up:

- get a [redpitaya STEMlab 125-14](https://www.redpitaya.com/Catalog/p20/stemlab-125-14-starter-kit?cat=c99)

- download the SD card image [red_pitaya_radar_digitizer_image.zip] (https://github.com/jbrzusto/dar/red_pitaya_radar_digitizer_image.zip

- unzip the SD card image onto a blank 4 GB micro SD card, such as the one which comes with the
  redpitaya STEMlab.  Make sure to use a fresh good quality card.

- after a successful unzip, the top level directory of the micro SD card should have these files

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

### Redpitaya Software Customization

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


### Hooking up the radar to the redpitaya (aka VID, TRG, ACP, ARP)

This requires a front-end circuit to:


-   use high-speed (125 MS/s) 14-bit ADC-A for video;
    set jumper for +/- 1 Volt input
    use resistor network for impedance matching

-   use high-speed 14-bit ADC-B for trigger
    set jumper for +/- 20 Volt input

-   use low-speed (100 kS/s) 12-bit ADC for ARP / heading (Analog input 1 = E2:pin 14);
    use resistor network to adjust range, limit current draw.  Input range should be -1 to +1 V

-   use low speed 12-bit ADC for ACP / azimuth (Analog input 0 = E2:pin 13);
    use resistor network to adjust range, limit current draw.  Input range should be -1 to +1 V

### Calibrating the digitizer for the radar

This requires having the RP hooked up to the radar.  You connect your laptop to the
RP via ethernet, and send your web browser to

`http://redpitaya.local` or `http://10.42.0.56` (or something else, if you've
reconfigured the RP's network)

On that screen, choose the "Digdar Marine Radar Scope" application.
You'll see this (except that the network address is different than what you'll use):

![initial digdar screen](https://github.com/jbrzusto/dar/blob/master/img/initial_screen.png)

To get rid of the noisy and unhelpful video signal, click on **Video** to deselect that signal,
and then click on **Autoscale** to rescale.  You'll then see the trigger, ACP and ARP signals like this:

![trig_acp_arp_no_video] (https://github.com/jbrzusto/dar/blob/master/img/trig_acp_arp_no_video.png)

To focus on calibrating one signal, deselct ARP and Trigger to see only ACP, then click on the
**Range** button to let you adjust the time and voltage ranges.  Increase the time to see
ACP pulses clearly.

![adjust_range] (https://github.com/jbrzusto/dar/blob/master/img/adjust_range.png)

To see a summary of how ACP, ARP and trigger are being captured, click on the **Measure** button
to see a window like this:

![show_signal_measurements] (https://github.com/jbrzusto/dar/blob/master/img/show_signal_measurements.png)

The measurement for **Video** is not usually helpful, but under **Trigger**, the *Rate* should be
the radar's PRF.  Do not worry about *Capturing*; it is showing only the rate at which pulses are
being captured by the web browser.

Under **ACP / Azi**, you should see the appropriate number of ACP per sweep (4096 for Bridgemaster E,
450 for Furuno radars).

Under **ARP / Hdg**, the *Rate* should be the radar rotation rate, typically around 24-25 RPM.

If these signals are not being captured correctly, you need to adjust the trigger thresholds
for individual signals.  This is done one signal at a time, by selecting that signal:
  - set Capturing and Plot **When** to **At every pulse**
  - set **Pulses From** to Trigger, ACP, or ARP
  - set the **Range** to a reasonable value
  - adjust the **Detect Pulse At** (excitation level) and **Relax At** (relaxation level) up or down
    until you see an appropriate rate in the **Measure** window

To watch raw (untriggered) signal, you can choose *Continuously* in the **When** menu.
Then hit **Autoscale** to make the scale usable.  You should probably increase the **Range**
to be large as well.  It can go up to 5 seconds which is useful for checking the ARP pulse.

Once you are confident in your capture of these signals, you can save parameters to
on-board digitizer storage, or download to a local file, via **Load/Store Parameters**.
The factory defaults might or might not be useful for your radar.
