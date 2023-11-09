```bash
LOCATION=northeurope
RG=rg-containerconf-demo
KV=akv-containerconf-demo
CR=cr0containerconf0demo
USER_ID=$(az ad signed-in-user show --query id -o tsv)

az group create \
  --name $RG \
  --location $LOCATION

az keyvault create \
  --name $KV \
  --location $LOCATION \
  --resource-group $RG \
  --enable-rbac-authorization

KV_ID=$(az keyvault show \
  --name $KV \
  --resource-group $RG \
  --query id \
  -o tsv)

az role assignment create \
--assignee $USER_ID \
--role "Key Vault Crypto Officer" \
--scope $KV_ID

az keyvault key create \
  --name cosign \
  --kty EC \
  --size 256 \
  --vault-name $KV

az acr create \
  --name $CR \
  --resource-group $RG \
  --location $LOCATION \
  --sku standard \
  --admin-enabled false \
  --anonymous-pull-enabled

CR_ID=$(az acr show \
  --name $CR \
  --resource-group $RG \
  --query id \
  -o tsv)

az role assignment create \
--assignee $USER_ID \
--role "AcrPush" \
--scope $CR_ID

az acr login --name $CR
```