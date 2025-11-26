# azure-vm-basics-cli
Mini Azure Project – Deploy a Windows VM via Azure CLI

This mini-lab demonstrates how to deploy a Windows and a Linux Server virtual machine **fully via Azure CLI**, without using the Azure Portal.  
It is part of my AZ-104 learning path and builds the foundation for later networking and security projects.

---

##  Goals
- Create a Resource Group  
- Deploy a Windows Server VM via Azure CLI  
- Automatically create NIC, NSG, Public IP  
- Open RDP (3389) via CLI  
- Connect to the VM via Remote Desktop  

##  Prerequisites
- Azure Subscription  
- Azure CLI or Azure Cloud Shell  
- Basic understanding of regions & SKUs

  ---

## Section 1 - Deploy Windows VM (Mini-Lab)


## Step 1 – Create Resource Group
```bash
az group create --name app-grp --location westeurope --output table
```

---

## Step 2 - Deploy Windows VM
```bash
az vm create \
  --resource-group app-grp \
  --name firstvm01 \
  --image Win2022Datacenter \
  --size Standard_B2s \
  --admin-username username \
  --admin-password 'password' \
  --location westeurope \
  --public-ip-sku Standard \
  --output table
```

---

## Step 3 - Open RDP Port (3389)
```bash
az vm open-port \
  --resource-group app-grp \
  --name firstvm01 \
  --port 3389 \
  --priority 1001
```

---

## Step 4 - Retrieve Public IP
```bash
az vm show \
  --resource-group app-grp \
  --name firstvm01 \
  --show-details \
  --query publicIps \
  -o tsv
```

---

## Step 5 connect via RDP

Open the Remote Desktop client (mstsc on Windows),  
enter the public IP from Step 4, and log in using:

- **Username:** the admin username you specified  
- **Password:** the secure password you created

---

## Step 6 - Cleanup (optional)
```bash
az group delete --name app-grp --yes --no-wait
```

## Learnings

- Understanding Azure regions & capacity constraints

- Deploying VMs purely via CLI

- Working with NSGs and inbound rules

- Handling errors like unavailable SKUs

- First full compute deployment without portal usage

  ---

## Section 2 - Resize the Virtual Machine (Mini-Lab)

This mini-lab shows how to resize an existing VM using the Azure CLI.  
The VM was created in Section 1 and is resized within the same VM family.


## Step 1 – Deallocate the VM
```bash
az vm deallocate \
  --resource-group app-grp \
  --name firstvm01
```

---

## Step 2 – Check available resize options (optional but recommended)
```bash
az vm list-vm-resize-options \
  --resource-group app-grp \
  --name firstvm01 \
  -o table
```

---

## Step 3 – Resize the VM (example: B2ms)
```bash
az vm resize \
  --resource-group app-grp \
  --name firstvm01 \
  --size Standard_B2ms
```

---

## Step 4 – Start the VM again
```bash
az vm start \
  --resource-group app-grp \
  --name firstvm01
```

---

## Step 5 – Verify new size
```bash
az vm show \
  --resource-group app-grp \
  --name firstvm01 \
  --query "hardwareProfile.vmSize" \
  -o tsv
```

## Learnings (Resize)

- A VM must be deallocated before resizing.

- az vm list-vm-resize-options shows only compatible SKUs for the existing VM.

- Not every VM size can be converted to every other size (resource disk vs. non-resource disk).

- Resizing within the same family (e.g. B1s → B2ms) is the typical real-world scenario.


---

## Section 3 - Install IIS Web Server on the VM (Mini-Lab)

This mini-lab extends the VM deployment by installing a basic IIS web server on the Windows Server VM created in Section 1.
The installation is performed remotely using Azure CLI with Run Command.


## Step 1 - Ensure the VM is running
```bash
az vm start \
  --resource-group app-grp \
  --name firstvm01
```

---

## Step 2 – Install IIS using Run Command
```bash
az vm run-command invoke \
  --resource-group app-grp \
  --name firstvm01 \
  --command-id RunPowerShellScript \
  --scripts "Install-WindowsFeature -name Web-Server -IncludeManagementTools"
```

---

## Step 3 – Open Port 80 (HTTP) in the NSG
```bash
az vm open-port \
  --resource-group app-grp \
  --name firstvm01 \
  --port 80 \
  --priority 1002
```

---

## Step 4 – Retrieve Public IP
```bash
az vm show \
  --resource-group app-grp \
  --name firstvm01 \
  --show-details \
  --query publicIps \
  -o tsv
```

---

## Step 5 – Test the Web Server

Open a browser and enter the public IP:

```bash
http://PUBLIC-IP
```

---

## Step 6 – Cleanup (optional)
```bash
az group delete --name app-grp --yes --no-wait
```

---

## Learnings (Web Server)

- Installing software/workloads on VMs remotely

- Using az vm run-command for automation

- Opening additional NSG ports securely

- Understanding IIS as a sample web workload

- Seeing workload availability through public access

- Note: Port 80 is used here only for demonstration. in production environments, HTTPS and certificates should be used.

---

## Section 4 - Deploy a Linux VM with SSH Key Authentication (Mini-Lab)

This mini-lab demonstrates how to deploy an Ubuntu Linux virtual machine using SSH key authentication instead of a password.
The VM is created via the Azure CLI and accessed securely using a locally generated SSH key pair.


## Step 1 - Generate SSH Keypair (on local machine or Cloud Shell)
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/azure_key
```
No passphrase required for this lab.
This generates:
Private key: ~/.ssh/azure_key
Public key: ~/.ssh/azure_key.pub

---

## Step 2 - Store public key into variable
```bash
PUBLIC_KEY="$(cat ~/.ssh/azure_key.pub)"
```

---

## Step 3 - Create Linux VM using SSH Key authentication (Azure Shell)
```bash
az vm create \
  --resource-group app-grp \
  --name linuxvm01 \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --authentication-type ssh \
  --ssh-key-values "$PUBLIC_KEY" \
  --public-ip-sku Standard \
  --output table
```
Note: The VM must be deployed in the same region as the Resource Group

---

## Step 4 - Open port 22 (SSH)
```bash
az vm open-port \
  --resource-group app-grp \
  --name linuxvm01 \
  --port 22 \
  --priority 1003
```

---

## Step 5 - Retrieve Public IP
```bash
az vm show \
  --resource-group app-grp \
  --name linuxvm01 \
  --show-details \
  --query publicIps \
  -o tsv
```

---

## Step 6 – Connect via SSH using the private key
```bash
ssh -i ~/.ssh/azure_key azureuser@PUBLIC-IP
```

successful login confirms:
- SSH-key authentication
- Linux VM reachable
- Inbound NSG rules work

---

## Step 7 - optional verification: password login disabled (on local machine or Cloud Shell)
```bash
sudo sshd -T | grep passwordauthentication
```

Expected Output:
```nginx
passwordauthentication no
``` 

---

## Step 8 – Cleanup (optional)
```bash
az group delete --name app-grp --yes --no-wait
```

---

## Learnings (Linux + SSH)
- Generating and handling SSH keypairs
- secure authentication without passwords
- Logging into Linux VM via SSH
- Understanding NSG access requirements
- Comfort inside a Linux terminal in Azure
- Recognizing default security posture in cloud images


















