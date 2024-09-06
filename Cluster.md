Here’s a detailed `README.md` file that includes the prerequisites, steps, and manifest files to create an AKS (Azure Kubernetes Service) cluster with Terraform. It also includes necessary comments within the Terraform files.

---

## README: AKS Cluster Deployment Using Terraform

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

#### 3. Initialize Terraform

Before deploying, initialize the Terraform workspace to download the required provider plugins:

```bash
terraform init
```

#### 4. Modify Variables (Optional)

You can modify the variable values in the `terraform.tfvars` file to suit your needs (e.g., resource group name, AKS cluster name, node count, etc.). This file might look like this:

```hcl
resource_group_name = "aks-resource-group"
location            = "East US"
node_count          = 2
```

#### 5. Apply Terraform Configuration

Run the following command to deploy the AKS cluster:

```bash
terraform apply
```

Once applied, Terraform will provision the Azure resources, including the AKS cluster.

---

### Terraform Files

#### `main.tf`

This is the main configuration file for Terraform that defines the AKS cluster and other related Azure resources.

```hcl
provider "azurerm" {
  features {}
}

# Resource group for the AKS cluster
resource "azurerm_resource_group" "aks" {
  name     = var.resource_group_name
  location = var.location
}

# Azure Kubernetes Service (AKS) cluster
resource "azurerm_kubernetes_cluster" "aks" {
  name                = "aks-cluster"
  location            = azurerm_resource_group.aks.location
  resource_group_name = azurerm_resource_group.aks.name
  dns_prefix          = "akscluster"

  default_node_pool {
    name       = "default"
    node_count = var.node_count   # The number of nodes in the AKS cluster
    vm_size    = "Standard_DS2_v2"
  }

  identity {
    type = "SystemAssigned"   # Use system-assigned managed identity for the AKS cluster
  }

  network_profile {
    network_plugin    = "azure"   # Use Azure CNI for the AKS cluster
    load_balancer_sku = "standard"
  }
}

# Output the kubeconfig to manage the AKS cluster
output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_admin_config_raw
  sensitive = true
}
```

#### `variables.tf`

This file defines the input variables that can be customized when deploying the AKS cluster.

```hcl
variable "resource_group_name" {
  description = "The name of the resource group to create."
  type        = string
}

variable "location" {
  description = "The Azure region in which to create resources."
  type        = string
  default     = "East US"
}

variable "node_count" {
  description = "The number of nodes in the default node pool."
  type        = number
  default     = 1
}
```

#### `outputs.tf`

This file defines the outputs from the Terraform deployment, such as the `kube_config` to interact with the AKS cluster.

```hcl
output "kube_config" {
  value     = azurerm_kubernetes_cluster.aks.kube_admin_config_raw
  description = "Kubeconfig to interact with the AKS cluster"
  sensitive = true
}
```

#### `terraform.tfvars`

This file contains the values for the variables defined in `variables.tf`. You can customize these values based on your requirements.

```hcl
resource_group_name = "aks-resource-group"
location            = "East US"
node_count          = 2
```


