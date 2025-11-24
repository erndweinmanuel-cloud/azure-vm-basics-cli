# azure-vm-basics-cli
Mini Azure Project – Deploy a Windows VM via Azure CLI

This mini-lab demonstrates how to deploy a Windows Server virtual machine **fully via Azure CLI**, without using the Azure Portal.  
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

## Step 6 - Cleanup
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








