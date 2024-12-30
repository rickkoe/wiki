---
title: FlashSystem Host Configuration Best Practices
description: 
published: true
date: 2023-01-23T16:08:27.087Z
tags: ibm, flashsystem, best practice
editor: markdown
dateCreated: 2022-02-11T01:11:34.900Z
---

# General Information

This document is intended to be a consolidation of information from multiple sources listed in the [Documentation Links](/ibm-flashsystem).  Please consider this document to be a Quick Reference Guide or “cheat sheet”.

## Queue Depths

Queue depth is used to control the number of concurrent operations that occur on different storage resources. Queue depth is the number of I/O operations that can be run in parallel on  
a device.Queue depths apply at various levels of the system, at the disk or flash level, at the storage controller level and at the per volume and host bus adapter (HBA) level on the host. For  
example, each Spectrum Virtualize node has a queue depth of 10,000. A typical disk drive will operate efficiently at a queue depth of 8. Most host volume queue depth defaults will be  
around 32. Guidance for limiting queue depths in large SANs that was described in previous documentation has been replaced with calculations for overall I/O group based queue depth  
considerations. There isn’t a set rule for setting a queue-depth value per host HBA or per volume. The requirements for your environment will be driven by the intensity of each workload. You should ensure that one application or host cannot run away and use the entire controller queue.  However, if you have a specific host application that requires the lowest latency and highest  
throughput, then you should consider giving it a proportionally larger share than others.  In configurations with multiple hosts, configure each host with a similar queue depth to maintain fairness between the hosts.

-   For most hosts, set the HBA queue depth to 32.
-   For hosts that are significantly busier or where not many hosts are configured on the storage system, set the HBA queue depth to 128.

# AIX

## Configuring for fast fail and dynamic tracking

Issue this to set the Fibre Channel SCSI I/O Controller Protocol Device event error recovery policy to fast\_fail for each Fibre Channel adapter:  *This example command is for adapter fscsi0.*

```plaintext
chdev -l fscsi0 -a fc_err_recov=fast_fail command 
```

> By default, dynamic tracking is enabled on all systems running AIX 7.1
{.is-info}

Issue this command to enable dynamic tracking for each Fibre Channel device:  *This example command is for adapter fscsi0.*

```plaintext
chdev -l fscsi0 -a dyntrk=yes 
```

## Configuration Recommendations for AIX

The following device settings can be changed with the chdev AIX command.

-   **reserve\_policy=no\_reserve**  
    The default reserve policy is single\_path (SCSI-2 reserve). Unless there is a specific need for reservations, use no\_reserve.
-   **algorithm=shortest\_queue**   
    If coming from SDD PCM, AIX defaults this to fail\_over. You cannot set algorithm to shortest\_queue unless reservation policy is no\_reserve.
-   **queue\_depth=32**

The default queue depth is 20. IBM recommends 32.

-   **rw\_timeout=30**  
    IBM recommends 30.

>60 was the default for the SDD PCM and 30 is the default for AIX PCM.
{.is-info}

## Configuring Multi-Pathing

The recommended multi-path driver to use on IBM AIX and VIOS when attached to Spectrum Virtualize storage devices is the default AIX PCM.

If the SDDPCM driver is already installed on your system, refer to the SDDPCM users guide for instructions on removing the SDDPCM from your system.

[http://www-01.ibm.com/support/docview.wss?uid=ssg1S7000303](http://www-01.ibm.com/support/docview.wss?uid=ssg1S7000303)

To configure multiple paths for all LUNs, run the cfgmgr command to scan for new devices.

If you are using the AIX Path Control Module (AIXPCM) and want to check that the multipath device is available, run the lsdev command and search for the hdisk# line with the MPIO IBM #### FC Disk label against it.

## Verifying Multi-Pathing

[_https://www.ibm.com/docs/en/aix/7.2?topic=l-lsmpio-command_](https://www.ibm.com/docs/en/aix/7.2?topic=l-lsmpio-command)

You can run the **lsmpio** command without any flags or with the **\-l** flag to display the path operational status.

```plaintext
lsmpio
```

The output is similar to the output that is displayed by running the **lspath** command as shown in the following command:

```plaintext
lspath -F hdiskN
```

The valid values of the status column are *Enabled*, *Disabled*, *Failed*, or *Missing*. The extended path\_status field might contain one or more three-letter status abbreviations to provide more detailed path status.

## Queue Depth

To list the current queue\_depth attribute value for the hdisk2 disk, type the following command:

```plaintext
lsattr -E -l hdisk2 -a queue_depth
```

Setting Queue Depth JBOD disks and RAID disks on PCIe2 and PCIe3 adapter families

Use the chdev command with the queue\_depth attribute on the JBOD or RAID disk as in the following example:

```plaintext
chdev -1 hdiskN -a " queue\_depth =value "
```

Where hdiskN represents the name of a JBOD or RAID disk and value is the value you assign for the disk queue depth.

## Known AIX issues and limitations

The restriction of Direct Memory Access (DMA) resources is one of the known issues and limitations with the system and an IBM® Power Systems AIX® host.

The system does not impose a size limitation on volumes for AIX hosts. Check your AIX documentation to determine whether the AIX system itself restricts volume size.

On a heavily loaded system, the following symptoms can indicate that the host is low on direct memory access (DMA) resources:

-   You might see errors that indicate that the host bus adapter (HBA) was unable to activate an I/O request on the first attempt.
-   You might see lower-than-expected performance with no errors being logged.

To reduce the incidence of these messages, you can increase the resources by modifying the maximum transfer size attribute for the adapter as follows:

1.  Type the lsattr -El HBA -a max\_xfer\_size command to view the current setting, where HBA is the name of the adapter that is logging the error. For this example, the HBA is fcs0.
2.  Type the chdev -l fcs0 -P -a max\_xfer\_size=0x1000000 command to increase the size of the setting.
>Note: To view the range of allowable values for the attribute, type lsattr -Rl fcs0 -a max\_xfer\_size.
{.is-info}
3. Restart the host to put these changes into effect.

# VMware

VMware has a built-in multipathing driver which supports Spectrum Virtualize ALUA preferred path algorithms.

Consider the following points:
- The storage array type should be ALUA (VMW\_SATP\_ALUA).
- Path selection policy should be RoundRobin.

The Round Robin Input/Output Operations Per Second (IOPS) should be changed from 1000 to 1, to evenly distribute I/Os across as many ports on the system as possible. For more information about how to change this setting, see [_VMware Adjusting Round Robin IOPS limit from default 1000 to 1_](https://kb.vmware.com/s/article/2069356).

## Changing the queue depth for QLogic, Emulex, and Brocade HBAs

[_https://kb.vmware.com/s/article/1267_](https://kb.vmware.com/s/article/1267)

## Adjusting the IOPS parameter

To adjust the IOPS parameter from the default 1000 to 1, run this command:

In ESXi 6.x:

for i in \`esxcfg-scsidevs -c |awk '{print 1}' | grep naa.xxxx\\\`; do esxcli storage nmp psp roundrobin deviceconfig set --type=iops --iops=1 --device=i; done

Where, *.xxxx* matches the first few characters of your naa IDs.

To verify if the changes are applied, run this command:

```plaintext
esxcli storage nmp device list
```

You see output similar to:

Path Selection Policy: VMW\_PSP\_RR  
Path Selection Policy Device Config: {policy=iops,iops=1,bytes=10485760,useANO=0;lastPathIndex=1: NumIOsPending=0,numBytesPending=0}  
Path Selection Policy Device Custom Config:  
Working Paths: vmhba33:C1:T4:L0, vmhba33:C0:T4:L0

*Note: You do not need to restart the host for the changes to take effect.*

# RedHat Enterprise Linux (RHEL)

You must ensure the following configuration settings to restore the path if they are lost during an upgrade or other hardware maintenance that results in a node being offline:

## SCSI INQUIRY TIMEOUT

Add scsi\_mod.inq\_timeout=70 to the kernel boot command line through GRUB configuration. By adding the scsi\_mod.inq\_timeout=70 parameter, the change in the parameter is persistent from a server reboot. Linux hosts can also regain system node paths when lost. This change can be done by completing the following steps on RHEL 7/8:

1.  To make the change permanent, do the following:

vi /etc/sysconfig/grub

1.  Add to the GRUB\_CMDLINE\_LINUX line:

scsi\_mod.inq\_timeout=70

1.  Run the following command to rewrite the boot record:

-   On BIOS-based machines:  # grub2-mkconfig -o /boot/grub2/grub.cfg
-   On UEFI-based machines:  # grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

1.  reboot

VERIFY USING THIS COMMAND:

systool -m scsi\_mod -A inq\_timeout

if systool is not installed:

yum install sysfsutils

## SCSI COMMAND TIMEOUT

Set the udev rules for SCSI command timeout to 120s. This is the recommended setting for all versions of Linux.

To increase the SCSI command timeout for the system, create the following udev rule:

1.  Set SCSI command timeout to 120s (default == 30 or 60) for IBM 2145 devices

SUBSYSTEM=="block", ACTION=="change", ENV{ID\_VENDOR}=="IBM",ENV{ID\_MODEL}=="2145", RUN+="/bin/sh -c 'echo 120 >/sys/block/%k/device/timeout'"

1.  After you set up your volumes, confirm that they are set for 120 seconds. Locate the block device paths by running:

multipath -ll | grep sd

1.  Then run:

cat /sys/block/sdX/device/timeout (where X is each 2145 block device path).

1.  To reload the udev rules without rebooting (or dynamically) you can run the following commands:

udevadm control -R

/sbin/udevadm trigger --type=devices --action=add

## Multipath settings in multipath.conf

Edit /etc/multipath.conf with the following parameters and confirm the changes by entering:

```plaintext
multipathd -k
multipathd> show config
Red Hat Linux versions 5.x, 6.0, and 6.1
vendor "IBM"
product "2145"
path_grouping_policy "group_by_prio"
path_selector "round-robin 0"
prio_callout "/sbin/mpath_prio_alua /dev/%n" #Used by Red Hat 5.x
prio "alua"
path_checker "tur"
failback "immediate"
no_path_retry 5
rr_weight uniform
rr_min_io 1000
dev_loss_tmo 120    
Red Hat Linux versions 6.2 and higher
vendor "IBM"
product "2145"
path_grouping_policy "group_by_prio"
path_selector "round-robin 0" # Used by Red Hat 6.2
prio "alua"
path_checker "tur"
failback "immediate"
no_path_retry 5
rr_weight uniform
rr_min_io_rq "1"
dev_loss_tmo 120
Red Hat Linux version 7.x and 8.x
vendor "IBM"
product "2145"
path_grouping_policy "group_by_prio"
path_selector "service-time 0" # Used by Red Hat 7.x
prio "alua"
path_checker "tur"
failback "immediate"
no_path_retry 5
rr_weight uniform
rr_min_io_rq "1"
dev_loss_tmo 120   
```

## Useful Linux Commands

**Find FC Port WWPNs in RHEL**

$ grep -v "zZzZ" -H /sys/class/fc\_host/host\*/\*\_name/sys/class/fc\_host/host6/fabric\_name:0x100000051e900105    << switch port name, wwpn/sys/class/fc\_host/host6/node\_name:0x200000e08b87de9a      <<    HBA node name, wwnn/sys/class/fc\_host/host6/port\_name:0x**210000e08b87de9a**      <<    HBA port name, wwpn/sys/class/fc\_host/host6/symbolic\_name:QLE2462 FW:v7.03.00 DVR:v8.07.00.18.07.2-k

**Scan for new devices in RHEL without rebooting**

```plaintext
for i in /sys/class/scsi_host/host*/scan; do echo "- - -" > $i; done
```

```
[root@rick-3650m3 class\]# multipath -ll
mpatha (360050768108101cc880000000000427a) dm-3 IBM     ,2145
size=70G features='1 queue\_if\_no\_path' hwhandler='0' wp=rw
|-+- policy='service-time 0' prio=50 status=active
| |- 5:0:1:0 sdc 8:32 active ready running
| \`- 7:0:1:0 sde 8:64 active ready running
\`-+- policy='service-time 0' prio=10 status=enabled
|- 5:0:0:0 sdb 8:16 active ready running
\`- 7:0:0:0 sdd 8:48 active ready running
```

# Windows

## Multipath Module

Use Microsoft Multipath I/O (MPIO) with Microsoft Device Specific Module (MS DSM), which

is included in the Windows Server operating system. The older Subsystem Device Driver

Device Specific Module (SDDDSM) is no longer supported.

## Changing the disk timeout on Microsoft Windows Server

To change the disk timeout value on Windows Server operating systems, follow steps in the Windows Server host and in the registry browsing tool.

On your Windows Server hosts, change the **disk I/O timeout** value to 60 in the Windows registry, as follows:

1.  In Windows, click **Start** > **Run**.
2.  In the dialog text box, type regedit and click **Enter**.
3.  In the registry browsing tool, locate the HKEY\_LOCAL\_MACHINE\\System\\CurrentControlSet\\Services\\Disk\\TimeOutValuekey.
4.  Confirm that the value for the key is 60 (decimal value). If necessary, change the value to 60.

## Configure Qlogic HBA

To configure the HBA BIOS, either use the QLogic HBA manager software or reboot into the Fast!UTIL tool. Configure the following settings:

-   Host Adapter BIOS: Disabled (unless the machine is configured for SAN Boot)
-   Adapter Hard Loop ID: Disabled
-   Connection Options: 1 - point to point only
-   LUNs Per Target: 0
-   Port Down Retry Count: 15

Set the execution throttle to a suitable queue depth for your environment, for example, a value of 100. For more information about the execution throttle and queue depth, see QLogic documentation.

If you are using Native Multipathing Plug-In (NMP), set Enable Target Reset to No. See Table 1 to include the required parameters for the registry key.

| Key | Required parameters |
| --- | --- |
| HKEY\_LOCAL\_MACHINE > SYSTEM > CurrentControlSet > Services > ql2xxx > Parameters > Device > DriverParameters | Buschange=0;FixupInquiry=1  <br>  <br>Note: If you are using QLogic driver version 9.1.2.11 or later, Buschange cannot be set to zero. See your device driver documentation for details. |
| Table 1. Registry key parameters for QLogic models |     |

## Configure Emulux HBA

After you install the Emulex host bus adapter (HBA) and the driver on hosts that run a Microsoft Windows Server operating system, you must configure the HBA.

For the Emulex HBA StorPort driver, accept the default settings and set topology to 1 (1=F\_Port Fabric). For the Emulex HBA FC Port driver, use the default settings and change the parameters that are given in Table 1.

Note: The parameters that are shown in parentheses correspond to the parameters in HBAnywhere.

| Parameters | Recommended Settings |
| --- | --- |
| Query name server for all N-ports (BrokenRSCN) | Enabled |
| LUN mapping (MapLuns) | Enabled (1) |
| Automatic LUN mapping (MapLuns) | Enabled (1) |
| Allow multiple paths to SCSI targets (MultipleSCSIClaims). | Enabled |
| Scan in device ID order (ScanDeviceIDOrder) | Disabled |
| Translate queue full to busy (TranslateQueueFull). | Enabled |
| Retry timer (RetryTimer) | 2000 milliseconds |
| Maximum number of LUNs (MaximumLun) | Equal to or greater than the number of the system LUNs that are available to the HBA. |
| Table 1. Configuration file parameters for the Emulex HBA |     |