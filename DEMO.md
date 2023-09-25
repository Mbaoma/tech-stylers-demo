In this demo, I will deploy a 3-tier application to Azure Kubernetes Services (AKS)

### Steps
- Create a Docker image
```
$ docker login
$ docker build -t hub-username/image-name .
```

- Login to your Azure account
```
$ az login
```

- Create a resource group and an ACR
```
$ az group create --name myResourceGroup --location eastus
$ az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

- Login to the ACR to fetch your image's login server
```
$ az acr update --name techstylersacr --admin-enabled true
$ az acr login --name <acrName>
$ az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
```

- [Push](https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-acr?tabs=azure-cli) the image to Azure Container Registry
```
$ docker tag docker-image:tag <acrLoginServer>/image-name:tag
$ docker images
$ docker push <acrLoginServer>/image-name:tag
```

- Get your image name and tag
$ az acr repository list --name <ACRname>    
$ az acr repository show-tags --name <ACRname> --repository <repo>

- Create and connect to the AKS CLI
[Guide](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli)
```
$ az aks create -g myResourceGroup -n myAKSCluster --enable-managed-identity --node-count 1 --enable-addons monitoring --generate-ssh-keys
$ az aks install-cli
$ az account set --subscription xxxx-xxx-xx-xx-xxxx
$ az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
$ kubectl get nodes
```

- Create a secret
```
$ kubectl create secret generic <secret-name> --from-literal key=value
```

- Create the deployments
```
$ kubectl apply -f k8s-configs
```