# KVM Storage

-----

## **Understanding Storage Pools and LVM 🧠**

  * **KVM Storage Pool**: This is a location managed by **libvirt** where you store your virtual machine disk images. Instead of just having loose files, a pool provides a structured way for KVM to manage storage. It can be a directory, an LVM Volume Group, an NFS share, or other storage types.
  * **LVM (Logical Volume Manager)**: LVM is a powerful storage management tool in Linux. It adds a flexible layer between your physical hard drives and your file systems.
      * **Physical Volume (PV)**: A physical disk or partition (like `/dev/sda`) that is initialized for LVM use.
      * **Volume Group (VG)**: A pool of storage created by combining one or more PVs. This is the central storage container.
      * **Logical Volume (LV)**: A "virtual" partition that is carved out from a VG. This is what you format with a file system and use. LVs can be easily resized, which is a major advantage.

-----

## **Common KVM Storage Formats 💿**

When you create a virtual machine, its virtual disk is stored as a file. The format of this file determines its features and performance. Here are five formats commonly used in the industry:

  * **qcow2 (QEMU Copy On Write 2)**: This is the most advanced and widely used format for KVM. It supports features like thin provisioning (files start small and grow as needed), AES encryption, zlib compression, and live snapshots, making it incredibly flexible for development and production environments.
  * **raw**: This format is a simple, bit-for-bit image of a disk. It offers the best performance because it has almost no overhead. However, it lacks features like snapshots, and the file size is pre-allocated (e.g., a 100GB disk will immediately consume 100GB of space on the host). It's best for performance-critical VMs where advanced features aren't needed.
  * **VMDK (Virtual Machine Disk)**: The primary format for VMware products (vSphere/ESXi). KVM's ability to read and write to VMDK files is crucial for interoperability and migrating virtual machines between VMware and KVM platforms.
  * **VHDX (Virtual Hard Disk v2)**: The disk format used by Microsoft's Hyper-V hypervisor. Support for VHDX in KVM is essential for environments that run both Windows and Linux-based virtualization and need to move workloads between them.
  * **VDI (Virtual Disk Image)**: The default format for Oracle's VirtualBox. Like VMDK and VHDX, compatibility with VDI allows users to easily migrate virtual machines created in VirtualBox to a KVM-based environment.

-----

## **Examples: Creating Different Storage Formats 🛠️**

You can create virtual disk images in any of these formats using the `qemu-img` command-line tool. The `-f` flag specifies the format, followed by the filename and the virtual disk size.

  * **Creating a `qcow2` Image (Recommended for KVM):**

    ```bash
    qemu-img create -f qcow2 my-vm-disk.qcow2 30G
    ```

    This creates a 30GB virtual disk that will grow in size on the host as data is added.

  * **Creating a `raw` Image (for Performance):**

    ```bash
    qemu-img create -f raw performance-disk.raw 50G
    ```

    This immediately creates a 50GB file on your host filesystem.

  * **Creating a `VMDK` Image (for VMware Compatibility):**

    ```bash
    qemu-img create -f vmdk esxi-compatible.vmdk 40G
    ```

  * **Creating a `VHDX` Image (for Hyper-V Compatibility):**

    ```bash
    qemu-img create -f vhdx hyper-v-compatible.vhdx 60G
    ```

  * **Creating a `VDI` Image (for VirtualBox Compatibility):**

    ```bash
    qemu-img create -f vdi virtualbox-compatible.vdi 20G
    ```

-----

## **Key LVM and Virsh Commands ⚙️**

Here are the main commands for creating an LVM-based storage pool:

  * `pvcreate {Drive}`: Defines a drive as a Physical Volume (PV).
  * `vgcreate {vg_name} {Drive}`: Creates a Volume Group (VG) from a PV.
  * `lvcreate -n {lv_name} -L {lv_size} {vg_name}`: Creates a Logical Volume (LV) within a VG.
  * `mkfs -t {filesystem_type} {LV Drive}`: Formats the LV with a specified filesystem.
  * `virsh pool-define-as {name} --type {type} --target {target_filesystem}`: Defines a new storage pool in KVM.
  * `virsh pool-list`: Lists existing storage pools.
  * `virsh pool-start`: Starts a defined pool.
  * `virsh pool-autostart`: Configures a pool to start automatically on boot.

-----

## **Step-by-Step: Creating the Storage Pool 🛠️**

This scenario demonstrates creating a partition for a guest VM using an LVM Partition on `/dev/sda`.

#### **1. Create the Physical Volume (PV)**

First, you must assign the physical disk `/dev/sda` to be a Physical Volume.

```bash
# pvcreate /dev/sda
```

#### **2. Create the Volume Group (VG)**

Next, create a Volume Group named `guest` from the PV you just created.

```bash
# vgcreate guest /dev/sda
```

#### **3. Create the Logical Volume (LV)**

Now, create a 4GB Logical Volume named `ubuntu_g` inside the `guest` Volume Group.

```bash
# lvcreate -n ubuntu_g -L 4000 guest
```

#### **4. Format the Logical Volume**

Format the new LV with the `ext4` filesystem.

```bash
# mkfs -t ext4 /dev/mapper/guest-ubuntu_g
```

#### **5. Configure Automatic Mounting**

To make the filesystem that was just created automatically mount to `/var/lib/libvirt/images`, add an entry to `/etc/fstab`.

```bash
# echo "/dev/mapper/guest-ubuntu_g  /var/lib/libvirt/images   ext4  defaults   0   0" >> /etc/fstab
```

#### **6. Mount the Filesystem**

Mount all filesystems listed in `/etc/fstab` to apply the changes immediately.

```bash
# mount -a
```

#### **7. Define the KVM Storage Pool**

Finally, create the **Storage Pool** from the Logical Volume that was created. This command defines a directory-based pool named `ubuntu_pool` that points to the mounted LV.

```bash
# virsh pool-define-as ubuntu_pool --type dir --target /var/lib/libvirt/images
```

After defining the pool, don't forget to start it and set it to autostart using the `virsh pool-start` and `virsh pool-autostart` commands.