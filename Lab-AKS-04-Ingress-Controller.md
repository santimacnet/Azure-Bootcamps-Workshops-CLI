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

### Paso3: Creando RUTAS DE INGRESS EN AKS

Desplegar las rutas desde DASHBOARD o FICHERO (The ingress resource configures the rules that route traffic to one of the two applications):

```
$ kubectl apply -f aks-helloworld-ingress.yml -n ingress-basic
ingress.networking.k8s.io/hello-world-ingress created
ingress.networking.k8s.io/hello-world-ingress-static created
```

Codigo del fichero:
```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /hello-world-one(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-two
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /(.*)
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress-static
  namespace: ingress-basic
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/rewrite-target: /static/$2
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /static(/|$)(.*)

```

Comprobacion de creacion de ingress y las rutas:
```
$ kubectl get ingress --namespace ingress-basic
NAME                         HOSTS   ADDRESS         PORTS   AGE
hello-world-ingress          *       20.54.143.197   80      2m11s
hello-world-ingress-static   *       20.54.143.197   80      2m11s


$ kubectl describe ingress --namespace ingress-basic
Name:             hello-world-ingress
Namespace:        ingress-basic
Address:          20.54.143.197
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /hello-world-one(/|$)(.*)   aks-helloworld-one:80 (10.240.0.13:80)
              /hello-world-two(/|$)(.*)   aks-helloworld-two:80 (10.240.0.29:80)
              /(.*)                       aks-helloworld-one:80 (10.240.0.13:80)
Annotations:  kubernetes.io/ingress.class: nginx
              nginx.ingress.kubernetes.io/rewrite-target: /$1
              nginx.ingress.kubernetes.io/ssl-redirect: false
              nginx.ingress.kubernetes.io/use-regex: true
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  3m26s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress
  Normal  CREATE  3m26s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress
  Normal  UPDATE  2m57s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress
  Normal  UPDATE  2m57s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress


Name:             hello-world-ingress-static
Namespace:        ingress-basic
Address:          20.54.143.197
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *
              /static(/|$)(.*)   aks-helloworld-one:80 (10.240.0.13:80)
Annotations:  kubernetes.io/ingress.class: nginx
              nginx.ingress.kubernetes.io/rewrite-target: /static/$2
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age    From                      Message
  ----    ------  ----   ----                      -------
  Normal  CREATE  3m26s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress-static
  Normal  CREATE  3m26s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress-static
  Normal  UPDATE  2m57s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress-static
  Normal  UPDATE  2m57s  nginx-ingress-controller  Ingress ingress-basic/hello-world-ingress-static
```

Si abrimos un navegador y navegamos a las rutas veremos las aplicaciones funcionando.


