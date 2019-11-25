**PRACTICA AZURE KUBERNETES AKS BASADO EN BOOTCAMP GOOGLE**
------------------------------------------------------------------

Tutorial didáctico con fines de demostración y formacion para eventos y meetups sobre AKS y los comandos basicos de Kubectl.

Para este tutorial es necesario tener los siguientes requerimientos:
- AKS creado en Azure con permisos administrador
- saber utilizar la shell.azure.com y herramientas
- saber utilizar Kubectl para trabajar con Kubernetes
- saber utilizar Dashboard de Kubernetes para consultar Wordloads


### Configurar acceso con Suscripcion Azure para abrir Dashboard Kubernetes

Abrir una shell de Azure para conectarnos al Dashboard Kubernetes
```
$ az account list
$ az account set --subscription ****-****-***-***
$ az aks browse --name <nombre-aks> --resource-group <nombre-rg> 
```

Abrir otra shell de Azure para ejecutar comandos con Kubectl
```
$  kubectl get nodes
```

### Paso1: Implementando aplicación Bootcamp en AKS

...

### Paso2:  Explorando aplicación creada y su estado

...


### Paso3: Exponer Services para acceder aplicación creada

...


### Paso4: Escalar aplicación creada desde 1-pod hasta 5-pods

...

### Paso5: Rolling updates pasar de aplicacion-v1 a aplicacion-v2

...

### Conclusiones
Hemos visto un caso practico de como realizar en 5 pasos un deployment en Kubernetes (AKS).


