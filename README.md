# 📘 Terraform Azure Core Infrastructure Deployment
 <img width="1400" height="730" alt="Thumbnail (3)" src="https://github.com/user-attachments/assets/cf2946eb-91e0-4f49-a3cc-1d1caba1d477" />

## 🎥 Watch Me Build This!

### Part 1  
👉 **[Watch Me Build This!](https://www.loom.com/share/d05bc2aa88bc4d8d98b1a178d7377cb6)**

### Part 2  
👉 **[Watch Me Build This!](https://www.loom.com/share/131b286744154eee9d83d29234d7f06b)**



> This project is demonstrated through a short Loom video, showing the full build, verification in the Azure Portal, and complete teardown.

---

## 🚀 Overview

This project provisions a complete Azure environment using **Terraform Infrastructure-as-Code (IaC)**.

The deployment includes:

- Resource Group  
- Virtual Network (VNet)  
- Subnet  
- Network Security Group (NSG) with SSH rule  
- Public IP (Standard SKU)  
- Network Interface (NIC)  
- Storage Account (with globally unique name)  
- Linux Virtual Machine (Ubuntu 22.04, password authentication)

It demonstrates the full lifecycle:

**Deploy → Verify → Destroy**

---

## 🧱 Architecture (Conceptual)

The Terraform config creates:

- A Resource Group as the container  
- A VNet + Subnet for networking  
- An NSG with an inbound SSH rule on port 22  
- A Standard Public IP for external access  
- A NIC attached to the subnet and public IP  
- A Storage Account for general/infra usage  
- A Linux VM running Ubuntu 22.04

---

## 📂 File Structure

```text
terraform-azure-core-infra/
├── main.tf
├── variables.tf
├── outputs.tf
├── README.md
└── LICENSE
```
🛠️ How to Use This Project
1. Prerequisites

   - Terraform installed

   - Azure CLI installed

   - An Azure subscription

   - Logged in via az login
2. Initialize Terraform
 ```text
 terraform init
```
3. Format and Validate
```text
terraform fmt
terraform validate
```
4. Preview the Plan
```text
terraform plan -out tfplan
```
5. Apply the Deployment
```text
terraform apply tfplan
```
After this, you’ll have:

Resource Group: rg-coretf-lab

VM: coretf-vm

VNet, Subnet, NSG, NIC, Public IP, Storage Account
6. Destroy the Environment
```text
terraform destroy -auto-approve
```
This removes all resources that were created by this configuration.


---

🧪 Troubleshooting Notes

These are real issues encountered and resolved while building this lab:

Public IP SKU Error

Free-tier subscriptions often do not allow Basic IPv4 public IPs.

Fix:
Use
 sku = "Standard" for azurerm_public_ip.
---
Storage Account Name Error

Storage account names must:

Be all lowercase

Use only letters and numbers

Be globally unique

Be 3–24 characters long

Fix:
Removed the dash and appended a random string:

```text
coretfstorage${random_string.storage_suffix.result} 
```
---
SSH Key Path Error

Terraform couldn’t find the SSH key file referenced in the config.

Fix (for this lab):
Switched to password authentication to keep the demo simple and focused on Terraform.

---

Resource Renaming/Recreation

Changing naming (like name_prefix or hardcoded names) can cause Terraform to destroy and recreate resources.

Fix:
Accepted as normal behavior when refactoring; Terraform handled the replacements automatically.

---

## 🧠 What I Learned

This project helped me practice how real infrastructure is deployed in an enterprise setting.  
Using Terraform forced me to think in terms of desired state, version-controlled configs, and repeatable builds instead of clicking around the Azure Portal.

Working through naming constraints, SKU limitations, and resource recreation showed how cloud teams handle real Azure errors and refactor infrastructure safely.  
Deploying, validating, and destroying the environment reinforced the full IaC lifecycle — which is exactly how production teams avoid configuration drift.



## 🔧 How This Applies in Real Cloud Work

In enterprise environments, Terraform is essential for:

- Enforcing consistent resource standards across environments  
- Automating Azure deployments with predictable outputs  
- Rebuilding or scaling environments quickly  
- Preventing manual mistakes common in portal-driven setups  
- Managing cloud infrastructure as code through Git workflows  

This lab reflects the same workflow used by Cloud, DevOps, and Platform Engineering teams in real organizations.

---

📦 Full Terraform Configuration

Below are the three main Terraform files used in this project for quick reference.

main.tf
```text

terraform {
  required_version = ">= 1.3.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Random suffix for globally unique storage account name
resource "random_string" "storage_suffix" {
  length  = 5
  upper   = false
  special = false
}

# Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "rg-coretf-lab"
  location = var.location

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "vnet" {
  name                = "coretf-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Subnet
resource "azurerm_subnet" "subnet" {
  name                 = "coretf-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Public IP (Standard SKU)
resource "azurerm_public_ip" "pip" {
  name                = "coretf-pip"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Standard"

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Network Security Group
resource "azurerm_network_security_group" "nsg" {
  name                = "coretf-nsg"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  security_rule {
    name                       = "Allow-SSH"
    priority                   = 1001
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "22"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Network Interface
resource "azurerm_network_interface" "nic" {
  name                = "coretf-nic"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Attach NSG to NIC
resource "azurerm_network_interface_security_group_association" "nic_nsg" {
  network_interface_id      = azurerm_network_interface.nic.id
  network_security_group_id = azurerm_network_security_group.nsg.id
}

# Storage Account (name corrected for Azure rules)
resource "azurerm_storage_account" "sa" {
  name                     = "coretfstorage${random_string.storage_suffix.result}"
  resource_group_name      = azurerm_resource_group.rg.name
  location                 = azurerm_resource_group.rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

# Linux VM (Ubuntu, password auth)
resource "azurerm_linux_virtual_machine" "vm" {
  name                = "coretf-vm"
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
  size                = "Standard_B1s"

  admin_username = var.admin_username
  admin_password = var.admin_password
  disable_password_authentication = false

  network_interface_ids = [
    azurerm_network_interface.nic.id
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  tags = {
    environment = "lab"
    project     = "terraform-azure-core-infra"
  }
}

```
variables.tf

```text
variable "name_prefix" {
  description = "Prefix for all resources"
  type        = string
  default     = "coretf-"
}

variable "location" {
  description = "Azure region for all resources"
  type        = string
  default     = "eastus"
}

variable "admin_username" {
  description = "Admin username for the VM"
  type        = string
  default     = "azureuser"
}

variable "admin_password" {
  description = "Admin password for the VM"
  type        = string
  default     = "SomeP@ssw0rd123!"
}

```
outputs.tf

```text
output "resource_group_name" {
  description = "Name of the deployed resource group"
  value       = azurerm_resource_group.rg.name
}

output "vm_public_ip" {
  description = "Public IP assigned to the Linux VM"
  value       = azurerm_public_ip.pip.ip_address
}

output "storage_account_name" {
  description = "Name of the deployed storage account"
  value       = azurerm_storage_account.sa.name
}
```





