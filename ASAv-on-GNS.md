# Configuring Cisco ASAv 9.x on GNS3 1.4.x
Recently I went through an interesting experience of Cisco ASA setup in GNS3. I must say it was a real challenge, but finally, not an impossible task. There is a lot of particularities you must take into account, all depending from ASA version to GNS3 release. In this post, I will focus on how to configure an ASAv firewall to run as a QEMU VM in new GNS3 version suite 1.4.x. As of date of this writing I was able to access ASAv image version 9.5(2.204) and the GNS3 1.4.5 setup.

The 1.4.x suite of GNS3 is a relatively new appearing into the scene, and how is typical with new software releases, surely expected to have some bugs and/or incompatibilities. That’s why my first attempts was made in previous software suite of 1.3.x version (actually 1.3.13 version). It was a wrong way, because how I realized soon, for ASAv to be configured, a VNC console should be attached, or in 1.3.13 I didn’t found how to do that (is not excluded that I missed something). As a consequence, I quickly switched to newest GNS3 1.4.5 version. The following text assume a newly setup of GNS3 version 1.4.5 with no additional settings, except maybe only for the default location for project and binary images folder – I prefer to reconfigure these on *C:\GNS3\Projects* and respectively *C:\GNS3\Images*.

## Cisco ASA virtual appliance (ASAv)
Cisco ASAv is a re-imaged version of Cisco ASA specifically designed to run as a VM on top of some hypervisor. In fact, the same ASA code is running, but in different form factor. There are versions for vSphere, Hyper-V and KVM. Just because GNS3 use QEMU as a VM emulator we will employ the KVM image of ASAv. By the way, ASAv is the image Cisco use in their notable virtual labs VIRL. Not all ASA versions are available in a VM format - I suppose only those starting with 9.x, thereby if you want to try some older versions, e.g. popular ASA 8.4(2), you will need to experience another approach (a new article devoted to this subject should come). It’s worth noting that the ASAv have some limitations compared to classical ASA, in particular you wouldn’t be able to build firewall clusters (failover or A/A), test multiple context mode feature or play with Etherchannel. For this scenarios, I usualy use an 8.4(2) ASA setup – which, by the way should run only in QEMU 0.11.0 which in turn can’t be started in GNS 1.4.x only in previous suite 1.3.x !!!.

So, before we start we need to obtain somewhat the ASAv image. If you are fortunate enough to have access to Cisco downloads (a service contract associated with your profile is needed) then just go to *cisco.com - All downloads - Products – Security – Firewalls - Adaptive Security Appliances (ASA) - Adaptive Security Virtual Appliance (ASAv)* and download the qcow2 (KVM) image of ASAv for your preferred version.

![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20download%20page%20for%20Cisco%20ASAv.png)

In case you do not have access to official Cisco downloads, yet I recommend to try a simple Internet search, good chances are to find somewhere a leaked image (usualy on some China resources). To be honest, I can’t understand why Cisco restrict downloads to this type of software, anyway, next after setup you will need a license key to go over the limitations of unlicensed state of appliance (bandwith limitation to 100kbps). It would be fine if Cisco would allow download and free use of appliance in unlicensed state, respectively for production usage a suitable license should be bought.

## Configuring ASAv template on GNS3

A step by step guide follow:
##### Start new *QEMU VM Template wizard* with following parameters:
1. Type: Default
2. Name: ASAv-8.5(2.204) or any meaningful title
3. Qemu binary: qemu-system-x86_64w.exe (v2.4.0)
4. RAM: 2048 MB
5. Disk Image (hda): C:\GNS3\images\QEMU\asav952-204.qcow2

>I will recommend to store original OS images in other folder than that used by GNS3 for image storage. When you specify an image to be used by GNS3 a copy of that original file would be automatically copied to GNS3 binary image folder location.

#### Edit newly created QEMU Template:
1. General settings – Symbol :/symbols/asa.svg
2. General settings – Category: Security Devices
3. General settings – Console Type: VNC

>In my testing, I tried to change vCPUs from 1 to 4, but nothing more than *1514 Illegal Instruction (core dumped) ...* error message got in ASAv, hence don’t touch it, we will set the number of vCPUs in other place for ASAv to be an SMP virtual machine.

>Switching the console to VNC type one it’s like directly connect with a keyboard and a monitor to the virtual machine. Initial ASAv configuration don’t allow access to the serial console port so at least at this stage, the only possible option is VNC. Don’t forget, the ASAv was designed to play in a VM with a full console. Even so, we will configure serial console port to ASAv as well.

4. Network – Adapters: 6x (default e1000 type)
5. Advanced Settings – Additional settings – Options:

`-cpu Haswell -smp 4,sockets=4,cores=1,threads=1`

> I have successfully used this string for all my Intel CPU. The microarchitecture (Haswell, Nehalem and so on) seems to no matter – successfully ran on different CPU generation with no problems. For AMD CPUs, community recommend to use (haven’t tested):

`-cpu Opteron_G5 -smp 4,sockets=4,cores=1,threads=1`

> The default option’s value: –nographic, should be cleared. This will be guarantee an automatic VNC console opening.

6. (Optional) Activate CPU throttling – Percentage of CPU allowed: 80%
7. Advanced Settings – uncheck: Use as a linked base VM.

QEMU ASAv VM template settings summary:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20summary.png)

QEMU ASAv VM template general settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20general%20settings.png)

QEMU ASAv VM template network settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20hdd.png)

QEMU ASAv VM template general settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20network.png)

QEMU ASAv VM template advanced settings:<br />
![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20advanced%20settings.png)

I think I will provide some additional inputs about the setting named: Use as a linked base VM. By default, QEMU VMs works as a linked VM which means that every time you create a new QEMU VM (in our case ASAv) in your project a linked virtual disk is created to the original qcow2 image. All the modifications are thus recorded in that new file but yet unmodified block are read from original image. Through this, we can create hundreds of new QEMU VMs without needing to clone the virtual disk (that’s the similar to the technology used in VDI).Given the fact that during the life of an ASAv VM, disk modifications are really very few, result that the disk overhead created by each new ASAv are truly negligible. If you disable linked VM mode (uncheck the: Use as a linked base VM) the QEMU VM will interact directly with original qcow2 virtual disk (all writes will be recorded here). As a consequence a single QEMU VMs from this template can be started (just try to drag and drop a second ASAv to workspace and you will see an error message).

Why then we intentionally disabled linked base VM mode? First off, we need this only during ASAv template making and after this we will switch back to linked mode. Our interest is to do a series of configuration changes (first boot, serial console, ASDM image upload) in the original image file which we want to keep in all new ASAv instances created from this template.

Surely, the same results can be achieved by making template in linked mode (linked qcow2 virtual disk) and then committing all the changes to the original qcow2 image via qemu-img.exe tool, but, I think it is harder, just disabling and then re-enabling the VM’s linked mode settings seems to be much easier ..., the choice is yours.

To check the virtual disk that is mounted to QEMU VM just drag a new ASAv to an empty project, right click ASAv device – and choose *show in file manager*. An explorer window to qcow2 image opens – with linked mode disabled this would be the template image *asav952-204.qcow2* located in binary image folder, whereas for linked mode this would be a qcow2 image (somewhere in project’s folder) linked to the original template - base virtual disk image. Also, additionaly you can check what qcow2 images are involved via Windows resource monitor – CPU – Associated Handles – filter by QEMU string.

8. Drag a new instance of ASAv 9.5(2.204) to the working space on an empty project in GNS3. No topology are needed to continue, just single, unconnected ASAv device.

9. Power-ON newly instantiated ASAv device (right-click - start) and immediately open the console (right-click - console).  In opened VNC terminal and a loading progress (Linux) should pe observed.

10. On Boot Loader phase choose the option: bootflash:/asa952-204-smp-k8.bin with no configuration load (anyway no configuration yet exists).


![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20bootloader.png)

11. In the meantime, it would be interesting to do some analyzing in Resource Monitor. First, to confirm the SMP nature of started QEMU VM look at the number of threads/CPU associated with *qemu-system-x86_64w.exe* process (CPU - Processes) – should be more than 4x thread/CPU in use, and second, to confirm the non-Linked mode of operation for the ASAv VM do a search in Associated Handles for a qemu key (CPU-Associated Handles) – in non-Linked mode, the VM should interact directly with the original qcow2 image: asav952-204.qcow2 (a screen is inserted below).

![alt-text](/assets/images/screen%20resource%20monitor%20-%20qemu%20treads%20plus%20handler%20not%20linked%20mode%20-%202.png)

After device load, the number of vCPU can be checked by the `show cpu usage` commmad.

```
ciscoasa# sh cpu usage
CPU utilization for 5 seconds = 1%; 1 minute: 1%; 5 minutes: 0%

Virtual platform CPU resources
------------------------------
Number of vCPUs              :     4
Number of allowed vCPUs      :     0
vCPU Status                  :  Noncompliant: Over-provisioned
```

12. If you carefully track the booting progress you will see that the appliance will discover that it starts for the first time (*Initial bootup detected ...*) and for the system variables to be applied an automatic reboot will come.  So first time booting will end up with an automatic reboot. On the second boot, also choose the option *with no configuration load* in Bootloader Dialog. First and second time booting could take some time to progress so be patient and wait them to complete – sometimes it may seem that the appliance hung, try to wait several minutes before doing a forced powering off.

13. If everything goes smoothly, after the second boot, you should reach the traditional Cisco command line prompter (empty password for privileged mode). At this stage, we will enable the serial console for the appliance. By default, the ASAv works only with traditional VM console (monitor/keyboard directly connected to x86 hardware) and additional steps needed to enable console via serial ports. More about that you can read here [ASAv Quick Start Guide, 9.5](http://www.cisco.com/c/en/us/td/docs/security/asa/asa95/asav/quick-start/asav-quick/asav-vmware.html), section *Configure a Network Serial Console Port*.

For serial console to be on, a file named *use_ttyS0* should exist in root of disk0. It doesn’t matter the content, just to be present. The simplest mode to create such a file is to make a copy of an existing file – the documentation suggest to clone from *coredump.cfg* file, like shown below:

```
ciscoasa(config)# cd coredumpinfo
ciscoasa(config)# copy coredump.cfg disk0:/use_ttyS0
```

14. Theoretically, here, we can do also some additional configurations, one that we want to keep in all the ASAv instances derived from this template. For example, we can copy here the ASDM image to disk0 to not be bother with that in the future. I will skip this step.
15. Reload de appliance (type reload in privileged mode). You will see that the command prompt can’t anymore be accessed via de VNC console. I mean, the console will open, but, at one moment the interaction will be handover to the serial console and no more activity going to be possible by VNC. The last message recorded in VNC confirm that: *Lina to use serial port /dev/ttyS0 for console IO*.

![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20console%20handover.png)

16. Now, after all the modifications to the ASAv image, we can switch the template back to his original Linked mode of operation. Also, we will switch the console settings to *telnet type*. Do the configuration changes in template settings, not in ASAv instance. The ASAv device from our temporary project can be safety removed, it has already done his job.

## Using newly created ASAv template

To use the newly created ASAv template, just drag the template icon to the workspace, do your connections and power-on the device. You can use multiple ASAv devices running simultaneous with no problem, on my PC (i7-4970s CPU with12GB RAM) I ran five concurrent instances - all started ok and became usable shortly (less than 1 min).

Just because we don’t mention *–nographic* in template’s *Advanced Settings – Additional Settings* the VNC console will automatically open every time you start the device. If you close that window, the appliance will power-off automatically. The VNC console don’t interfere with serial console which you can open via context menu. If you add the *-nographic* option, the VM will start silent without a VNC console. Anyway, my preference is to leave the VNC console to open automatically, at least for the begging just to have an additional visibility of the process.

After you load the ASAv device, you will periodically be announced by a missing license warning message: *Warning: ASAv platform license state is Unlicensed …* It is because the appliance don’t have a license key applied and it works in unlicensed state. As mentioned above, for lab and test scenarios, an unlicensed state are more than sufficient. In this state, you will get all the ASAv features but at the same time be limited to 100 Kbps interface bandwith.

It is interesting to see what virtual disks files are involved for an ASAv device started from our completed template. Beacause the template was configured as a Linked Mode VM, a linked virtual disk plus the base disk should be used, a fact confirmet by the screen below:

![alt-text](/assets/images/configuring%20Cisco%20ASAv%209.x%20on%20GNS3%201.4.x%20-%20qemu%20ASAv%20template%20-%20linked%20mode%20virtual%20disks.png)

[screencast](https://www.youtube.com/watch?v=oG1jAHdxyMg) for the process described above:

<a href="http://www.youtube.com/watch?feature=player_embedded&v=oG1jAHdxyMg
" target="_blank"><img src="http://img.youtube.com/vi/oG1jAHdxyMg/0.jpg"
alt="IMAGE ALT TEXT HERE" width="240" height="180" border="0" /></a>
