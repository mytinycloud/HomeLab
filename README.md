# 📖 Homelab Network Design Document

**Version:** 1.0
**Status:** Mark VI Architecture Plan
**Last Updated:** <!-- ANSIBLE_MANAGED_BLOCK_START: last_updated -->*This timestamp is automatically updated.*<!-- ANSIBLE_MANAGED_BLOCK_END: last_updated -->

### **Revision History**

| Version | Date | Author | Changes |
| :--- | :--- | :--- | :--- |
| 0.1.0 - 0.5.9 | 2025-09-04 | James | Initial draft, iterative additions of security, services, and operational concepts. |
| 0.6.0 | 2025-09-04 | James | Added Document Control, IP Schema, and Glossary. |
| **1.0** | **2025-09-04** | **James** | **Created definitive Port Map from user-provided list. Corrected all naming and table formatting.** |

---

### 1.0 Introduction & Design Philosophy

This document outlines the final architecture of the homelab network following a significant upgrade to a high-availability, multi-gigabit infrastructure. This plan represents the design for the sixth major iteration (Mark VI) of the homelab and is intended to be the single source of truth for its physical, logical, and operational state.

The core design philosophy is built on five key pillars:

* **High Availability:** Eliminate single points of failure at the core network, firewall, and server connectivity layers.
* **Performance:** Utilize 10GbE connectivity, Layer 3 switching, and Quality of Service to provide a high-speed, low-latency, and responsive experience.
* **Security:** Enforce a defense-in-depth strategy with strict segmentation, proactive threat detection, and centralized identity management.
* **Operational Discipline:** Employ best practices for monitoring, logging, backups, and secrets management to ensure a stable and maintainable environment.
* **Scalability & Automation:** Create a flexible foundation that is managed via Infrastructure as Code and documented through automated processes.

**Naming Convention:** All systems and key infrastructure components adhere to a naming scheme based on the *Dune* universe by Frank Herbert.

---

### 2.0 Network diagram

This document should be used in conjunction with the following visual diagrams:
![Network diagram](/Network%20Diagram.svg)
---

### 3.0 Usability & Operational Requirements

This section defines the non-functional requirements that govern the end-user and administrative experience of the homelab.

#### 3.1 Service Accessibility

* **Requirement:** Access to all web-based services shall be seamless and secure.
* **Implementation:** A Single Sign-On (SSO) system (`Authelia`) will provide a unified login portal. Users should only need to authenticate once per session to access all authorized applications. Two-factor authentication will be enforced at this central point.

#### 3.2 User Experience & Performance

* **Requirement:** The network must feel responsive at all times. Bulk data transfers must not negatively impact the performance of real-time or interactive applications.
* **Implementation:** A Quality of Service (QoS) policy on the `pfSense` firewall will prioritize latency-sensitive traffic (gaming, video calls) over bulk downloads, ensuring a consistently low-latency experience for interactive tasks.

#### 3.3 Administrative Experience

* **Requirement:** The system should be manageable "by exception." Day-to-day administration should be minimal, with the system actively reporting issues rather than requiring constant manual checks.
* **Implementation:** Proactive monitoring (`Uptime Kuma`), centralized logging (`Loki`), and detailed metrics (`Prometheus`/`Grafana`) will provide a comprehensive overview of system health. Alerts will be configured to notify the administrator of any service failures or anomalous conditions.

#### 3.4 Service Availability & Communication

* **Requirement:** Critical services should be highly available. In the event of an outage, the status of the system should be clearly and easily accessible.
* **Implementation:** The core network, firewall, and server connectivity are designed with full redundancy. An `Uptime Kuma` status page will provide a clear, at-a-glance view of the health of all services, serving as the single source of truth during any troubleshooting.

---

### 4.0 Physical Topology & Hardware Specifications

This section details the physical hardware, connectivity, and component specifications for all core infrastructure devices.

#### 4.1 Core Network Switches

The core is comprised of two Allied Telesis x510 Series switches operating as a single logical unit via the VCStack™ feature, interconnected in a redundant ring topology.

<!-- ANSIBLE_MANAGED_BLOCK_START: switch_specifications -->
* **`SIETCH-TABR` (VCStack Master)**
    * **Role:** Core Switch, L3 Router for internal VLANs.
    * **Model:** Allied Telesis x510-52GTX
    * **Ports:** 48x 1GbE RJ45, 4x 10GbE SFP+
    * **Stacking Ports:** S1/51, S2/52
* **`SIETCH-JACURUTU` (VCStack Member)**
    * **Role:** Access Switch.
    * **Model:** Allied Telesis x510-28GTX
    * **Ports:** 24x 1GbE RJ45, 4x 10GbE SFP+
    * **Stacking Ports:** S1/27, S2/28
<!-- ANSIBLE_MANAGED_BLOCK_END: switch_specifications -->

#### 4.2 Proxmox Virtualization Hosts

The compute cluster consists of two Proxmox VE hosts with distinct roles based on their hardware capabilities.

<!-- ANSIBLE_MANAGED_BLOCK_START: host_specifications -->
* **`Arrakis` (Primary Application Host)**
    * **Role:** High-Performance Workloads, Secondary Firewall.
    * **CPU:** AMD Ryzen 5 5600 (6 Cores / 12 Threads)
    * **RAM:** 96 GB DDR4 (Non-ECC)
    * **Storage:**
        * **Boot/System:** 1x 240GB Crucial SSD
        * **Bulk Data:** 1x 8TB Seagate HDD
    * **Network:**
        * **Data Link (10G LACP):** 1x Dual Port 10GbE SFP+ NIC
        * **VM Link (1G LACP):** 1x Quad Port 1GbE NIC
        * **Management:** 1x Onboard 1GbE (dedicated)
    * **Boot Mode:** EFI
* **`Caladan` (Lightweight Services Host)**
    * **Role:** Lightweight Services, Primary Firewall.
    * **CPU:** Intel Core i3-2120 (2 Cores / 4 Threads)
    * **RAM:** 8 GB DDR3 (Non-ECC)
    * **Storage:**
        * **Boot/System:** 1x 240GB Crucial SSD
    * **Network:**
        * **Data Link (1G LACP):** 2x Single Port 1GbE NICs
        * **Management:** 1x Onboard 1GbE (dedicated)
    * **Boot Mode:** Legacy BIOS
<!-- ANSIBLE_MANAGED_BLOCK_END: host_specifications -->

#### 4.3 Network Attached Storage (NAS)

The primary backup target and bulk file storage is provided by a centralized NAS.

<!-- ANSIBLE_MANAGED_BLOCK_START: nas_specifications -->
* **`IX` (Synology NAS)**
    * **Role:** Primary Backup Target, Bulk File Storage.
    * **Model:** Synology DS920+
    * **CPU:** Intel Celeron J4125 (4 Cores)
    * **RAM:** 4 GB DDR4
    * **Storage:** All-Flash Array (Configuration to be detailed by Ansible).
    * **Network (LACP):** 2x Onboard 1GbE RJ45
<!-- ANSIBLE_MANAGED_BLOCK_END: nas_specifications -->

#### 4.4 Port Map & Cabling Plan

This section provides a prescriptive guide for all physical infrastructure connections to the VCStack. This serves as the definitive source of truth for physical cabling and the desired state for automated configuration validation.

<!-- ANSIBLE_MANAGED_BLOCK_START: port_map -->
| Device | Port | Connects to Device | Connects to Port | VLAN(s) / Config | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **SIETCH-TABR** | **S1/51** | SIETCH-JACURUTU | S2/28 | VCStack Link | Stack Interconnect 1 |
| **SIETCH-TABR** | **S2/52** | SIETCH-JACURUTU | S1/27 | VCStack Link | Stack Interconnect 2 |
| **SIETCH-TABR** | **port1.0.50** | Arrakis | SFP+ Port 1 | LACP Group 1, Trunk | Server 10G Data Link 1 |
| **SIETCH-TABR** | **port1.0.47** | ISP ONT | N/A | Access VLAN 99 | WAN Transit from ISP |
| **SIETCH-TABR** | **port1.0.45** | Arrakis | Onboard 1GbE | Access VLAN 10 | Arrakis Management Link |
| **SIETCH-TABR** | **port1.0.44** | IX (NAS) | Onboard NIC 1 | LACP Group 2, Trunk | NAS Data Link 1 |
| **SIETCH-TABR** | **port1.0.43** | Wireless AP | N/A | Access VLAN 100 | Untrusted Wireless |
| **SIETCH-TABR** | **port1.0.42** | Arrakis | 1G NIC Port 1 | LACP Group 3, Trunk | Arrakis VM Link 1 |
| **SIETCH-TABR** | **port1.0.41** | Arrakis | 1G NIC Port 2 | LACP Group 3, Trunk | Arrakis VM Link 2 |
| **SIETCH-TABR** | **port1.0.39** | Caladan | 1G NIC 1 | LACP Group 4, Trunk | Caladan Data Link 1 |
| **SIETCH-JACURUTU**| **port2.0.26** | Arrakis | SFP+ Port 2 | LACP Group 1, Trunk | Server 10G Data Link 2 |
| **SIETCH-JACURUTU**| **port2.0.24** | Caladan | Onboard 1GbE | Access VLAN 10 | Caladan Management Link |
| **SIETCH-JACURUTU**| **port2.0.21** | Arrakis | 1G NIC Port 3 | LACP Group 3, Trunk | Arrakis VM Link 3 |
| **SIETCH-JACURUTU**| **port2.0.20** | Arrakis | 1G NIC Port 4 | LACP Group 3, Trunk | Arrakis VM Link 4 |
| **SIETCH-JACURUTU**| **port2.0.19** | IX (NAS) | Onboard NIC 2 | LACP Group 2, Trunk | NAS Data Link 2 |
| **SIETCH-JACURUTU**| **port2.0.18** | Caladan | 1G NIC 2 | LACP Group 4, Trunk | Caladan Data Link 2 |
| **SIETCH-JACURUTU**| **port2.0.4** | Aqara Hub | N/A | Access VLAN 100 | IoT Hub |
| **SIETCH-JACURUTU**| **port2.0.2** | Hue Hub | N/A | Access VLAN 100 | IoT Hub |
| **SIETCH-JACURUTU**| **port2.0.1** | Client PC | N/A | Access VLAN 20 | Primary Client Access |
<!-- ANSIBLE_MANAGED_BLOCK_END: port_map -->

---

### 5.0 Storage Architecture

This section details the strategy for data storage, outlining the different tiers, their purposes, and the underlying technologies.

#### 5.1 Proxmox Host Local Storage

Each Proxmox node utilizes its local storage for specific, performance-sensitive tasks.

* **`Arrakis` Storage Strategy:**
    * **System Pool (LVM-Thin on SSD):** The 240GB SSD is used for the Proxmox OS and provides an LVM-Thin pool for performance-critical VM/LXC root disks.
    * **Bulk Pool (LVM on HDD):** The 8TB HDD is configured as a separate LVM volume, passed through to specific VMs for large, less performance-sensitive data.
* **`Caladan` Storage Strategy:**
    * **System Pool (LVM-Thin on SSD):** The 240GB SSD houses the Proxmox OS and an LVM-Thin pool for all lightweight VMs and LXCs that run on this node.

#### 5.2 Shared Network Storage

Centralized, network-accessible storage is provided by the Synology NAS.

* **Technology:** `IX` (Synology DS920+) provides storage pools presented to the network.
* **Primary Role: Backup Target:** A dedicated NFS share is presented to the Proxmox cluster for all VM and application backups.
* **Secondary Role: Bulk File Shares:** SMB shares are provided for client devices on the network for general purpose file sharing.

---

### 6.0 Logical Topology & Security

The network employs a multi-layered security and routing architecture.

#### 6.1 Macro-segmentation & Routing

VLANs provide the primary layer of network separation.

* **Layer 3 Switch (Performance Routing):** The `SIETCH-TABR` VCStack performs all inter-VLAN routing for trusted, high-speed internal networks.
* **Firewall (Security Routing):** The `pfSense` cluster handles all routing for untrusted networks, management interfaces, and all traffic destined for the internet.

#### 6.2 VLAN Schema
<!-- ANSIBLE_MANAGED_BLOCK_START: vlan_schema -->
| VLAN ID | Network Name | CIDR | Gateway / Router | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **10** | **Management** | `10.0.10.0/24` | **pfSense** | Proxmox hosts, IPMI, switch management. |
| **11** | **Transit** | `10.0.11.0/30` | N/A | P2P link between L3 Switch and pfSense. |
| **99** | **WAN Transit** | N/A | N/A | L2 segment for ISP ONT traffic to pfSense WAN. |
| **2** | **Storage** | `10.0.2.0/24` | **L3 Switch** | NFS/iSCSI traffic for `IX`. |
| **3** | **Media** | `10.0.3.0/24` | **L3 Switch** | Media servers, game servers, containers. |
| **4** | **VMs** | `10.0.4.0/24` | **L3 Switch** | General purpose lab VMs & databases. |
| **5** | **Kubernetes** | `10.0.5.0/24` | **L3 Switch** | Dedicated network for Kubernetes cluster nodes and services. |
| **20** | **Client** | `10.0.20.0/24` | **L3 Switch** | Trusted personal devices. |
| **30** | **Monitoring** | `10.0.30.0/24` | **pfSense** | Prometheus, Grafana, and other monitoring tools. |
| **100**| **Untrusted** | `10.0.100.0/24`| **pfSense** | IoT devices & all current wireless clients. |
| **200**| **DMZ** | `10.0.200.0/24`| **pfSense** | Legacy or non-tunneled public services. |
| **998**| **Native** | N/A | N/A | Isolates untagged traffic on trunk ports. |
| **999**| **Parking** | N/A | N/A | Isolates all unused physical switch ports. |
<!-- ANSIBLE_MANAGED_BLOCK_END: vlan_schema -->

#### 6.3 IP Addressing Schema

To ensure consistency and predictability, IP addresses are allocated according to the following prescriptive scheme. All gateways will reside at `.1`.

| VLAN / Name | Subnet | Static Range | DHCP Pool | Key Static Assignments |
| :--- | :--- | :--- | :--- | :--- |
| **10 / Management** | `10.0.10.0/24` | `.10-.49` | `.100-.254` | `.10`: Arrakis, `.11`: Caladan, `.20`: SIETCH-TABR |
| **2 / Storage** | `10.0.2.0/24` | `.10-.49` | (None) | `.10`: IX |
| **3 / Media** | `10.0.3.0/24` | `.10-.49` | `.100-.254` | `.10`: Docker Host, `.11`: Plex Server |
| **4 / VMs** | `10.0.4.0/24` | `.10-.49` | `.100-.254` | `.10`: PostgreSQL-DB, `.20`: Authelia, `.30`: Ubuntu Lab |
| **5 / Kubernetes**| `10.0.5.0/24` | `.10-.49` | (None) | `.10`: k8s-cp-01, `.11`: k8s-w-01, `.12`: k8s-w-02<br>*`.200-.220` reserved for MetalLB* |
| **20 / Client** | `10.0.20.0/24` | `.50-.99` | `.100-.254` | (Static range reserved for devices like printers) |
| **30 / Monitoring** | `10.0.30.0/24` | `.10-.49` | (None) | `.10`: Uptime Kuma, `.11`: Loki, `.12`: Prometheus, `.13`: Grafana |
| **100 / Untrusted**| `10.0.100.0/24`| `.10-.49` | `.100-.254` | `.10`: Home Assistant |
| **200 / DMZ** | `10.0.200.0/24`| `.10-.49` | `.100-.254` | (Reserved for future public-facing servers) |

#### 6.4 Defense-in-Depth Firewall Strategy

Security is enforced at three distinct layers:

* **Layer 1: Network Firewall (pfSense):** The perimeter firewall.
* **Layer 2: Hypervisor Firewall (Proxmox VE):** Provides micro-segmentation.
* **Layer 3: Guest OS Firewall (ufw / Windows Defender):** Protects applications inside each VM.

#### 6.5 Proactive Threat Management (IDS/IPS)

* **Technology:** **Suricata** will be deployed as a package on the `pfSense` firewall cluster, operating in-line on the WAN interface to actively block threats.

#### 6.6 DHCP & DNS Services

* **DHCP Strategy:** A hybrid DHCP server/relay model will be used, centralized on `pfSense`.
* **DNS Strategy:** The **Unbound** resolver on `pfSense`, augmented with **pfBlockerNG**, will provide secure, ad-blocking DNS for all clients.

#### 6.7 Port Security & Default Configuration

* **Native VLAN:** All trunk ports will use `VLAN 998` as the native VLAN. VLAN 1 will not be used.
* **Parking VLAN:** All unused physical switch ports will be shut down and assigned to `VLAN 999`.

---

### 7.0 Firewall & Access Control Policies

The guiding principle is **"deny by default."** No traffic is permitted unless explicitly allowed.

#### 7.1 L3 Switch (Trusted Zone) Access Control

The policy within the trusted zone (`VLANs 2, 3, 4, 5, 20`) is **ALLOW ALL** to facilitate maximum performance. All non-local traffic is forwarded to pfSense for inspection.

#### 7.2 pfSense Firewall (Security Zone) Access Control

The pfSense cluster enforces security for all traffic crossing trust boundaries.

| FROM &#9660; / TO &#9654; | Internet | Management (10) | Untrusted (100) | Monitoring (30) | Kubernetes (5) | Trusted Zone (2,3,4,20) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Management (10)** | **ALLOW** | N/A | **ALLOW** | **ALLOW** | **ALLOW** `[6]` | **ALLOW** |
| **Untrusted (100)** | **ALLOW** | **DENY** | N/A | **DENY** | **DENY** | **DENY** |
| **Monitoring (30)** | **ALLOW** | **DENY** | **ALLOW** `[1]`| N/A | **ALLOW** | **ALLOW** `[1]` |
| **Kubernetes (5)** | **ALLOW** | **DENY** | **DENY** | **ALLOW** | N/A | **ALLOW** `[5]` |
| **Trusted Zone (2,3,4,20)**| **ALLOW** | **ALLOW** `[2]` | **ALLOW** `[3]` | **ALLOW** `[4]` | **ALLOW** `[6]` | (Handled by L3 Switch) |
| **Internet** | N/A | **DENY** | **DENY** | **DENY** | **DENY** | **DENY** |

**Rule Exceptions:** `[1]` Monitoring probes. `[2]` Admin access. `[3]` IoT control. `[4]` Dashboard access. `[5]` K8s access to storage/DBs. `[6]` Admin access to K8s API (TCP/6443).

---

### 8.0 Performance Management

A **QoS** policy using **FQ_CODEL** will be implemented on the `pfSense` WAN interface to prioritize real-time traffic over bulk transfers.

---

### 9.0 Network Monitoring & Visibility

* **SNMP for Metrics:** The `Prometheus` server will scrape detailed performance and error counters from the VCStack via SNMP.
* **Port Mirroring for Analysis:** A port will be reserved on the stack for on-demand traffic mirroring (SPAN) for deep packet inspection.

---

### 10.0 Core Software Architecture
<!-- ANSIBLE_MANAGED_BLOCK_START: software_architecture -->
| Platform | Component | Target Version | Notes |
| :--- | :--- | :--- | :--- |
| **Hypervisor** | Proxmox VE | 8.x | All nodes must run the same version. |
| **Network OS** | AlliedWare Plus | 5.5.x or later | Both switches must run identical firmware. |
| **Firewall** | pfSense Plus | Latest Stable | Deployed as a VM cluster. |
| **IDS/IPS** | Suricata | Latest Stable | Deployed as a pfSense package. |
| **Container Orchestration** | K3s | Latest Stable | Lightweight Kubernetes distribution. |
| **IaC Provisioning**| Terraform | Latest Stable | Used with the Proxmox provider. |
| **IaC Configuration**| Ansible | Latest Stable | Manages VM/LXC configuration. |
| **Monitoring** | Prometheus / Grafana| Latest Stable | Deployed as LXC containers. |
<!-- ANSIBLE_MANAGED_BLOCK_END: software_architecture -->

---

### 11.0 Service Inventory & Application Stacks

#### 11.1 Standalone Services
<!-- ANSIBLE_MANAGED_BLOCK_START: service_inventory -->
| Service Name | Host Node | VLAN | Purpose | Software & Versions |
| :--- | :--- | :--- | :--- | :--- |
| **Docker Host** | Arrakis | Media (3) | Container host. Applications will be configured to use the central PostgreSQL DB. | - Docker Engine: `26.1.1`<br>- Portainer-CE: `2.20.1`<br>- Cloudflare Tunnel: `2024.x.x` |
| **Plex Server** | Arrakis | Media (3) | High-performance media streaming. | - OS: `Ubuntu 24.04 LTS`<br>- Plex Media Server: `1.40.1` |
| **PostgreSQL-DB** | Arrakis | VMs (4) | (LXC) Centralized database server for applications. | - OS: `Ubuntu 24.04 LTS`<br>- PostgreSQL: `16.x` |
| **Home Assistant**| Arrakis | Untrusted (100)| Home automation and IoT management. | - Home Assistant OS: `12.3` |
| **Authelia** | Caladan | VMs (4) | (LXC) Centralized Identity Provider for SSO. | - Authelia: `4.38.0` |
| **Uptime Kuma** | Caladan | Monitoring (30)| (LXC) Uptime monitoring and status page. | - Uptime Kuma: `1.23.11` |
| **Loki** | Caladan | Monitoring (30)| (LXC) Centralized log aggregation server. | - Loki: `2.9.5` |
| **Prometheus** | Caladan | Monitoring (30) | (LXC) Metrics collection. | - Prometheus: `2.51.1` |
| **Grafana** | Caladan | Monitoring (30) | (LXC) Visualization dashboards for metrics and logs. | - Grafana: `10.4.2` |
<!-- ANSIBLE_MANAGED_BLOCK_END: service_inventory -->

#### 11.2 Kubernetes Cluster

The cluster will run K3s, a lightweight distribution. The nodes are provisioned as VMs across the Proxmox cluster.

| Node Name | Role | Host Node | VLAN | IP Address | vCPU | RAM | Disk |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **k8s-cp-01** | Control Plane | Arrakis | Kubernetes (5) | `10.0.5.10` | 2 | 4 GB | 30 GB |
| **k8s-w-01** | Worker | Arrakis | Kubernetes (5) | `10.0.5.11` | 4 | 8 GB | 50 GB |
| **k8s-w-02** | Worker | Caladan | Kubernetes (5) | `10.0.5.12` | 2 | 3 GB | 50 GB |

#### 11.3 VM & Container Templates
<!-- ANSIBLE_MANAGED_BLOCK_START: template_inventory -->
| Template Name | Type | Base OS | Key Features |
| :--- | :--- | :--- | :--- |
| **ubuntu-2404-cloudinit** | VM | Ubuntu 24.04 LTS | - Cloud-Init enabled<br>- `qemu-guest-agent` installed<br>- Hardened with basic security profiles |
<!-- ANSIBLE_MANAGED_BLOCK_END: template_inventory -->

---

### 12.0 Enterprise Services & Operational Discipline

* **Centralized Logging:** **Loki** will aggregate logs from all systems.
* **Identity and Access Management (IAM):** **Authelia** will provide SSO and 2FA.
* **Secrets Management:** **Ansible Vault** will be used to encrypt all secrets at rest.
* **Uptime Monitoring & Alerting:** **Uptime Kuma** will provide real-time status checks and notifications.

---

### 13.0 Documentation Automation Workflow

This document is a living document, automatically updated by an **n8n** workflow that orchestrates **Ansible** to gather and inject live state data.

---

### 14.0 Backup and Recovery Strategy

A multi-layered 3-2-1 backup strategy is in place, utilizing Proxmox backups, Ansible for application data, and `IX` as the primary target, with off-site sync to the cloud.

---

### 15.0 Future State Integrations

* **Dedicated ZFS Storage Server:** Build `CHOAM`, a dedicated TrueNAS server.
* **Third Proxmox Node (`SALUSA SECUNDUS`):** Add a third node to enable Proxmox HA.
* **VLAN-Aware Wi-Fi 7 AP:** Deploy a new AP to enable proper wireless segmentation.
* **Migrate Services:** Systematically migrate containerized services from the standalone Docker host to the Kubernetes cluster.

---

### 16.0 Glossary of Terms

| Term | Definition |
| :--- | :--- |
| **ACL** | Access Control List. A set of rules applied to a switch or router interface to filter traffic. |
| **CARP** | Common Address Redundancy Protocol. Used by pfSense for firewall failover. |
| **IaC** | Infrastructure as Code. Managing infrastructure through machine-readable definition files. |
| **IDS/IPS**| Intrusion Detection/Prevention System. A security appliance that monitors for malicious activity. |
| **K8s** | Kubernetes. An open-source container orchestration system for automating software deployment, scaling, and management. |
| **LACP** | Link Aggregation Control Protocol. A protocol for bundling multiple network links into one logical link. |
| **LXC** | Linux Containers. An operating-system-level virtualization method for running multiple isolated Linux systems. |
| **SSO** | Single Sign-On. An authentication scheme that allows a user to log in with a single ID to any of several related software systems. |
| **SVI** | Switched Virtual Interface. A virtual routed interface on a Layer 3 switch that represents a VLAN. |
| **VCStack**| Virtual Chassis Stacking. Allied Telesis technology to combine multiple physical switches into one logical switch. |
| **VLAN** | Virtual Local Area Network. A method of logically segmenting a physical network. |