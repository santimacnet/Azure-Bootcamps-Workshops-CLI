**PRACTICA AKS KUBERNETES PARA TECHDAYS DE MICROSOFT**
-------------------------------------------------------

Tutorial para eventos y meetups sobre AKS donde veremos:
- Paso1: Creando un Service Principal para AKS
- Paso2: Creando un cluster de AKS
- Paso3: Configurar Kubectl y Credenciales acceso AKS
- Paso4: Configurar Helm y repositorios de Charts 
- Paso5: Instalar WordPress con MariaDB con Helm

Chuleta: https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf

Requerimientos Tutorial:
- Azure y Kubernetes fundamentos basicos
- Conocimientos Azure CLI, shell.azure.com, Kubectl y Helm
- Suscripcion de Azure con permisos Admin para Azure Active Directory

Todo se realizará directamente desde la Shell de Azure mediante comandos Azure CLI.


### Configurar Suscripcion Azure para Workshop

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***
```

### Paso1: Creando un Service Principal para AKS

Contexto: Cuando creamos un AKS, Azure automaticamente genera un Service Principal de AAD, pero esto no siempre ocurre (arrggg!!) y entonces tenemos que generarlo nosotros previamente para indicarlo en el comando: az aks create... 

El Service Principal es necesario para interactuar con otros recursos de Azure, como un Load Balancer, registry ACR, Virtual Networks,etc y si AKS no dispone de uno en el momento de la creación fallará y nos dara un error: "AADSTS700016: Application with identifier xxxxx-xxxxx-xxxxx-xxxxxx was not found in the directory xxxxx-xxxxx-xxxxx-xxxxxx.

Explicacion en Microsoft Docs y Proyecto GitHub Service principals for AKSm que podemos consultar en este enlace: 
https://docs.microsoft.com/en-us/azure/aks/kubernetes-service-principal

Creamos el Service Principal y guardamos en notepad la respuesta JSON con AppID y Password, este SP se guarda en Azure Active Directory en la opcion de Aplicaciones Registradas:

```
$ az ad sp create-for-rbac --skip-assignment --name AKSClusterSantiPruebasServicePrincipal
...respuesta json...
```

### Paso2: Creando un cluster de AKS

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

### Paso3: Configurar Kubectl y Credenciales acceso AKS

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

### Paso4: Configurar Helm y repositorios de Charts 

Helm, es "The package manager for Kubernetes", como explican en su web oficial https://helm.sh y actualmente se encuentra en la version Helm 3.0: https://helm.sh/blog/helm-3-released

Helm, a parte de un package manager de Kubernetes, proporciona una forma fácil de instalar aplicaciones empaquetadas denominadas Charts y facilita el proceso del ciclo de vida de implementaciones de dichas aplicaciones en Kubernetes.

Para ello, configuramos varios repositorios comunes de Charts de Helm y asegurarmos que estan actualizados con los ultimos paquetes para trabajar desde nuestro entorno: 
```
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com
$ helm repo add azure-samples https://azure-samples.github.io/helm-charts
$ helm repo add azure-marketplace https://marketplace.azurecr.io/helm/v1/repo
"stable" has been added to your repositories

$ helm repo update
Update Complete. ⎈ Happy Helming!⎈

$ helm search repo stable
...listado de charts del repo stable para instalar en Kubernetes
```

Mas detalles: https://v3.helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository


### Paso5: Instalar WordPress y MariaDB con Helm

Para ver la potencia de Helm, instalaremos WordPress desde el repo de azure-marketplace/wordpress con un solo comando:

```
$ helm install aks-blogdemo azure-marketplace/wordpress --set global.imagePullSecrets={emptysecret}
NAME: aks-blogdemo
LAST DEPLOYED: Sun Mar 22 21:08:17 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w aks-blogdemo-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default aks-blogdemo-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default aks-blogdemo-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)

```

Ahora consultamos estado pods hasta que esten ready, suele tardar 2 minutos aprox.

```
$ kubectl get pods -w

NAME                                      READY   STATUS    RESTARTS   AGE
aks-blogdemo-mariadb-0                    1/1     Running   0          16m
aks-blogdemo-wordpress-778c69ff58-5gtlk   1/1     Running   1          16m
otros pods...
```
