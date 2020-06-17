**PRACTICA AKS KUBERNETES CON INGRESS CONTROLLER**
------------------------------------------------------------------

Tutorial para eventos, meetups y formación sobre AKS donde veremos:

- Paso1: Entendiendo que es un Controlador Ingress 
- Paso2: 
- Paso3: 
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
- Enrutamiento basado en rutas Fanout
- Enrutamiento basado en Virtual Hosting Name
- Seguridad TLS (Terminacion de TLS/SSL en capa de transporte)


Con Ingress, los usuarios **no se conectan directamente a un Servicio**, llegan al controlador de Ingress y a partir de ahí la solicitud se reenvía al Servicio deseado segun las reglas definidas.

En el siguiente ejemplo, las solicitudes de los usuarios a **blue.example.com y green.example.com** llegan al Ingress y a partir de ahí, se enviarían a webserver-blue-svc y webserver-green-svc respectivamente. 

Ejemplo de una regla basado en nombre: 
```yml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: virtual-host-ingress
  namespace: default
spec:
  rules:
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
  name: fan-out-ingress
  namespace: default
spec:
  rules:
  - host: example.com
    http:
      paths:
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

