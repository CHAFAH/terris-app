
## AKS Cluster Deployment Using Terraform

This guide provides step-by-step instructions for setting up an Azure Kubernetes Service (AKS) cluster using Terraform. The deployment also includes Helm charts to manage application deployment on the AKS cluster.

### Prerequisites

Before proceeding, make sure you have the following tools installed on your local machine:

1. **Azure Account**:
   - You can sign up for a free Azure account [here](https://azure.microsoft.com/free/).
2. **Azure CLI**:
   - Install the Azure CLI by following the instructions [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).
3. **Terraform**:
   - Install Terraform by following the official guide [here](https://learn.hashicorp.com/tutorials/terraform/install-cli).
4. **kubectl**:
   - Install Kubernetes CLI (`kubectl`) to manage Kubernetes clusters.
   - Instructions can be found [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
5. **Helm**:
   - Install Helm to manage Kubernetes applications.
   - Instructions can be found [here](https://helm.sh/docs/intro/install/).

---

### Steps to Create the AKS Cluster

#### 1. Clone the Repository

Clone the GitHub repository containing the Terraform and Helm chart files:

```bash
git clone https://github.com/your-repo/aks-terraform.git
cd aks-terraform
```

#### 2. Configure Azure Authentication

Ensure that you are authenticated in Azure using the Azure CLI:

```bash
az login
```

### Terraform Files

Below is a more detailed set of Terraform configuration files that are properly structured to create an AKS cluster on Azure, deploy your application using Helm charts, and securely expose the necessary ports. This configuration includes proper commenting and explanations.

### `main.tf`

```hcl
# Main Terraform configuration for deploying AKS Cluster and other resources

[provider "azurerm" {
  features {}
}

# Resource group definition
resource "azurerm_resource_group" "rg" {
  name     = "my-aks-resource-group"
  location = "East US"
}

# Virtual network definition
resource "azurerm_virtual_network" "vnet" {
  name                = "aks-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

# Subnet for the AKS cluster
resource "azurerm_subnet" "aks_subnet" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Network security group for AKS
resource "azurerm_network_security_group" "nsg" {
  name                = "aks-nsg"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}

# Allow necessary ports for the application (frontend, backend, mongo)
resource "azurerm_network_security_rule" "allow_app_ports" {
  name                        = "AllowAppPorts"
  priority                    = 100
  direction                   = "Inbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "*"
  destination_port_ranges     = ["3000", "3001", "27017"]
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.rg.name
  network_security_group_name = azurerm_network_security_group.nsg.name
}

# AKS cluster definition
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-cluster"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  dns_prefix          = "myakscluster"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_DS2_v2"
    vnet_subnet_id = azurerm_subnet.aks_subnet.id
  }

  identity {
    type = "SystemAssigned"
  }
}

# Output the AKS credentials for use with kubectl
output "kube_config" {
  value = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive = true

```

### `variables.tf`

```hcl
# Variable file for the AKS deployment
variable "resource_group_name" {
  type        = string
  description = "The name of the resource group"
  default     = "my-aks-resource-group"
}

variable "location" {
  type        = string
  description = "The location of the resources"
  default     = "East US"
}
```

### `outputs.tf`

```hcl
# Outputs the Kubernetes config for accessing the cluster
output "kube_config" {
  description = "Kube config file to access the cluster"
  value       = azurerm_kubernetes_cluster.aks.kube_config_raw
  sensitive   = true
}

# Output the AKS cluster name
output "aks_cluster_name" {
  description = "The name of the AKS Cluster"
  value       = azurerm_kubernetes_cluster.aks.name
}
```

---
---

### Steps to Apply the Terraform and Helm Charts

#### **Step 1: Initialize Terraform**

```bash
terraform init
```

This will initialize Terraform and download the necessary providers.

#### **Step 2: Apply Terraform Configuration**

```bash
terraform apply
```

This command will create the AKS cluster, VPC, subnets, and security rules to allow traffic on ports `3000`, `3001`, and `27017`.

#### **Step 3: Get AKS Credentials**

Once the Terraform apply completes, you can get the Kubernetes configuration to interact with the cluster:

```bash
terraform output kube_config > ~/.kube/config
```

This allows `kubectl` to interact with the newly created AKS cluster.


1. **Set up the Kubernetes context** using the kubeconfig file from the Terraform output:

```bash
az aks get-credentials --resource-group aks-resource-group --name aks-cluster
```
---

### Conclusion

This guide provides the detailed steps to create an AKS cluster using Terraform, deploy services using Helm, and access the services via `NodePort`. You can modify the values and configurations based on your specific requirements.

