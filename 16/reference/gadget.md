---
title: Gadget snap syntax
---
A gadget snap requires two files in the `meta` directory of your snap.

* a `gadget.yaml` file that describes the device partitioning and boot system
* a `snap.yaml` file that describes the gadget snap and how it will be presented in stores

## The gadget.yaml file

Three YAML keys are used to describe your target device:

* `device-tree` (string): the Device Tree Blob path.
* `device-tree-origin` (string): `kernel` or `gadget` (default) depending if the DTB is located in the kernel or in the gadget snap.
* `volumes` (YAML sub-section, required): the volumes layout, where each disk image is represented as a YAML sub-section.

### The `volumes` sub-section

Each volume is described by:

* a name (required)
* a partition structure (required)
* a bootloader definition (`grub`, `u-boot`)
* a partitioning schema eg. `mbr`

#### Example: a raspberry pi 3 gadget.yaml

    device-tree: bcm2710-rpi-3-b.dtb
    volumes:
      pi3:
        schema: mbr
        bootloader: u-boot
        structure:
          - type: 0C
            filesystem: vfat
            filesystem-label: system-boot
            size: 128M
            content:
              - source: boot-assets/
                target: /

Used with the following directory structure:

    .
    ├── boot-assets
    │   ├── (...)
    │   └── bcm2710-rpi-3-b.dtb
    ├── meta
    │   ├── (...)
    │   ├── gadget.yaml
    │   └── snap.yaml
    ├── (...)
    └── README

`bootloader` (string): `grub` or `u-boot`.

This tells snapd whether to create grubenv or uboot.env

`volumes` (YAML sub-section)

Where each volume is a distinct disk image represented by a YAML sub-section, for example:

    volumes:
      pi3:
        schema: mbr
        bootloader: u-boot
        structure:
          - type: 0C
            filesystem: vfat
            filesystem-label: system-boot
            size: 128M
            content:
              - source: boot-assets/
                target: /

Image name (YAML subsection):

    volumes:                      # each volume is a distinct disk image
        name-of-the-image:   # XXX: figure out size limit if we want to write this somewhere (MBR, GPT?)
             - name: sbl1
               type: DEA0BA2C-CBDD-4805-B4F9-F428251C3E98 #
               offset: 512
               data: sbl1.mbn
            - name: foxy
               type: vfat
               size: 1024M
             - name: system-boot # filesystem label
               type: vfat
               size: 512M
               content:
                   - uboot.env
                   - EFI/  # subdirs allowed
             - name: uboot
               type: raw
               data: u-boot.img
               offset: 393216
               offset-write: mbr+30
            - name: foo
              type: raw
              size: 12MB
              content:
                  - data: one.img
                  - data: two.img # if no offset specified, goes immediately after preceding block
                    offset: 1234
            - name: bar
              type: dump
              data: foo.img
              offset: foo+50
           -

        name-of-the-other-image:
             - name: writable
               label: writable
               type: ext4


    Example: grub

    Example: beaglebone
