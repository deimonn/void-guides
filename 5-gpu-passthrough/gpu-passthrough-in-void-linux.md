# GPU passthrough in Void Linux

This is a guide to PCI passthrough in Void Linux, based on my own experience trying to achieve it.

## Preface

My focus was primarily in setting up a Windows VM for gaming and other graphics intensive apps, so there's some things you must keep in mind:

- I very much assume a Linux host and Windows guest
- I assume an Nvidia card is the one that will be passed through to the guest, as they seem to be more stable for the task than AMD cards
  - The Nvidia card still needs to be plugged into a monitor (e.g. second source of your main monitor); this guide does not delve into setting up Looking Glass
- I assume that a dedicated AMD card is being used for the host; this guide does not deal with issues regarding integrated graphics
- You will probably want a beefy CPU to keep up with both machines (at least 8 cores/16 threads running at 3.0GHz or faster is recommended)

If it is of interest to you, here's the build I went with:

- Motherboard
  - MSI PRO Z790-A MAX WIFI
- GPUs
  - Nvidia RTX 4070 (on PCIe 5.0 x16 slot)
  - AMD RX 590 (on PCIe 4.0 x4 slot)
- CPU
  - Intel Core i7-14700F
- RAM
  - Corsair Vengeance 64GB 5600MHz DDR5

It is prudent to research your components thoroughly beforehand; before you buy something, I recommend you look online for any issues others may have had with it while attempting PCI passthrough.

## Procedure

### BIOS configuration

Your motherboard must support virtualization and IOMMU for passthrough to be possible. Most modern motherboards do.

Modern motherboards also come with all of the following settings enabled, but just in case:

- In your BIOS, ensure **VT-d**/**Virtualization** and **IOMMU** parameters are enabled for your CPU.
- Check that **IOMMU** is enabled on your system by running `sudo dmesg | grep -i IOMMU` and seeing the output

### Figuring out IOMMU groups

The [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Ensuring_that_the_groups_are_valid) provides a nifty little script for figuring out the IOMMU groups. Simply write the following to a file, like `print-iommu-groups`:

```Bash
#!/bin/bash
shopt -s nullglob
for g in $(find /sys/kernel/iommu_groups/* -maxdepth 0 -type d | sort -V); do
    echo "IOMMU Group ${g##*/}:"
    for d in $g/devices/*; do
        echo -e "\t$(lspci -nns ${d##*/})"
    done;
done;
```

Run it with `bash print-iommu-groups`. This will let you see which devices belong to which groups, and it should output something like:

```
IOMMU Group 0:
	00:00.0 Host bridge [0600]: Intel Corporation Raptor Lake-S 8+12 - Host Bridge/DRAM Controller [8086:a740] (rev 01)
IOMMU Group 1:
	00:01.0 PCI bridge [0604]: Intel Corporation Raptor Lake PCI Express 5.0 Graphics Port (PEG010) [8086:a70d] (rev 01)
IOMMU Group 2:
	00:08.0 System peripheral [0880]: Intel Corporation GNA Scoring Accelerator module [8086:a74f] (rev 01)
IOMMU Group 3:
	00:0a.0 Signal processing controller [1180]: Intel Corporation Raptor Lake Crashlog and Telemetry [8086:a77d] (rev 01)
IOMMU Group 4:
	00:14.0 USB controller [0c03]: Intel Corporation Raptor Lake USB 3.2 Gen 2x2 (20 Gb/s) XHCI Host Controller [8086:7a60] (rev 11)
	00:14.2 RAM memory [0500]: Intel Corporation Raptor Lake-S PCH Shared SRAM [8086:7a27] (rev 11)
IOMMU Group 5:
	00:16.0 Communication controller [0780]: Intel Corporation Raptor Lake CSME HECI #1 [8086:7a68] (rev 11)
IOMMU Group 6:
	00:17.0 SATA controller [0106]: Intel Corporation Raptor Lake SATA AHCI Controller [8086:7a62] (rev 11)
IOMMU Group 7:
	00:1c.0 PCI bridge [0604]: Intel Corporation Raptor Lake PCI Express Root Port #1 [8086:7a38] (rev 11)
IOMMU Group 8:
	00:1c.3 PCI bridge [0604]: Intel Corporation Raptor Lake PCI Express Root Port #4 [8086:7a3b] (rev 11)
IOMMU Group 9:
	00:1c.4 PCI bridge [0604]: Intel Corporation Device [8086:7a3c] (rev 11)
IOMMU Group 10:
	00:1f.0 ISA bridge [0601]: Intel Corporation Raptor Lake LPC/eSPI Controller [8086:7a04] (rev 11)
	00:1f.3 Audio device [0403]: Intel Corporation Raptor Lake High Definition Audio Controller [8086:7a50] (rev 11)
	00:1f.4 SMBus [0c05]: Intel Corporation Raptor Lake-S PCH SMBus Controller [8086:7a23] (rev 11)
	00:1f.5 Serial bus controller [0c80]: Intel Corporation Raptor Lake SPI (flash) Controller [8086:7a24] (rev 11)
IOMMU Group 11:
	01:00.0 VGA compatible controller [0300]: NVIDIA Corporation AD104 [GeForce RTX 4070] [10de:2786] (rev a1)
	01:00.1 Audio device [0403]: NVIDIA Corporation AD104 High Definition Audio Controller [10de:22bc] (rev a1)
IOMMU Group 12:
	03:00.0 Ethernet controller [0200]: Intel Corporation Ethernet Controller I225-V [8086:15f3] (rev 03)
IOMMU Group 13:
	04:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere [Radeon RX 470/480/570/570X/580/580X/590] [1002:67df] (rev e1)
	04:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] [1002:aaf0]
```

At the very least, each GPU should be in its own group for passthrough to be possible.

### Binding the Nvidia card to VFIO

Next, we need to get Linux to bind the Nvidia card to VFIO so it can be used by the guest machine. It will no longer be available to the host.

If for any reason you installed the `nouveau` or `nvidia` drivers before this step, remove them first.

1.  Append to file `/etc/modprobe.d/vfio.conf`:

    ```
    options vfio-pci ids=<PCI_IDS_TO_PASSTHROUGH>
    softdep drm pre: vfio-pci
    ```

    Replace `<PCI_IDS_TO_PASSTHROUGH>` with the comma-separated list of PCI devices you wish to passthrough; e.g. `10de:2786,10de:22bc`

2.  Append to file `/etc/dracut.conf.d/hostonly.conf`:

    ```
    hostonly=yes
    ```

3.  Regenerate the `initramfs` to apply the changes:

    ```Shell
    sudo xbps-reconfigure -f linux6.12
    ```

4.  Check that your system still boots fine, is using the AMD card for rendering, and that the Nvidia card belongs to VFIO.

    You can check the last point with:

    ```Shell
    sudo dmesg | grep -i vfio
    ```

    You should see something like:

    ```
    [    1.025281] VFIO - User Level meta-driver version: 0.3
    [    1.037883] vfio-pci 0000:01:00.0: vgaarb: deactivate vga console
    [    1.037893] vfio-pci 0000:01:00.0: vgaarb: VGA decodes changed: olddecodes=io+mem,decodes=io+mem:owns=none
    [    1.039202] vfio_pci: add [10de:2786[ffffffff:ffffffff]] class 0x000000/00000000
    [    1.085865] vfio_pci: add [10de:22bc[ffffffff:ffffffff]] class 0x000000/00000000
    ```

### Creating the virtual machine

Now we have to set up the virtual machine that we will pass the GPU to.

1.  Begin by performing the setup steps of [Setting up virtual machines](../3-extra-software/setting-up-virtual-machines.md) (everything before **Creating a virtual machine**) if you haven't already.

2.  Install `edk2-ovmf`:

    ```Shell
    sudo xbps-install edk2-ovmf
    ```

3.  Go to "Create a virtual machine"; the steps are pretty self-explanatory, just make sure to check **Customize configuration before install** before clicking "Finish" on the last slide.

4.  The configuration screen will appear. You'll have to procure the following settings:

    - In the "Overview" section, **Firmware** must be set to **UEFI**.
    - In the "CPUs" section, **Copy host CPU configuration** must be checked.

5.  Click "Begin installation" and install the OS as per usual.

6.  Install and set up [VirtIO for Windows](../3-extra-software/setting-up-virtual-machines.md#virtio-for-windows). Switching the disk to "SCSI" and NIC to "virtio" is optional but recommended.

7.  With the machine off, click "Add Hardware", and add your Nvidia GPU as a "PCI Host Device" along with any other PCI devices you wish to pass.

8.  You'll have to detach virtual integration devices next.

    Go to hardware details, in the "Overview" section, switch to the XML tab. You may have to enable XML editing in preferences.

    Remove all of the following tags and their contents:

    - `channel`:

      ```XML
      <channel type="spicevmc">
        ...
      </channel>
      ```

    - `input`:

      ```XML
      <input type="tablet" bus="usb">
        ...
      </input>
      ```

    - `graphics`:

      ```XML
      <graphics type="spice" autoport="yes">
        ...
      </graphics>
      ```

    - `audio`:

      ```XML
      <audio id="1" type="spice"/>
      ```

    - `redirdev` (there are multiple):

      ```XML
      <redirdev bus="usb" type="spicevmc"> ... </redirdev>
      ```

    - `video`:

      ```XML
      <video> ... </video>
      ```

    Press **Apply** to validate and format the changes.

9.  Disable `memballoon`, which causes major performance issues for VFIO passthrough.

    Find the following in your overview XML:

    ```XML
    <memballoon model="virtio">
      <address type="pci" domain="0x0000" bus="0x04" slot="0x00" function="0x0"/>
    </memballoon>
    ```

    And replace it with:

    ```XML
    <memballoon model="none"/>
    ```

10. Next, we'll set up your keyboard and mouse to toggle between the host and the guest by pressing both CTRL keys at the same time.

    First determine which devices are your keyboard and mouse in `/dev/input/by-id`; you only care for devices with `event` in their name. Try to `cat` each one to see which produce output when you wiggle your mouse or type some letters.

    If no device in `/dev/input/by-id` produces any output when `cat`'d, it is possible you have some intercepting service set up, like keymapper (see [Keymapper and PCI passthrough](../3-extra-software/key-remapping-with-keymapper.md#keymapper-and-pci-passthrough)). You will have to either disable it or configure it in some manner.

    Search for the following two lines:

    ```XML
    <input type="mouse" bus="ps2"/>
    <input type="keyboard" bus="ps2"/>
    ```

    Replace `"ps2"` with `"virtio"` for both of them. Then, below them, add:

    ```XML
    <input type="evdev">
      <source dev="/dev/input/by-id/<MOUSE>"/>
    </input>
    <input type="evdev">
      <source dev="/dev/input/by-id/<KEYBOARD>" grab="all" repeat="on" grabToggle="ctrl-ctrl"/>
    </input>
    ```

    Replace `<MOUSE>` and `<KEYBOARD>` with the corresponding names as found in `/etc/input/by-id`.

11. Lastly, we'll set up audio forwarding. I am assuming you are using PipeWire, as per the [Installation Guide](../1-installation/guide.md).

    In the overview XML, look for:

    ```XML
    <audio id="1" type="none"/>
    ```

    Replace it with:

    ```XML
    <audio id="1" type="pipewire" runtimeDir="/run/user/1000">
      <input name="qemuinput"/>
      <output name="qemuoutput"/>
    </audio>
    ```

    If your user ID is not `1000`, you will have to change it. You can retrieve your user ID by running `id -u`.

12. Open the `/etc/libvirt/qemu.conf` file for editing and look for:

    ```
    #user = "example"
    ```

    Remove the `#` and replace `example` with your user name. You may have to reboot the computer for the change to enter into effect.

13. You can finally start the VM. If everything went well, the monitor attached to your Nvidia GPU should display the Windows login screen. You should also test sound and input.

Assuming your venture was successful, congratulations!

If not, you may wish to look into the [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF) to see if it can help you troubleshoot the issue. At worst, some research will be required. Good luck!

## Performance improvements

There are a couple more things that you can do to further improve virtual machine performance that are relatively unintrusive.

### vCPU allocation

Although this may sound counter-intuitive, it is worthwhile to reserve a couple of cores solely for host use.

This is because many operations (like disk I/O) still need to be handled by QEMU on the host, so if all resources are hogged by the guest and the host can't process things fast enough, the guest too will lag.

You should probably reserve at least 2 cores for host operation, but how many you actually need depends on how much work your host will continue doing in the background.

### CPU pinning

QEMU by default emulates each vCPU by just spawning a thread, which is treated like any other thread by the scheduler. As such, the local caches are lost everytime Linux reschedules the thread to a different physical CPU.

You can remediate this issue by pinning the threads to specific physical CPUs.

1.  First, you must figure out your CPU topology. You can do this by running `lscpu -e`, which will output something like:

    ```
    CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ   MINMHZ       MHZ
      0    0      0    0 0:0:0:0          yes 5300.0000 800.0000  925.2790
      1    0      0    0 0:0:0:0          yes 5300.0000 800.0000  800.0000
      2    0      0    1 4:4:1:0          yes 5300.0000 800.0000 1099.9680
      3    0      0    1 4:4:1:0          yes 5300.0000 800.0000  897.8860
      4    0      0    2 8:8:2:0          yes 5300.0000 800.0000 1100.0000
      5    0      0    2 8:8:2:0          yes 5300.0000 800.0000  800.0000
      6    0      0    3 12:12:3:0        yes 5300.0000 800.0000 1100.0000
      7    0      0    3 12:12:3:0        yes 5300.0000 800.0000  800.0000
      8    0      0    4 16:16:4:0        yes 5400.0000 800.0000  800.0000
      9    0      0    4 16:16:4:0        yes 5400.0000 800.0000  800.0000
     10    0      0    5 20:20:5:0        yes 5400.0000 800.0000 1100.0000
     11    0      0    5 20:20:5:0        yes 5400.0000 800.0000  800.0000
     12    0      0    6 24:24:6:0        yes 5300.0000 800.0000 1093.6479
     13    0      0    6 24:24:6:0        yes 5300.0000 800.0000  876.2070
     14    0      0    7 28:28:7:0        yes 5300.0000 800.0000 1045.5710
     15    0      0    7 28:28:7:0        yes 5300.0000 800.0000 1100.0110
     16    0      0    8 32:32:8:0        yes 4200.0000 800.0000  798.6430
     17    0      0    9 33:33:8:0        yes 4200.0000 800.0000  799.2990
     18    0      0   10 34:34:8:0        yes 4200.0000 800.0000  798.6340
     19    0      0   11 35:35:8:0        yes 4200.0000 800.0000  799.4930
     20    0      0   12 36:36:9:0        yes 4200.0000 800.0000  800.0000
     21    0      0   13 37:37:9:0        yes 4200.0000 800.0000  800.0000
     22    0      0   14 38:38:9:0        yes 4200.0000 800.0000  800.2680
     23    0      0   15 39:39:9:0        yes 4200.0000 800.0000  800.0000
     24    0      0   16 40:40:10:0       yes 4200.0000 800.0000  800.0000
     25    0      0   17 41:41:10:0       yes 4200.0000 800.0000  800.9920
     26    0      0   18 42:42:10:0       yes 4200.0000 800.0000  799.7670
     27    0      0   19 43:43:10:0       yes 4200.0000 800.0000  800.0000
    ```

    - The `CPU` column is the number that you'll use to identify that particular physical CPU.

    - The `CORE` column specifies to which physical core a CPU belongs; note that because of hyperthreading, two CPUs can share one core. This fact will become important later.

    - The `L1d:L1i:L2:L3` column enumerates the caches for each level. When two CPUs share L1/L2 caches, you will want to pin them **together**, or you'll make performance worse instead of improving it.

      If you have more than one L3 cache, you may wish to pin the whole group sharing that L3 cache together for better performance. If you don't (like in the example output above), ignore it.

      Your goal is to minimalize cache sharing between the host and the guest.

2.  Having figured out which physical CPUs you wish to pin for the guest, look for a `<vcpu>` element in the overview XML:

    ```XML
    <vcpu placement="static">16</vcpu>
    ```

    Below it, append the following:

    ```XML
    <cputune>
      <vcpupin vcpu="<VCPU>" cpuset="<CPUSET>"/>
    </cputune>
    ```

    Repeat the `<vcpupin>` line as many times as you have vCPUs allocated, where:

    - `vcpu="<VCPU>"` is the zero-based index of the vCPU.
    - `cpuset="<CPUSET>"` is the physical CPU you want to assign to it; this would be the `CPU` column of the `lscpu -e` output.

3.  You can pin a core for the emulator running on the host, which also helps:

    ```XML
    <cputune>
      ...
      <emulatorpin cpuset="<CPUSET>"/>
    </cputune>
    ```

    You can specify multiple CPUs using a comma, like `cpuset="0,1"`, as you still want to keep CPUs sharing L1/L2 caches together.

4.  If you switched your disk to SCSI during [Creating the virtual machine](#creating-the-virtual-machine), you can benefit from having a pinned I/O thread too. Add it with:

    ```XML
    <cputune>
      ...
      <iothreadpin iothread="1" cpuset="<CPUSET>"/>
    </cputune>
    ```

    Also add the following below the closing `</cputune>` tag:

    ```XML
    <iothreads>1</iothreads>
    ```

    You can use the same `<CPUSET>` as for the emulator, since the pinned I/O thread also runs on the host.

    Further setup is required to actually use the thread; see [Dedicated I/O thread](#dedicated-io-thread).

5.  Look for the `<cpu>` block, which will look something like:

    ```XML
    <cpu mode="host-passthrough" check="none" migratable="on"/>
    ```

    Replace `migratable="on"` with `migratable="off"`.

    Afterwards, expand it to a full element, and inside of it insert:

    ```XML
    <topology sockets="1" cores="<CORES>" threads="<THREADS>"/>
    <cache mode="passthrough"/>
    ```

    Like this:

    ```XML
    <cpu mode="host-passthrough" check="none" migratable="off">
      <topology sockets="1" cores="<CORES>" threads="<THREADS>"/>
      <cache mode="passthrough"/>
    </cpu>
    ```

    - Replace `<THREADS>` with `2` if **all** the CPUs you pinned for the guest do hyperthreading (i.e. there's two CPUs per physical core); otherwise just replace it with `1`.
    - Replace `<CORES>` with the number of vCPUs you allocated, unless you set `threads="2"` - in which case this number should be *half* the number of allocated vCPUs.

6.  Press **Apply** to validate and format your additions.

Example configuration:

```XML
<vcpu placement="static">16</vcpu>
<cputune>
  <vcpupin vcpu="0" cpuset="0"/>
  <vcpupin vcpu="1" cpuset="1"/>
  <vcpupin vcpu="2" cpuset="2"/>
  <vcpupin vcpu="3" cpuset="3"/>
  <vcpupin vcpu="4" cpuset="4"/>
  <vcpupin vcpu="5" cpuset="5"/>
  <vcpupin vcpu="6" cpuset="6"/>
  <vcpupin vcpu="7" cpuset="7"/>
  <vcpupin vcpu="8" cpuset="8"/>
  <vcpupin vcpu="9" cpuset="9"/>
  <vcpupin vcpu="10" cpuset="10"/>
  <vcpupin vcpu="11" cpuset="11"/>
  <vcpupin vcpu="12" cpuset="12"/>
  <vcpupin vcpu="13" cpuset="13"/>
  <vcpupin vcpu="14" cpuset="14"/>
  <vcpupin vcpu="15" cpuset="15"/>
  <emulatorpin cpuset="16"/>
  <iothreadpin iothread="1" cpuset="17"/>
</cputune>
<cpu mode="host-passthrough" migratable="off">
  <topology sockets="1" cores="8" threads="2"/>
  <cache mode="passthrough"/>
</cpu>
<iothreads>1</iothreads>
```

### Dedicated I/O thread

You can set up SCSI to use a dedicated I/O thread as discussed in CPU pinning above.

1. In the overview XML, look for your SCSI `<disk>` device. It will look something like:

  ```XML
  <disk type="file" device="disk">
    <driver name="qemu" type="qcow2"/>
    <source file="/var/lib/libvirt/images/win10.qcow2"/>
    <target dev="sdc" bus="scsi"/>
    <address type="drive" controller="0" bus="0" target="0" unit="2"/>
  </disk>
  ```

  To the `<driver>` element, append `io="threads" cache="writeback" discard="unmap"`, like:

  ```XML
  <driver name="qemu" type="qcow2" io="threads" cache="writeback" discard="unmap"/>
  ```

2. Next, look for a `<controller type="scsi">`, like:

  ```XML
  <controller type="scsi" index="0" model="virtio-scsi">
    <address type="pci" domain="0x0000" bus="0x05" slot="0x00" function="0x0"/>
  </controller>
  ```

  Append `<driver iothread="1" queues="8"/>` to the `<controller>` contents to enable the I/O thread:

  ```XML
  <controller ...>
    ...
    <driver iothread="1" queues="8"/>
  </controller>
  ```

### Enabling `topoext`

**AMD CPUs only.** Hyper-threading for AMD CPUs is disabled on QEMU by default, so you need to enable it manually.

In the `<cpu>` block described in [CPU pinning](#cpu-pinning), below the `<topology>` element, append:

```XML
<feature policy="require" name="topoext"/>
```

### Further reading

These are far from the only things you can do to improve performance, but you may slowly end up in a path of both diminishing returns and host-intrusive configurations.

For example, you can [improve performance by ~2%](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Static_huge_pages) by statically reserving memory at boot solely for the VM, with the caveat that it permanently locks the memory away from the host even when the VM is not running.

Should you require such level of optimization, I recommend you read the [Performace tuning](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF#Performance_tuning) section of the Arch Wiki.

## Final notes

### Ignoring disks in snapshots

If you have disks that you wish to leave as-is on snapshots (for example, such that only the disk containing Windows is snapshotted), you can simply append `snapshot="no"` to the `<disk>` device to ignore, like:

```XML
<disk type="file" device="disk" snapshot="no">
  ...
</disk>
```

### Physical disk passthrough

To passthrough a physical disk, append the following next to your other `<disk>` devices in your overview XML:

```XML
<disk type="block" device="disk" snapshot="no">
  <driver name="qemu" type="raw" cache="none" io="native"/>
  <source dev="/dev/disk/by-id/<ID>"/>
  <target dev="<DEV>" bus="sata"/>
</disk>
```

- Replace `<ID>` with your disk's ID (run `ls /dev/disk/by-id` to see all).
- Replace `<DEV>` with a linux-style device name continuing off the previous one; e.g. if the previous disk is `sdc`, make it `sdd`.

Take care not to mount any partitions you make inside of it on the Linux side while the VM is still running, or you'll corrupt the filesystem.

If you're not going to be accessing the disk from Linux at all, you can always append the following to your `/etc/fstab` to prevent yourself from accidentally mounting it:

```
/dev/disk/by-id/<ID>-part<N> /mnt/<ID>-part<N> auto noauto,x-udisks-auth 0 0
```

Replace `<ID>` and `<N>` as appropiate. This should hide the partition from most file managers.

### Can't open app due to virtualization

If some applications refuse to open on detecting that you're running on a virtual machine, you may want to go through the steps of [Hiding virtualization details](hiding-virtualization-details.md). It won't work for all applications, but it can help with many.
