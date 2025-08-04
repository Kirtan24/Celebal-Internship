# 🚀 Azure Hub-and-Spoke Architecture – Step-by-Step CLI Implementation

This document outlines the complete CLI-based setup of a secure and scalable **Hub-and-Spoke network architecture** on Microsoft Azure. This implementation includes the creation of resource groups, virtual networks (VNets), peering, subnets, Azure Firewall, Bastion access, NSGs, VMs, routing policies, and logging configuration.

---

## 🛠 1. Create Resource Group

```bash
az group create --name HubSpoke-RG --location centralindia
```

## 🌐 2. Create Hub Virtual Network with Subnets

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

## 🌐 3. Create Spoke Virtual Networks

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

## 🔄 4. Create VNet Peering

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

## 🔥 5. Deploy Azure Firewall

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

📌 **Firewall Private IP**: 11.0.1.4 (used in routing rules below)

## 🚦 6. Create User Defined Routes (UDRs)

### Spoke1

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

### Spoke2

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

## 🛡 7. Create Network Security Groups (NSGs)

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

## 🔐 8. Deploy Azure Bastion

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

## 💻 9. Create Virtual Machines

### Spoke1 VM

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

### Spoke2 VM

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

## 📜 10. Firewall Rules (Configured via Portal)

✅ Allow SSH from 11.0.2.0/26 to 12.0.0.0/24 and 13.0.0.0/24 on TCP port 22

✅ Allow ICMP for ping-based testing

✅ Allow outbound access to specific domains via Application Rules:
- *.microsoft.com
- *.ubuntu.com
- *.github.com
- *.google.com

## 📊 11. Log Analytics & Monitoring

### Create Log Analytics Workspace

```bash
az monitor log-analytics workspace create \
  --resource-group HubSpoke-RG \
  --workspace-name HubSpoke-LogAnalytics \
  --location centralindia
```

---

## 🎯 Architecture Overview

This implementation creates a secure hub-and-spoke network topology with the following components:

- **Hub VNet (11.0.0.0/16)**: Contains Azure Firewall and Bastion services
- **Spoke1 VNet (12.0.0.0/24)**: Workload network with VM
- **Spoke2 VNet (13.0.0.0/24)**: Workload network with VM
- **VNet Peering**: Enables communication between hub and spokes
- **Azure Firewall**: Centralized security and routing control
- **Azure Bastion**: Secure RDP/SSH access without public IPs
- **User Defined Routes**: Forces traffic through the firewall
- **Network Security Groups**: Additional layer of security
- **Log Analytics**: Centralized monitoring and logging

## 🔧 Post-Deployment Steps

1. Configure firewall rules via Azure Portal
2. Test connectivity between VMs through Bastion
3. Verify traffic is routing through the firewall
4. Set up monitoring and alerting
5. Configure backup and disaster recovery as needed

## 📝 Notes

- All resources are deployed in Central India region
- Private IP addresses are statically assigned for predictable routing
- Passwords are used for simplicity - consider using SSH keys in production
- Firewall rules need to be configured manually via the Azure Portal
- Log Analytics workspace is created for future monitoring setup
