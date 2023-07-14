# AKSWinContainersPVSample
Sample of Windows Pod using the Storage Account with PV and PVC

#### Change these parameters to your values

```azurecli
$AKS_RESOURCE_GROUP="accenture-rg"
$STORAGE_ACCOUNT_NAME="accstrgacctnik"
$SHARE_NAME="shareacctr"
$LOCATION="eastus"
```
#### Create storage account
```azurecli
az storage account create --name $STORAGE_ACCOUNT_NAME `
                          --resource-group $AKS_RESOURCE_GROUP `
                          --sku Standard_LRS
```
#### Create file share
```azurecli
az storage share create --name $SHARE_NAME `
                        --connection-string `
                        $(az storage account show-connection-string `
                        --name $STORAGE_ACCOUNT_NAME `
                        --resource-group $AKS_RESOURCE_GROUP -o tsv)
```

#### Get the Storage Key for Kubernetes Secret
```azurecli
$STORAGE_KEY=$(az storage account keys list `
       --resource-group $AKS_RESOURCE_GROUP `
       --account-name $STORAGE_ACCOUNT_NAME `
       --query "[0].value" -o tsv)
```

#### View the storage account name and storage key to make sure they are stored in the variables
```azurecli
$STORAGE_ACCOUNT_NAME
$STORAGE_KEY
```

#### Create the kubernetes secret file 
```azurecli
kubectl create secret generic azure-secret `
      --from-literal=azurestorageaccountname=$STORAGE_ACCOUNT_NAME `
      --from-literal=azurestorageaccountkey=$STORAGE_KEY `
      --dry-run=client `
      -o yaml > azure-secret.yaml
```

#### Apply the Kubernetes secret in the cluster
```azurecli
kubectl apply -f azure-secret.yaml
```

#### Check the share name to modify the Persistent Volume Yaml
```azurecli
echo $SHARE_NAME
```

#### Change the Persistant Volume file **azurefile-pv.yml** and replace the shareName value.

```yaml
      azureFile:
        secretName: azure-secret
        shareName: <place the value here>
        readOnly: false
      mountOptions:
```

#### Create the persistent volume
```azurecli
kubectl apply -f azurefile-pv.yml
```


#### Create the persistent volume Claim
```azurecli
kubectl apply -f azurefile-pvc.yml
```

#### Create the pod
```azurecli
kubectl apply -f test-pod.yml
```


#### Validate that pod is up and running
```azurecli
kubectl get pods 
```


## Done!