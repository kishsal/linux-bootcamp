# Create and use an SSH public-private key pair for Linux VMs in Azure

## Create an SSH key pair
```
ssh-keygen -m PEM -t rsa -b 4096

az vm create --name VMname --resource-group RGname --image UbuntuLTS --generate-ssh-keys

```

## Provide an SSH public key when deploying a VM
```
cat ~/.ssh/id_rsa.pub

az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values mysshkey.pub
```

## SSH into your VM
```
ssh azureuser@myvm.westus.cloudapp.azure.com
```

## To obtain the host fingerprint via the portal, use the Run Command feature to execute the command:
```
 ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub | awk '{print $2
 ```