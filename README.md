# Steps to connect to AKS cluster and deploy Ingress controller, web, api and ingress route services

## Step 1: Connect to Jump box
a. Go to the Jumpbox VM
b. Click on bastion option
c. Key in Username
d. Key in password 

## Step 2: One time setup

a. Connect to Jump box
b.Go to https://kubernetes.io/docs/tasks/tools/install-kubectl/ and download the latest version of kubectl for windows
c. Go to https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest and download for windows and follow the 
installation Steps
d. Go to https://github.com/helm/helm/releases/tag/v3.2.4 and install windows. Unzip and copy helm.exe to c:\aksdeploy folder


## Step 3: Deploy Ingress controller to AKS cluster (https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip)

a. az login
b. az account set --subscription <SubscriptionID>
c. az aks get-credentials --resource-group <ResourceGroupName> --name <AKSClusterName>
d. kubectl create namespace ingress-basic #Create a namespace for your ingress resources
e. helm repo add stable https://kubernetes-charts.storage.googleapis.com/ #Add the official stable repository
f. #Use Helm to deploy an NGINX ingress controller 
 
   #change the following in internal-ingress.yaml
      for static IP for Ingress controller, uncomment loadBalancerIP: <IP Address> and replace with internal or external IP 
      Set service.beta.kubernetes.io/azure-load-balancer-internal: "true" if internal
      Set service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "AKS-SN" #change to your subnet name  
      save changes and run the following command:

      helm install nginx-ingress stable/nginx-ingress --namespace ingress-basic -f internal-ingress.yaml --set controller.replicaCount=2 --set controller.nodeSelector."beta.kubernetes.io/os"=linux --set defaultBackend.nodeSelector."beta.kubernetes.io/os"=linux --set controller.extraArgs.enable-ssl-passthrough=""

    #validate that it created nginx-ingress-controller (LoadBalancer) and nginx-ingress-default-backend(ClusterIP) 

       kubectl get service -l app=nginx-ingress --namespace ingress-basic

## STEP 4: DEPLOY Web, API and ingress route using YAML

    api-service.yaml
        change image: acrimg.azurecr.io/api-service:latest to your ACR image   
        save change and run kubectl apply -f api-service.yaml

    web-service.yaml
        image: acrimg.azurecr.io/api-service:latest #change to your ACR image   
        save change and run kubectl apply -f web-service.yaml    

    ingress-route.yaml
        change host: web.example.com with actual web host name    
        change host: api.example.com with actual api host name
        save changes and run kubectl apply -f ingress-route.yaml 

## STEP 5: Cleanup

i. helm list --namespace ingress-basic
ii. helm uninstall nginx-ingress --namespace ingress-basic
