Referring to the ACA quick start doc, [quick start thru portal](https://learn.microsoft.com/en-us/azure/container-apps/quickstart-portal) and [quick start thru bash](https://learn.microsoft.com/en-us/azure/container-apps/get-started?tabs=bash )

# create the azure container app through portal 
+ create a container app environment 'my-app-env' with
  Zone redundancy: **disabled**
  log space: workspacemy-resource-group23e
  Public Network Access: **enabled**
+ created a new virtual net: my-app-vn (10.0.0.0/16)
+ created a new subnet: my-app-vn-sbn (10.0.0.0/23 Range: 10.0.0.0 - 10.0.1.255)
+ created container app: 'my-app-api'
+ select the image registry 'my-image-registry.azurecr.io'
+ select the pushed image 'your_docker_image'
+ fill the 'Arguments override' with the following parameters
-m, src.database.postgres, --address, 0.0.0.0, --port, 5000, --db_host, your-db-host, --db_port, 5432, --db_name, your-db-name, --db_username, yourusername, --db_password, $(DB_PASSWORD)
+ ingress setting: 
  **enabled**
  **accepting traffic from anywhere**
  ingress type: **HTTP**
  transport: **AUTO**
  Insecure connections: **allowed**
  Target port: **5000**
  Session affinity: **not enabled**

# set the azure secret through 'containerapp' tool
+ install the extension 'containerapp'
```shell
az extension add --name containerapp --upgrade --allow-preview true
```
+ set db-password as secret on azure
```shell
az containerapp secret set --name my-app-api --resource-group my-resource-group --secrets db-password='<db-password>'
az containerapp update --name my-app-api --resource-group my-resource-group --set-env-vars DB_PASSWORD=secretref:db-password 
```
 
# export the configuration file so that it can be reused to deploy the aca easily

```shell
az containerapp export --name my-app-api --resource-group my-resource-group --file my-app-api.exported.yaml
```
But my azure cli is too old, doesn't support the "export", have to upgrade it 
```shell
az upgrade
```
After upgrading, the extension 'containerapp' is part of the azure cli tool, but it doesn't support 'export'. I have to dump all the configuration
```shell
az containerapp show --name my-app-api --resource-group my-resource-group --output yaml > my-app-api.raw.yaml 
```
but this raw yaml cannot be used directly to deploy an aca, have to clean it up.

 