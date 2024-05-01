---
title: Creating an Encrypted Portable Container for ZeroNet
layout: post
series: ZeroNet
---

I have two Linux machines that I constantly switch between, and I had a problem syncing my [ZeroNet](https://zeronet.dev/)[^1] data between my machines. Additionally, I didn't want to share my data with a third-party server. So, I decided to make a portable USB stick for my ZeroNet that could not only be used with my devices but also with any other device running Linux. This approach also solved another problem I had with ZeroNet, which was its lack of encryption. Now, I could encrypt my USB stick and my ZeroNet data inside it without any complications.

And I thought that sharing the process of doing so would save a lot of time for someone experiencing the same problem. So here's how I set up my portable container:

## 1. Wiping the USB Stick

Always before encrypting any devices, you should securely wipe the device so that the previous data on it becomes unrecoverable in case you lose the device or it gets stolen. To wipe this USB stick, all I did was fill it with zeros until there was no space left on it:

To do so, you first need to identify the device that you want to wipe:

For that, you can run the `lsblk` command which will show all your devices and their names:

```bash
$ lsblk -p
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
...
/dev/sdc      8:32   1  29.3G  0 disk 
```

The output shows me that the USB stick that I'm going to use is located at `/dev/sdc`.

Then, use the `dd` command to read from `/dev/zero` or `/dev/urandom` into it until it's filled with zeros:

```bash
sudo dd if=/dev/zero of=/dev/sdc status=progress bs=128M
```

**Note:** You should replace `/dev/sdc` with your own device.

I recommend reading from `/dev/urandom` as most modern Flash Memory devices use compression features, which can compress a pattern of zeros and prevent the device from getting fully and securely wiped. However, with `/dev/urandom`, you'll get fairly fast-generated pseudo-random data that cannot be compressed, making it the safer approach.

You should wait until you get the "No space left on device" message from the `dd` command. Then, you can proceed to encrypt the device.

## 2. Encrypting the USB Stick

To encrypt the device, I used [LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup)[^2] as I only have Linux machines and I don't need my USB stick to be readable on Windows machines. However, if you want to make it cross-platform, you can use [VeraCrypt](https://veracrypt.de/en/Beginner%27s%20Tutorial.html) instead.

**Note: ** To encrypt the device using the LUKS format, you'll need to install the `cryptsetup` package (usually it is already installed in most Linux distributions).

```bash
sudo cryptsetup -y -v --type luks2 luksFormat /dev/sdc
```

By running the command above, you'll be prompted to accept the risks of overwriting data on your device. Then, you'll be asked for a passphrase, which will be used to decrypt and unlock the USB stick.

After you set up encryption, you need to open the device using `luksOpen` and create a file system on it. For the file system, I've used BTRFS, which is a modern and stable enough filesystem for my purpose. You can use EXT4 as well if you want something older and more stable, but the journaling of EXT4 can cause some additional wear and tear on your device.

```bash
sudo cryptsetup luksOpen /dev/sdc DeviceName
```

After you open and decrypt the device, it will be available at `/dev/mapper/DeviceName`, and you can use this path to create a file system on the device:

```bash
sudo mkfs.btrfs /dev/mapper/DeviceName -L WhateverLabelYouWant -f
```

Now you can mount your encrypted device and put the ZeroNet on it (it will mount your device on `/mnt/zeronet_container`):

```bash
sudo mkdir /mnt/zeronet_container
sudo mount /dev/mapper/DeviceName /mnt/zeronet_container
```

## 3. Setting-Up ZeroNet Bundle

Next, you want to download the ZeroNet Bundle, which includes all the executables inside it and doesn't need any additional packages to work. For that, I've downloaded the ZeroNetX Bundle, which is an actively maintained fork of ZeroNet:

```bash
cd /mnt/zeronet_container
wget https://github.com/ZeroNetX/ZeroNet/releases/latest/download/ZeroNet-linux.zip
unzip ZeroNet-linux.zip
cd ZeroNet-linux
```

Then, create a `zeronet.conf` file in the `Zeronet-linux` directory to enable TOR support:

```toml
force_encryption = True
tor_proxy = 127.0.0.1:9150
tor_controller = 127.0.0.1:9151
tor = always
```

**Note:** The ports `9150` and `9151` are default ports for the TOR Browser, which we will include in our container. This way, when you open your Tor Browser to access the ZeroNet, the ZeroNet will use the browser's controller port to communicate with Tor, minimizing the need for setting up Tor and granting access to ZeroNet to use its controller port.

## 4. Setting-Up a Portable TOR Browser

Making a portable Tor Browser is fairly easy. You only need to download the Tor Browser onto your USB stick and extract it there. Every time you want to use it, run it from there.

To download the Tor Browser for Linux, you should go to [https://www.torproject.org/download/](https://www.torproject.org/download/) and after downloading it, extract it into your USB stick.

## 5. Now your Portable ZeroNet is Ready to Use

After you set everything up, you can just plug your USB stick into your machine, open it up, and run the Tor Browser and ZeroNet from there.

Additionally, you can run your ZeroNet with `FireJail` to isolate it from the rest of your system and provide some sandboxing for it.

Another method, if you want to use your ZeroNet on untrusted machines portably, is through using Tails OS. You can download and boot the Tails OS on your USB stick and enable Persisted Encrypted Storage on it. Then, put your ZeroNet Bundle on the Persisted storage inside Tails and boot the device on any machine to use your ZeroNet without leaving any trace on the machine or having to trust the machine (to some extent).

---

My blog and all of its content are available under the `CC by SA 4.0` License on my GitHub. If you notice any problems or have any improvements for the blog or its content, you're always welcome to open a pull request.

[^1]: ZeroNet is a peer-to-peer web-like network, which I have [covered in detail in my blog](/blog/posts/a-very-technical-look-at-zeronet). 
[^2]: LUKS (Linux Unified Key Setup) is a disk encryption specification that provides an easy-to-use, platform-independent method for securing data on storage devices. It allows users to encrypt entire partitions or storage devices, ensuring that data remains protected even if the device is lost or stolen
