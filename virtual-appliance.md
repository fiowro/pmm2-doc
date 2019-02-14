# PMM Server as a Virtual Appliance

Percona provides a *virtual appliance* for running PMM Server in a virtual
machine.  It is distributed as an *Open Virtual Appliance* (OVA) package, which
is a `tar` archive with necessary files that follow the *Open
Virtualization Format* (OVF).  OVF is supported by most popular virtualization
platforms, including:

* [VMware - ESXi 6.5](https://www.vmware.com/products/esxi-and-esx.html)
* [Red Hat Virtualization](https://www.redhat.com/en/technologies/virtualization)
* [VirtualBox](https://www.virtualbox.org/)
* [XenServer](https://www.xenserver.org/)
* [Microsoft System Center Virtual Machine Manager](https://www.microsoft.com/en-us/cloud-platform/system-center)

<details>
  <summary style="font-size:1.25em;"><strong>Supported Platforms for Running the PMM Server Virtual Appliance</strong></summary>

The virtual appliance is ideal for running PMM Server on an enterprise
virtualization platform of your choice. This page explains how to run the
appliance in VirtualBox and VMware Workstation Player. which is a good choice
to experiment with PMM at a smaller scale on a local machine.  Similar
procedure should work for other platforms (including enterprise deployments on
VMware ESXi, for example), but additional steps may be required.

The virtual machine used for the appliance runs CentOS 7.

**Warning:** *The appliance must run in a network with DHCP, which will automatically assign an IP address for it. To assign a static IP manually, you need to acquire the root access as described in [How to set the root password when PMM Server is installed as a virtual appliance](https://www.percona.com/doc/percona-monitoring-and-management/faq.html#pmm-deploying-server-virtual-appliance-root-password-setting). Then, see the documentation for the operating system for further instructions: [Configuring network interfaces in CentOS](https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html).

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

To use this script, make sure to replace this placeholder with the the name of the image that you have downloaded from the [Download Percona Monitoring and Management](https://www.percona.com/downloads/pmm) site. This script also assumes that you have changed the working directory (using the cd command, for example) to the directory which contains the downloaded image file.

</details>

<details>
  <summary><strong> VirtualBox Using the GUI </strong></summary>

The following procedure describes how to run the PMM Server appliance using the graphical user interface of VirtualBox:

1. Download the OVA. The latest version is available at the [Download Percona Monitoring and Management](https://www.percona.com/downloads/pmm) site.

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

1. Download the OVA. The latest version is available at the [Download Percona Monitoring and Management](https://www.percona.com/downloads/pmm) site.

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

<details>
  <summary style="font-size:1.25em;"><strong>Identifying PMM Server IP Address</strong></summary>

When run PMM Server as virtual appliance, The IP address of your PMM Server
appears at the top of the screen above the login prompt. Use this address to
acces the web interface of PMM Server.

![alt text](https://www.percona.com/doc/percona-monitoring-and-management/_images/command-line.login.1.png "The IP address appears above the login prompt")

PMM Server uses DHCP for security reasons, and thus you need to check the PMM
Server console in order to identify the address.  If you require configuration
of a static IP address, see [Configuring network interfaces in CentOS](https://www.centos.org/docs/5/html/Deployment_Guide-en-US/s1-networkscripts-interfaces.html).

</details>

<details>
  <summary style="font-size:1.25em;"><strong>Accessing PMM Server</strong></summary>

To run the PMM Server, start the virtual machine and open in your browser the
URL that appears at the top of the terminal when you are logging in to the
virtual machine.

![alt text](https://www.percona.com/doc/percona-monitoring-and-management/_images/command-line.login.1.png "The URL to open in the browser appears at the top of the terminal when running the virtual machine")

If you run PMM Server in your browser for the first time, you are requested to
supply the user and a new password. Optionally, you may also provide your SSH
public key.

![alt text](https://www.percona.com/doc/percona-monitoring-and-management/_images/pmm.server.password-change.png "Set the user and password to access the PMM Server web interface")

Click Submit and enter your user name and password in the dialog window
that pops up. The PMM Server is now ready and the home page opens.

![alt text](https://www.percona.com/doc/percona-monitoring-and-management/_images/pmm.home-page.png "PMM Server home page")

You are creating a username and password that will be used for two purposes:

1. authentication as a user to PMM - this will be the credentials you need in order
   to log in to PMM.
2. authentication between PMM Server and PMM Clients - you will
   re-use these credentials when configuring pmm-client for the first time on a
   server, for example:

   Run this command as root or by using the **sudo** command

```bash
   $ pmm-admin config --username= --password= --server=1.2.3.4
```

</details>

<details>
  <summary style="font-size:1.25em;"><strong>Accessing the Virtual Machine</strong></summary>

To access the VM with the *PMM Server* appliance via SSH,
provide your public key:

1. Open the URL for accessing PMM in a web browser.

   The URL is provided either in the console window or in the appliance log.

2. Submit your **public key** in the PMM web interface.

After that you can use `ssh` to log in as the `admin` user.
For example, if *PMM Server* is running at 192.168.100.1
and your **private key** is :`~/.ssh/pmm-admin.key`,
use the following command:

```bash
   ssh admin@192.168.100.1 -i ~/.ssh/pmm-admin.key
```

</details>

<details>
  <summary style="font-size:1.25em;"><strong>Next Steps</strong></summary>

[Verify that PMM Server is running](https://www.percona.com/doc/percona-monitoring-and-management/deploy/index.html#deploy-pmm-server-verifying) by connecting to the PMM web interface using the IP address
assigned to the virtual appliance, then [install PMM Client](https://www.percona.com/doc/percona-monitoring-and-management/deploy/index.html#deploy-pmm-client-installing) on all database hosts that you want to monitor.

</details>

