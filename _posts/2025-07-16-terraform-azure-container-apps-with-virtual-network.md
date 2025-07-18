---
layout:     post
title:      "Azure Container App TCP port access"
subtitle:   "How to configure the resources needed in Terraform"
date:       2025-07-17 18:00:00
author:     "Juan Carlos Mayo"
header-img: "img/post-default-bg.png"
---

If you want to expose a TCP port from a Container App Azure will require you to deploy it within a [Virtual Network](https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview#tcp). The interesting part is that that is pretty much all you need. No Application Gateway, Private DNS Zone, Private Link... Accessibility is still managed by the Container App ingress configuration so even though it is publicly accessible it is easy to set restrictions on who can access your application.

<p align="center">
    <img src="{{ site.baseurl }}/img/post-terraform-azure-container-apps-with-virtual-network/container-app-tcp.png" />
</p>

What it is important to note is that the Container App Environment must be configured to use a [workload profile](https://learn.microsoft.com/en-us/azure/container-apps/workload-profiles-overview), which will enable us to link it to a subnet, and hence to a virtual network.

In Terraform, the resources can be configured in the following separate files.

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

Note that the parameter `infrastructure_resource_group_name` is not known in advance. Instead, it is defined upon creation. It might be inferable because it seems to follow this pattern `ME_<container-app-environment-name>_<resource-group-name>_<region>` but in practice it might require a staggered approach in which you should comment it out on the first deployment. `terraform plan` will make it visible after that, and that is when you can add it.

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

The definition of an Azure Container App can vary a lot depending on your needs. Environment variables, secrets, registry and ingress configuration, volume mounts... Below is a simplified example that highlights what is needed to make the TCP port available.

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

That is all. Once deployed, Azure Container Apps will provide you with an fqdn you can use to connect to the service. Something along this format `<container-app-name>.<unique-word-combination>.<region>.azurecontainerapps.io`. Just note, that depending on the service you are running, and on the way you are connecting to it, you might need to append the port to that fqdn.