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

- Create secrets for the application
```
$ kubectl create secret generic <secret-name> --from-literal key=value
```

- Create ACR login secrets 
```
$ kubectl create secret docker-registry my-acr-secret> \
  --docker-server=<your-acr-login-server> \
  --docker-username=<your-acr-username> \     
  --docker-password=<your-acr-password-or-token> \
  --docker-email=<your-email>
```

- Install NGINX ingress controller
```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm install ingress-nginx ingress-nginx/ingress-nginx
```

- Get the Public IP address
```
$ kubectl get service -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].status.loadBalancer.ingress[0].ip}'
```

- Create the deployments
```
$ kubectl apply -f k8s-configs
$ kubectl apply -f k8s-configs/app
$ kubectl apply -f k8s-configs/mongo
$ kubectl apply -f k8s-configs/redis
```

<img width="1431" alt="image" src="https://github.com/brikis98/terraform-up-and-running-code/assets/49791498/87f4d698-950c-4f24-877f-362424709972">
