# 🚀 Azure Hub-and-Spoke Architecture – Step-by-Step CLI Implementation

This document provides a **complete CLI-based implementation** of a secure and scalable **Hub-and-Spoke network architecture** on Microsoft Azure.

The setup includes **Virtual Networks (VNets), VNet Peering, Subnets, Azure Firewall, Bastion Host, User-Defined Routes (UDRs), Network Security Groups (NSGs), Virtual Machines, and centralized monitoring with Log Analytics**.

---

## 🏗 What is Hub-and-Spoke Architecture?

The **Hub-and-Spoke topology** is a network design where:

* A **Hub VNet** acts as the central point for connectivity, security, and management.
* **Spoke VNets** connect to the Hub VNet via peering, and they host workloads (applications, VMs, services).
* All communication between spokes is routed **through the Hub** for centralized control.

### 🔎 Purpose

* Centralize **security and routing** using Azure Firewall/NVA
* Enable **controlled communication** between workloads across spokes
* Provide **shared services** in the hub (e.g., Bastion, monitoring, DNS, logging)
* Simplify **governance and compliance** with a clear security boundary

### ⚙️ How it Works

1. **Spoke VNets** do not directly connect to each other.
2. Traffic between spokes must pass through the **Hub Firewall**, ensuring inspection and policy enforcement.
3. **User Defined Routes (UDRs)** force all outbound/inbound traffic from spokes through the Hub.
4. **Azure Bastion** in the Hub provides secure management access to workloads without exposing public IPs.
5. **Log Analytics** centralizes monitoring and diagnostic logging.

---

## 🖼 Architecture Diagram

<img width="2546" height="1616" alt="image" src="https://github.com/user-attachments/assets/760fc096-261d-41ce-b70c-bc052eb6b40f" />

---

## 🛠 Step-by-Step CLI Deployment

### 1️⃣ Create Resource Group

```bash
az group create --name HubSpoke-RG --location centralindia
```

---

### 2️⃣ Create Hub Virtual Network with Subnets

```bash
az network vnet create \
  --resource-group HubSpoke-RG \
  --name Hub-VNet \
  --address-prefix 11.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 11.0.1.0/26

az network vnet subnet create \
  --resource-group HubSpoke-RG \
  --vnet-name Hub-VNet \
  --name AzureBastionSubnet \
  --address-prefix 11.0.2.0/26
```

---

### 3️⃣ Create Spoke Virtual Networks

```bash
az network vnet create \
  --resource-group HubSpoke-RG \
  --name Spoke1-VNet \
  --address-prefix 12.0.0.0/24 \
  --subnet-name Workload \
  --subnet-prefix 12.0.0.0/24

az network vnet create \
  --resource-group HubSpoke-RG \
  --name Spoke2-VNet \
  --address-prefix 13.0.0.0/24 \
  --subnet-name Workload \
  --subnet-prefix 13.0.0.0/24
```

---

### 4️⃣ Configure VNet Peering

```bash
# Spoke1 ↔ Hub
az network vnet peering create --name Spoke1ToHub --resource-group HubSpoke-RG \
  --vnet-name Spoke1-VNet --remote-vnet Hub-VNet --allow-vnet-access

az network vnet peering create --name HubToSpoke1 --resource-group HubSpoke-RG \
  --vnet-name Hub-VNet --remote-vnet Spoke1-VNet --allow-vnet-access

# Spoke2 ↔ Hub
az network vnet peering create --name Spoke2ToHub --resource-group HubSpoke-RG \
  --vnet-name Spoke2-VNet --remote-vnet Hub-VNet --allow-vnet-access

az network vnet peering create --name HubToSpoke2 --resource-group HubSpoke-RG \
  --vnet-name Hub-VNet --remote-vnet Spoke2-VNet --allow-vnet-access
```

---

### 5️⃣ Deploy Azure Firewall

```bash
az network public-ip create \
  --name FirewallPublicIP \
  --resource-group HubSpoke-RG \
  --sku Standard \
  --location centralindia \
  --allocation-method Static

az network firewall create \
  --name HubFirewall \
  --resource-group HubSpoke-RG \
  --location centralindia

az network firewall ip-config create \
  --firewall-name HubFirewall \
  --name FWConfig \
  --public-ip-address FirewallPublicIP \
  --resource-group HubSpoke-RG \
  --vnet-name Hub-VNet
```

📌 **Firewall Private IP**: `11.0.1.4` (used in UDRs below)

---

### 6️⃣ Create User Defined Routes (UDRs)

#### Spoke1

```bash
az network route-table create \
  --name Spoke1-RouteTable \
  --resource-group HubSpoke-RG \
  --location centralindia

az network route-table route create \
  --resource-group HubSpoke-RG \
  --route-table-name Spoke1-RouteTable \
  --name RouteToFirewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 11.0.1.4

az network vnet subnet update \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke1-VNet \
  --name Workload \
  --route-table Spoke1-RouteTable
```

#### Spoke2

```bash
az network route-table create \
  --name Spoke2-RouteTable \
  --resource-group HubSpoke-RG \
  --location centralindia

az network route-table route create \
  --resource-group HubSpoke-RG \
  --route-table-name Spoke2-RouteTable \
  --name RouteToFirewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 11.0.1.4

az network vnet subnet update \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke2-VNet \
  --name Workload \
  --route-table Spoke2-RouteTable
```

---

### 7️⃣ Create Network Security Groups (NSGs)

```bash
az network nsg create \
  --resource-group HubSpoke-RG \
  --name Spoke1-NSG \
  --location centralindia

az network nsg create \
  --resource-group HubSpoke-RG \
  --name Spoke2-NSG \
  --location centralindia

az network vnet subnet update \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke1-VNet \
  --name Workload \
  --network-security-group Spoke1-NSG

az network vnet subnet update \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke2-VNet \
  --name Workload \
  --network-security-group Spoke2-NSG
```

---

### 8️⃣ Deploy Azure Bastion

```bash
az network public-ip create \
  --name BastionPublicIP \
  --resource-group HubSpoke-RG \
  --sku Standard \
  --location centralindia \
  --allocation-method Static

az network bastion create \
  --name HubBastion \
  --public-ip-address BastionPublicIP \
  --resource-group HubSpoke-RG \
  --vnet-name Hub-VNet \
  --location centralindia
```

---

### 9️⃣ Create Virtual Machines

#### Spoke1 VM

```bash
az vm create \
  --name Spoke1-VM \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke1-VNet \
  --subnet Workload \
  --image UbuntuLTS \
  --admin-username azureuser \
  --authentication-type password \
  --admin-password 'Kirtan@12345' \
  --nsg Spoke1-NSG \
  --private-ip-address 12.0.1.4
```

#### Spoke2 VM

```bash
az vm create \
  --name Spoke2-VM \
  --resource-group HubSpoke-RG \
  --vnet-name Spoke2-VNet \
  --subnet Workload \
  --image UbuntuLTS \
  --admin-username azureuser \
  --authentication-type password \
  --admin-password 'Kirtan@12345' \
  --nsg Spoke2-NSG \
  --private-ip-address 13.0.1.4
```

---

### 🔟 Firewall Rules (via Portal)

✅ Allow SSH from `11.0.2.0/26` → `12.0.0.0/24` & `13.0.0.0/24` (TCP 22)
✅ Allow ICMP (ping)
✅ Allow outbound to trusted domains (`*.microsoft.com`, `*.ubuntu.com`, `*.github.com`, `*.google.com`)

---

### 1️⃣1️⃣ Log Analytics & Monitoring

```bash
az monitor log-analytics workspace create \
  --resource-group HubSpoke-RG \
  --workspace-name HubSpoke-LogAnalytics \
  --location centralindia
```

---

## ✅ Architecture Overview

* **Hub VNet (11.0.0.0/16):** Firewall + Bastion
* **Spoke1 VNet (12.0.0.0/24):** Workload + VM
* **Spoke2 VNet (13.0.0.0/24):** Workload + VM
* **Firewall:** Centralized routing & security policies
* **Bastion:** Secure SSH/RDP without public IPs
* **NSGs + UDRs:** Enforce traffic flow and security boundaries
* **Log Analytics:** Central monitoring & diagnostics

---

## ⚖️ Pros & Cons of Hub-and-Spoke

### ✅ Advantages

* Centralized **security enforcement** via Firewall/NVA
* Easier **monitoring, governance, and compliance**
* Scalability – add more spokes easily
* Reduced complexity compared to full mesh networking
* Centralized **shared services** (Bastion, DNS, monitoring)

### ❌ Disadvantages

* **Hub as a bottleneck** (all traffic flows through it)
* Firewall adds **latency & cost**
* More complex **routing configuration** (UDRs, peering)
* Single point of failure if the Hub isn’t highly available

---

## 🔧 Post-Deployment Checklist

1. Configure & validate firewall rules
2. Test **VM-to-VM communication via Bastion**
3. Verify **traffic inspection** through firewall
4. Set up **alerts in Log Analytics**
5. Enable **backup & disaster recovery**

---
