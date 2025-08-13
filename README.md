# Automated Secure Azure Infrastructure with Terraform

### Created by Drayton Rolle

---

## 📝 Project Summary

This project demonstrates the automation of a secure cloud infrastructure deployment on Microsoft Azure using Terraform. The goal was to take a manually-built, secure environment and codify it using Infrastructure as Code (IaC) principles. This ensures the deployment is repeatable, version-controlled, and free from human error.

This project showcases advanced cloud engineering skills, including IaC, cloud networking, security, and automation, which are highly sought after in DevOps and Cloud Engineering roles. It proves the ability to translate manual configurations into efficient, automated workflows.

---

## 🚀 Key Concepts Demonstrated

* **Infrastructure as Code (IaC):** Used Terraform to define and manage all cloud resources declaratively.
* **Automation:** Automated the entire provisioning process of a multi-component environment with a few commands.
* **Resource Management:** Leveraged Terraform's state management to create, plan, and destroy resources in a controlled manner.
* **Secure Networking:** Programmatically defined a Virtual Network (VNet) with segmented subnets and a Network Security Group (NSG) to act as a firewall.
* **Secure Authentication:** Utilized SSH keys for secure, passwordless authentication to the virtual machine, a best practice for automated systems.

---

## 🛠️ Tools & Technologies

* **Terraform:** The core IaC tool used to define and deploy the infrastructure.
* **Microsoft Azure:** The cloud platform where the resources were deployed.
* **Azure CLI:** Used for authentication between the local machine and Azure.
* **Visual Studio Code:** The code editor used to write the Terraform configuration files.

---

## 📖 Step-by-Step Guide

This guide provides all the steps needed to replicate this project.

### Phase 0: Prerequisites & Setup

1.  **Install Visual Studio Code:** Download and install from [code.visualstudio.com](https://code.visualstudio.com/).
2.  **Install Terraform:** Download the appropriate binary for your OS from [terraform.io/downloads.html](https://www.terraform.io/downloads.html) and add it to your system's PATH.
3.  **Install Azure CLI:** Download and install the MSI for your OS from the [Azure CLI documentation](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
4.  **Authenticate with Azure:** Open a new terminal and run `az login`, then follow the browser prompts to sign in to your Azure account.

### Phase 1: Writing the Infrastructure Code

1.  **Create Project Folder:** Create a new folder for your project (e.g., `Terraform-Azure-Project`).
2.  **Open in VS Code:** Open this folder in Visual Studio Code.
3.  **Create `main.tf`:** Create this file and add the following code to configure the provider and resource group.
    ```terraform
    terraform {
      required_providers {
        azurerm = {
          source  = "hashicorp/azurerm"
          version = "~>3.0"
        }
      }
    }
    provider "azurerm" {
      features {}
    }
    resource "azurerm_resource_group" "rg" {
      name     = "Automated-SecuredAppRG"
      location = "East US"
    }
    ```
4.  **Create `network.tf`:** Create this file and add the following code to define the VNet, subnets, and NSG.
    ```terraform
    resource "azurerm_virtual_network" "vnet" {
      name                = "Automated-AppVNet"
      address_space       = ["10.0.0.0/16"]
      location            = azurerm_resource_group.rg.location
      resource_group_name = azurerm_resource_group.rg.name
    }
    resource "azurerm_subnet" "web" {
      name                 = "WebSubnet"
      resource_group_name  = azurerm_resource_group.rg.name
      virtual_network_name = azurerm_virtual_network.vnet.name
      address_prefixes     = ["10.0.1.0/24"]
    }
    resource "azurerm_subnet" "app" {
      name                 = "AppSubnet"
      resource_group_name  = azurerm_resource_group.rg.name
      virtual_network_name = azurerm_virtual_network.vnet.name
      address_prefixes     = ["10.0.2.0/24"]
    }
    resource "azurerm_subnet" "db" {
      name                 = "DbSubnet"
      resource_group_name  = azurerm_resource_group.rg.name
      virtual_network_name = azurerm_virtual_network.vnet.name
      address_prefixes     = ["10.0.3.0/24"]
    }
    resource "azurerm_network_security_group" "nsg" {
      name                = "WebSubnet-nsg"
      location            = azurerm_resource_group.rg.location
      resource_group_name = azurerm_resource_group.rg.name
      security_rule {
        name                       = "Allow_Web_Traffic"
        priority                   = 100
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_ranges    = ["80", "443"]
        source_address_prefix      = "Internet"
        destination_address_prefix = "*"
      }
    }
    resource "azurerm_subnet_network_security_group_association" "nsg_association" {
      subnet_id                 = azurerm_subnet.web.id
      network_security_group_id = azurerm_network_security_group.nsg.id
    }
    ```
5.  **Generate SSH Key:** If you don't have one, open a terminal and run `ssh-keygen -t rsa -b 4096`, accepting the defaults.
6.  **Create `vm.tf`:** Create this file and add the following code to define the virtual machine.
    ```terraform
    resource "azurerm_network_interface" "nic" {
      name                = "WebServer-nic"
      location            = azurerm_resource_group.rg.location
      resource_group_name = azurerm_resource_group.rg.name
      ip_configuration {
        name                          = "internal"
        subnet_id                     = azurerm_subnet.web.id
        private_ip_address_allocation = "Dynamic"
      }
    }
    resource "azurerm_linux_virtual_machine" "vm" {
      name                  = "Automated-WebServer01"
      resource_group_name   = azurerm_resource_group.rg.name
      location              = azurerm_resource_group.rg.location
      size                  = "Standard_B1s"
      admin_username        = "draytonadmin"
      network_interface_ids = [azurerm_network_interface.nic.id]
      admin_ssh_key {
        username   = "draytonadmin"
        public_key = file("~/.ssh/id_rsa.pub")
      }
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
    }
    ```

### Phase 2: Deployment & Cleanup

1.  **Navigate to Project Folder:** Open a terminal and `cd` into your project directory.
2.  **Initialize Terraform:** Run `terraform init`.
3.  **Plan Deployment:** Run `terraform plan` to review the resources that will be created.
4.  **Apply Plan:** Run `terraform apply` and type `yes` when prompted to build the infrastructure.
5.  **Verify:** Log in to the Azure Portal to see the created resources in the `Automated-SecuredAppRG` resource group.
6.  **Destroy Infrastructure:** When finished, run `terraform destroy` to cleanly delete all resources from Azure.

---

## 📸 Portfolio Screenshots

**1. Successful `terraform init`**
*Shows the successful initialization of the Terraform backend and provider plugins.*
![Terraform Init](images/Terraform%20init.png)

**2. Successful `terraform plan`**
*The end of the plan output, confirming that 9 resources were planned for creation.*
![Terraform Plan](images/Terraform%20plan.png)

**3. Successful `terraform apply`**
*The end of the apply output, confirming that all 9 resources were successfully added.*
![Terraform Apply](images/Apply%20Output.png)

**4. Final Resources in Azure Portal**
*A screenshot of the `Automated-SecuredAppRG` resource group in the Azure Portal, showing all the resources created by Terraform.*
![Azure Portal Resources](images/Azure%20Portal%20Resources.png)
