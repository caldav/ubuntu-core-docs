---
title: "Image building"
---
This document will walk you through all the steps to build an image for a device. You will learn how to:

- Create and upload a signature key to the store
- Create a model assertion for your target device
- Compose and build a custom image using `ubuntu-image`

## Your signature key

Before starting with building the image, you need to create a key to sign your future store uploads.

### 1. Create a key

As a first step, you have to generate a key that will be linked to your Ubuntu Store account. To do so, run:

    snap create-key

You can pass an optional name for the key:

    snap create-key foo

This command will ask you for a password to protect the key.

It will take some time, as it's creating a 4096 bit long key and needs some entropy to complete. To speed up the process, you can install the `rng-tools` package beforehand.

Now, you can list your keys with:

    snap keys

### 2. Upload the key to the store

Next, you have to upload it to the store, effectively linking it to your account. During this step, you will be asked to select an existing key and login with your store account credentials.

    $ snapcraft register-key

The key is now registered with the store and you can start the actual image building.

## Image building

An ubuntu-core image is composed of at least three snaps installed together: OS, gadget and kernel.

* The OS snap contains ubuntu-core and will be downloaded during image creation.
* The kernel and gedget snaps can be locally built or pulled from the store.

The composition of your image is described through a "model assertion", a signed JSON document.

### 1. Create a model assertion

To build an image, it is required you have a signed model assertion.

### Example

As an example, here is one for a device based on a Raspberry Pi 3 board. The JSON file is named `pi3-model.json`, with the following content:

    {
      "type": "model",
      "authority-id": "<your account id>",
      "brand-id": "<your account id>",
      "series": "16",
      "model": "my-pi3",
      "architecture": "armhf",
      "gadget": "pi3",
      "kernel": "pi2-kernel",
      "timestamp": "$(date -Iseconds --utc)"
    }

#### Keys description

- `type`: the assertion type you are creating
- `series`: the ubuntu-core series in use
- `model`: a lower-case name for your target device
- The `gadget` and `kernel` values refer to snaps already existing in the store.
- `authority-id`, `brand-id` refer to your store account id. You will find it on [your account page](https://myapps.developer.ubuntu.com/dev/account/), in the `Account-Id` field.

### 2. Sign your model assertion

Now you have to sign the model assertion with a key, by piping your json model through the `snap sign -k <key name>` command and outputing a model file, the actual assertion you will use to build your image.

    cat pi3-model.json | snap sign -k default &> pi3.model

This command will ask you for the password you used on key creation to secure the key.

### 3. Build the image

You can now create your image with the `ubuntu-image` tool.

To install it, run:

    snap install --edge --devmode ubuntu-image

Then, create the image:

    sudo /snap/bin/ubuntu-image -c beta -o pi3-test.img pi3.model

#### Arguments

- `-c`: the channel snaps are downloaded from
- `-o`: the output image file

You can include specific snaps pre-installed by default in the image by using the `--extra-snaps` argument. For example:

    sudo /snap/bin/ubuntu-image -c beta -o pi3-test.img pi3.model --extra-snaps rocketchat-server --extra-snaps nextcloud

### 4. Your image is ready

You can use a tool like `dd` to write the image to an SDcard and boot your device.

    sudo dd if pi3-test.img of=/dev/sdb bs=32M

#### First boot tips

* On the first boot you have to sign in with a valid Store account to make use of the device.
* `console-conf` will download the SSH key registered with your Store account and configure it so you can log into the device via `ssh <account name>@<device address>` without a password.
* There is no default `ubuntu` user on these images, but you can run `sudo passwd <account name>` to set a password in case you need a local console login.

#### Known problems and limitations:

* You need a monitor or a serial cable plugged to the device to be able to go through the first use setup process handled by `console-conf`.
