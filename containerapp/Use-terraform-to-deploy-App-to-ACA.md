---
title: Use terraform to deploy App to ACA
categories: Azure
description: Use terraform to deploy App to ACA
date: 2023-08-31 19:57:56
tags:
password:
message:
---

Here's a step-by-step demonstration for deploying an NGINX app to Azure Container Apps (ACA) using Terraform:

1. **Install Terraform:**

Make sure you have Terraform installed on your system. You can follow the installation instructions here: https://learn.hashicorp.com/tutorials/terraform/install-cli

2. **Create a Working Directory:**

Create a new directory for your Terraform project and navigate to it in your terminal.

3. **Create the Terraform Configuration:**

Create a file named `main.tf` in your project directory with the following content:

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_container_app" "nginx_app" {
  name                = "mynginxapp"
  resource_group_name = "MyResourceGroup"
  location            = "East US"

  os_type = "linux"

  image_source_type = "publicDockerHub"
  public_image_name = "nginx:latest"

  dns_name_label = "mynginxapp"

  app_settings = {
    "PORT" = "80"
  }
}
```

Replace `"MyResourceGroup"` and `"East US"` with your desired resource group and location. You can customize other settings as needed.

4. **Initialize Terraform:**

In your terminal, navigate to your project directory and run the following command to initialize Terraform:

```bash
terraform init
```

5. **Deploy the NGINX App:**

Run the following command to deploy the NGINX app to Azure Container Apps:

```bash
terraform apply
```

Terraform will show you the changes it's going to make. If everything looks good, type `yes` to confirm and proceed with the deployment.

6. **Access the Deployed App:**

After the deployment is complete, you can access the deployed NGINX app using the DNS name label you provided (`mynginxapp`). Open a web browser and navigate to `http://mynginxapp.<region>.azurecontainer.io`, where `<region>` is the Azure region you specified.

That's it! You've successfully deployed an NGINX app to Azure Container Apps using Terraform. Remember that this example provides a basic deployment and there are many additional configuration options available in Terraform to customize your deployment according to your needs.

Please note that Terraform and Azure services often receive updates, so make sure to consult the official documentation for the latest information and best practices.
