---
layout:     post
title:      "Azure Container App TCP port access"
subtitle:   "Terraform basic configuration"
date:       2025-07-17 18:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

If you want to expose a TCP port from a Container App Azure will require you to deploy it within a [Virtual Network](https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview#tcp). The interesting part is that - unless you have other requirements - is pretty much all you need. No Application Gateway, Private DNS Zone, Private Link... And you can still control access through the Container App ingress configuration, making it very easy to manage access rights.

<p align="center">
    <img src="{{ site.baseurl }}/img/post-terraform-azure-container-apps-with-virtual-network/container-app-tcp.png" />
</p>

To attach a virtual network to a container app, the associated container app environment must be configured to use a [workload profile](https://learn.microsoft.com/en-us/azure/container-apps/workload-profiles-overview), which is then linked to a subnet within the virtual network.

In Terraform, this means configuring four separate resources which you can see split below in three separate files.

`network.tf`

```terraform
# network.tf
resource "azurerm_virtual_network" "my_vnet" {
  name                = "my-vnet"
  location            = "westeurope"
  resource_group_name = "my-rg"
  address_space       = ["10.0.0.0/24"]
}

resource "azurerm_subnet" "my_subnet" {
  name                              = "my-subnet"
  resource_group_name               = "my-rg"
  virtual_network_name              = azurerm_virtual_network.my_vnet.name
  address_prefixes                  = ["10.0.0.64/26"]
  private_endpoint_network_policies = "Disabled"

  delegation {
    name = "Microsoft.App/environments"
    service_delegation {
      name    = "Microsoft.App/environments"
      actions = ["Microsoft.Network/virtualNetworks/subnets/join/action"]
    }
  }
}
```

<hr>

`container-app-env.tf`

Note that the parameter `infrastructure_resource_group_name` is not known in advance. It is a platform-managed resource group created to host infrastructure resources, and it will be defined during the creation of the container app environment. It might be inferable because it seems to follow this pattern `ME_<container-app-environment-name>_<resource-group-name>_<region>`. But in practice I have followed a staggered deployment approach in which it is commented out on the first deployment. Once the container app environment is created successfully, you can inspect the Container App Enviornment JSON definition - where it is defined as `infrastructureResourceGroup` - or run `terraform plan` to see it, and add it to your configuration.

```terraform
# container-app-env.tf
resource "azurerm_container_app_environment" "my_cae" {
  name                               = "my-cae"
  resource_group_name                = "my-rg"
  location                           = "westeurope"
  log_analytics_workspace_id         = azurerm_log_analytics_workspace.my_law.id    # Needs to be created separately
  infrastructure_subnet_id           = azurerm_subnet.my_subnet.id

  workload_profile {
      name                  = "Consumption"
      workload_profile_type = "Consumption"
    }
  
  # infrastructure_resource_group_name =
}
```

<hr>

`container-app.tf`

The definition of an Azure Container App can vary a lot depending on your needs. It might require environment variables, secrets, registry and ingress configuration, volume mounts... This is a simplified example running a basic setup that exposes publicly the TCP port 4840.

```terraform
# container-app.tf
resource "azurerm_container_app" "my_ca" {
  name                         = "my-container-app"
  container_app_environment_id = azurerm_container_app_environment.example.id
  resource_group_name          = azurerm_resource_group.example.name
  revision_mode                = "Single"
  workload_profile_name        = "Consumption"  # Needs to match the workload_profile defined in the container app environment

  ingress {
    allow_insecure_connections = false
    fqdn = null
    transport = "tcp"
    target_port = 4840
    external_enabled = true
    exposed_port = null
  }

  template {
    container {
      name   = "my-container-app"
      image  = "my-image"
      cpu    = 0.25
      memory = "0.5Gi"
    }
  }
}
```

<hr>

That is all. Once deployed, Azure Container Apps will provide you with an fqdn you can use to connect to the service. Something along this format `<container-app-name>.<unique-word-combination>.<region>.azurecontainerapps.io`. Bear in mind that depending on the service you are running, and on the way you are connecting to it, you might need to append the port to that fqdn.