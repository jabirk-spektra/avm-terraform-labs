# Exercise 1: Set Up the Azure Infrastructure Using Azure Verified Modules

### Estimated Duration:

## 📘 Scenario

As a Cloud Infrastructure Engineer at Contoso, you have been Contosotasked with establishing the organization's foundational Azure infrastructure using Azure Verified Modules (AVMs) for Terraform. To ensure consistency, security, and scalability across deployments, Contoso has standardized on Infrastructure as Code (IaC) practices and reusable Microsoft-validated Terraform modules.

In this exercise, you will prepare the Terraform development environment, authenticate to Azure, and progressively deploy core Azure resources, including a Resource Group, Log Analytics Workspace, Virtual Network, Key Vault, and Storage Account. By leveraging AVMs, you will build a secure and modular infrastructure that serves as the foundation for hosting workloads in subsequent exercises.

## 📖 Overview

In this exercise, you will configure the local Terraform development environment and use Azure Verified Modules (AVMs) to provision foundational Azure infrastructure. You will begin by verifying the required tools, authenticating to Azure, and initializing the Terraform configuration. Next, you will deploy essential Azure resources such as a Log Analytics Workspace, Virtual Network with subnets, Azure Key Vault, and an Azure Storage Account using reusable Terraform modules.

Throughout the exercise, you will validate each deployment in the Azure portal and use Git and GitHub to version-control your Terraform configurations. By the end of the exercise, you will have successfully deployed a secure, production-ready Azure infrastructure that provides the networking, security, monitoring, and storage services required for the remaining lab activities.

## 🎯 Objectives

In this exercise, you will complete the following tasks:

- Task 1: Prepare the Terraform Development Environment
- Task 2: Deploy the Base Infrastructure
- Task 3: Configure the Virtual Network and Subnets
- Task 4: Deploy the Key Vault
- Task 5: Deploy the Storage Account

## Task 1: Prepare the Terraform Development Environment

In this task, you will prepare the local development environment required for Terraform deployments in Visual Studio Code, install the required extensions, and verify that Terraform is installed.

1. Open **Visual Studio Code** on your Lab-VM.

   ![](../../images/vs.png)

1. Once the IDE opens, if you see the ***Welcome to VS Code*** sign-in pop-up for GitHub, simply close the window by clicking the **X** in the upper-right corner.

   ![](../../images/vsc-welcome-window-close.png)

1. In VS Code, ensure that the following extensions are installed:
   
   - [HashiCorp Terraform](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform) — syntax highlighting, validation, and IntelliSense for `.tf` files.
  
     ![](../../images/vsc-terraform-lab-hashicorp-ext.png)

1. From the **File** menu in VS Code, choose **Open Folder**.

   ![](../../images/vsc-open-folder.png)

1. Navigate to `C:\Users\azureuser`, select the **TerraformLabs** folder and then click **Select folder**.

   ![](../../images/vsc-select-folder-terraformlabs-01.png)

1. Now you will see another screen Do you trust the authors of the files in this folder?. Click **Trust Folder & Continue**.

   ![](../../images/vsc-trust-folder-terraformlabs-01.png)

1. Open the integrated terminal by selecting **Terminal → New Terminal**.

   ![](../../images/vsc-terraform-lab-new-terminal.png)

1. In the integrated terminal, verify that Terraform is installed by running the following command:

   ```bash
   terraform version
   ```

   This command displays the currently installed Terraform CLI version. You should see **Terraform version 1.9.x** or later installed in the environment.

   ![](../../images/vsc-terraform-version-01.png)

1. Sign in to Azure from the integrated terminal:

   ```
   az login
   ```

1. On the *Let’s get you signed in pop-up*, select **Work or school account**, then click **Continue**. You may need to minimize any open applications to bring this window into view.

   ![](../../images/az-select-work-or-school-account.png)

1. You'll see the Sign into Microsoft Azure tab. Here, enter your credentials:

   - **Email/Username:** <inject key="AzureAdUserEmail"></inject>
  
     ![](../../images/az-enter-username.png)
  
1. Next, enter the Temporary Access Pass:

   - **Temporary Access Pass:** <inject key="AzureAdUserPassword"></inject>
  
     ![](../../images/az-enter-tap.png)

1. On the *Sign in to all apps, websites, and services on this device?*, click **No, this app only**.

   ![](../../images/az-no-this-app-only.png)

1. You are now signed in to the Azure portal from your Visual Studio Code terminal. In Visual Studio Code integrated terminal, when prompted to select a subscription and tenant, press **Enter** to accept the default selection.

   ![](../../images/lf-09.png)

## Task 2 - Deploy the Base Infrastructure

In this part we are going to setup our Terraform root module and deploy an Azure Resource Group and Log Analytics Workspace ready for the rest of the lab. In this part we introduce our first Azure Verified Module, the `avm.log_analytics_workspace` module.

> **Important:** Do not clear or close the PowerShell terminal, as doing so may interrupt the Terraform session and affect the remaining lab tasks.

The Log Analytics Workspace is used as the target for diagnostic settings for all our other resources. This is where we are sending our logging telemetry.

1. In the **New Terminal**, run the following command to create a new directory named **avm-lab**.

   ```pwsh
   mkdir avm-lab
   ```

   ![](../../images/t2s1.png)

1. Navigate to the directory `avm-lab`.

   ```pwsh
   cd avm-lab
   ```

   ![](../../images/t2s2.png)

1. Copy the files from the **Part 1** folder into the **avm-lab** folder by running the following command:

      ```pwsh
      copy ../labs/part01-base/* .
      ```
   
   ![](../../images/t2s3.png)

   -  Your file structure should look like this:

      ```plaintext
      📂terraformlabs
      ┣ 📂avm-lab
      ┃ ┣ 📜.gitignore
      ┃ ┣ 📜avm.log_analytics_workspace.tf
      ┃ ┣ 📜locals.tf
      ┃ ┣ 📜main.tf
      ┃ ┣ 📜outputs.tf
      ┃ ┣ 📜terraform.tf
      ┃ ┗ 📜variables.tf
      ```
   -  Expand the **avm-lab** directory by clicking the dropdown arrow next to it to view the **Terraform** files.

      ![](../../images/t2s4.png)

1. Examine the `terraform` block in `terraform.tf` and note that we are referencing the `azurerm` and `random` providers.

   ![](../../images/t2s5.png)

   | Concept | Description |
   |:--------|:------------|
   | **AzureRM Provider (azurerm)** | The Azure Resource Manager provider enables Terraform to create, update, and manage Azure resources such as Virtual Networks, Virtual Machines, Storage Accounts, and Key Vaults. |
   | **Random Provider (random)** | Provides utilities to generate random values such as strings, passwords, integers, and unique resource names. It does not create Azure resources. | 

1. Examine the files below: 

   * `locals.tf`

     ![](../../images/locals.png)

      | Concept | Description |
      |:--------|:------------|
      | **locals** | Defines reusable local values that can be referenced throughout the Terraform configuration, reducing duplication and improving consistency. |
      | **name_replacements** | Creates a collection of values that will be substituted into the resource naming templates. |
      | **workload** | Represents the application or workload name (for example, demo) that becomes part of the Azure resource names. |
      | **environment** | Represents the deployment environment (for example, dev, test, or prod) and is included in the resource names to distinguish environments. | 
      | **location** | Specifies the Azure region (for example, swedencentral or eastus) that is incorporated into the resource names. | 
      | **sequence** | Generates a three-digit sequence number (for example, 001, 002, 003) using the format ("%03d", ...) function to ensure unique and consistently formatted resource names. |
      | **resource_names** | Generates the final resource names by replacing the placeholders in the resource name templates with the values defined in name_replacements. | 

   * `variables.tf` 
   
     ![](../../images/variable.png)

      | Concept | Description |
      |:--------|:------------|
      | **variable "location"** | Defines the Azure region where all resources will be deployed (for example, swedencentral or eastus).|
      | **type = string** | Specifies that the variable accepts a text value.|
      | **validation** | Ensures the value entered for the variable follows the required format before Terraform executes.|
      | **variable "resource_name_workload"** | Defines the workload name that is used as part of the Azure resource naming convention.|
      | **default = "demo"** | Assigns a default workload name if no value is provided by the user.|

   * `outputs.tf`

     ![](../../images/output.png)

      | Concept | Description |
      |:--------|:------------|
      | **output "resource_names"** | Displays the names of the Azure resources created by Terraform after the deployment completes.|
      | **local.resource_names** | Retrieves the generated resource names from the locals.tf file.|
      | **output "resource_ids"** | Displays the Azure Resource IDs of the deployed resources after the deployment completes.|
      | **module.resource_group.resource_id** | Returns the Resource ID of the deployed Resource Group.|
      | **module.log_analytics_workspace.resource_id** | Returns the Resource ID of the deployed Log Analytics Workspace.|

   * `main.tf`

     ![](../../images/main.png)

      | Concept | Description |
      |:--------|:------------|
      | **module "resource_group"** | Uses the Azure Verified Module (AVM) to deploy an Azure Resource Group.|
      | **source** | Specifies the Terraform Registry location of the Azure Verified Module used to create the Resource Group.|
      | **version** | Specifies the version of the AVM module to ensure a consistent deployment.|
      | **location = var.location** | Deploys the Resource Group in the Azure region specified by the location variable.|
      | **name = local.resource_names.resource_group_name** | Assigns the Resource Group name generated in the locals.tf file.|
      | **tags = var.tags** | Applies the user-defined tags to the Resource Group for easier organization and management.|

1. Examine the `avm.log_analytics_workspace.tf` file and note the `source` and `version` properties.

   ![](../../images/avm.png)

      | Concept | Description |
      |:--------|:------------|
      | **source** | Specifies the Terraform Registry location of the Azure Verified Module (AVM) that Terraform downloads and uses to deploy the Log Analytics Workspace.|
      | **version** | Specifies the version of the AVM module to use, ensuring a consistent and predictable deployment by avoiding unexpected changes from newer module versions.|

1. Create an environment variable to set the location variable:

      ```pwsh
      $env:TF_VAR_location = "<azure region>"
      ```

1. Replace `<azure region>` with a valid Azure location of your choice (e.g. eastus,eastus2,centralus,canadaeast,westus,westus3).

      ```pwsh
      $env:TF_VAR_location = "eastus"
      ```

      ![](../../images/t2s9.png)   

1. Navigate to the left side of the Visual Studio, and select **File (1)**, and click on **New File (2)**.

   ![](../../images/tf-02.png)

1. In the Search bar, give the name as **terraform.tfvars** and press **Enter**.

   ```
   terraform.tfvars
   ```

   ![](../../images/tf-03.png)

1. In the Create New File dialog box, verify that the file name is terraform, ensure the file is created in the `C:\Users\azureuser\TerraformLabs\avm-lab` directory, and then click Create File.

   ![](../../images/t2s12.png)

1. After the file is created, add the following code to it, and save the file `Ctrl + S`.

      ```hcl
      tags = {
        type = "avm"
        env  = "demo"
      }
      ```

      ![](../../images/tf-05.png)

1. Run the following command to initialize the Terraform configuration.

   ```
   terraform init
   ```
   
   - You should see: `Terraform has been successfully initialized!`

     ![](../../images/t2s14.png)

1. Run the following command to preview the resources that will be created and generate a Terraform plan file.

   ```
   terraform plan -out tfplan
   ```
   Expected output:

   ```
   Plan: 6 to add, 0 to change, 0 to destroy.
   ```

   ![](../../images/t2s15.png)   

   >**Note**: Scroll up in the **Terminal** to view the command output.

1. Run the following command to create the resources based on the generated Terraform plan file.

   ```
   terraform apply tfplan
   ```

   ![](../../images/t2s16.png)

   >**Note**: The **terraform apply tfplan** command applies the execution plan saved in the **tfplan file** and deploys the Azure resources exactly as defined in the previously generated Terraform plan.

   Expected output:

   ```
   Apply complete! Resources: 6 added, 0 changed, 0 destroyed.
   ```

   ![](../../images/t2s16-1.png)

   >**Note**: Scroll up in the **Terminal** to view the command output.

1. Take note of the outputs from the `terraform apply` command, they should look like this:

   ![](../../images/tf-10.png)

1. Navigate to the Azure portal. In the search bar, type **Log Analytics workspace (1)**, and then select **Log Analytics workspaces (2)** from the search results.

   ![](../../images/lw-01.png)

1. Click on the newly created **Log Analytics workspace** resource.

   ![](../../images/lw-02.png)

1. Navigate to the **GitHub repository** in your browser. Copy the **GitHub repository URL** from the browser's address bar. You will use this URL to configure the repository in **Visual Studio Code**.

   ![](../../images/github-login.png)

1. Use the copied GitHub repository URL to set the remote URL for the required repository.

   ```
   git remote set-url origin https://github.com/Cloudlabs-Enterprises/avm-terraform-labs-<inject key="Deployment-ID" enableCopy="false"/>
   ```

   ![](../../images/github-01.png)

1. Configure your **Git Author** identity by following the on-screen instructions before proceeding.

    ```pwsh
    git config --global user.email <inject key="GitHub User Name" enableCopy="true"/>
    ```

   ```
    git config --global user.name odl-user-<inject key="Deployment-ID" enableCopy="false"/>
    ```

    ![](../../images/t2s23.png)

1. Run the following command to initialize a new Git repository with the default branch set to main.

   ```
   git init -b main
   ```

   ![](../../images/t2s20.png)

1. Run the following command to stage all the files for commit.

   ```
   git add .
   ```

   ![](../../images/t2s21.png)

1. Run the following command to commit the staged files to the Git repository.

   ```
   git commit -m "Initial commit"
   ```

   ![](../../images/t2s22.png)

1. Run the command to upload the files to GitHub repository

   ```pwsh
   git push origin main
   ```

   ![](../../images/t2s25.png)

1. If a prompt appears asking you to connect to GitHub after running the command, click **Sign in with your browser** to continue.

   ![](../../images/connect-to-github.png)

1. If the Authorize Git Credential Manager pop-up appears, click **Authorize git-ecosystem** to continue.

   ![](../../images/agcm.png)

1. Review the files in the **GitHub repository.**

   ![](../../images/t2s28.png)

## Task 3 - Configure the Virtual Network and Subnets

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

## Task 4 - Deploy the Key Vault

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

## Task 5 - Deploy the Storage Account

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

## 🧾 Summary

In this lab, you completed the following:

* Authenticated with Azure and configured the target subscription for the deployment.
* Initialized the Terraform configuration and deployed the foundational Azure infrastructure using Azure Verified Modules (AVM).
* Provisioned core Azure resources, including a Resource Group, Log Analytics Workspace, Virtual Network, Key Vault, and Storage Account.
* Validated the deployed resources in the Azure portal to confirm successful deployment.
* Tracked infrastructure changes by initializing a Git repository and committing the Terraform configuration after each deployment stage.

You have successfully completed the lab. Click **Next >>** in the lower-right corner to proceed to the next lab.

![](../../images/next-button.png)