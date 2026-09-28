# KVM Network Configuration

-----

## **Key Networking Terminology 📚**

Understanding these terms is crucial for configuring KVM networking:

  * **libvirt**: This is the software that provides a management layer for KVM. It's an API and daemon that allows you to interact with the hypervisor.
  * **virsh**: This is the primary command-line tool you use to interact with **libvirt**. The commands in this guide are **virsh** commands.
  * **Network Bridge**: A virtual device that connects multiple network interfaces (like your host's physical NIC and your VMs' virtual NICs) so they can communicate as if they were on the same physical network segment.
  * **NAT (Network Address Translation)**: A method that allows VMs to communicate with the external network through the host's IP address. The host "translates" traffic for the VMs, which are on a private subnet. This is simple but means VMs aren't directly accessible from the outside network.
  * **DHCP (Dynamic Host Configuration Protocol)**: A service that automatically assigns IP addresses and other network settings to devices. KVM's default NAT network runs its own DHCP server for VMs.

-----

## **KVM Network Types Explained ↔️**

While the guide below focuses on creating a bridged network, KVM offers several networking models:

  * **Bridged Networking**: This connects your VMs directly to the same network as your host. Each VM gets its own IP address from your main LAN's DHCP server, making it a fully independent device on the network. This is ideal for running servers that need to be accessible.
  * **NAT Forwarding (Default)**: This is the out-of-the-box setup. KVM creates a private virtual network for the VMs and uses NAT to let them access the outside world. The VMs can get out, but external devices can't easily initiate connections back in.
  * **Isolated Network**: This creates a completely private network that only the VMs on that host can access. They can communicate with each other and the host, but have no connection to the external network. This is useful for secure lab environments.
  * **MACVTAP**: A simpler alternative to bridging that directly connects a VM's virtual interface to the host's physical interface. It's efficient but has a key limitation: the host cannot communicate with the guest VM over a MACVTAP connection.

-----

## **Common `virsh` Network Commands ⚙️**

You can manage KVM's virtual networks using the **`virsh`** command-line tool. Here are the main commands you'll need:

  * **`virsh net-list`**: See the currently active virtual networks for guests.
  * **`virsh net-list --all`**: See all available virtual networks, both active and inactive.
  * **`virsh net-edit {interface}`**: Edit the configuration of a virtual network.
  * **`virsh net-dumpxml {interface}`**: View the XML configuration for a specific virtual network.
  * **`virsh net-start {interface}`**: Activate a virtual network.
  * **`virsh net-destroy {interface}`**: Stop a virtual network.
  * **`virsh net-define --file {file}`**: Define a new virtual network from an XML configuration file.

-----

## **Creating a Bridged Network 🌉**

This step-by-step guide shows how to create a bridged network, allowing your VMs to appear on the same network as your host.

#### **1. Edit the Network Interfaces File**

You'll need to create a bridge interface. To do this, edit the `/etc/network/interfaces` file. Your physical network interface must be set to manual.

  * **For DHCP Configuration:**
    Add these lines to the file, replacing `(interface enp0s3 or eth0)` with your actual network interface name.

    ```ini
    # If using DHCP
    auto br0
    iface br0 inet dhcp
        bridge_ports (interface enp0s3 or eth0)
        bridge_stp off
        bridge_maxwait 0
    ```

  * **For Static IP Configuration:**
    Use this configuration if you want to assign a static IP to the bridge.

    ```ini
    # If using Static IP
    auto br0
    iface br0 inet static
        address (IP Address)
        netmask (Subnet Mask)
        bridge_ports (interface enp0s3 or eth0)
        bridge_stp off
        bridge_maxwait 0
    ```

#### **2. Restart Networking Services**

After saving the configuration, restart the networking service and reboot the system to apply the changes.

```bash
systemctl restart network
```

#### **3. Understand the Default NAT Network**

After creating a bridge, KVM by default will still use its NAT Forward network. This network has a virtual DHCP server that assigns an IP address to the Guest Machine. You can view its XML configuration with the `virsh net-dumpxml default` command.

Here is an example of the output:

```xml
<network>
  <name>default</name>
  <uuid>26e19a81-9616-40ca-a7f2-6c529766429f</uuid>
  <forward mode='nat'/>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:13:13:42'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

This command shows the XML markup of the default adapter for a Guest, which is NAT Forward.

#### **4. Create an XML File for the Bridge**

To use a Bridged Adapter for a Guest, you must create an XML file for the interface. Create a file named `bridge.xml` located at `/bridge.xml`.

Here is an example of the XML for the `br0` interface:

```xml
<network>
  <name>br0</name>
  <forward mode="bridge"/>
  <bridge name="br0"/>
</network>
```

#### **5. Define and Start the Bridge Network**

First, make the `br0` interface available to libvirt.

```bash
virsh net-define --file /bridge.xml
```

Next, start the `br0` network and set it to start automatically on boot.

```bash
virsh net-start br0; virsh net-autostart br0;
```