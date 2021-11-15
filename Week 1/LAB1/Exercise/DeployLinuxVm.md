# Create a RG
```
az group create --name linux-rg --location eastus
```

# az vm create
```
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

# Open port 80 for web traffic
```
az vm open-port --port 80 --resource-group linux-rg --name myVM
```

# Connect to virtual machine
```
ssh azureuser@40.68.254.142
```

# Install web server
```
sudo apt-get -y update
sudo apt-get -y install nginx
```

# Remove web server
```
sudo apt-get -y remove nginx
```

# Clean up resources
```
az group delete --name linux-rg
```
 
