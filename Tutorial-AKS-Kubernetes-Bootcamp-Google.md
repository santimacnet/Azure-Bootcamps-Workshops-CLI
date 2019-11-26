**PRACTICA AKS KUBERNETES BASADO EN BOOTCAMP GOOGLE**
------------------------------------------------------------------

Tutorial didáctico con fines de demostración y formacion para eventos y meetups sobre AKS y los comandos basicos de Kubectl.

Para este tutorial es necesario tener los siguientes requerimientos:
- AKS creado en Azure con permisos administrador
- saber utilizar shell.azure.com y Azure CLI
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
$ kubectl cluster-info
$ kubectl get nodes
```

### Paso1: Implementando aplicación Bootcamp en AKS

```
$ kubectl create deployment k8-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments  // ver deployment ej: k8-bootcamp
$ kubectl get rs           // ver replicaset ej: k8-bootcamp-6969f6d786
$ kubectl get pods         // ver pod creado ej: k8-bootcamp-6969f6d786-z842s 
$ kubectl get services     // no lo vemos porque no ha sido creado
$ kubectl get all          // ver todo los objetos con un solo comando
```

### Paso2:  Explorando aplicación para Troubleshooting y Debugging 

```
$ kubectl logs <nombre-pod>            // ver logs del pod-contenedor
$ kubectl describe pods <nombre-pod>   // detalle del pod creado indicando "nombre-pod"
$ kubectl exec <nombre-pod> env        // ejecutar comando env  en pod/contenedor para ver todas las vars entorno definidas
$ kubectl exec -ti <nombre-pod> bash   // ejecutar comando bash en pod/contenedor para tener una shell interactiva
  # curl localhost:8080                // veremos: Hello Kubernetes bootcamp! | Running on: k8-bootcamp-*****-*** | v=1
  # exit
```

### Paso3: Exponer Services para acceder aplicación creada

```
$ kubectl get services    // no vemos el servicio porque no fue creado
$ kubectl expose deployment/k8-bootcamp --type="LoadBalancer" --port 8080   // exponer como Load Balancer en AKS no usar NodePort
$ kubectl get services    // ahora veremos el servicio expuesto como LoadBalancer (pending)

$ kubectl describe services/k8-bootcamp   // describir datos del service para obtener el Endpoint "ip:puerto"
$ curl http://EXTERNALIP:PUERTO           // veremos "Hello Kubernetes bootcamp! | Running on: k8-bootcamp-*****-*** | v=1

$ kubectl describe deployment/k8-bootcamp  // describir datos del label-selector creado por defecto
$ kubectl get pods -l app=k8-bootcamp      // obtener pods con filtro por label
```

### Paso4: Escalar aplicación creada desde 1-pod hasta 5-pods

```
$ kubectl get pods -o wide
$ kubectl scale deployments/k8-bootcamp --replicas=5
$ kubectl get deployments
$ kubectl get pods -o wide -l app=k8-bootcamp  
$ kubectl describe deployments/k8-bootcamp
$ kubectl describe services/k8-bootcamp  

$ curl http://EXTERNALIP:puerto // veremos como funciona el load-balancing mirando pod-name diferente

$ kubectl scale deployments/k8-bootcamp --replicas=3 // escalar hacia abajo el numero de pods
$ kubectl get pods -o wide -l app=k8-bootcamp  
```

### Paso5: Rolling updates pasar de aplicacion-v1 a aplicacion-v2

```
$ kubectl get deployments
$ kubectl describe deployment/k8-bootcamp  // obtener detalles deployment actual
$ kubectl get pods -o wide

$ kubectl set image deployments/k8-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
$ kubectl get pods -o wide               // veremos status y IPs de los pods 
$ kubectl describe services/k8-bootcamp  // veremos ExternalIP y Endpoints de nuevos pods creados
$ curl http://EXTERNALIP:puerto          // veremos la respuesta desde aplicacion-V2

$ kubectl set image deployments/k8-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v1
$ kubectl get pods -o wide               // veremos status y IPs de los pods 
$ kubectl describe services/k8-bootcamp  // veremos ExternalIP y Endpoints de nuevos pods creados
$ curl http://EXTERNALIP:puerto          // veremos la respuesta desde aplicacion-V1
```

### Conclusiones
Hemos visto un caso practico de como realizar en 5 pasos deployments en Kubernetes (AKS).

