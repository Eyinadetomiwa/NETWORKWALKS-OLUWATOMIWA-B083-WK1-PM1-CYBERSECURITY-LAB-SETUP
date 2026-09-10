# Networkwalks Cybersecurity Internship — Task WK1-PM1
## Practical Lab Setup: VirtualBox & Kali Linux Environment Configuration

### Overview
This project documents the implementation and verification of an isolated, functional cybersecurity lab environment using **Oracle VirtualBox** and **Kali Linux**, completed for **Week 1 (Task WK1-PM1)** of the Networkwalks Cybersecurity Internship. 

The configuration includes a dedicated virtual NAT network, static IP assignment on an isolated subnet, resolution of kernel-level duplicate address detection (DAD) timeout conflicts, bidirectional host-to-guest integrations, and a clean baseline virtual machine snapshot.

---

### Network Architecture & Specifications

| Parameter | Assigned Value | Description |
| :--- | :--- | :--- |
| **Virtualization Platform** | Oracle VirtualBox | Host virtualization hypervisor |
| **Guest OS** | Kali Linux (x86_64) | Primary penetration testing and analysis workstation |
| **Virtual Network Name** | `NatNetwork` | Isolated internal NAT network |
| **Subnet CIDR** | `10.0.0.0/24` | Private IPv4 address space |
| **Gateway Address** | `10.0.0.1` | Virtual network default gateway |
| **Kali Static IP** | `10.0.0.2` | Static address assigned to `eth0` |
| **Subnet Mask** | `255.255.255.0` (`/24`) | Class C network mask |
| **DNS Resolvers** | `8.8.8.8`, `10.0.0.1` | Domain Name System resolvers |
| **DHCP Service** | Enabled | Active on virtual NAT network |
| **Adapter Mode** | Promiscuous (`Allow All`) | Allows full network frame monitoring |

---

### Step-by-Step Implementation & Verification

#### Step 1: NAT Network Creation & Initialization
Due to interface restrictions in modern VirtualBox versions, the custom NAT network was provisioned and validated using the `VBoxManage` command-line utility.

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" natnetwork add --netname NatNetwork --network "10.0.0.0/24" --enable --dhcp on
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" natnetwork list
```

*Verification:*
The output confirmed that `NatNetwork` was successfully enabled with an IP prefix of `10.0.0.0/24`, gateway `10.0.0.1`, and active DHCP server.

![NAT Network Configuration](screenshots/01_natnetwork_cli_verification.png)

---

#### Step 2: Virtual Machine Network Adapter Binding
The virtual machine network adapter (`nic1`) was bound directly to the custom NAT network with promiscuous mode enabled:

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" modifyvm "kali-linux-2026.1-virtualbox-amd64" --nic1 natnetwork --nat-network1 "NatNetwork" --nicpromisc1 allow-all
```

*Verification:*
The VirtualBox Manager details confirmed Adapter 1 attached to `NAT Network, 'NatNetwork'`.

![VM Network Settings](screenshots/02_vm_network_adapter_details.png)

---

#### Step 3: Kali Static IPv4 Configuration
Inside the Kali Linux guest OS, NetworkManager was configured via the graphical connection editor (`nm-connection-editor`) to enforce manual static addressing on `eth0`:

* **Method:** Manual
* **Address:** `10.0.0.2`
* **Netmask:** `24`
* **Gateway:** `10.0.0.1`
* **DNS:** `8.8.8.8`

![IPv4 Manual Settings](screenshots/03_kali_manual_ipv4_config.png)

---

#### Step 4: Network Activation & DAD Timeout Resolution
When activating the static IP, NetworkManager encountered an address reservation failure (`DAD timeout`) common on virtualized bridge adapters. The issue was permanently addressed by disabling IPv4 duplicate address detection timeout:

```bash
# Disable DAD timeout validation
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0

# Cycle the network connection
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"

# Verify ICMP transit and DNS resolution
ping -c 4 8.8.8.8
```

*Verification:*
The connection activated cleanly, and the ICMP ping test executed with **0% packet loss** across 4 transmitted packets.

![IP Activation and Ping Test](screenshots/04_activation_and_ping_verification.png)

---

#### Step 5: Shared Folders & Guest System Integrations
To facilitate secure file transfer between the host system and the isolated Kali environment:
1. Bidirectional Shared Clipboard and Drag-and-Drop were enabled under VM settings.
2. The host `Downloads` directory was mounted as a permanent, auto-mounting shared folder named `Downloads`.
3. Read/write access permissions were assigned to the `kali` user account:

```bash
# Add user to VirtualBox shared folders group
sudo usermod -aG vboxsf kali
```

*Verification:*
The shared host directory mounted automatically to `/media/sf_Downloads`, confirmed via directory listing and the Thunar File Manager.

![Shared Folder Verification](screenshots/05_shared_folder_verification.png)

---

#### Step 6: Baseline Virtual Machine Snapshot
To ensure a reliable rollback state prior to deploying target machines in Phase 2, a complete baseline snapshot was captured:

* **Snapshot Name:** `Baseline-Setup-Wk1`
* **Description:** Baseline snapshot for Week 1 (Task WK1-PM1). Configured NATNetwork (10.0.0.0/24), static IP (10.0.0.2), resolved DAD timeout via NetworkManager, enabled bidirectional clipboard, and verified auto-mounted shared folder (/media/sf_Downloads).

![Baseline Snapshot](screenshots/06_baseline_snapshot.png)

---

### Repository Structure
```text
├── README.md
└── screenshots/
    ├── 01_natnetwork_cli_verification.png
    ├── 02_vm_network_adapter_details.png
    ├── 03_kali_manual_ipv4_config.png
    ├── 04_activation_and_ping_verification.png
    ├── 05_shared_folder_verification.png
    └── 06_baseline_snapshot.png
```

---

### Conclusion & Next Steps
Phase 1 of the lab architecture is fully operational and isolated. The next phase will introduce vulnerable target operating systems (Windows 10/11, Windows Server, Metasploitable) on the `10.0.0.0/24` subnet for ethical penetration testing and security assessment workflows.
