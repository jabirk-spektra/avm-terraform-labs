# Exercise 2 - Building a Secure, Private Network Foundation with Azure Verified Modules

## Task 1 - Configure the Virtual Network and Subnets

In this part we are going to add a virtual network and subnets to our Terraform configuration by leveraging the Azure Verified Module for Virtual Network. The Virtual Network is going to be used to provide private connectivity between and to our virtual machine, key vault and storage account.

>IMPORTANT: This lab is incremental, you must not delete any files from the previous lab (especially the `terraform.tfstate` file). You must copy the files from the next lab into the `avm-lab` folder and only replace the existing files when prompted.

**Azure Networking Concepts:**

| Concept | Description |
|:--------|:------------|
| **Address Space** | The private IP CIDR block for the entire VNet (e.g. `10.0.0.0/22`). |
| **Subnet** | A logical subdivision of the VNet's address space. Resources are deployed into subnets. |
| **Region scope** | VNets exist within a single Azure region. |
| **CIDR Notation** | Specifies IP ranges using slash notation such as `/16` or `/24`. |

1. Copy the files from the **Part 2** folder into the **avm-lab** folder. This will add new files and replace the existing files where applicable.

      ```pwsh
      copy ../labs/part02-virtual-network/* .
      ```

   ![](../../images/t3s1.png)

   -  Your file structure should now look like this if you have followed the instructions correctly (this structure will continue to grow as you progress through the lab):

      ```plaintext
      📂terraformlabs
      ┣ 📂avm-lab
      ┃ ┣ 📂.git (hidden)
      ┃ ┣ 📂.terraform
      ┃ ┣ 📜.gitignore
      ┃ ┣ 📜.terraform.lock.hcl
      ┃ ┣ 📜avm.log_analytics_workspace.tf
      ┃ ┣ 📜avm.nat_gateway.tf
      ┃ ┣ 📜avm.network_security_group.tf
      ┃ ┣ 📜avm.virtual_network.tf
      ┃ ┣ 📜locals.tf
      ┃ ┣ 📜main.tf
      ┃ ┣ 📜outputs.tf
      ┃ ┣ 📜terraform.tf
      ┃ ┣ 📜terraform.tfstate
      ┃ ┣ 📜terraform.tfvars
      ┃ ┣ 📜tfplan
      ┃ ┗ 📜variables.tf
      ```

1. Open the **terraform.tfvars (1)** file, replace it with the following **code (2)**, and then save the file using `Ctrl + s`.

   ![](../../images/t3s2.png)

      ```hcl
      address_space = "10.0.0.0/22"
      subnets = {
        AzureBastionSubnet = {
          size                       = 26
          has_nat_gateway            = false
          has_network_security_group = false
        }
        private_endpoints = {
          size                       = 28
          has_nat_gateway            = false
          has_network_security_group = true
        }
        virtual_machines = {
          size                       = 24
          has_nat_gateway            = true
          has_network_security_group = false
        }
      }
      tags = {
        type = "avm"
        env  = "demo"
      }
      ```
   
   | Configuration | Description |
   |:--------|:------------|
   | `AzureBastionSubnet` | Creates a dedicated subnet for the Azure Bastion service. |
   | `size = 26` | Allocates a /26 subnet address range for Azure Bastion. |
   | `private_endpoints` | Creates a subnet for hosting Azure Private Endpoints. |
   | `virtual_machines` | Creates a subnet for deploying Virtual Machines. |
   | `has_network_security_group = true` | Associates a Network Security Group (NSG) with the Private Endpoints subnet. |

1. Run the following command to initialize the Terraform configuration and install the Azure Verified Module (AVM) for Virtual Networks along with the required networking resources.

   ```
   terraform init
   ```

   - You should see: `Terraform has been successfully initialized!`

     ![](../../images/t3s3.png)

1. Open the **avm.ip_addresses.tf (1)** file and note the use of a utility module here, pay close attention to the **source and version (2)** properties. A utility module is a helper module that doesn't deploy anything itself, but is used to calculate common values.

   ![](../../images/t3s4.png)

   | Configuration | Description |
   |:--------|:------------|
   | `source` | Specifies the Terraform Registry location of the Azure Verified Utility Module (AVM Utility) that Terraform downloads to calculate IP address ranges and subnet prefixes. |
   | `version` | Specifies the version of the utility module to use, ensuring a consistent and predictable deployment by avoiding unexpected changes from newer module versions. |

1. Open the **avm.virtual-network.tf (1)** file and look at each of the properties, paying close attention to the **source and version (2)** properties.

   ![](../../images/t3s5.png)

   | Configuration | Description |
   |:--------|:------------|
   | `source` | Specifies the Terraform Registry location of the Azure Verified Module (AVM) that Terraform downloads and uses to deploy the Azure Virtual Network. |
   | `version` | Specifies the version of the Azure Verified Module to use, ensuring a consistent and predictable deployment by avoiding unexpected changes from newer module versions. |

1. Examine the diagnostics settings in **locals.tf** and take note that this same setting will be applied to all of the AVM modules in the lab.

   ![](../../images/t3s6.png)

   | Configuration | Description |
   |:--------|:------------|
   | `diagnostic_settings` | Defines the diagnostic settings that are applied to Azure resources to collect logs and metrics. |
   | `Log Analytics Workspace` | Configures Azure resources to send their diagnostic logs and metrics to the deployed Log Analytics Workspace for centralized monitoring. |
   | `Reusable Local Value` | Stores the diagnostic settings as a local value so the same configuration can be reused across multiple Azure Verified Modules (AVMs), ensuring consistent monitoring for all deployed resources. |

1. In order to find more detail about AVM modules, you can navigate to their documentation. For example, you can find the documentation for the Virtual Network module [here](https://registry.terraform.io/modules/Azure/avm-res-network-virtualnetwork/azurerm/latest). From there you can navigate to the source code and see the module's implementation [here](https://github.com/Azure/terraform-azurerm-avm-res-network-virtualnetwork).

1. Run the following command in the terminal to apply the Terraform configuration and deploy the Azure resources.

   >**Note:** This command applies the Terraform configuration and automatically approves the deployment without prompting for confirmation.

   ```
   terraform apply -auto-approve
   ```

   ![](../../images/t3s9.png)

   Expected output:

   ```
   Apply complete! Resources: 19 added, 0 changed, 0 destroyed.
   ```

   ![](../../images/t3s9-1.png)

1. After the command completes successfully, you should see output similar to the following:

   ![](../../images/t3s9-2.png)

   >**Note**: Scroll up in the **Terminal** to view the command output.
   
1. Review the deployed resources in the Azure portal. In the global search bar, type **Virtual network (1)**, and then select **Virtual networks (2)** from the search results.

   ![](../../images/vnet-01.png)

1. Select the newly created **Virtual network**.

   ![](../../images/vnet-02.png)

1. Review the **Virtual network** that was created as part of the Terraform deployment.

   ![](../../images/vnet-03.png)

1. Navigate back to Visual Studio Code, and in the terminal, run the following command to commit the changes to the Git repository:

   ```
   git add .
   ```

   ```
   git commit -m "Add virtual network and subnets"
   ```
   
   ```
   git push origin main
   ```

   ![](../../images/t3s13.png)

   ![](../../images/t3s13-1.png)

1. Review the files in the **GitHub repository.**

   ![](../../images/github-page.png)

## Task 2 - Deploy the Key Vault

In this part we are going to add a Key Vault to our Terraform configuration by leveraging the Azure Verified Module for Key Vault. The Key Vault is going to be used to store the customer managed key for our storage account and the SSH private key for our virtual machine.

1. Copy the files from the **Part 3** folder into the **avm-lab** folder. When prompted, keep the existing files and choose to overwrite only the files that are being updated.

      ```pwsh
      copy ../labs/part03-key-vault/* .
      ```

      ![](../../images/t4s1.png)  

1. Run the following command to initialize the Terraform configuration and install the Azure Verified Module (AVM) for Key Vault.

   ```
   terraform init
   ```

   - You should see: `Terraform has been successfully initialized!`

     ![](../../images/t4s2.png)

1. Open the **avm.key_vault.tf (1)** file and look at each of the properties, paying close attention to the **private_endpoints (2)** and **role_assignments (3)** variables.

   ![](../../images/t4s3.png)

   | Configuration | Description |
   |:--------|:------------|
   | `private_endpoints` | Configures a Private Endpoint for the Key Vault, allowing it to be accessed securely over the Virtual Network instead of the public internet. |
   | `Private DNS Zone` | Links the Private Endpoint to a Private DNS Zone, enabling the Key Vault name to resolve to its private IP address. |
   | `Subnet Association` | Deploys the Private Endpoint into the private_endpoints subnet of the Virtual Network. |
   | `role_assignments` | Assigns Azure Role-Based Access Control (RBAC) permissions to users or identities for the Key Vault. |
   | `Key Vault Administrator` | Grants the specified user or identity full administrative permissions to manage the Key Vault, including keys, secrets, and certificates. |

1. Run the following command to apply the Terraform configuration and deploy the Azure resources.

   >**Note:** This command applies the Terraform configuration and automatically approves the deployment without prompting for confirmation.

   ```
   terraform apply -auto-approve
   ```

   ![](../../images/t4s5.png)

   Expected output:

   ```
   Apply complete! Resources: 15 added, 0 changed, 0 destroyed.
   ```

   ![](../../images/t4s5-1.png)

1. After the command completes successfully, you should see output similar to the following:

   ![](../../images/t3s9-2.png)

   >**Note**: Scroll up in the **Terminal** to view the command output.

1. Navigate back to the Azure portal. In the search bar, type **Key vault (1)**, and then select **Key vaults (2)** from the search results to view the newly created resource.

   ![](../../images/kv-04.png)

1. Select the newly created Key vault, and review the resource.

   ![](../../images/kv-05.png)

1. Commit the changes to git: 

   ```
   git add .
   ```
   
   ```
   git commit -m "Add key vault"
   ```
   
   ```
   git push origin main
   ```

   ![](../../images/t4s7.png)

   ![](../../images/t4s7-1.png)
   
1. Review the files in the **GitHub repository.**

   ![](../../images/t4s8.png)

## Task 3 - Deploy the Storage Account

In this part we are going to add a Storage Account to our Terraform configuration by leveraging the Azure Verified Module for Storage Account. The Storage Account is the main component of our demo lab and we will interact with it later on.

1. Copy the files from the **Part 4** folder into the **avm-lab** folder.

      ```pwsh
      copy ../labs/part04-storage-account/* .
      ```
   
      ![](../../images/t5s1.png)

1. Run to install the AVM module for Storage Account.

   ```
   terraform init
   ```

   - You should see: `Terraform has been successfully initialized!`

     ![](../../images/t5s2.png)

1. Open the **avm.storage-account.tf (1)** file and look at each of the **properties (2)**, paying close attention to the **managed_identities**, **customer_managed_key** and **containers** variables.

   ![](../../images/t5s3.png)

   | Configuration | Description |
   |:--------|:------------|
   | `managed_identities` | Configures managed identities for the Storage Account, allowing it to securely authenticate with other Azure services without storing credentials. |
   | `customer_managed_key` | Configures the Storage Account to use a customer-managed encryption key stored in Azure Key Vault instead of the default Microsoft-managed key. |
   | `containers` | Creates blob containers within the Storage Account and defines their access level. In this configuration, a private container named demo is created. |

1. Run the following command to apply the Terraform configuration and deploy the Azure resources.

   >**Note:** This command applies the Terraform configuration and automatically approves the deployment without prompting for confirmation.
   
   ```
   terraform apply -auto-approve
   ```

   ![](../../images/t5s5.png)

   Expected output:

   ```
   Apply complete! Resources: 20 added, 2 changed, 0 destroyed.
   ```

   ![](../../images/t5s5-1.png)

1. After the command completes successfully, you should see output similar to the following:

   ![](../../images/t3s9-2.png)

   >**Note**: Scroll up in the **Terminal** to view the command output.

1. Navigate to the Azure portal. In the global search bar, type **Storage accounts (1)**, and then select **Storage accounts (2)** from the search results.

   ![](../../images/sa-05.png)

1. Select the **Storage account (1)** created by the Terraform deployment, and review its **details (2)**.

   ![](../../images/sa-06.png)

1. Commit the changes to git:

   ```
   git add .
   ```
   ```
   git commit -m "Add storage account"
   ```
   
   ```
   git push origin main
   ```
   
   ![](../../images/t5s8.png)

   ![](../../images/t5s8-1.png)

1. Review the files in the **GitHub repository.**

   ![](../../images/t5s8-2.png)

---

