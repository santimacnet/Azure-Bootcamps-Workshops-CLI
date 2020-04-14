**PRACTICA COMANDOS KUBECTL GENERALES **
-------------------------------------------------------

Abrir una shell en Azure directamente para ejecutar estos comandos, necesitamos un AKS funcionando.

### Comandos para PODS

```
$ kubectl get pods
$ kubectl get pods -o wide
$ kubectl get pods --all-namespaces
$ kubectl get pods --all-namespaces -o wide
$ kubectl get pods -n kube-system
$ kubectl get pods --show-labels
```

Ref: https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images


### Comandos para SERVICES

```
$ kubectl get svc
$ kubectl get svc -o wide
```


### Comandos para NODOS

```
$ kubectl get nodes
$ kubectl get nodes -o wide
```

### Deploy HelloAPP en Kubernetes

```
# Crear dos deployment directamente con "run" y nombre del pod "web" y "web2"
$ kubectl run web --image=gcr.io/google-samples/hello-app:1.0 --port=8080
$ kubectl run web2 --image=gcr.io/google-samples/hello-app:2.0 --port=8080

# Crear el service directamente con "expose" de tipo "NodePort"
$ kubectl expose deployment web --target-port=8080 --type=NodePort

# Validar que el service se ha creado correctamente como tipo "NodePort" (2 opciones)
$ kubectl get service web
$ kubectl get svc -o wide

```


