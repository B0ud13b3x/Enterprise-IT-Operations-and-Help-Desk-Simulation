
# Enterprise IT Operations & Help Desk Simulation
<p align="center">
<img src="https://cdn.worldvectorlogo.com/logos/active-directory-1.svg" alt="Active Directory logo">
</p>

<p align="center">
 <b>🛠️ Windows Server 2022 | Active Directory | Networking | ITSM (osTicket)</b>
</p>

<p align="center">
<img align="middle" src="https://osticket.com/wp-content/uploads/2021/03/osticket-supsys-new-1-e1616621912452.png" alt="osTicket">
</p>

## 🎯 Project Objective
This project is a high-fidelity simulation of an enterprise-level IT environment. It demonstrates the transition from **Infrastructure Deployment** (building the corporate backbone) to **Active Operations** (managing user lifecycles and technical incidents). By integrating a Windows Server 2022 Domain Controller with a functional ITSM platform (osTicket), I simulate the complete lifecycle of a technical incident from user submission to administrative resolution.



## 📐 Network Architecture & Environment
![Network Diagram](./my%20first%20lab.png) 
*Architecture showing the isolation of internal corporate clients from the public internet through a multi-homed Windows Server Gateway.*

### Key Components:
*   **Hypervisor:** Oracle VirtualBox
*   **Domain Controller:** Windows Server 2022 (AD DS)
*   **Gateway/Router:** Server 2022 configured with **Dual NICs** (NAT for internet access and an Internal Network for security).
*   **Ticketing System:** osTicket (ITSM)
*   **Client Fleet:** Windows 10 Pro and Windows 11 Pro workstations



## 🛡️ Technical Implementation

### 1. Identity & Access Management (AD DS)
*   [**Domain Configuration:**](./active-directory/README.md) Architected `mydomain.com` as a simulated corporate forest.
*   **OU Hierarchy:** Designed a granular Organizational Unit (OU) structure for departments (IT, HR, Sales, Finance) to practice least-privilege permission management.
*   **Group Policy Objects (GPOs):** Implemented security baselines, including 90-day password rotation policies and account lockout thresholds (3 failed attempts).

### 2. Network Infrastructure & Routing
*   **NAT/RAS:** Configured the Domain Controller as a secure gateway using Remote Access Services to bridge the internal network to the public web.
*   **DHCP & DNS:** Built an automated IP addressing system (DHCP Scope) and configured Forward/Reverse lookup zones for seamless internal resource access.

### 3. Help Desk Operations (osTicket)
*   **Ticket Lifecycle:** Deployed and configured **osTicket** to manage incoming support requests.
*   **SLA Management:** Defined Service Level Agreements (SLAs) for different incident severities (e.g., Critical: 1hr resolution; Low: 24hr).
*   **Knowledge Base (KB):** Authored 5+ step-by-step troubleshooting guides to improve First Contact Resolution (FCR) rates.

### 4. Automation via PowerShell
*   **Bulk User Creation:** Developed a PowerShell script to programmatically create **1,000+ users** from a data file, simulating a large-scale enterprise onboarding event.


## 🛠️ The Troubleshooting Journal (STAR Method)
*In IT, knowing how to fix what breaks is more valuable than building it perfectly the first time.*

### Case Study: Network Connectivity Restoration
*   **Situation:** A Windows 11 client joined to the domain successfully but lost internet access, though it could still "ping" the Domain Controller.
*   **Task:** Identify and resolve the routing failure between the internal virtual network and the external NAT interface.
*   **Action:** Used `ipconfig /all` to verify the Default Gateway was pointing to the DC’s internal IP. Investigated the **Remote Access Services (RAS)** console and identified a stale NAT routing table.
*   **Result:** Restarted the RAS service and verified the static IP on the internal NIC. Full internet connectivity was restored to the client while remaining secured behind the DC.



## 🧰 Skills & Tools Demonstrated
*   **Identity Management:** Active Directory DS, GPO, PowerShell Scripting.
*   **Networking:** TCP/IP, DNS, DHCP, VPN/Routing, NAT/RAS.
*   **Support & Documentation:** Ticketing (osTicket), Technical Writing, Knowledge Base Management.
*   **Operating Systems:** Windows Server 2022, Windows 10/11 Pro, Linux (Ubuntu).



## 🚀 Active Lab Scenarios (In-Progress)
Current work involves simulating the "Top 10" most common help desk tickets, including:
1.  **User Account Lockout:** Diagnosing and unlocking accounts via AD DS.
2.  **Shared Drive Mapping:** Using GPOs to automatically map departmental drives.
3.  **Software Deployment:** Deploying enterprise applications (e.g., 7-Zip) via Group Policy.
