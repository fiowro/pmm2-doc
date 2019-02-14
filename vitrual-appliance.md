# PMM Server as a Virtual Appliance

Percona provides a *virtual appliance* for running PMM Server in a virtual
machine.  It is distributed as an *Open Virtual Appliance* (OVA) package, which
is a `tar` archive with necessary files that follow the *Open
Virtualization Format* (OVF).  OVF is supported by most popular virtualization
platforms, including:

* [https://www.vmware.com/products/esxi-and-esx.html](VMware - ESXi 6.5)
* [https://www.redhat.com/en/technologies/virtualization](Red Hat Virtualization)
* [https://www.virtualbox.org/](VirtualBox)
* [https://www.xenserver.org/](XenServer)
* [https://www.microsoft.com/en-us/cloud-platform/system-center](Microsoft System Center Virtual Machine Manager)

<details>
  <summary style="font-size:1.25em;"><strong>Supported Platforms for Running the PMM Server Virtual Appliance</strong></summary>

The virtual appliance is ideal for running PMM Server on an enterprise
virtualization platform of your choice. This page explains how to run the
appliance in VirtualBox and VMware Workstation Player. which is a good choice
to experiment with PMM at a smaller scale on a local machine.  Similar
procedure should work for other platforms (including enterprise deployments on
VMware ESXi, for example), but additional steps may be required.

The virtual machine used for the appliance runs CentOS 7.

**Warning:** *The appliance must run in a network with DHCP, which will automatically assign an IP address for it. To assign a static IP manually, you need to acquire the root access as described in [https://www.percona.com/doc/percona-monitoring-and-management/faq.html#pmm-deploying-server-virtual-appliance-root-password-setting](How to set the root password when PMM Server is installed as a virtual appliance). Then, see the documentation for the operating system for further instructions: [https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html](Configuring network interfaces in CentOS).

Instructions for setting up the virtual machine for different platforms:

<details>
  <summary><strong> VirtualBox Using the Command Line </strong></summary>

Instead of using the VirtualBox GUI, you can do everything on the command line. Use the `VBoxManage` command to import, configure, and start the appliance.

The following script imports the PMM Server appliance from `PMM-Server-1.6.0.ova` and configures it to bridge the en0 adapter from the host. Then the script routes console output from the appliance to `/tmp/pmm-server-console.log`. This is done because the script then starts the appliance in headless (without the console) mode.

To get the IP address for accessing PMM, the script waits for 1 minute until the appliance boots up and returns the lines with the IP address from the log file.

```bash
   # Import image
   VBoxManage import pmm-server-|VERSION NUMBER|.ova
   
   # Modify NIC settings if needed
   VBoxManage list bridgedifs
   VBoxManage modifyvm 'PMM Server [VERSION NUMBER]' --nic1 bridged --bridgeadapter1 'en0: Wi-Fi (AirPort)'
   
   # Log console output into file
   VBoxManage modifyvm 'PMM Server [VERSION NUMBER]' --uart1 0x3F8 4 --uartmode1 file /tmp/pmm-server-console.log
   
   # Start instance
   VBoxManage startvm --type headless 'PMM Server [VERSION NUMBER]'
   
   # Wait for 1 minute and get IP address from the log
   sleep 60
   grep cloud-init /tmp/pmm-server-console.log

```

In this script, `[VERSION NUMBER]` is the placeholder of the version of PMM Server that you are installing. By convention **OVA** files start with *pmm-server-* followed by the full version number such as 1.17.0.

To use this script, make sure to replace this placeholder with the the name of the image that you have downloaded from the [https://www.percona.com/downloads/pmm](Download Percona Monitoring and Management) site. This script also assumes that you have changed the working directory (using the cd command, for example) to the directory which contains the downloaded image file.

</details>

<details>
  <summary><strong> VirtualBox Using the GUI </strong></summary>

The following procedure describes how to run the PMM Server appliance using the graphical user interface of VirtualBox:

1. Download the OVA. The latest version is available at the [https://www.percona.com/downloads/pmm](Download Percona Monitoring and Management) site.

2. Import the appliance. For this, open the File menu and click Import Appliance and specify the path to the OVA and click Continue. Then, select Reinitialize the MAC address of all network cards and click Import.

3. Configure network settings to make the appliance accessible from other hosts in your network.

   **Note:** *All database hosts must be in the same network as PMM Server, so do not set the network adapter to NAT.*

  If you are running the appliance on a host with properly configured network settings, select Bridged Adapter in the Network section of the appliance settings.

4. Start the PMM Server appliance and set the root password (required on the first login).

   If it was assigned an IP address on the network by DHCP, the URL for accessing PMM will be printed in the console window.

</details>

<details>
  <summary><strong> VMware Workstation Player </strong></summary>

The following procedure describes how to run the PMM Server appliance using VMware Workstation Player:

1. Download the OVA. The latest version is available at the [https://www.percona.com/downloads/pmm](Download Percona Monitoring and Management) site.

2. Import the appliance.

   1. Open the File menu and click Open.

   2. Specify the path to the OVA and click Continue.

      **Note:** *You may get an error indicating that import failed. Simply click Retry and import should succeed.*

3. Configure network settings to make the appliance accessible from other hosts in your network.

   If you are running the applianoce on a host with properly configured network settings, select **Bridged** in the **Network connection** section of the appliance settings.

4. Start the PMM Server appliance and set the root password (required on the first login)

   If it was assigned an IP address on the network by DHCP, the URL for accessing PMM will be printed in the console window.

5. Set the root password as described in the section.

</details>

</details>


