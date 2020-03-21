**PRACTICA AKS KUBERNETES PARA TECHDAYS DE MICROSOFT**
-------------------------------------------------------

Tutorial para formacion eventos y meetups sobre AKS donde veremos:
- Como crear un clúster de AKS con autoescalado desde CLI
- Como implementar los contenedores desde Docker Hub
- Como configurar el monitoring
- Como escalar una aplicación en AKS

Para este tutorial es necesario tener los siguientes requerimientos:
- Azure fundamentos basicos
- Kubernetes fundamentos basicos
- Conocimientos shell.azure.com y Azure CLI
- Conocimientos Kubectl para trabajar con Kubernetes
- Suscripcion de Azure con permisos Admin para Azure Active Directory

Todo se realizará desde la Shell de Azure directamente.


### Configurar Suscripcion Azure para Workshop

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***
```

### Paso1: Creando un Service Principal para AKS

Contexto: Cuando creamos un AKS, Azure automaticamente genera un Service Principal de AAD, pero a veces esto no ocurre por algun motivo y entonces tenemos que generarlo nosotros previamente para indicarlo en el comando: az aks create... 

El Service Principal es necesario para interactuar con otros recursos de Azure, como un Load Balancer, registry ACR, Virtual Networks,etc y si AKS no dispone de uno en el momento de la creación fallará y nos dara un error: "AADSTS700016: Application with identifier xxxxx-xxxxx-xxxxx-xxxxxx was not found in the directory xxxxx-xxxxx-xxxxx-xxxxxx.

Microsoft Doc: Service principals for Azure Kubernetes Services (AKS): 
https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal

GitHub Notes: articles/aks/kubernetes-service-principal.md

Creamos el Service Principal y guardamos en notepad la respuesta JSON con AppID y Password, este SP se guarda en Azure Active Directory en la opcion de Aplicaciones Registradas:

```
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterSantiPruebasServicePrincipal
...respuesta json...
```

### Paso2: Creando un cluster Azure Kubernetes Service (AKS)

Creamos el grupo de recursos y AKS con la opcion de autoescalado, la creacion del cluster puede tardar entre 10 y 15 minutos.
```
$ az group create --name rg-santi-pruebas-cluster-aks --location westeurope

$ az aks create --resource-group rg-santi-pruebas-cluster-aks \
    --name aks-santi-cluster-pruebas \
    --location westeurope \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --vm-set-type VirtualMachineScaleSets \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --service-principal <appId> \
    --client-secret <password>

```

### Paso3: Desde la shell de Azure configurar Kubectl y Credenciales acceso AKS

Necesitamos configurar la herramienta de Kubectl para trabajar con AKS
```
$ az aks install-cli (para instalar kubectl si no lo tenemos en local)

$ kubectl version

$ az aks get-credentials --resource-group <nombre-rg> --name <nombre-aks-cluster> --admin
  Merged "aks-santi-cluster-pruebas-admin" as current context in /home/santimacnet/.kube/config

$ kubectl config current-context (para ver contexto correcto AKS)

$ kubectl cluster-info

$ kubectl get nodes
```

