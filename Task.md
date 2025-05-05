##  **Problem Statement:**

You are working as a DevOps Engineer for a cloud-native startup. Your team is building a secure Kubernetes environment on Azure. The CTO wants the AKS cluster to be deployed inside a custom Virtual Network (VNet) to control networking, restrict access, and enable peering in the future.

Your task is to:

1. Create a custom VNet named `aks-vnet` in the `East US` region.
2. Create **one subnets** inside this VNet:
   - `aks-subnet`: For AKS cluster nodes. ( 10.1.0.0/24)
3. Ensure the subnet in the same address space but with **non-overlapping CIDRs**.
4. Create an AKS cluster named `prod-aks` using the `aks-subnet`.
5. Ensure the AKS cluster uses the **Azure CNI** networking model so that pods get IPs from the subnet.
6. Attach the AKS cluster to the existing VNet (not the default one).

---

## ✅ **Solution Using Azure CLI:**

```bash
# 1. Set variables
RESOURCE_GROUP="aks-rg"
LOCATION="eastus"
VNET_NAME="aks-vnet"
AKS_SUBNET_NAME="aks-subnet"
APP_SUBNET_NAME="app-subnet"
VNET_ADDRESS_PREFIX="10.0.0.0/8"
AKS_SUBNET_PREFIX="10.240.0.0/16"
APP_SUBNET_PREFIX="10.241.0.0/16"
AKS_CLUSTER_NAME="prod-aks"
NODE_COUNT=2

# 2. Create Resource Group
az group create --name $RESOURCE_GROUP --location $LOCATION

# 3. Create Virtual Network and subnets
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --name $VNET_NAME \
  --address-prefix $VNET_ADDRESS_PREFIX \
  --subnet-name $AKS_SUBNET_NAME \
  --subnet-prefix $AKS_SUBNET_PREFIX

# 4. Create additional subnet
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $APP_SUBNET_NAME \
  --address-prefix $APP_SUBNET_PREFIX

# 5. Get AKS subnet ID
AKS_SUBNET_ID=$(az network vnet subnet show \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $AKS_SUBNET_NAME \
  --query id -o tsv)

# 6. Create AKS cluster with Azure CNI (advanced networking)
az aks create \
  --resource-group $RESOURCE_GROUP \
  --name $AKS_CLUSTER_NAME \
  --node-count $NODE_COUNT \
  --network-plugin azure \
  --vnet-subnet-id $AKS_SUBNET_ID \
  --generate-ssh-keys \
  --enable-managed-identity

# 7. Get AKS credentials
az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_CLUSTER_NAME
```

---

## 🔍 **Bonus Challenge for Learners (Optional):**

- Use `kubectl` to deploy a sample NGINX pod and verify the pod IP range belongs to the `aks-subnet`.
- Peer this VNet with another VNet and allow communication between them (advanced networking task).
- Create a Network Security Group (NSG) and attach it to the `app-subnet` with custom rules.
