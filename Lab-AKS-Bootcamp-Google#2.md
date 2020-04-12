**PRACTICA AKS KUBERNETES BASADO EN BOOTCAMP GOOGLE**
------------------------------------------------------------------

Tutorial didáctico con fines de demostración y formacion para eventos y meetups sobre AKS y los comandos basicos de Kubectl.

Para este tutorial es necesario tener los siguientes requerimientos:
- AKS creado en Azure con permisos administrador
- saber utilizar shell.azure.com y Azure CLI
- saber utilizar Kubectl para trabajar con Kubernetes

### Configurar Suscripcion Azure y acceso kubectl para Kubernetes

Abrir una shell de Azure para consultar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***

$ az aks get-credentials --resource-group <nombre-rg> --name <nombre-aks> --admin
```

### Practicando NAMESPACES

Desde la shell de Azure creamos los siguientes namespaces.

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
```

