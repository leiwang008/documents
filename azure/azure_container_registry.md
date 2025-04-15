+ create resource group and create azure container registry (ACR)
```shell
az login 
az group create --name my-resource-group --location eastus
az acr create --resource-group my-resource-group --name my-image-registry --sku Basic
```
+ login to ACR

```shell
$ az acr login --name my-image-registry
Login Succeeded
```
+ re-tag your image and push the re-tagged image to ACR

```shell
docker tag your_docker_image:latest my-image-registry.azurecr.io/your_docker_image:latest
docker push my-image-registry.azurecr.io/your_docker_image:latest
```

+ enable the authentication/login for this ACR
Go to your ACR in the Azure Portal and Under Settings → Access keys, Toggle Admin user to Enabled.