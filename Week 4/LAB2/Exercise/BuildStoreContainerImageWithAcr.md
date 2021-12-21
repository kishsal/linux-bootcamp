# Build and store container images with Azure Container Registry (https://docs.microsoft.com/en-us/learn/modules/build-and-store-container-images/)

1) Create a RG
   ```
   az group create --name learn-deploy-acr-rg --location eastus
   ```

2) Set a variable
   ```
   ACR_NAME=ksacr2021
   ```

3) Create new ACR
   ```
   az acr create --resource-group learn-deploy-acr-rg --name $ACR_NAME --sku Premium
   ```

4) Create a Dockerfile in VS Code
   ```
   code

   FROM    node:9-alpine
    ADD     https://raw.githubusercontent.com/Azure-Samples/acr-build-helloworld-node/master/package.json /
    ADD     https://raw.githubusercontent.com/Azure-Samples/acr-build-helloworld-node/master/server.js /
    RUN     npm install
    EXPOSE  80
    CMD     ["node", "server.js"]
    ```
    Save file as 'Dockerfile'

5) Build container image from the Dockerfile
    ```
    az acr build --registry $ACR_NAME --image helloacrtasks:v1 .
    ```

6) Verify the image
```
az acr repository list --name $ACR_NAME --output table
```

7) Enabled Admin account on ACR
```
az acr update -n $ACR_NAME --admin-enabled true
```

8) Retrieve username and password
```
az acr credential show --name $ACR_NAME
```

9) Deploy a container instance
```
az container create \
    --resource-group learn-deploy-acr-rg \
    --name acr-tasks \
    --image $ACR_NAME.azurecr.io/helloacrtasks:v1 \
    --registry-login-server $ACR_NAME.azurecr.io \
    --ip-address Public \
    --location eastus \
    --registry-username ksacr2021 \
    --registry-password bTaiSFwDawZrcF3cF1df03w7A+qEcNAD
```

10) Get the IP address of the ACI
```
az container show --resource-group  learn-deploy-acr-rg --name acr-tasks --query ipAddress.ip --output table
```

11) Open a browser and enter IP Address

12) Create a replicated region for ACR
```
az acr replication create --registry $ACR_NAME --location japaneast
```

13) Retrieve all container image replicas
```
az acr replication list --registry $ACR_NAME --output table
```

14) Delete RG