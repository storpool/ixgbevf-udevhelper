# ixgbe virtual function udev helper

## Description

Set of udev rules and script to help bringing up Virtual Function(VF) interfaces on Intel NICs handled by ixgbe(ixgbevf) Linux kernel module

## Author

* Anton Todorov (a.todorov@storpool.com)

## Installation

* Copy the udev rules file to /etc/udev/rules.d/:
```bash
cp 99-ixgbevf-udevhelper.rules /etc/udev/rules.d/
```
* Copy the udev helper script to /lib/udev/:
```bash
cp ixgbevf-udevhelper /lib/udev/
```
* Copy the config file to /etc/
```bash
cp ixgbevf-udevhelper.conf /etc/
```
* Reload udev rules
```bash
udevadm control --reload-rules
```

## Known issues

```
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0 enp4s0f0: SR-IOV enabled with 1 VFs
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1 enp4s0f1: SR-IOV enabled with 1 VFs
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1: not enough MMIO resources for SR-IOV
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0: not enough MMIO resources for SR-IOV
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0: Failed to enable PCI sriov: -12
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1: Failed to enable PCI sriov: -12
```

A workaround is to add also `pci=realloc` at the kernel cmdline.

## Configuration

### /etc/ixgbevf-udevhelper.conf

* Template used for naming VF interfaces
  Expandable variables:

    `_PFNAME_` - name of the phys interface
    `_VFID_` - VF id number, starting from 0
  Default: `VFNAME_TEMPLATE="_PFNAME__vf_VFID_"`

* List of PCI slots on which to enable VF

  Comma or space separated list of PCI_SLOT_NAMES:
    `PCI_SLOT_LIST="<PCI_SLOT_NAME>[,...]"`

* Number of VF interfaces to bring up.

  Comma or space separated list corresponding to each PCI_SLOT from above
    `PCI_NUMVFS_LIST="<number>[,...]"`

* VLAN\_ID for VF interface
  
  Optional set VLAN id to the VF interface
    `<EXPANDED_VFNAME_TEMPLATE>_VLAN=<VLAN_ID>`

* MTU for VF interface
  Optional set MTU to the VF interface
    `<EXPANDED_VFNAME_TEMPLATE>_MTU=<MTU>`

* Disable spoofchk on the VF interface

  Default is 'on', to turn off
    `<EXPANDED_VFNAME_TEMPLATE>_SPOOFCHK="off"`


For example to create two VF interfaces on eth2 and one VF interface on eth3 with corresponding PCI slot names 0000:04:00.0 and 0000:04:00.1 with VLANS 24 and 42 on eth2 and VLAN 1000 on eth3 and all with mtu 9000 with spoof check disabled on second VF on eth2:
```bash
PCI_SLOT_LIST="0000:04:00.0,0000:04:00.1"
PCI_NUMVFS_LIST=2,1
eth2_vf0_VLAN=24
eth2_vf1_VLAN=42
eth3_vf0_VLAN=1000
eth2_vf0_MTU=9000
eth2_vf1_MTU=9000
eth3_vf0_MTU=9000
eth2_vf1_SPOOFCHK=off
```

There is DEBUG variable that if set to anything will trigger the udev helper script to log more info to syslog.

Some CentOS kernels has issue with setting the MTU on the VFs before parent interface is up with proper MTU. To solve this uncomment the following to set the parent interface before setting the MTU on the slaves `SET_PARENT_MTU=yes`

### /etc/network/interfaces
* create configuration for the VF interface

Please note that the VF interfaces are manageable after the physical interface is up.

---
Copyright (c) 2015-2016 StorPool Storage AD
