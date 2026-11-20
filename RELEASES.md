# Download and install CaptainPBX

This repository publishes the CaptainPBX installer. The image installs a ready-to-run appliance: Debian, the control plane, and the media engine. Boot it on an empty disk and it writes CaptainPBX, then starts that disk.

## Current image

| | |
| --- | --- |
| File | [CaptainPBX-iso-v1.0.0.20261105120958.iso](https://sourceforge.net/projects/captainpbx/files/CaptainPBX-iso-v1.0.0.20261105120958.iso/download) |
| Size | 2.0 GB (2,088,960,000 bytes) |
| SHA256 | `a03a4763764dab2de91cd488e09b774b2cb46aa9b4247c62483d2900b247cb26` |

The image is published on SourceForge. Check the checksum after download.

```bash
sha256sum CaptainPBX-iso-v1.0.0.20261105120958.iso
```

## What you need

- A new virtual machine, or a disk you are willing to install onto
- 4 GiB RAM and 20 GiB of disk
- 64-bit guest. The image identifies itself so a wizard can pick a 64-bit Debian guest. Confirm 64-bit if the wizard asks.

Attach the ISO as a custom or virtual optical disk. Do not run an unattended guest installer from the hypervisor. That path ignores this image and installs a stock operating system instead.

## Install

1. Boot from the ISO.
2. Leave the menu alone. **Install CaptainPBX** starts after 5 seconds.
3. Wait while the disk is written. First boot continues on its own.
4. Eject or detach the ISO before the next power cycle, so the machine boots the hard disk.
5. Open `https://<ip>/` and finish the setup wizard.

What the installer does:

- **Empty disk:** writes CaptainPBX, then starts that disk. It does not ask you to partition.
- **Disk that already has a Linux root:** starts that disk. It does not wipe it. Detach the ISO and power cycle so the machine boots the hard disk directly.

The same steps apply on a cloud custom ISO and on a local hypervisor.

## First login

These passwords exist only until the setup wizard replaces the web login.

| Surface | User | Password |
| --- | --- | --- |
| Console or SSH | `root` | `captain` |
| Console or SSH | `captain` | `captain` |
| Web `https://<ip>/` | `captain` | `Ayeayecaptain` |

After the wizard, sign in with the account you created. The appliance opens SSH (22), HTTP and HTTPS (80/443), SIP (5060/5061), and RTP (10000–20000).

## What this image is

This is the installable CaptainPBX image for this tree. It is free to use as provided. It is not a license to rebrand the product. See [LICENSE](LICENSE).
