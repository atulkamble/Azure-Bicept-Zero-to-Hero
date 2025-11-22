# 📘 **AZURE BICEP – ZERO TO HERO**

### *Complete Learning Flow: Theory → Example → Code*

---

# 🟦 **CHAPTER 1 – INTRODUCTION TO BICEP**

## 🎯 **Theory Points to Remember**

* Bicep = **Infrastructure as Code** for Azure (replacement for ARM JSON).
* Easy syntax: **clean, readable, no JSON brackets**, no state files (unlike Terraform).
* Built by Microsoft; 100% integrated with ARM engine.
* Automates validation, dependency handling, and IntelliSense.
* File extension = **.bicep**
* Deployment through:

  ```bash
  az deployment group create ...
  ```

## 🧪 **Minimal Installation Commands**

```bash
az bicep install
az bicep version
```

---

# 🟦 **CHAPTER 2 – BICEP BASICS**

## 🎯 **Theory Points**

* Every template contains:

  * **param** (input values)
  * **var** (pre-calculated values)
  * **resource** (Azure resources)
  * **output** (return values)
* Default scope = **resource group**
* Naming rules: use interpolation `${}`

---

## 🧩 **Example Code – Minimal Storage Account**

```bicep
param name string
resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: name
  location: resourceGroup().location
  sku: { name: 'Standard_LRS' }
}
```

---

# 🟦 **CHAPTER 3 – PARAMETERS, VARIABLES, EXPRESSIONS**

## 🎯 **Theory Points**

* Parameters accept user-defined inputs.
* Use securestring for secrets.
* Variables save computed values.
* Functions simplify logic:

  * `uniqueString()`
  * `concat()`
  * `toLower()`
  * `resourceId()`

---

## 🧩 **Minimal Code – Parameters**

```bicep
param env string = 'dev'
param size int = 2
```

## 🧩 **Minimal Code – Variables**

```bicep
var storageName = 'st${uniqueString(resourceGroup().id)}'
```

---

# 🟦 **CHAPTER 4 – NETWORKING (VNet, Subnet, NSG, Public IP)**

## 🎯 **Theory Points**

* VNet = private network container.
* Subnets = subdivisions inside VNet.
* NSG = firewall for traffic.
* Public IP = external connectivity.
* NIC connects VM → Subnet → NSG.

---

## 🧩 **Minimal VNet + Subnet**

```bicep
resource vnet 'Microsoft.Network/virtualNetworks@2023-05-01' = {
  name: 'myVnet'
  properties: {
    addressSpace: { addressPrefixes: ['10.0.0.0/16'] }
    subnets: [
      {
        name: 'web'
        properties: { addressPrefix: '10.0.1.0/24' }
      }
    ]
  }
}
```

## 🧩 **Minimal Public IP**

```bicep
resource pip 'Microsoft.Network/publicIPAddresses@2022-05-01' = {
  name: 'myPip'
  properties: { publicIPAllocationMethod: 'Dynamic' }
}
```

## 🧩 **Minimal NSG Rule**

```bicep
resource nsg 'Microsoft.Network/networkSecurityGroups@2021-02-01' = {
  name: 'myNsg'
  properties: {
    securityRules: [
      {
        name: 'SSH'
        properties: {
          protocol: 'Tcp'
          destinationPortRange: '22'
          access: 'Allow'
          direction: 'Inbound'
          priority: 100
        }
      }
    ]
  }
}
```

---

# 🟦 **CHAPTER 5 – VIRTUAL MACHINES (Linux + Windows)**

## 🎯 **Theory Points**

* VM requires:

  * Subnet
  * NIC
  * NSG
  * OS image
* Username + Password or SSH key
* Extensions help install software automatically.
* VM size affects pricing.

---

## 🧩 **Minimal Linux VM**

```bicep
param adminUser string = 'azureuser'
param password string

resource vm 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: 'myVM'
  properties: {
    hardwareProfile: { vmSize: 'Standard_B1s' }
    osProfile: {
      adminUsername: adminUser
      adminPassword: password
    }
    storageProfile: {
      imageReference: {
        publisher: 'Canonical'
        offer: 'UbuntuServer'
        sku: '18.04-LTS'
        version: 'latest'
      }
    }
  }
}
```

---

# 🟦 **CHAPTER 6 – LOOPS & CONDITIONAL DEPLOYMENTS**

## 🎯 **Theory Points**

* Loops reduce repetitive code.
* Use `range()` for numeric loops.
* Use `if(condition)` for optional deployments.

---

## 🧩 **Minimal Loop – 3 Storage Accounts**

```bicep
resource loopStg 'Microsoft.Storage/storageAccounts@2023-01-01' = [for i in range(0,3): {
  name: 'st${i}${uniqueString(resourceGroup().id)}'
  sku: { name: 'Standard_LRS' }
}]
```

## 🧩 **Minimal Conditional Resource**

```bicep
param enableDiag bool = false

resource diag 'Microsoft.Insights/diagnosticSettings@2021-05-01' = if (enableDiag) {
  name: 'diag1'
  properties: { workspaceId: log.id }
}
```

---

# 🟦 **CHAPTER 7 – MODULES (Reusable Architecture)**

## 🎯 **Theory Points**

* Modules help break large templates.
* Parent → Child communication via parameters and outputs.
* Best practice folder structure:

  ```
  modules/
    vnet.bicep
    vm.bicep
    storage.bicep
  ```

---

## 🧩 **Minimal Module (storage.bicep)**

```bicep
param name string
resource stg 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: name
  sku: { name: 'Standard_LRS' }
}
```

## 🧩 **Call Module**

```bicep
module s './modules/storage.bicep' = {
  name: 'deployStg'
  params: { name: 'stgmod01' }
}
```

---

# 🟦 **CHAPTER 8 – KEY VAULT + SECRET MANAGEMENT**

## 🎯 **Theory Points**

* Stores secrets, keys, certificates.
* Secure parameter → stored without exposing value.
* Managed identity recommended for accessing secrets.

---

## 🧩 **Minimal Key Vault + Secret**

```bicep
resource kv 'Microsoft.KeyVault/vaults@2023-02-01' = {
  name: 'myKV'
  properties: {
    sku: { name: 'standard' }
    tenantId: tenant().tenantId
  }
}

resource secret 'Microsoft.KeyVault/vaults/secrets@2023-02-01' = {
  name: 'mysecret'
  parent: kv
  properties: { value: 'P@ssword123' }
}
```

---

# 🟦 **CHAPTER 9 – APP SERVICE & CONTAINERS**

## 🎯 **Theory Points**

* App Service Plan = compute tier.
* Web App = actual application.
* LinuxFxVersion sets runtime.
* Container Web Apps pull images from ACR.

---

## 🧩 **Minimal Web App (Python)**

```bicep
resource plan 'Microsoft.Web/serverfarms@2022-03-01' = {
  name: 'myPlan'
  sku: { name: 'B1'; tier: 'Basic' }
}

resource app 'Microsoft.Web/sites@2022-03-01' = {
  name: 'myApp'
  properties: {
    serverFarmId: plan.id
    siteConfig: { linuxFxVersion: 'PYTHON|3.10' }
  }
}
```

## 🧩 **Minimal Container Web App**

```bicep
resource app 'Microsoft.Web/sites@2022-03-01' = {
  name: 'containerApp'
  properties: {
    siteConfig: {
      linuxFxVersion: 'DOCKER|myacr.azurecr.io/app:latest'
    }
  }
}
```

---

# 🟦 **CHAPTER 10 – AKS (Azure Kubernetes Service)**

## 🎯 **Theory Points**

* Managed Kubernetes by Azure.
* Node pools define VM type/count.
* Use Managed Identity for authentication.
* Requires VNet + ACR integration.

---

## 🧩 **Minimal AKS Cluster**

```bicep
resource aks 'Microsoft.ContainerService/managedClusters@2023-05-02' = {
  name: 'myAks'
  properties: {
    dnsPrefix: 'demo'
    agentPoolProfiles: [
      { name: 'nodepool1'; vmSize: 'Standard_DS2_v2'; count: 2 }
    ]
    identity: { type: 'SystemAssigned' }
  }
}
```

---

# 🟦 **CHAPTER 11 – CI/CD (GitHub Actions + Azure DevOps)**

## 🎯 **Theory Points**

* CI/CD automates template deployment.
* Stages:
  1️⃣ Validate
  2️⃣ What-if
  3️⃣ Deploy
* Uses `parameters.json` for environment separation.

---

## 🧩 **Minimal GitHub Actions Bicep Deployment**

```yaml
- name: Deploy Bicep
  run: |
    az deployment group create \
      --resource-group rg1 \
      --template-file main.bicep
```

---

# 🟦 **CHAPTER 12 – ADVANCED ENTERPRISE TOPICS**

## 🎯 **Theory Points**

* Diagnostics → Log Analytics
* Private Endpoints → secure networking
* Auto Tagging → FinOps
* Cross-subscription → multi-tenant setups
* Policy-as-Code → governance enforcement

---

# 🏆 **CAPSTONE PROJECTS (FINAL)**

## ⭐ **Project 1: Enterprise AKS Architecture**

* VNet
* Subnets
* ACR
* AKS Cluster
* Ingress
* Key Vault
* Logs + Monitoring
* Modules-based deployment

---

## ⭐ **Project 2: Secure Web App Deployment**

* App Service Plan
* Web App (container)
* Key Vault secret binding
* SQL Database
* Private Endpoint
* Role Assignments

---

## ⭐ **Project 3: Multi-Environment CI/CD**

* Dev / QA / Prod
* Parameter files
* GitHub Actions Multi-Stage
* Modular Bicep Architecture

---


Just tell me what you want next.
