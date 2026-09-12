# Azure Storage (Core)

| Topic                               | What to Cover                                                                              |
| ----------------------------------- | ------------------------------------------------------------------------------------------ |
| **Storage Account Overview**        | Storage account purpose, account types, endpoints, containers, file shares, queues, tables |
| **Blob Storage (Hot/Cool/Archive)** | Containers, blobs, access tiers, upload/download, tier selection                           |
| **Azure Files**                     | SMB/NFS file shares, mounting shares, shared storage use cases                             |
| **Managed Disks**                   | OS disk, data disk, Standard HDD/SSD, Premium SSD, attaching disks to VMs                  |
| **Access Keys & SAS**               | Storage account keys, connection strings, SAS tokens, permissions and expiry               |
| **Lifecycle Management**            | Automatically move blobs between tiers or delete old data using rules                      |
| **Data Redundancy Basics**          | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS                                                       |
| **Storage Use Cases**               | Backups, static content, VM disks, shared files, logs, archives, application data          |

## 1. Storage Account Overview

An **Azure Storage Account** is the top-level Azure resource used to provide storage services.

```text
Storage Account
│
├── Blob Storage
│   └── Containers → Blobs
│
├── Azure Files
│   └── File Shares → Directories → Files
│
├── Queue Storage
│
└── Table Storage
```

Basic Azure CLI practice:

```bash
az login

az group create \
  --name storage-rg \
  --location centralindia

az storage account create \
  --name mystorage12345 \
  --resource-group storage-rg \
  --location centralindia \
  --sku Standard_LRS \
  --kind StorageV2
```

Check the account:

```bash
az storage account show \
  --name mystorage12345 \
  --resource-group storage-rg
```

## 2. Blob Storage — Hot, Cool & Archive

Blob Storage is **object storage** used for images, videos, documents, backups, logs and other unstructured data.

```text
Storage Account
      │
   Container
      │
 ┌────┼─────┐
Blob Blob  Blob
```

Access tiers:

| Tier        | Best For                   | Access               |
| ----------- | -------------------------- | -------------------- |
| **Hot**     | Frequently accessed data   | Fast/frequent        |
| **Cool**    | Infrequently accessed data | Occasional           |
| **Cold**    | Rarely accessed data       | Rare                 |
| **Archive** | Long-term archival         | Requires rehydration |

Create a container:

```bash
az storage container create \
  --name images \
  --account-name mystorage12345 \
  --auth-mode login
```

Upload:

```bash
az storage blob upload \
  --account-name mystorage12345 \
  --container-name images \
  --name logo.png \
  --file logo.png \
  --auth-mode login
```

List blobs:

```bash
az storage blob list \
  --account-name mystorage12345 \
  --container-name images \
  --auth-mode login \
  --output table
```

Change tier:

```bash
az storage blob set-tier \
  --account-name mystorage12345 \
  --container-name images \
  --name logo.png \
  --tier Cool \
  --auth-mode login
```

## 3. Azure Files

Azure Files provides managed **file shares** accessible using protocols such as SMB.

```text
Multiple Systems
     │
     ├──────────┐
     ▼          ▼
  Server 1   Server 2
     │          │
     └────┬─────┘
          ▼
     Azure Files
       File Share
```

Create a share:

```bash
az storage share create \
  --name sharedfiles \
  --account-name mystorage12345
```

Typical use cases include shared application files, lift-and-shift applications, centralized file storage and replacing/on-extending traditional file servers.

## 4. Managed Disks

Managed Disks provide persistent block storage primarily for Azure VMs.

```text
Azure VM
│
├── OS Disk
│
└── Data Disk
```

Common disk choices include **Standard HDD**, **Standard SSD**, **Premium SSD** and **Ultra Disk**, depending on workload and supported VM/region configurations.

Create a disk:

```bash
az disk create \
  --resource-group storage-rg \
  --name datadisk01 \
  --size-gb 32 \
  --sku Premium_LRS
```

Attach it:

```bash
az vm disk attach \
  --resource-group storage-rg \
  --vm-name myvm \
  --name datadisk01
```

Then inside Linux:

```bash
lsblk
sudo mkfs.xfs /dev/sdc
sudo mkdir /data
sudo mount /dev/sdc /data

df -h
```

> Always verify the actual device name with `lsblk` before formatting.

## 5. Access Keys & SAS

### Access Keys

Storage accounts provide account-level keys with broad access.

```bash
az storage account keys list \
  --resource-group storage-rg \
  --account-name mystorage12345
```

Because these keys are powerful credentials, avoid hardcoding them in application code.

### SAS — Shared Access Signature

SAS provides **delegated, restricted access**.

You can control:

```text
SAS
├── Resource
├── Permissions
│   ├── Read
│   ├── Write
│   ├── Delete
│   └── List
└── Expiration
```

Conceptually:

```text
Access Key
   ↓
Broad account access

SAS Token
   ↓
Limited resource + permissions + time
```

For Azure-hosted applications, prefer **Microsoft Entra ID + managed identities/RBAC** where supported rather than distributing account keys.

## 6. Lifecycle Management

Lifecycle policies automate blob tiering and deletion.

Example strategy:

```text
New Blob
   │
   ▼
 HOT
   │ 30 days
   ▼
 COOL
   │ 90 days
   ▼
 COLD / ARCHIVE
   │ 365 days
   ▼
 DELETE
```

Example policy:

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "archive-old-data",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ]
        }
      }
    }
  ]
}
```

Apply:

```bash
az storage account management-policy create \
  --account-name mystorage12345 \
  --resource-group storage-rg \
  --policy policy.json
```

## 7. Data Redundancy Basics

```text
LRS
Data → Copies within one datacenter/zone scope

ZRS
Data → Zone 1
     → Zone 2
     → Zone 3

GRS
Primary Region
     │
     └────────────► Secondary Region

GZRS
Zones in Primary Region
     │
     └────────────► Secondary Region
```

| Option      | Basic Idea                           |
| ----------- | ------------------------------------ |
| **LRS**     | Local redundancy                     |
| **ZRS**     | Redundancy across availability zones |
| **GRS**     | Geo-replication to secondary region  |
| **RA-GRS**  | GRS + read access to secondary       |
| **GZRS**    | Zone redundancy + geo-replication    |
| **RA-GZRS** | GZRS + read access to secondary      |

Easy way to remember:

**L = Local → Z = Zones → G = Geographic region**

## 8. Storage Use Cases

| Requirement                     | Azure Storage Choice |
| ------------------------------- | -------------------- |
| Images/videos/documents         | **Blob Storage**     |
| Static website content          | **Blob Storage**     |
| Shared folders                  | **Azure Files**      |
| VM OS/data storage              | **Managed Disks**    |
| Long-term backup/archive        | **Blob Archive**     |
| Application messages            | **Queue Storage**    |
| NoSQL key/attribute data        | **Table Storage**    |
| Frequently accessed objects     | **Hot tier**         |
| Rarely accessed objects         | **Cool/Cold**        |
| Long-term rarely retrieved data | **Archive**          |

### Recommended hands-on sequence

```text
Create Resource Group
        ↓
Create Storage Account
        ↓
Create Blob Container
        ↓
Upload Blob
        ↓
Change Access Tier
        ↓
Create Azure File Share
        ↓
Practice SAS
        ↓
Configure Lifecycle Policy
        ↓
Compare LRS / ZRS / GRS
        ↓
Create & Attach Managed Disk
```

This gives students one continuous **Azure Storage Core lab** covering nearly every topic in your table.


# 📦 Azure Storage Accounts: Full Setup with Codes

---

## 📖 Overview

Azure Storage Account is a cloud storage solution providing object, file, queue, table, and disk storage.

---

## 🛠️ Steps to Create a Storage Account

1. **Create a Resource Group**
2. **Create a Storage Account**
3. **Configure Storage Account Options** (like redundancy, access tier, networking)
4. **List, Update, Delete Storage Accounts**

---

## 📌 Azure Storage Services - Key Points

### 🔹 Overview

* Scalable, durable, highly available cloud storage.
* Storage Types: **Blob**, **File Shares**, **Queues**, **Tables**, **Managed Disks**.

### 🔹 Types of Azure Storage

1. **Blob Storage** – Object storage for unstructured data.
2. **File Shares** – Managed file shares accessible via SMB/NFS.
3. **Queue Storage** – Message queueing.
4. **Table Storage** – NoSQL key-value database.
5. **Azure Managed Disks** – Virtual disks for VMs.

### 🔹 Blob Types

* **Block Blob** – For large files.
* **Page Blob** – Random read/write (used for Azure VMs).
* **Append Blob** – Optimized for append operations.

### 🔹 Disk Types

* **Premium SSD (v2/v1)**
* **Standard SSD**
* **Standard HDD**

### 🔹 Access Keys & SAS

* **Access Keys**: Full access to storage account.
* **SAS Token**: Limited, time-restricted access.

### 🔹 Data Protection & Redundancy

* **Replication Types**: LRS, ZRS, GRS, GZRS, RAGRS, RAGZRS
* **Features**: Versioning, Soft Delete, Snapshot, Change Feed, Object Replication, Inventory, Lifecycle Policies.

### 🔹 Static Website Hosting & CDN

* Host static web apps via Storage Account.
* Integrate with Azure CDN for global delivery.

---

## 🕌 Azure CLI Commands

> Ensure you're logged in:

```bash
az login
```

**Create Resource Group**

```bash
az group create --name MyResourceGroup --location eastus
```

**Create Storage Account**

```bash
az storage account create \
  --name mystorageacct12345 \
  --resource-group MyResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot
```

**List Storage Accounts**

```bash
az storage account list --resource-group MyResourceGroup -o table
```

**Show Storage Account Details**

```bash
az storage account show --name mystorageacct12345 --resource-group MyResourceGroup
```

**Delete Storage Account**

```bash
az storage account delete --name mystorageacct12345 --resource-group MyResourceGroup
```

**List Storage Account Keys**

```bash
az storage account keys list --account-name mystorageacct12345 --resource-group MyResourceGroup -o table
```

---

## 🕌 Azure PowerShell Commands

> Ensure you're logged in:

```powershell
Connect-AzAccount
```

**Create Resource Group**

```powershell
New-AzResourceGroup -Name MyResourceGroup -Location "East US"
```

**Create Storage Account**

```powershell
New-AzStorageAccount -ResourceGroupName "MyResourceGroup" `
  -Name "mystorageacct12345" `
  -Location "East US" `
  -SkuName "Standard_LRS" `
  -Kind "StorageV2" `
  -AccessTier "Hot"
```

**List Storage Accounts**

```powershell
Get-AzStorageAccount -ResourceGroupName "MyResourceGroup"
```

**Delete Storage Account**

```powershell
Remove-AzStorageAccount -ResourceGroupName "MyResourceGroup" -Name "mystorageacct12345"
```

---

## 🕌 ARM Template

📄 `storageAccount.json`

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string"
    },
    "location": {
      "type": "string",
      "defaultValue": "eastus"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

**Deploy ARM Template**

```bash
az deployment group create --resource-group MyResourceGroup --template-file storageAccount.json --parameters storageAccountName=mystorageacct12345
```

---

## 🕌 Bicep Template

📄 `storageAccount.bicep`

```bicep
param storageAccountName string
param location string = 'eastus'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

**Deploy Bicep Template**

```bash
az deployment group create --resource-group MyResourceGroup --template-file storageAccount.bicep --parameters storageAccountName=mystorageacct12345
```

---

## 🕌 Terraform

📄 `main.tf`

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "MyResourceGroup"
  location = "East US"
}

resource "azurerm_storage_account" "example" {
  name                     = "mystorageacct12345"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  account_kind             = "StorageV2"
  access_tier              = "Hot"
}
```

**Deploy Terraform Configuration**

```bash
terraform init
terraform plan
terraform apply
```

---
Let's practice **Azure Storage Account Upload Scenarios** based on different **Azure Storage Types**. I’ll give you CLI-based examples for each type:

---

## **Azure Storage Types & Upload Practice**

| **Storage Type**               | **Use Case**                             | **Upload Command Example (Azure CLI)**          |
| ------------------------------ | ---------------------------------------- | ----------------------------------------------- |
| **Blob Storage**               | Large unstructured data (images, videos) | `az storage blob upload`                        |
| **File Storage (File Shares)** | Lift-and-shift apps, SMB shares          | `az storage file upload`                        |
| **Queue Storage**              | Message queues (not for files)           | Not applicable (use `az storage message put`)   |
| **Table Storage**              | NoSQL key-value pairs (not for files)    | Not applicable (use `az storage entity insert`) |

---

## **1. Blob Storage Upload**

```bash
# Variables
STORAGE_ACCOUNT_NAME=<your_storage_account>
CONTAINER_NAME=<your_container_name>
FILE_PATH=/path/to/file.txt

# Upload file to Blob Container
az storage blob upload \
  --account-name $STORAGE_ACCOUNT_NAME \
  --container-name $CONTAINER_NAME \
  --file $FILE_PATH \
  --name file.txt
```
```
az storage blob upload \
  --account-name atulkamble9796857478 \
  --container-name mycontainer \
  --file /Users/atul/Downloads \
  --name a.txt
```

---

## **2. File Storage Upload (File Shares)**

```bash
# Variables
STORAGE_ACCOUNT_NAME=<your_storage_account>
SHARE_NAME=<your_fileshare_name>
FILE_PATH=/path/to/file.txt

# Upload file to File Share
az storage file upload \
  --account-name $STORAGE_ACCOUNT_NAME \
  --share-name $SHARE_NAME \
  --source $FILE_PATH
```

---

## **3. Queue Storage (Add Message to Queue)**

```bash
# Variables
STORAGE_ACCOUNT_NAME=<your_storage_account>
QUEUE_NAME=<your_queue_name>
MESSAGE_CONTENT="Hello Azure Queue"

# Put message into queue
az storage message put \
  --account-name $STORAGE_ACCOUNT_NAME \
  --queue-name $QUEUE_NAME \
  --content "$MESSAGE_CONTENT"
```

---

## **4. Table Storage (Insert Entity)**

```bash
# Variables
STORAGE_ACCOUNT_NAME=<your_storage_account>
TABLE_NAME=<your_table_name>
PARTITION_KEY="SamplePartition"
ROW_KEY="1"
DATA='{"Name":"Atul","Role":"Architect"}'

# Insert entity into Table
az storage entity insert \
  --account-name $STORAGE_ACCOUNT_NAME \
  --table-name $TABLE_NAME \
  --entity PartitionKey=$PARTITION_KEY RowKey=$ROW_KEY Name=Atul Role=Architect
```

---

## **Common Prerequisites**

```bash
# Login to Azure
az login

# Set subscription (optional)
az account set --subscription <subscription_id>
```

---

## **Summary**

| **Storage Type** | **Command Used**           |
| ---------------- | -------------------------- |
| Blob Storage     | `az storage blob upload`   |
| File Storage     | `az storage file upload`   |
| Queue Storage    | `az storage message put`   |
| Table Storage    | `az storage entity insert` |

---

### Do you want a **Terraform automation script** for uploading files to Azure Blob & File Share?
