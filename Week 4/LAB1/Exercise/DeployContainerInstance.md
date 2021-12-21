# Deploy a container instance in Azure using the Azure CLI (https://docs.microsoft.com/en-us/azure/container-instances/container-instances-quickstart)

1) Create a RG
   ```
   az group create --name myResourceGroup --location eastus
   ```

2) Create a Container
   ```
   az container create --resource-group myResourceGroup --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label aci-demoks2021 --ports 80
   ```

3) Get the FQDN of the Container instance
   ```
   az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
   ```

4) Pull the container logs to troubleshoot
   ```
   az container logs --resource-group myResourceGroup --name mycontainer
   ```

5) Attached output stream
   ```
   az container attach --resource-group myResourceGroup --name mycontainer
   ```

6) Clean up resources
   ```
   az container delete --resource-group myResourceGroup --name mycontainer

   az container list --resource-group myResourceGroup --output table
   ```

7) Delete RG
   ```
    az group delete --name myResourceGroup
    ```

    