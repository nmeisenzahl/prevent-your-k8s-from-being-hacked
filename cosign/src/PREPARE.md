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
  --name containerconf \
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

# Docker
az acr login --name $CR

# nerdctl
export TOKEN=$(az acr login -n $CR --expose-token -ojson |jq -r .accessToken)
nerdctl login $CR.azurecr.io -p $TOKEN -u 00000000-0000-0000-0000-000000000000
```
