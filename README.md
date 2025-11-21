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

---

##  Prerequisites
- Azure Subscription  
- Azure CLI or Azure Cloud Shell  
- Basic understanding of regions & SKUs

  ---

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
  --size Standard_B1s \
  --admin-username  \
  --admin-password '' \
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







