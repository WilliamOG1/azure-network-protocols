<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Create Azure Virtual Machines
- Observe ICMP Traffic
- Configuring a Firewalll [Network Security Group]

<h2>Actions and Observations</h2>

## Part 1: Create Azure Virtual Machines

### Create a Resource Group
1. Navigate to [Azure Portal](https://portal.azure.com/)
2. Click on **"Resource groups"** in the left navigation
3. Click **"+ Create"** to create a new resource group
4. Name the resource group (e.g., `"NetworkAnalysisLab-RG"`)
5. Select an appropriate region
6. Click **"Review + create"** and then **"Create"**

### Create a Windows 10 Virtual Machine
1. In the Azure portal, search for **"Virtual machines"**
2. Click **"+ Create"** and select **"Azure virtual machine"**
3. Select the resource group created in the previous step
4. Name the VM (e.g., `"windows-vm"`)
5. Select **"Windows 10 Pro"** as the image
6. Choose an appropriate size (**Standard B2s** is sufficient)
7. Create a username and password
8. Check the licensing agreement box
9. On the **Networking** tab, ensure **"Create new"** is selected for **Virtual network**
10. Note the name of the automatically created **Virtual network**
11. Click **"Review + create"** and then **"Create"**

### Create a Linux (Ubuntu) Virtual Machine
1. Repeat the process to create another VM
2. Name the VM (e.g., `"linux-vm"`)
3. Select **"Ubuntu Server 20.04 LTS"** as the image
4. Choose an appropriate size (**Standard B1s** is sufficient)
5. Select **"Password"** as the Authentication type
6. Create a username and password
7. On the **Networking** tab, select the same **Virtual network** created with the Windows VM
8. Click **"Review + create"** and then **"Create"**

### Verify Network Configuration
1. Go to **"Virtual networks"** in the Azure portal
2. Click on the virtual network created
3. Click **"Subnets"** in the left navigation
4. Verify both VMs are in the same subnet

---

## Part 2: Observe ICMP Traffic

### Connect to Windows VM via Remote Desktop
1. In the Azure portal, go to the **Windows VM** you created
2. Click **"Connect"** and select **RDP**
3. Download the **RDP file**
4. Open the **RDP file** using **Remote Desktop**
5. Enter the credentials you created for the Windows VM

### Install and Configure Wireshark
1. Inside the **Windows VM**, open a web browser
2. Navigate to [Wireshark Download Page](https://www.wireshark.org/download.html)
3. Download and install Wireshark (select the **Windows installer**)
4. Accept all **default installation options**
5. Launch **Wireshark** after installation

### Capture and Analyze ICMP Traffic
1. In **Wireshark**, select the **main network interface** (usually `"Ethernet"`)
2. Click the **blue shark fin icon** to start capturing packets
3. In the **filter bar** at the top, type `"icmp"` and press **Enter**

### Ping the Ubuntu VM
1. In the **Azure portal**, go to the **Ubuntu VM**
2. Note the **private IP address** from the Overview page
3. In the **Windows VM**, open **Command Prompt**
4. Ping <Ubuntu-VM-private-IP>
5. Obserce ping requests and replied within WireShark
6. From The Windows 10 VM, open command line or PowerShell and attempt to ping a public website (such as www.google.com) and observe the traffic in WireShark

---

## Part 3: Configuring Network Security Groups and Observing Different Protocols

### Block ICMP Traffic with Network Security Group

1. Start a continuous ping to the Ubuntu VM:

    ```ssh
    ping <Ubuntu-VM-IP> -t
    ```

2. In the Azure portal, go to the Ubuntu VM.
3. Click on **"Networking"** in the left navigation.
4. Click on the **Network Security Group** name.
5. Click **"+ Add"** to add an inbound security rule.
6. Set the following values:

    - **Source:** Any  
    - **Source port ranges:** *  
    - **Destination:** Any  
    - **Service:** Custom  
    - **Protocol:** ICMP  
    - **Action:** Deny  
    - **Priority:** 300 (or any number lower than existing rules)  
    - **Name:** `Deny-ICMP`  

7. Click **"Add"**.

## Observe Blocked ICMP Traffic

1. Return to the Windows VM.
2. Notice that the ping requests are now timing out.
3. Observe in Wireshark that ICMP requests are sent but no replies are received.

## Re-enable ICMP Traffic

1. In the Azure portal, find the **Network Security Group**.
2. Select the rule you created.
3. Click **"Delete"** to remove the rule  
   *or*  
   Change **"Action"** to **"Allow"**.
4. Observe in the Windows VM that ping replies resume.
5. Stop the continuous ping by pressing **Ctrl+C**.

## Observe SSH Traffic

1. In Wireshark, clear the current capture by clicking the restart button.
2. In the filter bar, type: ```ssh``` or ```tcp.port == 22```
   and press **Enter**.
4. Open **PowerShell** in the Windows VM.
5. Connect to the Ubuntu VM:
    ```
    ssh username@<Ubuntu-VM-IP>
    ```
6. Enter the password when prompted.
7. Run basic Linux commands:

    ```sh
    ls
    pwd
    whoami
    ```

8. Observe the SSH traffic in Wireshark.
9. Type `exit` to close the SSH connection.

## Observe DHCP Traffic

1. In Wireshark, change the filter to: ```dhcp``` or ```udp.port ==67``` or ```udp.port ==68```

2. Open **PowerShell** as Administrator.
3. Run: ```ipconfig/renew```
4. Observe the DHCP traffic in Wireshark.
5. Note the **DHCP discover, offer, request, and acknowledgment** process.

## Observe DNS Traffic

1. In Wireshark, change the filter to: ```dns``` or ```udp.port == 53```
2. In **PowerShell**, run:

    ```sh
    nslookup google.com
    nslookup disney.com
    ```

3. Observe the **DNS queries and responses** in Wireshark.
4. Note the different types of DNS records returned.

## Observe RDP Traffic

1. In Wireshark, change the filter to: ```rdp``` or ```tcp.port == 3389```

2. Notice the continuous stream of RDP traffic.
3. Move your mouse or type in the Windows VM to increase activity.
4. Observe how **RDP traffic is constantly flowing** due to the live stream nature of the protocol.

# Lab Cleanup

## End the Lab Session

1. Close the **Remote Desktop connection** to the Windows VM.
2. Return to the Azure portal.

## Delete Lab Resources

1. Go to **"Resource groups"** in the Azure portal.
2. Select the resource group created for this lab.
3. Click **"Delete resource group"**.
4. Type the **resource group name** to confirm deletion.
5. Click **"Delete"**.
