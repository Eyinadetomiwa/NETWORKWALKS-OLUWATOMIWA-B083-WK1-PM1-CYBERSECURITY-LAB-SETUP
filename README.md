# Networkwalks Cybersecurity Internship — Task WK1-PM1
## Practical Lab Setup: VirtualBox & Kali Linux Environment Configuration

### Overview
This project documents the setup and verification of an isolated cybersecurity lab using Oracle VirtualBox and Kali Linux, completed for Week 1 (Task WK1-PM1) of the Networkwalks Cybersecurity Internship. 

The configuration includes a dedicated virtual NAT network, static IP assignment on an isolated subnet, resolution of duplicate address detection (DAD) timeout conflicts, bidirectional host-to-guest integrations, and a clean baseline snapshot.

---

### Network Architecture & Specifications

| Parameter | Assigned Value | Description |
| :--- | :--- | :--- |
| Virtualization Platform | Oracle VirtualBox | Host virtualization hypervisor |
| Guest OS | Kali Linux (x86_64) | Primary penetration testing workstation |
| Virtual Network Name | NatNetwork | Isolated internal NAT network |
| Subnet CIDR | 10.0.0.0/24 | Private IPv4 address space |
| Gateway Address | 10.0.0.1 | Virtual network default gateway |
| Kali Static IP | 10.0.0.2 | Static address assigned to eth0 |
| Subnet Mask | 255.255.255.0 (/24) | Class C network mask |
| DNS Resolvers | 8.8.8.8, 10.0.0.1 | Domain Name System resolvers |
| DHCP Service | Enabled | Active on virtual NAT network |
| Adapter Mode | Promiscuous (Allow All) | Allows full network traffic monitoring |

---

### Step-by-Step Implementation & Verification

#### Step 1: NAT Network Creation & Initialization
Due to GUI limitations in modern VirtualBox versions, the custom NAT network was created and verified via the VBoxManage command-line utility:

"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" natnetwork add --netname NatNetwork --network "10.0.0.0/24" --enable --dhcp on
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" natnetwork list

Verification:
The output confirmed that NatNetwork was enabled with an IP prefix of 10.0.0.0/24, gateway 10.0.0.1, and active DHCP.

![NAT Network Configuration](VBoxManage%20natnetwork%20list.png)

---

#### Step 2: VM Network Adapter Configuration
The virtual machine network adapter (nic1) was attached directly to the custom NAT network with promiscuous mode set to allow all:

"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "kali-linux-2026.1-virtualbox-amd64" --nic1 natnetwork --nat-network1 "NatNetwork" --nicpromisc1 allow-all

Verification:
The VirtualBox Manager details confirmed Adapter 1 attached to NAT Network, 'NatNetwork'.

![VM Network Settings](Virtual%20Machine%20Network%20Adapter%20Binding.png)

---

#### Step 3: Kali Static IPv4 Configuration
Inside Kali Linux, NetworkManager was configured via the graphical connection editor (nm-connection-editor) to apply manual static addressing on eth0:

* Method: Manual
* Address: 10.0.0.2
* Netmask: 24
* Gateway: 10.0.0.1
* DNS: 8.8.8.8

![IPv4 Manual Settings](Static%20IPv4%20Configuration.png)

---

#### Step 4: Network Activation & DAD Timeout Resolution
When activating the static IP, NetworkManager threw an address reservation error due to a known duplicate address detection (DAD) timeout. This was resolved by setting the DAD timeout to zero:

sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
ping -c 4 8.8.8.8

Verification:
The connection activated successfully, and the ICMP ping test returned 0% packet loss.

![IP Activation and Ping Test](DAD%20Bug%20Fix,%20Network%20Activation%20%26%20ICMP%20Connectivity.png)

---

#### Step 5: Shared Folders & Guest System Integrations
To easily move files between the host system and the Kali VM:
1. Bidirectional Shared Clipboard and Drag-and-Drop were enabled under VM settings.
2. The host Downloads directory was mounted as a permanent, auto-mounting shared folder named Downloads.
3. User permissions were granted inside Kali:

sudo usermod -aG vboxsf kali

Verification:
The shared directory mounted automatically to /media/sf_Downloads, verified via terminal listing and the File Manager.

![Shared Folder Verification](Shared%20Folder%20%26%20Filesystem%20Verification.png)

---

#### Step 6: Baseline Virtual Machine Snapshot
To ensure a rollback point before testing and installing target machines, a baseline snapshot was created:

* Snapshot Name: Baseline-Setup-Wk1
* Description: Baseline snapshot for Week 1 (Task WK1-PM1). Configured NATNetwork (10.0.0.0/24), static IP (10.0.0.2), resolved DAD timeout via NetworkManager, enabled bidirectional clipboard, and verified auto-mounted shared folder (/media/sf_Downloads).

![Baseline Snapshot](Baseline%20Virtual%20Machine%20Snapshot.png)

---

### Conclusion & Next Steps
The Phase 1 lab setup is complete and fully operational. Future phases will involve deploying target operating systems (Windows, Metasploitable) on the 10.0.0.0/24 subnet for penetration testing and practical assessment exercises.
