# Exercise 2: Deploy a Virtual Machine and Validate Secure Storage Access

### Estimated Duration:

## 📘 Scenario

With the foundational Azure infrastructure successfully deployed, Contoso is ready to provision a secure compute environment for administrators and applications. To align with the organization's security standards, the virtual machine must be deployed without a public IP address, leverage Azure Bastion for secure remote access, and use Managed Identity for authentication to Azure services.

In this exercise, you will deploy a Linux Virtual Machine and Azure Bastion using Azure Verified Modules (AVMs). You will securely connect to the virtual machine using an SSH private key stored in Azure Key Vault, authenticate to Azure using Managed Identity, and validate secure access to Azure Storage by uploading and managing blobs without relying on storage account keys or connection strings.

## 📖 Overview

In this exercise, you will extend the Azure infrastructure by deploying a Linux Virtual Machine, Azure Bastion, and the supporting resources required for secure administration and storage access. Using Azure Verified Modules (AVMs), you will update the existing Terraform configuration to provision the virtual machine and configure role assignments that enable secure access to Azure Storage.

After the deployment is complete, you will connect to the virtual machine through Azure Bastion using an SSH private key stored in Azure Key Vault. You will then install the Azure CLI, authenticate using the virtual machine's Managed Identity, and interact with Azure Storage by uploading, listing, downloading, and verifying blob data. By the end of the exercise, you will have validated a secure, passwordless authentication workflow that demonstrates Azure best practices for identity management, private connectivity, and Infrastructure as Code.

## 🎯 Objectives

In this exercise, you will complete the following tasks:

- Task 1: Deploy the Virtual Machine and Azure Bastion
- Task 2: Connect to the Virtual Machine Using Azure Bastion
- Task 3: Install the Azure CLI and Sign In
- Task 4: Upload and Manage Blobs in the Storage Account

## Task 1 - Deploy the Virtual Machine and Azure Bastion

In this part we are going to add a Virtual Machine to our Terraform configuration by leveraging the Azure Verified Module for Virtual Machine. The Virtual Machine is going to be used to interact with the Storage Account later. We are also going to add a role assignment to the storage module to assign permissions to the managed identity of the virtual machine to the storage container.

1. Copy the files from the **Part 5** folder into the **avm-lab** folder. This will add new files and update existing ones. If prompted, choose to overwrite the existing files while retaining all other files.

      ```pwsh
      copy ../labs/part05-virtual-machine/* .
      ```

      ![](../../images/e2t1s1-1.png)

1. Add the following variables to the **terraform.tfvars (1)** file, and then save the **changes (2)** by pressing `Ctrl + S`.

   ![](../../images/e1t1s2.png)

      >**Note:** You may need to choose a different virtual machine SKU depending on availability in your chosen region.

      ```hcl
      virtual_machine_sku = "Standard_B2s"
      virtual_machine_image = {
        publisher = "Canonical"
        offer     = "0001-com-ubuntu-server-jammy"
        sku       = "22_04-lts-gen2"
        version   = "latest"
      }
      ```

1. Run the following command to initialize the Terraform configuration and install the Azure Verified Modules (AVMs) for the Virtual Machine and Role Assignments.

   ```pwsh
   terraform init 
   ```
   - You should see: `Terraform has been successfully initialized!`

     ![](../../images/e2t1s3.png)   

1. Open the **avm.virtual_machine.tf (1)** file and look at each of the properties, paying close attention to the **generated_secrets_key_vault_secret_config (2)** and **network_interfaces (3)** variables.

   ![](../../images/e2t1s4.png)

   | Configuration | Description |
   |:--------|:------------|
   | `generated_secrets_key_vault_secret_config` | Configures the virtual machine to automatically store generated secrets, such as the SSH private key, in the specified Azure Key Vault. |
   | `network_interfaces` | Configures the network interface (NIC) for the virtual machine, including its name and IP configuration. The NIC is connected to the virtual_machines subnet of the Virtual Network. |

1. Apply the changes with Terraform: 

   ```pwsh
   terraform apply -auto-approve
   ```

   >**Note:** This command applies the Terraform configuration and automatically approves the deployment without prompting for confirmation.

   ![](../../images/t5s5.png)

   Expected output:

   ```
   Apply complete! Resources: 16 added, 8 changed, 1 destroyed.
   ```

   ![](../../images/e2t1s4-1.png)

1. After the command completes successfully, you should see output similar to the following:

   ![](../../images/t3s9-2.png)

   >**Note**: Scroll up in the **Terminal** to view the command output.

   >**Note:** The deployment may take several minutes to complete, as the virtual machine and its associated resources are being provisioned. Please wait until the command finishes successfully.

1. Navigate to the Azure portal. In the search bar, type **Virtual machines (1)**, and then select **Virtual machines (2)** from the search results.

   ![](../../images/vm-07.png)

1. Select the newly created **Virtual Machine (1)**, and review it's **Properties (2)**.

   ![](../../images/vm-08.png)

1. Commit the changes to git:

   ```
   git add .
   ```

   ```
   git commit -m "Add virtual machine and bastion"
   ```
   
   ```
   git push origin main
   ```

   ![](../../images/e2t1s8.png)

   ![](../../images/e2t1s8-1.png)

1. Review the files in the **GitHub repository.**

   ![](../../images/e2t1s8-2.png)


## Task 2 - Connect to the Virtual Machine Using Azure Bastion

In this part we are going to connect to the virtual machine via the Azure Bastion service using the SSH private key stored in the Key Vault.

1. Open the Azure portal. In the search bar, type **Virtual machine (1)**, and then select **Virtual machines (2)** from the search results.

   ![](../../images/vm-07.png)

1. On the Virtual machines page, select the Virtual machine that was deployed using Terraform.

    ![](../../images/vm-10.png)

1. On the **Overview (1)** page, click the drop-down arrow next to **Connect (2)**, and then select **Connect via Bastion (3)**.

    ![](../../images/vm-1-n.png)

1. Ensure that **SSH (1)** is selected as the Protocol, and then select **SSH Private Key from Azure Key Vault (3)** from the **Authentication Type (2)** drop-down menu.
   
   ![](../../images/vm-12.png)

1. Enter **azureuser (1)** in the Username field. Then, select the available **Subscription (2)**, **Azure Key Vault (3)**, and **Azure Key Vault Secret (4)** from their respective drop-down menus.

   ![](../../images/vm-11.png)

1. Select the **Open in new browser tab (1)** checkbox, and then click **Connect (2).**

   ![](../../images/vm-13.png)

1. After you click **Connect**, you may see the message **"A popup blocker is preventing a new window from opening. Please allow pop-ups and retry."**

   ![](../../images/e2t2s7.png)

1. In the top-right corner of your browser, click the ellipsis **(...) (1)**, and then select **Settings (2)**.

   ![](../../images/e2t2s8.png)

1. In the **Settings**  page, click on **Privacy, search, and services (1)**, and then click **Site permissions (2)**.

   ![](../../images/e2t2s9.png)

1. Click **All permissions**.

   ![](../../images/e2t2s10.png)

1. Next, click on **Pop-ups and redirects**.

   ![](../../images/e2t2s11.png)

1. Under the **Default behavior**, turn the toggle **Off** beside **Blocked (recommended)**.

   ![](../../images/e2t2s12.png)

1. Now, navigate back to the Azure portal, return to the virtual machine, and click **Connect**.

   ![](../../images/e2t2s13.png)

1. After clicking **Connect**, you will be redirected to a new browser tab, where the **Virtual Machine** will be displayed.

   ![](../../images/e2t2s14.png)

## Task 3 - Install the Azure CLI and Sign In

We are going to install the Azure CLI and login with the system assigned managed identity of the VM from the Azure Bastion SSH terminal.

1. Run the following command to install the Azure CLI.

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

   ![](../../images/vm-15.png)

1. Run to login with the system assigned managed identity.

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   az login --identity
   ```

   ![](../../images/vm-16.png)

## Task 4 - Upload and Manage Blobs in the Storage Account

We are going to create a blob in the storage account using the Azure CLI form the Azure Bastion SSH terminal.

1. Run the following command to create a file named hello.txt with the content **"hello world"**. 

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   echo "hello world" > hello.txt
   ```

   ![](../../images/blob-01.png)

1. Navigate to the Azure portal. In the global search bar, type **Storage accounts (1)**, and then select **Storage accounts (2)** from the search results.

   ![](../../images/sa-05.png)

1. Select the **Storage Account (1)**, copy the **Storage Account (2)** name , and save it in Notepad for use in the next steps.

   ![](../../images/sa-06.png)

   >**Note**: The storage account name may very.   

1. Run the following command to upload the **hello.txt** file to the storage account. Replace <storage-account-name> with the storage account name that you copied in **Step 3.**

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   az storage blob upload --account-name <storage-account-name> --container-name demo --file hello.txt --name hello.txt --auth-mode login
   ```

   ![](../../images/blob-02.png)

1. Run the following command to list the blobs in the container. Replace <storage-account-name> with the storage account name you copied in **Step 3.**

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   az storage blob list --account-name <storage-account-name> --container-name demo --auth-mode login
   ```

   ![](../../images/blob-03.png)

1. Run the following command to download the blob to a new file. Replace <storage-account-name> with the storage account name you copied in **Step 3.**

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   az storage blob download --account-name <storage-account-name> --container-name demo --name hello.txt --file hello2.txt --auth-mode login
   ```

   ![](../../images/blob-04.png)

1. Run to view the contents of the downloaded file.

   >**Note**: Copy the commands to Notepad, and then paste them into the virtual machine to run them.

   ```
   cat hello2.txt
   ```

   ![](../../images/blob-05.png)

## 🧾 Summary

In this exercise, you completed the following:

* Configured Terraform to deploy an Azure Virtual Machine, Azure Bastion, and supporting infrastructure using Azure Verified Modules (AVMs)
* Created and configured the `terraform.tfvars` file with networking, tagging, and virtual machine settings
* Initialized and deployed the infrastructure using the Terraform `init` and `apply` workflow
* Verified the deployed Virtual Machine and associated resources in the Azure portal
* Connected securely to the Virtual Machine using Azure Bastion with an SSH private key stored in Azure Key Vault
* Installed the Azure CLI on the Virtual Machine and authenticated using its System Assigned Managed Identity
* Uploaded, listed, downloaded, and verified a blob in Azure Storage using Azure CLI with Managed Identity authentication, validating secure access without using storage account keys

## 🎉 Congratulations!

You have successfully completed all five Azure Verified Modules (AVM) Terraform labs!

Throughout this workshop, you used **Terraform** and **Azure Verified Modules (AVMs)** to provision and manage Azure infrastructure following **Infrastructure as Code (IaC)** principles. You deployed core Azure resources including **Resource Groups, Virtual Networks, Subnets, NAT Gateways, Network Security Groups, Log Analytics Workspaces, Key Vaults, Storage Accounts, Azure Bastion, and Linux Virtual Machines** using reusable, production-ready Terraform modules.

You also implemented secure infrastructure practices by integrating **Azure Key Vault** for secret management, enabling **Managed Identity** for passwordless authentication, and configuring **role-based access control (RBAC)** to securely access Azure Storage. Throughout the labs, you gained hands-on experience with Terraform workflows (`init`, `plan`, and `apply`), variables, outputs, modules, state management, and Azure resource dependencies while following best practices for **automation, modularity, scalability, security, and maintainability** on Microsoft Azure.

