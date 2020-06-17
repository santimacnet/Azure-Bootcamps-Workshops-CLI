**PRACTICA AKS KUBERNETES CON INGRESS CONTROLLER **
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

- Los recursos de ingreso son completamente independientes con los servicios.
- Un controlador Ingress permite exponer los servicios de dentro del clúster al mundo exterior desde un solo punto de entrada. 
- Es la solucion recomendada para no tener que usar servicios NodePort o LoadBalancer en kubernetes.
- Con NodePort o LoadBalancer exponemos el servicio al exterior especificando y acoplandonos con el tipo de servicio. 

De acuerdo con kubernetes.io: "Un Ingress es una colección de reglas que permiten que las conexiones entrantes lleguen a los Servicios del clúster".

Con los Servicios, las reglas de enrutamiento están asociadas con un Servicio determinado. Existen mientras exista el Servicio, y hay muchas reglas porque hay muchos Servicios en el clúster. 
Si de alguna manera podemos desacoplar las reglas de enrutamiento de la aplicación y centralizar la administración de reglas, podemos actualizar nuestra aplicación sin preocuparnos por su acceso externo.
Esto se puede hacer usando el recurso Ingress.

Para permitir que la conexión entrante llegue a los Servicios del clúster, Ingress configura un equilibrador de carga HTTP / HTTPS de Capa 7 para los Servicios y proporciona lo siguiente:

- Balanceo de carga
- Reglas personalizadas
- Enrutamiento basado en rutas (fanout)
- Enrutamiento basados en Hosting virtual (nombre)
- Seguridad TLS (Terminacion de TLS/SSL en capa de transporte)


Con Ingress, los usuarios no se conectan directamente a un Servicio. Los usuarios alcanzan el punto final de Ingress y, a partir de ahí, la solicitud se reenvía al Servicio deseado.

En el siguiente ejemplo, las solicitudes de los usuarios a blue.example.com y green.example.com irían al mismo punto final de Ingress y, a partir de ahí, se enviarían a webserver-blue-svc y webserver-green-svc respectivamente. 

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

También podemos usar reglas de Fanout, cuando las solicitudes a example.com/blue y example.com/green se envíen a webserver-blue-svc y webserver-green-svc, respectivamente:

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



