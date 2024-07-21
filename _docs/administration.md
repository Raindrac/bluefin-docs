---
layout: page
title: Administrator's Guide
permalink: /administration
---

<div data-theme-toc="true"> </div>

## Day to Day Operation

Bluefin and Aurora are designed to be installed for the life of the hardware without reinstallation. Unlike traditional operating systems the image is always pristine and "clean", making upgrades less problematic. Updates are automatic and silent by default. 

## Secure Boot

Secure Boot is supported by default on our systems, providing an additional layer of security. After the first installation, you will be prompted to enroll the secure boot key in the BIOS.

Enter the password `ublue-os` when prompted to enroll our key.

If this step is not completed during the initial setup, you can manually enroll the key by running the following command in the terminal:

    ujust enroll-secure-boot-key

Secure boot is supported with our custom key. The pub key can be found in the root of the bazzite repository [here](https://github.com/ublue-os/bazzite/blob/main/secure_boot.der). If you'd like to enroll this key prior to installation or rebase, download the key and run the following:

```bash
sudo mokutil --timeout -1
sudo mokutil --import secure_boot.der
```

### Note:
If you encounter an issue with a password being recognized as incorrect, try using the `-` key on the numpad instead.

## Installing Applications

### Graphical Applications

Use the GNOME Software Center or KDE Discover applications to [install applications from Flathub](https://flathub.org/). Unlike stock Fedora, system updates and upgrades are not handled by this application, it's scope has been reduced to only install Flatpaks from Flathub. The [Warehouse](https://flathub.org/apps/io.github.flattool.Warehouse) tool is included for management.

![GNOME Software Center](upload://fsykftNe5xGcslvq4pETWwlVIA1.png)
![Discover|659x500](upload://ktGkdD7dmJEDfOipPNWaGQLSUAE.png)

### Command Line Applications

The [brew](https://brew.sh/) application is the package manager used for installing command line applications in Bluefin and Aurora. 

- [Homebrew Documentation](https://docs.brew.sh/)
- [Homebrew Packages](https://formulae.brew.sh/)
- [Cheatsheet](https://devhints.io/homebrew)

## System Updates

Bluefin and Aurora are designed to be "hands off". System updates apply weekly, and Flatpaks update twice a day [in the background](https://github.com/ublue-os/config/tree/main/files/usr/lib/systemd). Updates are applied when the system reboots. Therefore, it is recommended to routinely power off your device when it's not being used to ensure kernel updates are being applied. Application updates (like the browser) happen independently of this and don't require a reboot.

See [configuration of updates](https://universal-blue.discourse.group/docs?topic=80) if you want to change the default behavior.

Machine firmware updates are provided through the standard Software Center:

![Firmware updates](upload://cFCaYQvGAZGsNbKZDJeq6maBDFW.png)

- See also: https://universal-blue.discourse.group/docs?topic=513

### Upgrades and Throttle Settings

Bluefin publishes images for the current and last stable version of Fedora. This is to give users maximum flexibility by allowing them to rebase to the version they want. You can choose from three rolling tags, or lock to a specific version of Fedora.  

| | `gts` (default) | `stable` | `latest` |
|---|---|---|---|
| Fedora Version: | 39 | 40 | 40 | 
| GNOME Version: | 45 | 46 | 46 |
| Target User: | Most users | Enthusiasts | Advanced users | 
| System Updates: | Weekly | Weekly | Daily | 
| Application Updates: | Twice a Day | Twice a Day | Twice a Day | 
| Kernel: | Gated | Gated | Upstream | 

- `gts`: This is the default image and is always aliased to the previous stable version of Fedora. It targets the majority of users. 
- `stable`: This is for enthusiasts who want the latest version of the GNOME and Fedora. It is always aliased to the current version of Fedora but follows the Fedora CoreOS release schedule and not the Fedora Silverblue release schedule.
- `latest`: For users who want the very latest Fedora has to offer, daily updates, full open throttle. :smile: 

The major difference between `latest` and `stable` is when they update. `latest` will upgrade to the next major Fedora release as soon as it is available and it builds daily. `stable` will upgrade when CoreOS does it's userpace upgrade, which is usually a few weeks afterwards, and only builds weekly. 

#### Gated Kernel

The `gts` and `stable` tags feature a gated kernel. This kernel follows the same version as the [Fedora CoreOS stable channel](https://fedoraproject.org/coreos/release-notes?arch=x86_64&stream=stable), which is a slower cadence than default Fedora Silverblue.

#### Asus and Surface Devices

Asus and Surface devices use their own dedicated images and only follow the `:latest` tag. This is to ensure proper support for those devices.

#### Switching between channels

Use the `ujust rebase-helper` command to select rebase and select a specific channel: 

![image|690x308, 50%](upload://6V1oxfZ0qfB7VPaMFeqzckZFMc6.jpeg)

Or select `date` and choose an older image. 

![image|690x415, 50%](upload://tSQTtHoKh8wKNRiWgUTftYh61Xm.png)


#### Switching between tags manually
<summary>Here are the manual commands with rpm-ostree, we recommend becoming familiar with them if you find yourself rebasing often:</summary>
<details>
Before changing a channel it is recommended to remove any locally layered packages: 

```bash
rpm-ostree reset
```

Then run a status:

```bash
rpm-ostree status
```

and look for the image you are on, look for a terribly long line like this: `ostree-image-signed:docker://ghcr.io/ublue-os/bluefin:gts`

The `ghcr.io/ublue-os/bluefin:gts` is the important part, with `bluefin` being the image name, and the `:latest` being the image tag. That is the image you are currently on. Look for `:gts`, `:stable`, `:latest`, or in certain cases the version like `:39` or `:40`. Use the `bootc switch` command to move to a newer or older version:

### Examples

In this example we're rebasing to `:stable`, which is the latest stable release of Fedora (currently 40): 

```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bluefin:stable
```
To always be on the `:gts` (default) release:

```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bluefin:gts
```
Explicit version tags of the Fedora release are available for users who wish to handle their upgrade cycle manually:

```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bluefin:40
```
Additionally rebasing to a specific date tag is encouraged if you need to "pin" to a specific day or version:

```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bluefin:38-20231101
```

If you use an nvidia machine, remember that the `-nvidia` is important! (This is why it's important to note the image name when you ran that previous status command:

```bash
rpm-ostree rebase ostree-image-signed:docker://ghcr.io/ublue-os/bluefin-nvidia:stable
```

Check the [Fedora Silverblue User Guide](https://docs.fedoraproject.org/en-US/fedora-silverblue/) for more information.
</details>

## Overwriting System Defaults

Most Bluefin and Aurora system defaults are shipped on the base image along with Fedora configuration in `/usr/etc`. Most of these can be overriden by placing a file in `/etc`. 

For example, the Distrobox configuration is in `/usr/etc/distrobox/distrobox.ini`. Your customization options will be placed in `/etc/distrobox/distrobox.ini`. This is useful for situations where you need a copy of the original file for reference. 

Check the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) for more information on configuration options, in particular `~/.local` and `~/.config`. 

## Power Management

By default Bluefin will switch the power profile depending on if you're on AC or battery. You can adjust this setting by going to the logo menu -> extensions manager, and then selecting what you want.

![image|564x500](upload://8A27h2nogJfwsudfvwWdeSQzcyS.png)

You can also turn off the extensions entirely with this command:

- To turn it off: `ujust configure-auto-power-profile disable`
- To turn it on: `ujust configure-auto-power-profile enable`

## Remote Management

### Note

> This feature is incomplete and needs contributors to make it a reality

Bluefin and Aurora include Cockpit for machine management. We're hoping to include more out-of-the-box management templates, please [check this issue](https://github.com/ublue-os/bluefin/issues/271) if you're interested in volunteering.

## Verification

These images are signed with sigstore's [cosign](https://docs.sigstore.dev/cosign/overview/). You can verify the signature by downloading the `cosign.pub` key from [this repo](https://github.com/ublue-os/bluefin) and running the following command:

```bash
cosign verify --key cosign.pub ghcr.io/ublue-os/bluefin
```

## Building Locally (Bluefin Example)

1. Clone this repository and `cd` into the working directory

    ```bash
    git clone https://github.com/ublue-os/bluefin.git
    cd bluefin
    ```

2. Make modifications if desired

3. Build the image (Note that this will download and the entire image)

    ```bash
    podman build . -t bluefin
    ```

4. [Podman push](https://docs.podman.io/en/latest/markdown/podman-push.1.html) to a registry of your choice.

5. Rebase to your image to wherever you pushed it:

    ```bash
    sudo rpm-ostree rebase ostree-image-signed:docker://whatever/bluefin:latest
    ```