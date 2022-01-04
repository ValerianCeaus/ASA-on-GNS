# Configuring Cisco ASA 8.4(2) on GNS3 1.3.13

Steps required to configure and run a Cisco ASA 8.4(2) image on GN3 1.3.13.

For ASA 8.4(2) to run successful, a QEMU emulator of version 0.11.0 needs to be in place and functional or, as it seems, the QEMU 0.11.0 refuse to work in GNS3 1.4.x suite (even if it is installed here). Somewhere in forums have found that GNS team will no longer offer support for QEMU 0.11.0 in their product. Maybe in one of the future releases but certainly not now …

Why not just use the 2.4.0 version of QEMU present and functional in both GNS3 versions suites ? The truth is that it works, but with a little issue, a very slow speed through device interfaces. In my testings, a simple ASDM image copying can take several hours with a high chance for device crashing. Types and number of NICs, license or whatever else has no effect.

So, because of unavailability of QEMU 0.11.0 in GNS3 1.4.x I switched back to 1.3.x version suite. Why then, I didn’t use the 1.3.x suite in [my previous article for an ASAv instance configuration](/ASAv-on-GNS.md) ? It is because we needed the VNC console for ASAv initial configuration or that is present only in newer 1.4.x suite.

To conclude: for ASAv you will need GNS3 1.4.x (because of VNC console) and respectively for ASA 8.4(2) you will need the GNS3 1.3.11 (because of QEMU 0.11.0). At least in my testings, this offer me a stable and workable setup.

> Do not mix the GNS3 1.3.13 and GNS3 1.4.x on the same machine, simply because they wasn’t designed to work together, configuration that most probably lead to complete non-functional setup.

##### Why bothering also with ASA in addition to ASAv ? Doesn’t ASAv being sufficient ?

Well, the ASAv is a software designed to run in virtual infrastructure and such that, some features are no longer needed here. Take simple, what ASA doing by clustering, in virtual infrastructure with ASAv is accomplished by hypervisor’s High Availability features, multiple contexts are replaced by multiple standalone ASAv instances and so on. To be more precise, that’s the unsupported features that that the [official documentation](http://www.cisco.com/c/en/us/td/docs/security/asa/asa92/asav/quick-start/asav-quick/intro-asav.html) states: *The ASAv does not support the following features: clustering, multiple context mode, active/active failover, EtherChannels, Shared AnyConnect Premium Licenses*.

So in case you want to play with ASA clustering or multiple context mode you will need a classic ASA instance running in GNS3. ASAv is good but not always sufficient. Even so, ASAv should be your standard, especially since it is somewhat closer to VIRL style.

##### What images are needed to run ASA in GNS3 ?

Compared to ASAv where an original qcow (KVM) image was sufficient to configure the device in GNS3, for ASA the original bin image are not sufficient. The truth is that this original image needs to be unpacked, then some files modifications needs to be done and after that repack the content in a way suitable for QEMU. All of this can be done manually or scripted (see GNS3 forums for *Cisco Image Unpacker for Windows* or *repack.v4.sh* shell script for Linux) but imho, if you not search for a specific ASA version just do a search on the Internet for ASA 8.4(2) GNS3 files. Other versions should be also be available to. In any case, you should be left with two files: *asa-vmlinuz* (a Linux kernel) and *asa-initrd.gz* (a RAM disk file). These are the files used by QEMU in GNS3.

##### How to configure Cisco ASA 8.4(2) in GNS3 1.3.13 ?

1. Download the latest version for 1.3.x GNS3 version suite. To do that, go to GNS3 web site – avoid the download button witch will direct you only to GNS3 1.4.x download, instead, click on software  (top bar menu) – on the left, written with small font size, click on To download Version 1.3.13 of GNS3 Click Here (see screenshot below). After, authentication a download for GNS3-1.3.13-all-in-one.exe file should start.

![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20download%20page.png)

2. Install the 1.3.13 GNS3. No rocket science here, just follow the installation wizard. Setup will install one by one all the components needed: Dynamips, QEMU, GNS3, WinPcap, Wireshark and many others parts.

3. Initial GNS3 configuration. At this step, I usualy configure some general, non-essential, GNS3 preferences:
   1. *General – Local paths – projects/binary images* – redirected to a shorted path, e.g. *C:\GNS3\projects* and respectively *C:\GNS3\images*
   2. In *Topology View* change the *default label text style* to a less accentuated one.
   3. In *Miscellaneous* – disable the *Automatically check for update* and *Automatically send crash reports* options.
4. Create a new Cisco ASA device by starting *New QEMU VM template* from *Edit – Preferences – QEMU VMs – New* menu. Use the following parameters:
   1. Type: ASA 8.4(2)
   2. Name: ASA5520-8.4(2) or any meaningful title you choose
   3. Qemu binary: leave the default qemu.exe (v0.11.0)
   4. RAM: 1024MB or more, 1024 is the minimum
   5. Initial RAM disk (initrd): select RAM disk file, e.g. asa8420-initrd.gz
   6. Kernel image (vmlinuz): select Kernel image file, e.g. asa842-vmlinuz

I will recommend to store original OS images in other folder than that used by GNS3 for image storage. When you specify an image to be used by GNS3 a copy of that original file would be automatically copied to GNS3 binary image folder location.

After device creation, edit VM configuration and add up to 6 network adapters (network section). Leave the rest parameters untouched.

Device summary settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20ASA%20device%20summary%20settings.png)

Device general settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20ASA%20device%20general%20settings.png)

Device HDD:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20ASA%20device%20HDD.png)

Device network settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20ASA%20device%20network.png)

Device advanced settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASA%208.42%20on%20GNS3%201.3.13%20-%20ASA%20device%20advanced%20settings.png)

>The steps above was successfully tested inside a Virtual Machine with Windows Server 2012R2 as a guest OS. The VM was provisioned with 2x vCPU and 8GB RAM and run on an ESXi host. Physical server equipped with Intel Xeon E5320 1.86GHz CPU.

>There is no need for: *Expose hardware assisted virtualization to the guest OS …* No VT-x needed inside the Virtual Machine.

##### Use the newly created ASA device template

Now, you can use the newly created ASA device template, just drag the device to the map pane, build the topology and start it. Open console, if everything is ok you should see a typical ASA OS loading progress. After a minute, you will be faced with long awaited Cisco ASA CLI (use empty password for privileged exec mode).

You can use as many Cisco ASA instances in your projects as you want, sure no more than your hardware permit. Every time you instantiate a new Cisco ASA device, a flash disk device is created and mounted automatically for each Cisco ASA instance thus no additional steps for virtual disk creation needed. You can find the qcow virtual disk file (flash.qcow2) associated to your ASA device in *project folder/project-files/qemu/dev-uiid/* folder.

Usualy, the first step before using ASA is to put a license key. If you do a simple google search you will find several *freely flying* license keys. Just use one of them. For me, it turned out to be ok the following activation key: *activation-key 0x7212d04a 0xe041d3fe 0x1d22f820 0xea5440e4 0x8231dd9f* … which unlike other keys does not hung loading progress for several minutes after activation.
