**PRACTICA AKS KUBERNETES CREAR CLUSTER NUEVO**
------------------------------------------------------------------

Tutorial para charlas, eventos, meetups y formación sobre AKS donde veremos:

   - Paso1: Creando un Service Principal para AKS
   - Paso2: Creando cluster AKS con Load Balancer Standard
   - Paso3: Creando registry ACR y Service Principal
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

Creamos Service Principal y anotamos JSON con AppID y Password, este SP se guarda en Azure AD Aplicaciones Registradas:
```
# configurar suscripcion correcta si fuera necesario
$ az account list
$ az account set --subscription ****-****-***-***

# configurar service principal para AKS para luego attachar con ACR
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterPruebasServicePrincipal
...guardar datos respuesta json con password...
```

### Paso2: Creando cluster AKS con Load Balancer Standard

Definims variables entorno para los recursos
```
RG_NAME=AKS-demolab-recursos
RG_LOCATION=westeurope
ACR_NAME=acrhubdemolab
AKS_NAME=aks-cluster-demolab
```

Creamos el grupo de recursos y AKS la creacion del cluster puede tardar entre 5 y 10.
La configuración de **nodos en el grupo de nodos predeterminado con el parámetro --node-count 2** sirve para los servicios esenciales del sistema en AKS en este grupo de nodos, y los nodos adicionales contribuyen a la confiabilidad de las operaciones del clúster.
```
$ az group create --name $RG_NAME --location $RG_LOCATION

# crear LB-Standard ya que LB-Básico no se admite cuando se usan varios grupos de nodos en node pools
$ az aks create --name $AKS_NAME \
    --resource-group $RG_NAME \
    --location $RG_LOCATION \
    --enable-addons monitoring \
    --node-count 2
    --node-vm-size Standard_b2ms \
    --vm-set-type VirtualMachineScaleSets \
    --generate-ssh-keys \
    --load-balancer-sku standard \
    --service-principal <appId> \
    --client-secret <password>
```    
    
Verificamos AKS con Service Principal y grupos de nodos creados. Actualmente se recomienda usar identidades administradas por Azure AD:
https://santimacnet.wordpress.com/2020/04/20/aks-creando-un-cluster-con-identidad-administrada-asignada-por-azure-ad
```
$ az aks show --name $AKS_NAME --resource-group $RG_NAME 
$ az aks show --name $AKS_NAME --resource-group $RG_NAME -o table    
$ az aks show --name $AKS_NAME --resource-group $RG_NAME --query servicePrincipalProfile.clientId -o tsv)

IMPORTANTE: los clústeres de AKS se crean con una entidad de servicio que tiene un período de expiración de un año.
Referencia: https://docs.microsoft.com/es-es/azure/aks/update-credentials
```


NODEPOOLS: Escalar manualmente los nodos de los nodepools, así como la capacidad de escalar a cero grupos de nodos basados en costosas máquinas virtuales, es una buena estrategia para optimizar los costos en AKS cuando se administran directamente las demandas de carga de trabajo. 
```
# Listar nodos de AKS pero no vemos los nodepools
$ kubectl get nodes

# Listar grupos de nodos del cluster y ver tipo (system o user)
$ az aks nodepool list --resource-group $RG_NAME --cluster-name $AKS_NAME -o table
```
Ref: https://docs.microsoft.com/es-es/azure/aks/use-multiple-node-pools


### Paso3: Creando registry ACR y Service Principal

Creamos el grupo de recursos y AKS con la opcion de autoescalado, la creacion del cluster puede tardar entre 10 y 15 minutos.
```
# creamos el ACR para publicar imagenes
$ az acr create --name $ACR_NAME --resource-group $RG_NAME --location $RG_LOCATION --sku Basic

# atachamos ACR-ID para deployar imagenes entre ACR y AKS
$ ACR_ID=$(az acr show --name $ACR_NAME --resource-group $RG_NAME --query id -o tsv)
$ az aks update --name $AKS_NAME --resource-group $RG_NAME --attach-acr $ACR_ID
  - AAD role propagation - Running.... (tarda unos minutos)
  
# creamos Service Principal para utilizar con Azure DevOps Pipelines
$ AZDEVOPS_ACR_SP=AzureDevOpsPipelineACRServicePrincipal
$ AZDEVOPS_ACR_SP_PASSWORD=$(az ad sp create-for-rbac --name $AZDEVOPS_ACR_SP --scopes $ACR_ID --role acrpush --query password -o tsv)
$ echo $AZDEVOPS_ACR_SP_PASSWORD   # Guardarlo en sitio seguro para usarlo en los pipelines

# creamos Service Principal para utilizar con Developers que luego podemos revocar
OPCIONAL: pensar si realmente es necesario al ACR directamente o todo por Azure DevOps
    
# importamos imagen NGNIX para pruebas AKS
$ az acr import  -n $ACR_NAME -g $RG_NAME --source docker.io/library/nginx:latest --image nginx:v1
$ az acr repository list --name $ACR_NAME --output table
```

Nota: tambien podemos crear primero ACR y usar el parametro --attach-acr en la creacion de AKS

### Paso4: Configurar Kubectl y Credenciales acceso AKS

Configurar la herramienta de Kubectl para trabajar con AKS
```
$ az aks install-cli (para instalar kubectl si no lo tenemos en local)

$ az aks get-credentials --name $AKS_NAME --resource-group $RG_NAME --admin
  Merged ........ as current context in /home/santimacnet/.kube/config

#  ejecutar comandos Kubectl para verificar aks funcionando
$ kubectl version
$ kubectl config get-contexts (para ver todos los contextos AKS)
$ kubectl config current-context (para ver contexto correcto AKS)

$ kubectl cluster-info
$ kubectl get nodes

# realizar deploy nginx para pruebas desde ACR
$ kubectl run demo-nginx --image=acrhubdemolab.azurecr.io/nginx:v1 --port=80
$ kubectl expose deployment demo-nginx --port=80 --type=LoadBalancer

$ kubectl get all (ver IP publica para nginx)
```

### Paso5: Configurar acceso Dashboard AKS

```
# crear role para permisos con rbac
$ kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

# abrir dashboard como proxy
$ az aks browse --name $AKS_NAME --resource-group $RG_NAME

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


