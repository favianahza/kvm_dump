# KVM Prerequisites & Pre-Checks

Before installing and using KVM, it's essential to ensure your system meets the necessary requirements and to perform some preliminary checks.

-----

### **Hardware and Software Requirements**

To run KVM, your host machine must have the following:

  * **A processor with virtualization support**. This means an Intel processor with Intel Virtualization Technology (VT-x) or an AMD processor with AMD-V support. These extensions provide the hardware-level instructions needed to run virtual machines efficiently.
  * **A Linux operating system with a minimum kernel version of 2.6.20**. KVM was first integrated into the mainline Linux kernel with this version.
  * **Superuser (root) privileges**. Administrative access is required to install KVM packages and manage the hypervisor and its virtual machines.
  * **An ISO image or installation media for the guest operating system**. You will need this to install an OS onto your new virtual machine.

-----

### **Pre-Installation Checks**

Perform these checks from the command line to verify that your system is ready.

#### **1. Verify Processor Virtualization Support**

First, confirm that the **processor virtualization extensions are enabled** in your system's BIOS/UEFI. Then, run the following command to check if the CPU flags are visible to the operating system:

```bash
# grep -E 'svm|vmx' /proc/cpuinfo
```

  * **`vmx`** (Virtual Machine eXtensions) in the output indicates an **Intel** processor with virtualization support.
  * **`svm`** (Secure Virtual Machine) in the output indicates an **AMD** processor with virtualization support.

If this command produces no output, you may need to enable the virtualization settings in your computer's BIOS or UEFI.

#### **2. Check Kernel Version**

You can **see the kernel version** to ensure it is 2.6.20 or higher with this command:

```bash
# uname -a
```

This command prints detailed system information, including the kernel version, which will confirm if your OS is compatible.

#### **3. Update Your System**

Ensure your **host system is up to date** with the latest security patches and software updates. On Debian-based systems like Ubuntu, you can use:

```bash
# apt update
```

Keeping your system updated is a crucial security practice and ensures you have the latest stable packages for KVM and its dependencies. For other distributions like CentOS or Fedora, you would use `yum update` or `dnf update`.