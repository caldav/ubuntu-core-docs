---
title: "Image building"
---
This document will walk you through all the steps to build an image for your snappy device.

## Your signature key

A key is needed to sign your store uploads.

### Create a key

As a first step, you have to generate a key pair that will be linked to your Ubuntu Store account.

    $ snap create-key

This will ask you for a password to protect the key and will take some time to finish (it's creating a 4096 bit long key and needs some entropy to complete: you can move the mouse, type on the keyboard, etc.)

You can list your keys with:

    $ snap keys

### Upload the key to the store

Next, you have to upload it to the store, effectively linking it to your account. During this step, you will be asked to login with your store account credentials.

Note that this step requires at least snapcraft 2.17.

    $ snapcraft register-key

The key is now registered with the store and you can start the actual image building.

## Image building

An `ubuntu-core` image is composed of at least three snaps: OS, gadget and kernel. THis composition is declared through a "model assertion", a signed JSON document defining your image.

### Create a model assertion

To build an image, it is required to have a signed model assertion. The model assertion is a JSON document that defines your ubuntu-core image.

### Example

As an example, here is a model assertion for a Raspberry Pi 3.

It is a file named `pi3-model.json`, with the following content:

    {
      "type": "model",
      "authority-id": "<your account id>",
      "brand-id": "<your account id>",
      "series": "16",
      "model": "pi3",
      "architecture": "armhf",
      "gadget": "pi3",
      "kernel": "pi2-kernel",
      "timestamp": "$(date -Iseconds --utc)"
    }

In this example, the `gadget` and `kernel` values refer to snaps already existing in the store.

You have to replace < your account id > with your actual Store account id. You will find it on https://myapps.developer.ubuntu.com/dev/account/ when logged in. Just copy the value of the field `Account-Id`.

### Image signing

Now you have to sign the model assertion with your keys

    $ cat pi3-model.json | snap sign -k default &> pi3.model

This will ask you for the password you used on key creation to secure the key.

And finally you can create your image with the `ubuntu-image` tool:

    $ snap install --edge --devmode ubuntu-image
    $ sudo /snap/bin/ubuntu-image -c beta -o pi3-test.img pi3.model

You can include specific snaps by default in the image by using the `--extra-snaps` argument.

## Tips for first boot

The first boot setup is handled by the `console-conf` tool.

* On the first boot you have to sign in with a valid Store account to make use of the device.
* `console-conf` will download the SSH key registered with your Store account and configure it so you can log into the device via `ssh <account name>@<device address>` without a password.
* There is no default `ubuntu` user on these images, but you can run `sudo passwd <account name>` to set a password in case you need a local console login.

### Known problems and limitations:

* You need a monitor or a serial cable plugged to the device to be able to go through the first use setup process handled by `console-conf`.
* You can’t configure a WiFi connection from the first boot setup inside `console-conf` yet.
