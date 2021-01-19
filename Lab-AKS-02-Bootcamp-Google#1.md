**PRACTICA AKS KUBERNETES BASADO EN BOOTCAMP GOOGLE**
------------------------------------------------------------------

Tutorial para eventos, meetups y formación sobre AKS donde veremos:

    - Paso0: Implementando aplicación Bootcamp en AKS
    - Paso1: Explorando recursos y objetos de API-RESOURCES
    - Paso2: Explorando aplicación para Troubleshooting y Debugging 
    - Paso3: Exponer Services para acceder aplicación creada
    - Paso4: Escalar aplicación creada desde 1-pod hasta 5-pods
    - Paso5: Rolling updates pasar de aplicacion-v1 a aplicacion-v2

Chuleta: https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf

Requerimientos:

    - Azure, Kubectl, shell.azure.com y Kubernetes fundamentos basicos
    - AKS ya creado en Azure con permisos administrador
    - Suscripcion de Azure con permisos Admin para Azure Active Directory

### Paso0: Implementando aplicación Bootcamp en AKS
```
$ kubectl create deployment k8-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments  // ver deployment ej: k8-bootcamp
$ kubectl get rs           // ver replicaset ej: k8-bootcamp-6969f6d786
$ kubectl get pods         // ver pod creado ej: k8-bootcamp-6969f6d786-z842s 
$ kubectl get services     // no lo vemos porque no ha sido creado
$ kubectl get all          // ver todo los objetos con un solo comando
```

### Paso1: Explorando recursos y objetos de API-RESOURCES

No Todos los Objetos están en un Espacio de nombres 
La mayoría de los recursos de Kubernetes (ej. pods, services, replication controllers, y otros) están en algunos espacios de nombres. Sin embargo, los recursos que representan a los propios NAMESPACES, NODOS, VOLUMENES no están en NAMESPACES. 

```
# Para conocer recursos de Kubernetes 
$ kubectl api-resources 

# In a namespace
$ kubectl api-resources --namespaced=true

# Not in a namespace
$ kubectl api-resources --namespaced=false
```

### Paso2: Explorando aplicación para Troubleshooting y Debugging 

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

