**PRACTICA COMANDOS BASICOS KUBECTL y CREACION AKS**
------------------------------------------------------------------

Tutorial didáctico con fines de demostración, formacion y meetups

- BASADO EN TUTORIALES DE AKS Y BOOTCAMP GOOGLE

### Configurar AKS y acceso para kubectl 

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***

$ az group create --name rg-workshop-aks-lab --location westeurope
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterWorkshopLabServicePrincipal
$ az aks create --resource-group rg-workshop-aks-lab \
    --name aks-workshop-lab \
    --location westeurope \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --service-principal <appId> \
    --client-secret <password>

# Configurar acceso Kubectl
$ az aks get-credentials --resource-group rg-workshop-aks-lab --name aks-workshop-lab --admin
$ kubectl get componentstatus
$ kubectl get nodes -o wide

# Configurar Dashboard
$ kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

$ az aks browse --resource-group rg-workshop-aks-lab --name aks-workshop-lab 
```

### Practicando NAMESPACES

Desde la shell de Azure creamos los siguientes namespaces para varios nginx separados
Por defecto si no funciona --image=nginx usar estos otros en el nombre de la imagen
- docker.io/library/nginx:alpine 
- k8s.gcr.io/nginx:alpine 

```
# creamos los namespaces para la practica
$ kubectl create namespace env-dev
$ kubectl create namespace env-qa
$ kubectl create namespace env-prod

$ kubectl get namespaces

# creamos los pods para la practica (si no indicamos nada namespace=default)
$ kubectl run nginx --image=nginx  
$ kubectl run nginx --image=nginx --namespace env-dev
$ kubectl run nginx --image=nginx --namespace env-qa
$ kubectl run nginx --image=nginx --namespace env-prod

# saber donde estan desplegados los pods 
$ kubectl get pods --all-namespaces
$ kubectl get pods --namespace env-dev

# describir pods desplegados
$ kubectl describe pod nginx --namespace env-dev

# conectando al pod mediante port-forwarding (navegar en localhost:9090)
$ kubectl port-forward nginx-7bb7cd8db5-jk7cd 9090:80 --namespace env-dev
```


