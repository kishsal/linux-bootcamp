# Manage Linux VMs with the Azure CLI

## Create RG
```
az group create --name myResourceGroupVM --location eastus
```

## Create virtual machine
```
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

## Connect to VM
```
ssh azureuser@52.174.34.95
```

## Understand VM images
```
az vm image list --output table

az vm image list --offer CentOS --all --output table

az vm create --resource-group myResourceGroupVM --name myVM2 --image OpenLogic:CentOS:6.5:latest --generate-ssh-keys
```

## Find available VM sizes
```
az vm list-sizes --location eastus --output table
```

## Create VM with specific size
```
az vm create \
    --resource-group myResourceGroupVM \
    --name myVM3 \
    --image UbuntuLTS \
    --size Standard_F4s \
    --generate-ssh-keys
```

## Resize a VM
```
az vm show --resource-group myResourceGroupVM --name myVM --query hardwareProfile.vmSize

az vm list-vm-resize-options --resource-group myResourceGroupVM --name myVM --query [].name

az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_DS4_v2

az vm deallocate --resource-group myResourceGroupVM --name myVM

az vm resize --resource-group myResourceGroupVM --name myVM --size Standard_GS1

az vm start --resource-group myResourceGroupVM --name myVM
```

## Find the power state
```
az vm get-instance-view \
    --name myVM \
    --resource-group myResourceGroupVM \
    --query instanceView.statuses[1] --output table
```

## Get IP address
```
az vm list-ip-addresses --resource-group myResourceGroupVM --name myVM --output table
```

## Stop the VM
```
az vm stop --resource-group myResourceGroupVM --name myVM
```

## Start the VM
```
az vm start --resource-group myResourceGroupVM --name myVM
```

## Delete RG
```
az group delete --name myResourceGroupVM --no-wait --yes`
```


##


