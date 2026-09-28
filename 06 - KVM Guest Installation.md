# KVM - Guest Installation

## Understanding virt-install and osinfo

  * **`virt-install`**: This is a command-line tool designed to provision new KVM virtual machines. It automates the entire process, from creating the virtual hardware and storage to starting the OS installation, making it perfect for scripting and repeatable deployments.
  * **`osinfo-db`**: This is a database that contains metadata about various operating systems. Tools like `virt-install` use this information to automatically optimize the virtual machine's configuration (e.g., selecting the best virtual hardware like `virtio` devices) for the specific guest OS you are installing.

-----

## Mandatory virt-install Options

While there are many options available, `virt-install` requires a few mandatory arguments to create a VM. If these are not provided, the command will fail.

  * **`--name`**: You must provide a unique name for the virtual machine.
  * **`--ram`**: You must specify the amount of memory (in MiB) to allocate to the VM.
  * **Storage (`--disk` or other)**: You must define at least one storage device for the VM. The simplest way is using `--disk size=X` to have `virt-install` create a new disk image for you.
  * **Installation Source**: You must specify how the operating system will be installed. This can be done with `--location` (for network installs), `--cdrom` (for ISO files), or `--pxe` (for network booting).

Here is a bare-bones example using only the minimum required options. `virt-install` will use defaults for everything else (like 1 vCPU, NAT networking, and a basic graphical console).

```bash
virt-install \
  --name my-basic-vm \
  --ram 2048 \
  --disk size=20 \
  --cdrom /path/to/your/install.iso
```

-----

### Step 1: Install the osinfo Database

Before creating a guest, it's recommended to install the `osinfo` database tools.

```bash
apt install osinfo-db-tools osinfo-db libosinfo-1.0.0
```

-----

### Step 2: Install the Guest OS

You can install a guest using a single `virt-install` command. Below are examples for CentOS, Ubuntu, and Windows Server.

#### CentOS 7 Installation Example (Serial Console)

This command installs CentOS 7 in a headless mode, managed via a text-based serial console.

```bash
virt-install  --network bridge:br0 --name CentOS_7 \
 --os-variant=centos7.0 --ram=512 --vcpus=1  \
 --disk path=/var/lib/libvirt/images/testvm1-os.qcow2,format=qcow2,bus=virtio,size=5 \
  --graphics none  --location=/mnt/hgfs/CentOS-7-x86_64-Minimal-1810.iso \
  --extra-args="console=tty0 console=ttyS0,115200"  --check all=off
```

  * **`--graphics none`**: Specifies a text-only console.
  * **`--extra-args`**: Directs the installer's output to the serial console (`ttyS0`).

-----

## Other OS Installation Examples

### Ubuntu Server 22.04 Installation Example (Graphical)

This example installs Ubuntu Server using a graphical installer, which is common for modern Linux distributions.

```bash
virt-install \
  --name Ubuntu_Server_22.04 \
  --os-variant=ubuntu22.04 \
  --ram=2048 --vcpus=2 \
  --disk path=/var/lib/libvirt/images/ubuntu-22.04.qcow2,format=qcow2,bus=virtio,size=25 \
  --network bridge:br0 \
  --graphics spice \
  --location /path/to/your/ubuntu-22.04-live-server-amd64.iso
```

  * **`--graphics spice`**: Enables a high-performance graphical console that you can connect to with a client like `virt-viewer`.
  * **`--ram=2048 --vcpus=2`**: Allocates more resources suitable for a modern server OS.

### Windows Server 2022 Installation Example (with VirtIO Drivers)

Installing Windows requires an extra step: providing **VirtIO drivers** for the best performance. These drivers allow Windows to properly communicate with KVM's virtual hardware.

```bash
virt-install \
  --name Windows_Server_2022 \
  --os-variant win2k22 \
  --ram=4096 --vcpus=4 \
  --disk path=/var/lib/libvirt/images/ws2022.qcow2,format=qcow2,bus=virtio,size=80 \
  --disk path=/path/to/your/virtio-win.iso,device=cdrom \
  --network bridge:br0,model=virtio \
  --features acpi=on,apic=on,hyperv_mode=on \
  --graphics spice \
  --location /path/to/your/windows_server_2022.iso
```

  * **`--disk path=/path/to/your/virtio-win.iso,device=cdrom`**: This is crucial. It attaches the VirtIO driver ISO as a second CD-ROM. During the Windows installation, you must manually load the `viostor` (for disk) and `NetKVM` (for network) drivers from this disk.
  * **`--features ... hyperv_mode=on`**: Enables Hyper-V enlightenments, which significantly improves performance for Windows guests.
  * **`bus=virtio`** and **`model=virtio`**: Ensures the disk and network devices use the high-performance VirtIO backend, which relies on the drivers from the ISO.