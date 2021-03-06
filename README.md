# backurne

`backurne` is a handy tool for backuping RBD's image on RBD.\
Yep ! What is better, for backuping a Ceph cluster, than another Ceph cluster ?


It does not do much by itself, though, but orchestrate and relies heavily on other tools.\
It has a fine integration with Proxmox, but is able to backup "plain" (or "raw RBD") cluster as well.

Supported features
---
- **Snapshot-based backup**, with no agent strictly required on the VM.
- Backup inspection and restoration via **command line interface** as well as via **REST API**.
- **Support multiple retention policy** efficiently (both in term of storage and network bandwidth), dynamically configurable per host (proxmox-only) via REST API.
- Auto cleanup : deletion is never generated by a human, thus **no human mistakes**.
- **Compression** and **encryption** "on the wire" for enhanced efficiency and security.
- Peaceful integration with other snapshots (via Proxmox web interface or whatever).
- Multiple cluster support, with mixed type ("proxmox" and "plain").
- A couple of backups can be stored on the live clusters, for faster recovery.
- Optionnal **fsfreeze** support (proxmox-only) via Qemu-quest-agent.
- Backup deactivation via Proxmox's web interface.

- VM tracking, for those who uses a single Proxmox cluster with multiple Ceph backend.

Encryption and compression at rest are also seamlessly supported via Bluestore OSDs (see https://ceph.com/community/new-luminous-bluestore/)

Required packages
---

Core: python3-dateutil, python3-termcolor, python3-prettytable, python3-requests, python3-proxmoxer (from https://github.com/swayf/proxmoxer) \
For mapping (optional): kpartx, rbd-nbd (luminous or later)\
For the REST API: python3-flask, python3-flask-autoindex\
For bash autocompletion: jq


Installation
---

 - Check out the **Authentification** parts.
 - Clone the source, edit the configuration
 - Setup a Ceph cluster, used to store the backups
 - Profit ?

Configuration
---

See [custom.conf.sample](custom.conf.sample)

Authentification, and where should I run what
---

`backurne` interacts with the backup cluster via the `rbd` command line. It must have the required configuration at /etc/ceph/ceph.conf and the needed keyring.\
It is assumed that `backurne` will be run on a Ceph node (perhaps a monitor), but this is not strictly required (those communications will not be encrypted nor compressed).

`backurne` connects to proxmox's cluster via their HTTP API. No data is exchanged via this link, it is purely used for "control" (listing VM, listing disks, fetching informations etc).

`backurne` connects to every "live" Ceph clusters via SSH. For each cluster, it will connect to a single node, always the same, defined in Proxmox (and / or overwritten via the configuration).\
SSH authentification nor authorization is **not** handled by `backurne` in any way.\
It is up to you to configure ssh : either accept or ignore the host keys, place your public key on the required hosts etc.

Command line interface
---

See [cli.md](cli.md)

REST API
---

See [api.md](api.md)

Used technology
---

 - `RBD` is the core technology used by `backurne` : it provides snapshot export, import, diff, mapping etc.
 - `ssh` is used to transfert the snapshots between the live clusters and the backup cluster. `RBD` can be manipulated over TCP/IP, but without encryption nor compression, thus this solution was not kept.
 - `md5sum` (or other, see the configuration) is used to check the consistancy between snapshots. Cryptographic properties of md5 (well ..) are not required, but this tool is the most deployed, thus is the default. Using xxHash or murmur3 is a more efficient.
 - `rbd-nbd` is used to map a specific backup and inspect its content.
 - `kpartx` is used to explode partition tables into multiple block devices.


Note
---
On Proxmox, LXC is not yet supported. Only Qemu so far :/

The project is developed mainly for Debian Stretch and Proxmox, and is used here on these technologies.\
The "plain" feature as well as running `backurne` on other operating system is less tested, and may be less bug-proof.\
Bug report, merge requests and feature requests are welcome : some stuff are not implemented simply because I do not need them, not because it cannot be done nor because I do not want to code them.
