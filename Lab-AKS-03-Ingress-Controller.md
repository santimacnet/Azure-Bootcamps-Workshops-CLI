**PRACTICA AKS KUBERNETES CON NGINX INGRESS CONTROLLER**
------------------------------------------------------------------

Tutorial para eventos, meetups y formación sobre AKS con Ingress donde veremos:

- Paso1: Entendiendo que es un Controlador Ingress 
- Paso2: Instalando NGNIX Ingress Controler con Deployment 
- Paso3: Instalando NGNIX Ingress Controler con Helm-3
- Paso4: Instalando Recursos Ingress para enrutar trafico
- Paso5: Verificando instalacion y configuracion NGINX

Lectura previa para fundamentos:
- Conceptos: https://kubernetes.io/docs/concepts/services-networking/ingress
- Conceptos: https://kubernetes.github.io/ingress-nginx/troubleshooting

Mejoras para trabajar futuras versiones:
- Utilizar un servicio y pods en otro namespace en lugar de "default"
- Llegar a otros servvicios con "Cross Namespace Ingress Network", se necesita "kube-dns"
- Ejemplo con externalName: servicio-svc.namespace.svc.cluster.local
- Configurar certificados para https con certmanager


### Paso1: Entendiendo que es un Controlador Ingress 

De acuerdo con la definición de kubernetes.io: "Un Ingress es una colección de reglas que permiten que las conexiones entrantes lleguen a los Servicios del clúster".

Aqui comparto una serie de definiciones para entender el concepto:

- Un controlador Ingress permite exponer servicios ClusterIP al mundo exterior desde 1 solo punto de entrada
- Un controlador Ingress es completamente independiente de los servicios de Kubernetes
- Es la solucion recomendada para no tener que usar servicios NodePort o LoadBalancer en kubernetes
- Con LoadBalancer/NodePort exponemos el servicio al exterior acoplandonos con el tipo de servicio

Con los Servicios, las reglas de enrutamiento están asociadas con un servicio determinado y existen mientras exista el servicio 
Si de alguna manera podemos desacoplar las reglas de enrutamiento de la aplicación y centralizar la administración de reglas, podemos actualizar nuestra aplicación sin preocuparnos por su acceso externo.
Esto lo podemos hacer mediante el recurso Ingress.

![Diagrama Ingress](https://github.com/santimacnet/Azure-Bootcamps-Workshops-CLI/blob/master/images/lab-ingress-Controller-NGINX-diagrama.png)

Existen muchos tipos de Ingress Controller, en esta practica usamos NGINX donde existen 2 opciones, ambas son de código abierto y están  en GitHub:

- Uno es mantenido por la community Kubernetes: https://github.com/kubernetes/ingress-nginx
- Otro es mantenido por le empresa NGINX: https://github.com/nginxinc/kubernetes-ingress
- Listado completo: https://kubernetes.io/docs/concepts/services-networking/ingress-controllers

Ref: https://www.nginx.com/blog/wait-which-nginx-ingress-controller-kubernetes-am-i-using


Para permitir que la conexión entrante llegue a los Servicios del clúster, Ingress configura un balanceador HTTP/HTTPS para los servicios y proporciona lo siguiente:

- Balanceo de carga
- Reglas personalizadas
- Enrutamiento basado en Locations (Fanout)
- Enrutamiento basado en Virtual Host (Virtual Server)
- Seguridad TLS (Terminacion de TLS/SSL en capa de transporte)

Con un Ingress, los usuarios **no se conectan directamente a un Servicio**, llegan al controlador de Ingress y a partir de ahí la solicitud se reenvía al Servicio deseado segun las reglas definidas.

En el siguiente ejemplo, las solicitudes de los usuarios a **blue.example.com y green.example.com** llegan al Ingress y a partir de ahí, se enviarían a webserver-blue-svc y webserver-green-svc respectivamente. 

Ejemplo de una regla basado en virtual host: 
```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: virtual-host-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - backend:
          serviceName: nginx-main-svc
          servicePort: 80
  - host: blue.example.com
    http:
      paths:
      - backend:
          serviceName: webserver-blue-svc
          servicePort: 80
  - host: green.example.com
    http:
      paths:
      - backend:
          serviceName: webserver-green-svc
          servicePort: 80
```

También podemos usar reglas de Fanout, cuando las solicitudes a **example.com/blue y example.com/green** se envíen a webserver-blue-svc y webserver-green-svc, respectivamente:

```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: fanout-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-main-svc
          servicePort: 80
      - path: /blue
        backend:
          serviceName: webserver-blue-svc
          servicePort: 80
      - path: /green
        backend:
          serviceName: webserver-green-svc
          servicePort: 80
```

Diagrama del ejemplo:

![Diagrama Ingress](https://github.com/santimacnet/Azure-Bootcamps-Workshops-CLI/blob/master/images/lab-ingress-url-routing-image.jpg)

Un ejemplo en video muy didactico lo podemos ver aqui:

Ref: https://github.com/justmeandopensource/kubernetes/tree/master/yamls/ingress-demo


### Paso2: Instalando NGNIX Ingress Controler con Deployment

La instalación oficial se puede realizar con los siguientes pasos para un deploy normal, pero actualmente esta recomendado usar Helm3 como vemos en el paso3.

```
# Ejecutar el siguiente deployment
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud/deploy.yaml

namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created

# Verificar la instalacion
$ kubectl get deploy -n ingress-nginx
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           11m

$ kubectl get svc -n ingress-nginx
... obtener la IP del servicio LoadBalancer
```

Ref: https://kubernetes.github.io/ingress-nginx/deploy/#azure


### Paso3: Instalando NGNIX Ingress Controler con Helm-3 

La instalación oficial nos indica utilizar HELM para instalar NGNIX:

```
$ helm repo update
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
NAME: my-release
LAST DEPLOYED: Jun 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-release-ingress-nginx-controller'
...

$ helm install my-release ingress-nginx/ingress-nginx
$ helm list

$ kubectl get all --namespace default

$ kubectl get svc --namespace default
  ... obtener la IP del servicio LoadBalancer
```
Ref: https://kubernetes.github.io/ingress-nginx/deploy/#using-helm

### Paso4: Instalando Recursos Ingress para enrutar trafico

Vamos a desplegar recursos para ver en accion NGINX con las propias demos que ofrece el fabricante.

Creamos el archivo para el deployment de la practica CAFE o TE: demo-deployment.yml

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: coffee
spec:
  replicas: 2
  selector:
    matchLabels:
      app: coffee
  template:
    metadata:
      labels:
        app: coffee
    spec:
      containers:
      - name: coffee
        image: nginxdemos/nginx-hello:plain-text
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: coffee-svc
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: coffee
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tea
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tea 
  template:
    metadata:
      labels:
        app: tea 
    spec:
      containers:
      - name: tea 
        image: nginxdemos/nginx-hello:plain-text
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tea-svc
  labels:
spec:
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: tea
```

Creamos ahora el archivo para recursos ingress: demo-ingress.yml y rellenar la IP publica del servicio de NGINX INGRESS que se ha creado en el balanceador de AKS para ponerla en ip-ip-ip-ip
```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cafe-ingress
   annotations:
     kubernetes.io/ingress.class: nginx 
spec:
  rules:
  - host: labdemocafe.ip-ip-ip-ip.nip.io
    http:
      paths:
      - path: /tea
        backend:
          serviceName: tea-svc
          servicePort: 80
      - path: /coffee
        backend:
          serviceName: coffee-svc
          servicePort: 80
```

Aplicamos los archivos desde la consola para ver el resultado:
```
$ kubectl apply -f demo-deployment.yml

$ kubectl apply -f demo-ingress.yml
```

Abrimos un navegador y vistamos las rutas definidas para ver los resultados:
```
http://labdemocafe.40-127-237-79.nip.io/tea
http://labdemocafe.40-127-237-79.nip.io/coffee
```

### Paso5: Verificando instalacion y configuracion NGINX
```
# Check Ingress Resource Events
$ kubectl get deploy -n <namespace-of-ingress>
$ kubectl get ing -n <namespace-of-ingress>
$ kubectl describe ing <ingress-resource-name> -n <namespace-of-ingress>

# Check Logs y configuracion
$ kubectl get pods -n <namespace-of-ingress-controller>
$ kubectl logs -n <namespace> nginx-ingress-controller-67956bf89d-fv58j
$ kubectl exec -it -n <namespace-of-ingress-controller> nginx-ingress-controller-67956bf89d-fv58j cat /etc/nginx/nginx.conf
```

Enjoy the Lab!!
