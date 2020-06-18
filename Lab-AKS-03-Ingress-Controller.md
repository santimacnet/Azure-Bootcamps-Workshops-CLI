**PRACTICA AKS KUBERNETES CON INGRESS CONTROLLER**
------------------------------------------------------------------

Tutorial para eventos, meetups y formación sobre AKS donde veremos:

- Paso1: Entendiendo que es un Controlador Ingress 
- Paso2: Instalando NGNIX Ingress con Deployment en AKS
- Paso3: Instalando NGNIX Ingress con Helm-3 en AKS
- Paso4: 
- Paso5: 

Chuleta: https://kubernetes.github.io/ingress-nginx


### Paso1: Entendiendo que es un Controlador Ingress 

Aqui comparto una serie de definiciones para entender el concepto:

- Un controlador Ingress es completamente independiente de los servicios de Kubernetes
- Un controlador Ingress permite exponer servicios ClusterIP al mundo exterior desde 1 solo punto de entrada
- Es la solucion recomendada para no tener que usar servicios NodePort o LoadBalancer en kubernetes
- Con LoadBalancer/NodePort exponemos el servicio al exterior acoplandonos con el tipo de servicio

De acuerdo con la definición de kubernetes.io: "Un Ingress es una colección de reglas que permiten que las conexiones entrantes lleguen a los Servicios del clúster".

Con los Servicios, las reglas de enrutamiento están asociadas con un Servicio determinado. Existen mientras exista el Servicio, y hay muchas reglas porque hay muchos Servicios en el clúster. 
Si de alguna manera podemos desacoplar las reglas de enrutamiento de la aplicación y centralizar la administración de reglas, podemos actualizar nuestra aplicación sin preocuparnos por su acceso externo.
Esto lo podemos hacer mediante el recurso Ingress.

Para permitir que la conexión entrante llegue a los Servicios del clúster, Ingress configura un balanceador HTTP/HTTPS para los servicios y proporciona lo siguiente:

- Balanceo de carga
- Reglas personalizadas
- Enrutamiento basado en Locations (Fanout)
- Enrutamiento basado en Virtual Host (Virtual Server)
- Seguridad TLS (Terminacion de TLS/SSL en capa de transporte)

Con Ingress, los usuarios **no se conectan directamente a un Servicio**, llegan al controlador de Ingress y a partir de ahí la solicitud se reenvía al Servicio deseado segun las reglas definidas.

En el siguiente ejemplo, las solicitudes de los usuarios a **blue.example.com y green.example.com** llegan al Ingress y a partir de ahí, se enviarían a webserver-blue-svc y webserver-green-svc respectivamente. 

Ejemplo de una regla basado en nombre: 
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

Con el recurso Ingress que acabamos de crear, ahora deberíamos poder acceder a los servicios webserver-blue-svc o webserver-green-svc utilizando las URL blue.example.com y green.example.com. 

Ref: https://github.com/justmeandopensource/kubernetes/tree/master/yamls/ingress-demo

Diagrama del ejemplo:

![Diagrama Ingress](https://github.com/santimacnet/Azure-Bootcamps-Workshops-CLI/blob/master/images/lab-ingress-url-routing-image.jpg)


### Paso2: Instalando NGNIX Ingress con Deployment en AKS

La instalación oficial nos indica utilizar los siguientes pasos:

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


### Paso2: Instalando NGNIX Ingress con Helm-3 en AKS

La instalación oficial nos indica utilizar HELM para instalar NGNIX:

```
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
```

Verificando instalacion y configuracion
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

Ref: https://kubernetes.github.io/ingress-nginx/deploy/#using-helm


