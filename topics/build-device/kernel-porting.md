---
title: Kernel porting Guide
---

## Introduction

When dealing with 3rd-parties, or simply when trying to port Snappy Ubuntu Core to a new piece of hardware, the questions that we most frequently face are:

Q: What are the kernel config options that Snappy Ubuntu Core requires to run?

Q: What are the options that the kernel I use on my hardware must support to properly run Snappy Ubuntu Core?

Q: Is feature X mandatory to run Snappy Ubuntu Core?

The following is meant to answer these questions and serve as a guide to facilitate third parties on non-Ubuntu based kernels and most likely older kernel versions (eg. 3.10, 3.14) to enable the minimum kernel requirements in order to get an introductory experience with Snappy Ubuntu Core. This may also highlight areas where it would be expected that features/functionality would be absent/broken due to the kernels lacking the support.

The list of features that Snappy Ubuntu Core requires to work is the sum of all the features required by the building blocks and software that Snappy Ubuntu Core builds upon. Â With this in mind, the Snappy Ubuntu Core delta configuration was split in kconfig fragments (one per area / software) that people can apply on top of the base defconfig of their hardware.

DISCLAIMER: Â It should also be noted that these are NOT officially maintained. Â There is no routine security maintenance nor bug fixing done for these. Â For official support and maintenance, please contact <>.

## Requirements

### Kernel Config

Goal: Provide a series of config changes that developers can apply with scripts/kconfig/merge_config.sh on top of their defconfig file.

The instructions below assume you are using an Ubuntu 16.04 x86_64 workstation, have a recent version of snapcraft installed (>= 2.8.4), and have the tools required to build a kernel installed (eg. Ubuntu requires build-essential, toolchain, apt-get build-dep linux-image-`uname -r`, etc.).

Overview of the kconfig delta:

An example of the kconfig delta is available here ([https://git.launchpad.net/~p-pisati/ubuntu/+source/linux/tree/kernel/configs/snappy?h=snappy_v4.4](https://git.launchpad.net/~p-pisati/ubuntu/%2Bsource/linux/tree/kernel/configs/snappy?h%3Dsnappy_v4.4&sa=D&ust=1474626830425000&usg=AFQjCNERlK9yEX-MVaVWbuAbWjaLcIAsQA)) and is composed of:

1.  Generic.config - contains the features that we enforce in the Ubuntu config, and in general all the configs that â€˜make senseâ€™ to enable
2.  Security.config - security options that we want to turn on - AA, SECCOMP, STACKPROTECTOR, etcetc
3.  Systemd.config - features required by systemd (see also Â [https://github.com/systemd/systemd/blob/master/README](https://github.com/systemd/systemd/blob/master/README&sa=D&ust=1474626830426000&usg=AFQjCNFBWjpqfF4mKXw5quBh8budD3ImHg)Â - REQUIREMENTS section)
4.  Snappy.config - features that are required by ubuntu-core go here
5.  Containers.config - features required by lxc/docker (see also https://github.com/docker/docker/blob/master/contrib/check-config.sh)

Temporary git tree & base branches location:

*   [https://git.launchpad.net/~p-pisati/ubuntu/+source/linux/](https://git.launchpad.net/~p-pisati/ubuntu/%2Bsource/linux/&sa=D&ust=1474626830428000&usg=AFQjCNEVTvzAog1x2ZXWVkSAY2uAz3RmQw)

*   snappy_v4.4
*   snappy_v3.18
*   snappy_v3.14
*   snappy_v3.10
*   3.16? 4.1? â€¦

*   Android tree [https://code.launchpad.net/~wenchien/+git/android-kernel](https://code.launchpad.net/~wenchien/%2Bgit/android-kernel&sa=D&ust=1474626830429000&usg=AFQjCNGQBm-9opaB8mrjWnOvxoBuv__p0g)

*   snappy_v3.10 - based on android-3.10.y
*   snappy_v3.14 - based on android-3.14

All of these branches went under these modifications:

1.  The directory â€˜kernel/configs/snappyâ€™ containing the necessary kconfig delta (generic.config, security.config, snappy.config, systemd.config, containers.config) was added to the root of every branch
2.  The necessary kbuild bits to make the kernel build using the kconfig fragments in kernel/config/snappy were cherry-picked from upstream
3.  A snapcraft.yaml file was added to the root of every branch, ready to snap the kernel using snapcraft

Every one of these branches was tested with the aforementioned snapcraft.yaml, and it produced a working x86_64 kernel capable of booting a recent ubuntu-core amd64 image.

People enabling new hardware can base their derivative work out of these branches following these steps:

*   Clone the â€˜ubuntu-coreâ€™ official git tree:

*   git clone git://git.launchpad.net/~p-pisati/ubuntu/+source/linux

*   Pick the kernel version that suit your needs:

*   cd linux && git checkout snappy_v4.4

*   If the target hardware requires any custom patch, apply it now on top of this tree (in case of a big BSP stack, it makes more sense to rebase the BSP on top of this branch)
*   If the target hardware requires any custom kernel configuration, either create a new kconfig fragment in â€˜kernel/configs/snappyâ€™ and adjust the â€˜kdefconfigâ€™ section in snapcraft.yaml Â or apply the missing CONFIG directly in the â€˜kconfigsâ€™ section in snapcraft.yaml
*   If the target hardware uses a different base defconfig then â€˜x86_64_defconfigâ€™, change the first entry in the â€˜kdefconfigâ€™ section in snapcraft.yaml appropriately (e.g. multi_v7_defconfig):

*   kdefconfig: [multi_v7_defconfig, â€¦]

*   Modify the name of the kernel snap in snapcraft.yaml to reflect the target hardware and build the kernel snap:

*   snapcraft -d

*   Or in case of cross-compilation (e.g. armhf):

*   snapcraft -d --target-arch=armhf

### The Kconfig delta

At the time of this writing, here is what the kconfig delta contains - comments inline:

### Generic.config - distilled from debian.master/config/annotations - >= 3.10:

    #LP#1105230 and LP#1385510

    # CONFIG_SOUND_OSS_CORE_PRECLAIM is not set

    # upstart requires DEVTMPFS be enabled and mounted by default.

    CONFIG_DEVTMPFS=y

    CONFIG_DEVTMPFS_MOUNT=y

    # some /dev nodes require POSIX ACLs, like /dev/dsp

    CONFIG_TMPFS_POSIX_ACL=y

    # Ramdisk size should be a minimum of 64M

    CONFIG_BLK_DEV_RAM=y

    CONFIG_BLK_DEV_RAM_SIZE=65536

    # LVM requires dm_mod built in to activate correctly (LP: #560717)

    CONFIG_MD=y

    CONFIG_BLK_DEV_DM=y

    # sysfs: ensure all DEPRECATED items are off

    # CONFIG_SYSFS_DEPRECATED is not set

    # automatically add local version will cause packaging failure

    CONFIG_LOCALVERSION=""

    # Ensure IPV6 is y, if this is a module we get a module load for

    # every ipv6 packet, bad.

    CONFIG_IPV6=y

    # Ensure ECRYPT_FS is y as it cannot be autoloaded and it has complex

    # dependancies which can pull it =m at a whim.

    CONFIG_ECRYPT_FS=y

    # Required if /init is a shell script.

    CONFIG_BINFMT_SCRIPT=y

    # Newer udevs don't handle firmware loading, and having the userspace

    # fallback enabled in the kernel just results in big delays if we do

    # fall back.

    # See LP:1398458

    # CONFIG_FW_LOADER_USER_HELPER_FALLBACK is not set

    CONFIG_CRASH_DUMP=y

    CONFIG_RTC_DRV_CMOS=m

    CONFIG_NVRAM=m

    CONFIG_INPUT_UINPUT=y

    # needed by dbus

    CONFIG_SYSVIPC=y

    CONFIG_SYSVIPC_SYSCTL=y

    Security.config - distilled from debian.master/config/annotations -

    # CONFIG_COMPAT_BRK is not set

    # CONFIG_DEVKMEM is not set

    CONFIG_LSM_MMAP_MIN_ADDR=0

    CONFIG_SECURITY=y

    CONFIG_SECURITY_APPARMOR=y

    CONFIG_SECURITY_SELINUX=y

    CONFIG_SECURITY_SMACK=y

    CONFIG_SECURITY_YAMA=y

    CONFIG_SYN_COOKIES=y

    CONFIG_DEFAULT_SECURITY_APPARMOR=y

    CONFIG_SECCOMP=y

    CONFIG_SECCOMP_FILTER=y

    CONFIG_CC_STACKPROTECTOR=y

    CONFIG_CC_STACKPROTECTOR_REGULAR=y

    CONFIG_DEBUG_RODATA=y

    CONFIG_DEBUG_SET_MODULE_RONX=y

    CONFIG_STRICT_DEVMEM=y

    # CONFIG_COMPAT_VDSO is not set

    # CONFIG_ACPI_CUSTOM_METHOD is not set

    CONFIG_DEFAULT_MMAP_MIN_ADDR=32768

    Systemd.config - distilled from systemd/README - >= 3.10 for the mandatory options:

    # for more info, see REQUIREMENTS in systemd/README:

    CONFIG_DEVTMPFS=y

    CONFIG_CGROUPS=y

    CONFIG_INOTIFY_USER=y

    CONFIG_SIGNALFD=y

    CONFIG_TIMERFD=y

    CONFIG_EPOLL=y

    CONFIG_NET=y

    CONFIG_SYSFS=y

    CONFIG_PROC_FS=y

    CONFIG_FHANDLE=y

    # CONFIG_SYSFS_DEPRECATED is not set

    CONFIG_UEVENT_HELPER_PATH=""

    # CONFIG_FW_LOADER_USER_HELPER is not set

    CONFIG_DMIID=y

    CONFIG_BLK_DEV_BSG=y

    CONFIG_NET_NS=y

    CONFIG_DEVPTS_MULTIPLE_INSTANCES=y

    # Optional but strongly recommended:

    CONFIG_IPV6=y

    CONFIG_AUTOFS4_FS=y

    CONFIG_TMPFS_POSIX_ACL=y

    CONFIG_TMPFS_XATTR=y

    CONFIG_SECCOMP=y

    CONFIG_CGROUP_SCHED=y

    CONFIG_FAIR_GROUP_SCHED=y

    CONFIG_CFS_BANDWIDTH=y

    CONFIG_SCHEDSTATS=y

    CONFIG_SCHED_DEBUG=y

    CONFIG_EFIVAR_FS=y

    CONFIG_EFI_PARTITION=y

    # CONFIG_AUDIT is not set

    Snappy.config:

    CONFIG_RD_BZIP2=y

    CONFIG_RD_LZMA=y

    CONFIG_RD_XZ=y

    CONFIG_REGULATOR=y

    CONFIG_RFKILL=y

    CONFIG_RFKILL_INPUT=y

    CONFIG_RFKILL_REGULATOR=m

    CONFIG_RFKILL_GPIO=m

    CONFIG_RAW_DRIVER=m

    CONFIG_FANOTIFY=y

    CONFIG_AUTOFS4_FS=y

    # CONFIG_USB_FUNCTIONFS is not set

    # CONFIG_USB_ZERO is not set

    # CONFIG_USB_MASS_STORAGE is not set

    # CONFIG_USB_G_MULTI_CDC is not set

    CONFIG_CONFIGFS_FS=y

    CONFIG_KEYS=y

    CONFIG_ENCRYPTED_KEYS=y

    CONFIG_SQUASHFS=m

    CONFIG_SQUASHFS_FILE_DIRECT=y

    CONFIG_SQUASHFS_DECOMP_MULTI_PERCPU=y

    CONFIG_SQUASHFS_XATTR=y

    CONFIG_SQUASHFS_ZLIB=y

    CONFIG_SQUASHFS_LZ4=y

    CONFIG_SQUASHFS_LZO=y

    CONFIG_SQUASHFS_XZ=y

Containers.config - distilled from check-config.sh:

    # https://github.com/docker/docker/blob/master/contrib/check-config.sh

    CONFIG_NAMESPACES=y

    CONFIG_NET_NS=y

    CONFIG_PID_NS=y

    CONFIG_IPC_NS=y

    CONFIG_UTS_NS=y

    CONFIG_DEVPTS_MULTIPLE_INSTANCES=y

    CONFIG_CGROUPS=y

    CONFIG_CGROUP_CPUACCT=y

    CONFIG_CGROUP_DEVICE=y

    CONFIG_CGROUP_FREEZER=y

    CONFIG_CGROUP_SCHED=y

    CONFIG_CPUSETS=y

    CONFIG_MEMCG=y

    CONFIG_KEYS=y

    CONFIG_MACVLAN=m

    CONFIG_VETH=m

    CONFIG_BRIDGE=m

    CONFIG_BRIDGE_NETFILTER=m

    CONFIG_NF_NAT_IPV4=m

    CONFIG_IP_NF_FILTER=m

    CONFIG_IP_NF_TARGET_MASQUERADE=m

    CONFIG_NETFILTER_XT_MATCH_ADDRTYPE=m

    CONFIG_NETFILTER_XT_MATCH_CONNTRACK=m

    CONFIG_NF_NAT=m

    CONFIG_NF_NAT_NEEDED=y

    CONFIG_POSIX_MQUEUE=y

    # optional features

    CONFIG_USER_NS=y

    CONFIG_SECCOMP=y

    CONFIG_CGROUP_PIDS=y

    CONFIG_MEMCG_KMEM=y

    CONFIG_MEMCG_SWAP=y

    CONFIG_BLK_CGROUP=y

    CONFIG_BLK_DEV_THROTTLING=y

    CONFIG_IOSCHED_CFQ=y

    CONFIG_CFQ_GROUP_IOSCHED=y

    CONFIG_CGROUP_PERF=y

    CONFIG_CGROUP_HUGETLB=y

    CONFIG_NET_CLS_CGROUP=m

    CONFIG_CGROUP_NET_PRIO=y

    CONFIG_CFS_BANDWIDTH=y

    CONFIG_FAIR_GROUP_SCHED=y

    CONFIG_RT_GROUP_SCHED=m

    CONFIG_EXT4_FS=y

    CONFIG_EXT4_FS_POSIX_ACL=y

    CONFIG_EXT4_FS_SECURITY=y

    CONFIG_VXLAN=m

    CONFIG_AUFS_FS=m

    CONFIG_BTRFS_FS=m

    CONFIG_BLK_DEV_DM=y

    CONFIG_DM_THIN_PROVISIONING=m

    CONFIG_OVERLAY_FS=m

    CONFIG_OVERLAY_FS_V1=y

### AppArmor

*   John Johansen has AppArmor backport branches for v3.0 through v4.5\. They are getting stale and are missing a number of fixes. They are also missing some new features but Snappy should not require any of them.

*   [http://kernel.ubuntu.com/git/jj/linux-apparmor-backports/](http://kernel.ubuntu.com/git/jj/linux-apparmor-backports/&sa=D&ust=1474626830473000&usg=AFQjCNEOJFiNho0hRGobB-EEpusEA_R9cg)

*   Ppisati also has some unmaintained branches:

*   [http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h=snappy_v3.10](http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h%3Dsnappy_v3.10&sa=D&ust=1474626830474000&usg=AFQjCNGhKan2N3Rx8YhLBg6s_K4yGnaX0w)
*   [http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h=snappy_v3.14](http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h%3Dsnappy_v3.14&sa=D&ust=1474626830475000&usg=AFQjCNHQMHNZEqSlFloilSg--jjRR89XnQ)
*   [http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h=snappy_v3.16](http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h%3Dsnappy_v3.16&sa=D&ust=1474626830475000&usg=AFQjCNEwOOgDySUUx6rHg0EzjDCxYn4XLg)
*   [http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h=snappy_v3.18](http://kernel.ubuntu.com/git/ppisati/ubuntu-vivid.git/log/?h%3Dsnappy_v3.18&sa=D&ust=1474626830476000&usg=AFQjCNHi1qimDgOv1czLYnqwAXauXOqKgA)

### Seccomp

*   CONFIG_SECCOMP_FILTER && CONFIG_HAVE_ARCH_SECCOMP_FILTER
*   [Part of the Linux kernel since >= 3.5](http://kernelnewbies.org/Linux_3.5%23head-c48d6a7a26b6aae95139358285eee012d6212b9e&sa=D&ust=1474626830477000&usg=AFQjCNGoOP2qrlRqQxXfnpe5H2bME6kd5A)Â

### Cgroups

*   Systemd requires CONFIG_CGROUPS

*   Optionally: CONFIG_CGROUP_SCHED and CONFIG_FAIR_GROUP_SCHED (>= 3.9)

*   Lxc / Docker requires CONFIG_CGROUPS, CONFIG_CGROUP_CPUACCT, CONFIG_CGROUP_DEVICE, CONFIG_CGROUP_FREEZER, CONFIG_CGROUP_SCHED and CONFIG_MEMCG (>= 3.6)

*   Optionally: CONFIG_CGROUP_PIDS (4.3), CONFIG_MEMCG_KMEM (3.9), CONFIG_MEMCG_SWAP (3.9), CONFIG_MEMCG_SWAP_ENABLED (3.9), CONFIG_BLK_CGROUP (3.3), CONFIG_CGROUP_PERF (2.6.38), CONFIG_CGROUP_HUGETLB(3.9, became lockless in 3.19), CONFIG_NET_CLS_CGROUP (refactored in 3.13) and CONFIG_CGROUP_NET_PRIO (firstly introduced in 3.2 as NETPRIO_CGROUP, renamed and refactored around 3.13/3.14)

*   Ubuntu-core requires CONFIG_CGROUP_DEVICE (for device cgroup)

### Namespace

*   Systemd requires CONFIG_NAMESPACES, CONFIG_NET_NS (>= 3.4)
*   Lxc / Docker requires CONFIG_NAMESPACES, CONFIG_NET_NS, CONFIG_PID_NS, CONFIG_IPC_NS, CONFIG_UTS_NS (3.8 - introduced in 3.8 these features were constantly under refactoring and development until 3.14)

*   Optionally: CONFIG_USER_NS (>= 3.14)

*   Ubuntu-core requires CONFIG_NAMESPACES (for mount namespace for private /tmp)

### Bindmounts

*   Itâ€™s a 2.4.x feature.

### /dev/pts multi-instance support

*   Can this be covered by the Kernel Config requirement of CONFIG_DEVPTS_MULTIPLE_INSTANCES=y
*   Part of the Linux kernel since 2.6.38

### AUFS

*   The Ubuntu kernel team has an update script which could be used if on a 4.x or newer kernel:

*   [http://kernel.ubuntu.com/git/ubuntu/ubuntu-xenial.git/commit/?id=7967fb9f082040ad4e9c3001e676892e3c026d73](http://kernel.ubuntu.com/git/ubuntu/ubuntu-xenial.git/commit/?id%3D7967fb9f082040ad4e9c3001e676892e3c026d73&sa=D&ust=1474626830486000&usg=AFQjCNEHV8KqY5HZaA_r3XA_mTNfWGYYWA)

### OverlayFS

*   Note: This always seems to enter the conversation, but it is not currently required for Snappy.
*   Merged in 3.18, thereâ€™s a tree with backports of WIP patches for previous kernels with varying degree of features and support: Â [https://github.com/adilinden/overlayfs-patches](https://github.com/adilinden/overlayfs-patches&sa=D&ust=1474626830487000&usg=AFQjCNGxHPVIUfAOc2NOEzkD_rVKE8OvVA)

### Networking

*   Are there any specific kernel options about networking we need?
*   Lxc / Docker requires CONFIG_MACVLAN (3.9), CONFIG_VETH, CONFIG_BRIDGE, CONFIG_BRIDGE_NETFILTER, CONFIG_NF_NAT_IPV4, CONFIG_IP_NF_FILTER, CONFIG_IP_NF_TARGET_MASQUERADE, CONFIG_NETFILTER_XT_MATCH_ADDRTYPE, CONFIG_NETFILTER_XT_MATCH_CONNTRACK, CONFIG_NF_NAT, CONFIG_NF_NAT_NEEDED (>= 3.9)

*   Optionally: CONFIG_VXLAN (>= 3.16, left EXPERIMENTAL at 3.9)

### Misc

*   The fix for ["net_admin apparmor denial when using Go"](https://bugs.launchpad.net/snappy/%2Bbug/1465724&sa=D&ust=1474626830489000&usg=AFQjCNE2W_tGOZmc5yaAH89Y4E77kZMnFA)Â should eventually be included. Fixes have been [proposed upstream](http://thread.gmane.org/gmane.linux.kernel.lsm/27927&sa=D&ust=1474626830490000&usg=AFQjCNHRUAIUm2rLsG6L6RyudaaGgvlfhA).
*   Systemd requires CONFIG_DEVTMPFS, CONFIG_INOTIFY_USER, CONFIG_SIGNALFD, CONFIG_TIMERFD, CONFIG_EPOLL, CONFIG_SYSFS, CONFIG_PROC_FS, CONFIG_FHANDLE, CONFIG_DMIID, CONFIG_BLK_DEV_BSG (>= 3.8)

*   Optionally (but strongly recommended: CONFIG_IPV6, CONFIG_AUTOFS4_FS, CONFIG_TMPFS_XATTR, CONFIG_{TMPFS,EXT4,XFS,BTRFS_FS,...}_POSIX_ACL, CONFIG_CHECKPOINT_RESTORE, CONFIG_EFIVAR_FS, CONFIG_EFI_PARTITION (>= 3.8)

*   Lxc / Docker requires CONFIG_KEYS, CONFIG_POSIX_MQUEUE (>= 3.9)
*   Seccomp logging will need fixes in support of snappy --devmode, but those fixes are not yet available
*   May require [https://git.kernel.org/cgit/linux/kernel/git/tip/tip.git/commit/?h=x86/asm&id=9dea5dc921b5f4045a18c63eb92e84dc274d17eb](https://git.kernel.org/cgit/linux/kernel/git/tip/tip.git/commit/?h%3Dx86/asm%26id%3D9dea5dc921b5f4045a18c63eb92e84dc274d17eb&sa=D&ust=1474626830493000&usg=AFQjCNGl_aeVw5Kg9E9aBpjHl_R_UXSccw)Â for kernels <4.3 so we can continue to block socketcall() in seccomp. [https://bugs.launchpad.net/ubuntu/+source/libseccomp/+bug/1576066](https://bugs.launchpad.net/ubuntu/%2Bsource/libseccomp/%2Bbug/1576066&sa=D&ust=1474626830494000&usg=AFQjCNEkL0Ku6LMV7rCrzKCndcarZxQUUA)Â says that all that is needed on 16.04-based OS snap images is the kernel patch and an updated libseccomp. After some testing I (jdstrand) will update if this is all that is required.

Validation

Here we should describe and list what are the things we should test to validate the snappy features above (e.g. app confinement, rollback) have been properly enabled in the kernel. What are the existing tools we have now and what will be needed to develop.

Currently certification team has a checkbox snap, the tutorial can be found at [https://docs.google.com/document/d/1IxLSZkRITzuU--c7H6GSDfKCf4GDemaw5xFgzDDhExg/edit](https://docs.google.com/document/d/1IxLSZkRITzuU--c7H6GSDfKCf4GDemaw5xFgzDDhExg/edit&sa=D&ust=1474626830496000&usg=AFQjCNG3SkwDPd9AeGuxB1es4GIRRp_E_g). It consists of common snappy commands and usage. Some of the tests are manual.

Ubuntu Core QA team also has integration tests: [https://github.com/ubuntu-core/snappy/tree/master/integration-tests](https://github.com/ubuntu-core/snappy/tree/master/integration-tests&sa=D&ust=1474626830497000&usg=AFQjCNEsw49aefZho-8OeGsadTK3ezS7RA). Cert team is planning to inherit them into checkbox.
