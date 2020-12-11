# Terraform 101

## What is Terraform?
Terraform is an infra-as-code framework that figures out how to make your desired state into reality.

* Allows you to declare the desired state of your infrastructure 
* Will go figure out what the current state of your infrastructure is
* Diffs the current vs desired states and comes up with a series of actions that must be taken
* Performs the series of actions

## Architecture
![Overiew](assets/overview.png)
Source: https://medium.com/hackernoon/terraform-openstack-ansible-d680ea466e22

## Key Terms
* Statefile - a mapping of the resources declared in your code to what exists in the infrastructure provider.
* Plan - 
* Apply
* Provider

## Getting Started
This tutorial is going to focus on learning Terraform basics using Azure.
1. Install Terraform:
    ```
    brew tap hashicorp/tap && brew install hashicorp/tap/terraform
    ```
    If you already have Terraform, upgrade to the latest version:
    ```
    brew upgrade hashicorp/tap/terraform
    ```
1. Install the Azure CLI:
    ```
    brew update && brew install azure-cli
    ```
1. Login into Azure
    1. Authenticate your Azure CLI
        ```
        $ az login
        ```
    1. List your subscriptions and pick which one you want to use
        ```
        $ az account list --query "[].{name:name, subscriptionId:id}"
        ```
    1. Set your subscription
        ```
        $ az account set --subscription="<subscription_id>"
        ```

## Create your Terraform file
Let's call it `main.tf` and fill it with a single provider block. A provider is responsible for understanding API interactions and exposing resources. Most providers configure a specific infrastructure platform (either cloud or self-hosted).

In this case, we are pulling in the AzureRM provider.
```
provider "azurerm" {
  version = "~>2.0"
  features {}
}
```

## Initialize Terraform
The terraform init command is used to initialize a working directory containing Terraform configuration files.
```
$ terraform init
```

Two things happened behind the scenes:
* `.terraform` folder was created. This will eventually contain:
    * Binaries of any providers that are used in your code
    * `lock.json` - mapping of providers to specific commit hashes of the versions deployed
* `azurerm` provider was downloaded from the [Terraform Registry](https://registry.terraform.io).

Now we're ready to start defining resources.

## Resource Block
Resources are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects, such as virtual networks, compute instances, or higher-level components such as DNS records.

In this case, we are creating a new Azure `Resource Group` (note this is an Azure specific construct, similar to a folder for associating your Azure resources). Add in the following block to the end of your `main.tf` file:
```
resource "azurerm_resource_group" "example_rg" {
  name = "<uid>-test"
  location = "westus"
}

resource "azurerm_network_security_group" "example_nsg" {
  name                = "<uid>-test-nsg"
  location            = azurerm_resource_group.example_rg.location
  resource_group_name = azurerm_resource_group.example_rg.name
}

resource "azurerm_network_security_rule" "example_nsr" {
  name                        = "<uid>-test-nsr"
  priority                    = 100
  direction                   = "Outbound"
  access                      = "Allow"
  protocol                    = "Tcp"
  source_port_range           = "5555"
  destination_port_range      = "6666"
  source_address_prefix       = "*"
  destination_address_prefix  = "*"
  resource_group_name         = azurerm_resource_group.example_rg.name
  network_security_group_name = azurerm_network_security_group.example_nsg.name
}
```

## Terraform Plan
The terraform plan command is used to create an execution plan. Terraform performs a refresh, unless explicitly disabled, and then determines what actions are necessary to achieve the desired state specified in the configuration files.
```
$ terraform plan
```

Review the output to make sure the changes you're expecting are being made.

## Terraform Apply
If you're satisfied, you can now apply your changes. A plan will run again, and you will need to confirm the changes.
```
$ terraform apply
...
Plan: X to add, Y to change, Z to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
...
Apply complete! Resources: X added, Y changed, Z destroyed.
```

## Inspect Terraform state file
You should see a `terraform.tfstate` file created in the top-level directory. The state file is a critical component of the Terraform architecture. It keeps a mapping between the modules you defined your code and the actual resources created by cloud provider.

## Modify a resource
In `main.tf`, update the `destination_port_range` to `7777`. Now, let's re-run the apply and see what happens.
```
$ terraform apply
```

## Check for drift and re-sync
Go to the Azure Portal and update your NSG rule's source port range to `6666`. 

Now run Terraform Apply again and see what happens.
```
$ terraform apply
```

## Terraform Destroy
Finally, let's clean everything up and destroy all the resouces we created.
```
$ terraform destory
```

Check the Azure Portal to verify that all the resources have been deleted. Also, take a look at the `terraform.tfstate` file and see how it changed.
