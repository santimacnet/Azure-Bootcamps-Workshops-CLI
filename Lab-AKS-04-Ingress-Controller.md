**PRACTICA AKS - NGINX INGRESS BASIC **
------------------------------------------------------------------

Tutorial para eventos, meetups y formación sobre AKS con Ingress basado en la documentacion oficial de Microsoft.

Ref: https://docs.microsoft.com/en-us/azure/aks/ingress-basic


### Paso1: Desplegando NGINX con HELM 
```
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
   
# Ver el servicio creado y la  EXTERNAL-IP publica obtenida para conectar
kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller
    
NAME                                     TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                     
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.189.9   20.54.143.197   80:30006/TCP,443:32409/TCP    
```


### Paso2: Desplegando aplicaciones AKS-ONE y AKS-TWO como ClusterIP

Desplegar aplicacion AKS-ONE desde DASHBOARD o FICHERO: 
$ kubectl apply -f aks-helloworld-one.yml -n ingress-basic

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
        image: neilpeterson/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to AKS - ONE"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-one  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
```


Desplegar aplicacion AKS-TWO desde DASHBOARD o FICHERO: 
$ kubectl apply -f aks-helloworld-two.yml -n ingress-basic

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-two  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-two
  template:
    metadata:
      labels:
        app: aks-helloworld-two
    spec:
      containers:
      - name: aks-helloworld-two
        image: neilpeterson/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to AKS - TWO"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-two  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-two
```

Comprobacion de creacion de los Pods:

```
$ kubectl get pods --namespace ingress-basic
NAME                                                      READY   STATUS              RESTARTS   AGE
aks-helloworld-one-78866647fc-jnxx7                       1/1     Running             0          3m29s
aks-helloworld-two-769548854c-6r7rh                       1/1     Running             0          2m10s
```

### Paso3: ...




