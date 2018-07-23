# ixgbe and i40e virtual function udev helper

## Description

Set of udev rules and script to help bringing up Virtual Function(VF) interfaces on Intel NICs handled by ixgbe(ixgbevf) Linux kernel module

The newer NICs like X710 are using i40e kernel module which is handled too

## Author

* Anton Todorov (a.todorov@storpool.com)

## Installation

```bash
# Copy the udev rules file to /etc/udev/rules.d/:
cp 99-ixgbevf-udevhelper.rules /etc/udev/rules.d/
# Copy the udev helper script to /lib/udev/:
cp ixgbevf-udevhelper /lib/udev/
# Copy the config file to /etc/
cp ixgbevf-udevhelper.conf /etc/
# Reload udev rules
udevadm control --reload-rules
```

:exclamation: **_ethtool_ must be installed using the distribution's package manager.** Used to enable NIC hardware filtering.

## Known issues

```
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0 enp4s0f0: SR-IOV enabled with 1 VFs
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1 enp4s0f1: SR-IOV enabled with 1 VFs
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1: not enough MMIO resources for SR-IOV
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0: not enough MMIO resources for SR-IOV
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.0: Failed to enable PCI sriov: -12
Aug 10 11:43:24 server kernel: ixgbe 0000:04:00.1: Failed to enable PCI sriov: -12
```

A workaround is to add `pci=realloc` at the kernel cmdline.

```
Aug 10 12:21:24 server kernel: ixgbe 0000:02:00.0 enp2s0f0: SR-IOV enabled with 1 VFs
Aug 10 12:21:24 server kernel: ixgbe 0000:02:00.0: can't enable 1 VFs (bus 03 out of range of [bus 02])
Aug 10 12:21:24 server kernel: ixgbe 0000:02:00.0: Failed to enable PCI sriov: -12
```

A workaround is to add `pci=assign-busses` at the kernel cmdline.

## Configuration

### /etc/ixgbevf-udevhelper.conf

* Template used for naming VF interfaces
  Expandable variables:
  
    `_PFNAME_` - name of the phys interface
    `_VFID_` - VF id number, starting from 0
  
  Default: `VFNAME_TEMPLATE="_PFNAME__vf_VFID_"`

* List of PCI slots on which to enable VF
  
  Comma or space separated list of PCI_SLOT_NAMES:
  
    `PCI_SLOT_LIST="<PCI_SLOT_NAME>[,<PCI_SLOT_NAME>...]"`

* Number of VF interfaces to bring up.
  
  Comma or space separated list corresponding to each PCI_SLOT from above
  
    `PCI_NUMVFS_LIST="<number>[,<number>...]"`

* VLAN\_ID for VF interface
  
  Optional set VLAN id to the VF interface
  Must match the number of VFs enabled for the given PCI interface.
  
  The interface naming convention format:
    `PCI<PCI_SLOT_LIST ID>_VF_VLAN_LIST="<VLAN_ID>[,<VLAN_ID>...]"`

* MTU for VF interface
  
  Optional set MTU to the VF interface
  Must match the number of VFs enabled for the given PCI interface.
  
  The interface naming convention format:
    `PCI<PCI_SLOT_LIST ID>_VF_MTU_LIST="<mtu>[,<mtu>...]"`

* Disable spoofchk on the VF interface
  
  Must match the number of VFs enabled for the given PCI interface.
  
  The interface naming convention format:
    `PCI<PCI_SLOT_LIST ID>_VF_SPOOFCHK_LIST="off[,on...]"

  Default is 'on'

* Set VF trust for VF interface (to enable Multicast promiscuous mode)
  
  Must match the number of VFs enabled for the given PCI interface.
  
  The interface naming convention format:
    `PCI<PCI_SLOT_LIST ID>_VF_TRUST_LIST="on[,off...]"

  Default is 'off'

* Set custom unique name for VF interface
  
  Must match the number of VFs enabled for the given PCI interface.
  This option override VFNAME_TEMPLATE naming for given VF interface
  Leave single dash (-) where the default VFNAME_TEMPALTE should be used
  
  The interface naming convention format:
    `PCI<PCI_SLOT_LIST ID>_VF_TEMPLATE_LIST="spname0[,-...]"`

* Set MTU on the PF interface
  
  There are cases when it needs to be set like issues on CentOS/RHEL 7.2 driver
  Default is to set it to the VF_MTU value if there is VF_MTU value set

    `PCI_MTU_LIST="<mtu>[,<mtu>...]`

* When changing VF MTU bring the interface up first
  
  The VF interface will be bring up before setting it's MTU. To disable this behavior set
  
    `VF_UP_BEFORE_MTU='no'`

* Do not bring the primary interface up

  By default the helper is bringing up the primary interface before altering the VFs
  
    `<PCI_NO_UP_LIST="yes[,no...]"`

* By default the helper will rename the VF interface according to the Template.
  
  To leave the renaming of the VF interfaces to UDEV set the following variable:
  
    `RENAME_VF_INTERFACE=no`

* wait for VF information is sysfs to be populated
  
  On a slow/busy system the value could be increased
  
    `NUMVFS_WAIT=30`

For example to create two VF interfaces on eth2 and one VF interface on eth3 with corresponding PCI slot names 0000:04:00.0 and 0000:04:00.1 with VLANS 24 and 42 on eth2 and VLAN 1000 on eth3 and all with mtu 9000 with spoof check disabled on second VF on eth2:
```bash
PCI_SLOT_LIST="0000:04:00.0,0000:04:00.1"
PCI_NUMVFS_LIST=2,1
PCI0_VF_VLAN_LIST="24,42"
PCI1_VF_VLAN_LIST="1000"
PCI0_VF_MTU_LIST="9000,9000"
PCI1_VF_MTU_LIST="9000"
PCI0_VF_SPOOFCHK_LIST="on,off"
PCI1_VF_SPOOFCHK_LIST="on"
```

* hardware filtration in the NIC
  
  By default the helper tool enable the hardware filtering in the NIC.
  To disable set the following:
  
    `HARDWARE_FILTERING=no`

There is DEBUG variable that if set to anything will trigger the udev helper script to log more info to syslog.

### /etc/network/interfaces
* create configuration for the VF interface

Please note that the VF interfaces are manageable after the physical interface is up.

---
Copyright (c) 2015-2018 StorPool Storage AD
