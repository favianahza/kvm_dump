# KVM Installation

-----

## KVM Installation on Debian 

This installation guide uses **Debian 9** as the base operating system.

#### **1. Install KVM Packages**

First, install the necessary packages using the `apt` package manager. This command installs the core KVM components, networking utilities, and the virtualization daemon.

```bash
apt install qemu-kvm bridge-utils libvirt-daemon libvirt-daemon-system
```

  * **`qemu-kvm`**: Provides the backend machine emulation and virtualization.
  * **`bridge-utils`**: Contains tools to create and manage network bridges, allowing VMs to connect to the host's network.
  * **`libvirt-daemon-system`**: The `libvirtd` daemon that manages the virtual machines and hypervisor.

-----

## KVM Installation on CentOS 7 

For **CentOS 7**, the installation process uses the `yum` package manager and includes a different set of packages.

#### **1. Install KVM Packages**

Install the required packages, including graphical management tools and command-line utilities. The `-y` flag automatically confirms the installation.

```bash
yum install virt-install qemu-kvm libvirt libvirt-python \
   libguestfs-tools virt-install virt-manager -y
```

  * **`virt-install`**: A command-line tool for provisioning new virtual machines.
  * **`libguestfs-tools`**: A set of tools for accessing and modifying VM disk images.
  * **`virt-manager`**: A graphical user interface for managing virtual machines.

-----

## Post-Installation Steps

After installing the packages, you must enable and verify the KVM services.

#### **2. Enable the libvirtd Service**

Enable the service to ensure it starts automatically every time the system boots.

```bash
systemctl enable libvirtd
```

#### **3. Check KVM Daemon Status**

Check the KVM daemon status to confirm it is active and running.

  * **For Debian 9:**

    ```bash
    service libvirtd status
    ```

  * **For CentOS 7:**

    ```bash
    systemctl status libvirtd
    ```

#### **4. Verify KVM Module**

Finally, check that the KVM kernel module has been loaded correctly. This command provides details about the `kvm` module if it's active.

```bash
modinfo kvm
```