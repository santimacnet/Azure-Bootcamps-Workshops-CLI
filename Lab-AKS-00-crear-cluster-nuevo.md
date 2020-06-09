**PRACTICA AKS KUBERNETES CREAR CLUSTER NUEVO**
------------------------------------------------------------------

Tutorial para charlas, eventos, meetups y formación sobre AKS donde veremos:

   - Paso1: Creando un Service Principal para AKS
   - Paso2: Creando cluster de AKS 
   - Paso3: Creando registry de imagenes ACR
   - Paso4: Configurar Kubectl y Credenciales acceso AKS
   - Paso5: Configurar acceso Dashboard AKS
   - Paso6: Acceso Dashboard mediante port-forward


Requerimientos Tutorial:

    - Azure y Kubernetes fundamentos basicos
    - Azure CLI, Kubectl y shell.azure.com
    - Suscripcion de Azure con permisos Admin para Azure Active Directory
    - Chuleta: https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf


### Paso1: Creando un Service Principal para AKS

Contexto: Cuando creamos un AKS, Azure automaticamente genera un Service Principal de AAD, pero esto no siempre ocurre (arrggg!!) y entonces tenemos que generarlo nosotros previamente para indicarlo en el comando: az aks create... 

El Service Principal es necesario para interactuar con otros recursos de Azure, como un Load Balancer, registry ACR, Virtual Networks,etc y si AKS no dispone de uno en el momento de la creación fallará y nos dara un error: "AADSTS700016: Application with identifier xxxxx-xxxxx-xxxxx-xxxxxx was not found in the directory xxxxx-xxxxx-xxxxx-xxxxxx.

Explicacion en Microsoft Docs y Proyecto GitHub Service principals for AKS que podemos consultar en este enlace: 
https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal

Creamos el Service Principal y guardamos en notepad la respuesta JSON con AppID y Password, este SP se guarda en Azure Active Directory en la opcion de Aplicaciones Registradas:

```
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterSantiPruebasServicePrincipal
...respuesta json...
```

### Paso2: Creando cluster de AKS

Definims variables entorno para los recursos
```
RG_NAME=rg-aks-cluster-demo
RG_LOCATION=westeurope
ACR_NAME=acrhubdemo
AKS_NAME-demonew

```

Creamos el grupo de recursos y AKS con la opcion de autoescalado, la creacion del cluster puede tardar entre 10 y 15 minutos.
```
$ az group create --name $RG_NAME --location $RG_LOCATION

$ az aks create --resource-group $RG_NAME \
    --name aks-cluster-pruebas \
    --location westeurope \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --service-principal <appId> \
    --client-secret <password>
    
# verificamos AKS actualizado y version definida
$ az aks list -o table
$ az aks show --resource-group $RG_NAME --name aks-cluster-pruebas -o table    
```

### Paso3: Creando registry de imagenes ACR

Creamos el grupo de recursos y AKS con la opcion de autoescalado, la creacion del cluster puede tardar entre 10 y 15 minutos.
```
# creamos el ACR para publicar imagenes
$ az acr create --name $ACR_NAME --resource-group $RG_NAME --location $RG_LOCATION --sku Basic

# atachamos AKS para deployar imagenes
$ az aks update --name aks-cluster-pruebas --resource-group $RG_NAME --attach-acr $ACR_NAME
  - AAD role propagation - Running.... (tarda unos minutos)
  
# importamos imagen NGNIX para pruebas AKS
$ az acr import  -n $ACR_NAME -g $RG_NAME --source docker.io/library/nginx:latest --image nginx:v1
$ az acr repository list --name $ACR_NAME --output table
```

Nota: tambien podemos crear primero ACR y usar el parametro --attach-acr en la creacion de AKS

### Paso4: Configurar Kubectl y Credenciales acceso AKS

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***
```

Configurar la herramienta de Kubectl para trabajar con AKS
```
$ az aks install-cli (para instalar kubectl si no lo tenemos en local)

$ az aks get-credentials --resource-group <nombre-rg> --name <nombre-aks-cluster> --admin
  Merged ........ as current context in /home/santimacnet/.kube/config

#  ejecutar comandos Kubectl para verificar aks funcionando
$ kubectl version
$ kubectl config current-context (para ver contexto correcto AKS)
$ kubectl cluster-info
$ kubectl get nodes

# realizar deploy nginx para pruebas desde ACR
$ kubectl run demo-nginx --image=acrhubdemo.azurecr.io/nginx:v1 --port=80
$ kubectl expose deployment demo-nginx --port=80 --type=LoadBalancer

$ kubectl get all (ver IP publica para nginx)
```

### Paso5: Configurar acceso Dashboard AKS

```
# crear role para permisos con rbac
$ kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

# abrir dashboard como proxy
$ az aks browse --resource-group $RG_NAME --name <nombre-aks>

# ver lista de roles definidos
$ kubectl get clusterrolebinding 

# quitar permisos para no acceder al Dashboard
$ kubectl delete clusterrolebinding kubernetes-dashboard -n kube-system
```

### Paso6: Acceso Dashboard mediante port-forward

Esta opción reenvía las conexiones de un puerto local a un puerto en un pod. En comparación con kubectl proxy, kubectl port-forward es más genérico, ya que puede reenviar TCP tráfico mientras que kubectl proxy solo puede reenviar tráfico HTTP.

Usar esto es realmente útil para pruebas/depuración (conectarnos directamente al pod y depurarlo), pero NUNCA como alternativa y exponer aplicaciones para entornos de producción.

```
$ kubectl -n kube-system get pods
-------------------------------------------------------------------------------
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-698c77c5d7-j5bbq                1/1     Running   0          22h
coredns-698c77c5d7-wq6sr                1/1     Running   0          22h
coredns-autoscaler-79b778686c-xbjbd     1/1     Running   0          22h
kube-proxy-6pm6r                        1/1     Running   0          22h
kube-proxy-rh8rr                        1/1     Running   0          22h
[kubernetes-dashboard-74d8c675bc-qgd7l]   1/1     Running   1          22h
metrics-server-69df9f75bf-rkpsp         1/1     Running   2          22h
omsagent-5cjg9                          1/1     Running   0          22h
omsagent-b977c                          1/1     Running   1          22h
omsagent-rs-85cd58d7f4-vtcg8            1/1     Running   1          22h
tunnelfront-f7d9d5fcc-kmxhd             1/1     Running   0          22h

$ kubectl port-forward kubernetes-dashboard-74d8c675bc-qgd7l 9090:9090 -n kube-system
```
Abrimos navegador en localhost:9090 y veremos el Dashboard de Kubernetes
Doc oficial:https://docs.microsoft.com/es-es/azure/aks/kubernetes-dashboard


