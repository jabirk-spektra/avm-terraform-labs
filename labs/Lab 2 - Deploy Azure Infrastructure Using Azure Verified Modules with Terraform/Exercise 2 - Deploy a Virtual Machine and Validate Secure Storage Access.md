# Exercise 2: Deploy a Virtual Machine and Validate Secure Storage Access

### Part 5 - Virtual machine and Bastion

In this part we are going to add a Virtual Machine to our Terraform configuration by leveraging the Azure Verified Module for Virtual Machine. The Virtual Machine is going to be used to interact with the Storage Account later. We are also going to add a role assignment to the storage module to assign permissions to the managed identity of the virtual machine to the storage container.

1. Copy the files from the [part 5](labs/part05-vm/) folder into the `avm-lab` folder, remembering to retain the existing files and just add an overwrite when prompted.

      ```pwsh
      copy ../avm-terraform-labs/labs/part05-virtual-machine/* .
      ```

1. Add the following variables to your `terraform.tfvars` file:

      >NOTE: You may need to choose a different virtual machine sku depending on availability in your chosen region. If you are running this lab in Skillable, SKUs are restricted to names beginning with `Standard_B2*` or `Standard_D2*` only.

      ```hcl
      virtual_machine_sku = "Standard_D2s_v4"
      virtual_machine_image = {
        publisher = "Canonical"
        offer     = "0001-com-ubuntu-server-jammy"
        sku       = "22_04-lts-gen2"
        version   = "latest"
      }
      ```

1. Run `terraform init` to install the AVM modules for Virtual Machine and Role Assignments.
1. Apply the changes with Terraform. NOTE: We are applying this now, because the bastion can take a few minutes to deploy.
1. Navigate to the `Source Control` tab in Visual Studio Code and review the changes to the files.
1. Open the `avm.virtual-machine.tf` file and look at each of the properties, paying close attention to the `generated_secrets_key_vault_secret_config` and `network_interfaces` variables.
1. Apply the changes with Terraform: `terraform apply -auto-approve`.
1. Review the deployed resources in the Azure Portal.
1. Commit the changes to git: `git add . && git commit -m "Add virtual machine and bastion"`.

### Part 6 - Connect to the VM via Bastion

In this part we are going to connect to the virtual machine via the Azure Bastion service using the SSH private key stored in the Key Vault.

1. Open the Azure Portal and navigate to the VM.
1. Click on the `Connect` button and select `Bastion`.
1. Choose `SSH Private Key from Azure Key Vault` in the `Authentication Type` dropdown.
1. Enter `azureuser` in the `Username` field.
1. Select you subscription from the `Subscription` drop down.
1. Select the Key Vault you created in the lab in the `Azure Key Vault` drop down.
1. Select the secret you created in the lab in the `Azure Key Vault Secret` drop down.
1. Click `Connect`. You may see a pop-up blocked message, click on the pop-up blocked icon in the address bar and select `Always allow pop-ups and redirects from https://portal.azure.com`.
1. A new browser window will open with a terminal session to the VM.

### Part 7 - Install the Azure CLI and login

We are going to install the Azure CLI and login with the system assigned managed identity of the VM from the Azure Bastion SSH terminal.

1. Run `curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash` to install the Azure CLI.
1. Run `az login --identity` to login with the system assigned managed identity.

### Part 8 - Create a blob in the storage account

We are going to create a blob in the storage account using the Azure CLI form the Azure Bastion SSH terminal.

1. Run `echo "hello world" > hello.txt` to create a file with some content.
1. Run `az storage blob upload --account-name <storage-account-name> --container-name demo --file hello.txt --name hello.txt --auth-mode login` to upload the file to the storage account.
1. Run `az storage blob list --account-name <storage-account-name> --container-name demo --auth-mode login` to list the blobs in the container.
1. Run `az storage blob download --account-name <storage-account-name> --container-name demo --name hello.txt --file hello2.txt --auth-mode login` to download the blob to a new file.
1. Run `cat hello2.txt` to view the contents of the downloaded file.

      Here are the commands to run, so you can copy to notepad and replace the placeholder with the storage account name you created in the lab. Then run the commands in the terminal:

      ```bash
      echo "hello world" > hello.txt
      az storage blob upload --account-name replace_me --container-name demo --file hello.txt --name hello.txt --auth-mode login
      az storage blob list --account-name replace_me --container-name demo --auth-mode login
      az storage blob download --account-name replace_me --container-name demo --name hello.txt --file hello2.txt --auth-mode login
      cat hello2.txt
      ```
