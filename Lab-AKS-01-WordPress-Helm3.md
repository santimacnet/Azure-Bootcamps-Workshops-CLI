**PRACTICA AKS KUBERNETES PARA TECHDAYS DE MICROSOFT**
-------------------------------------------------------

Tutorial para eventos y meetups sobre AKS donde veremos:

- Paso1: Configurar Helm3 y repositorios de Charts 
- Paso2: Instalar WordPress y MariaDB con Helm
- Paso3: Borrando WordPress y MariaDB con Helm
- Paso4: Probando Helm con otras aplicaciones del Hub
- Bonus: Otros comandos interesantes de Helm

Requerimientos Tutorial:

- AKS ya creado en Azure con permisos administrador
- Azure, suscripcion, Kubernetes, Kubectl y Helm3 fundamentos basicos
- Chuleta: https://linuxacademy.com/site-content/uploads/2019/04/Kubernetes-Cheat-Sheet_07182019.pdf

### Paso1: Configurar Helm3 y repositorios de Charts 

Helm, es "The package manager for Kubernetes", como explican en su web oficial https://helm.sh y actualmente se encuentra en la version Helm 3.0: https://helm.sh/blog/helm-3-released

Helm, a parte de un package manager de Kubernetes, proporciona una forma fácil de instalar aplicaciones empaquetadas denominadas Charts y facilita el proceso del ciclo de vida de implementaciones de dichas aplicaciones en Kubernetes.

Para ello, configuramos varios repositorios comunes de Charts de Helm y asegurarmos que estan actualizados con los ultimos paquetes para trabajar desde nuestro entorno: 
```
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com
$ helm repo add azure-samples https://azure-samples.github.io/helm-charts
$ helm repo add azure-marketplace https://marketplace.azurecr.io/helm/v1/repo
".........." has been added to your repositories

$ helm repo update
Update Complete. ⎈ Happy Helming!⎈

$ helm search repo stable
...listado de charts del repo stable para instalar en Kubernetes
```

Mas detalles: https://v3.helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository


### Paso2: Instalar WordPress y MariaDB con Helm

Para ver la potencia de Helm, instalaremos WordPress **con un solo comando** desde 2 repositorios Helm.

Ejecutamos comando de instalación y obtendremos las indicaciones detalladas para acceder a nuestro Wordpress.

Caso1: Instalamos Wordpress desde stable:
```
# creamos con param --generate-name nos pone automaticamente NAME: wordpress-1591957704
$ helm install stable/wordpress --generate-name

# leer las NOTES de instalacion para configurarlo 
WARNING: This chart is deprecated
NAME: wordpress-1591957704
LAST DEPLOYED: Fri Jun 12 10:28:26 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES: ...

** Please be patient while the chart is being deployed **

To access your WordPress site from outside the cluster follow the steps below:
.... detalles de los pasos de acceso, login y configuracion a Wordpress

```
Ref Helm Pelao: https://www.youtube.com/watch?v=CPjfb-I_BKU

Caso2: Instalamos Wordpress desde Azure Marketplace:
```
# creamos con nombre: aks-blogdemo y veremos las notas con NAME: aks-blogdemo
$ helm install aks-blogdemo azure-marketplace/wordpress --set global.imagePullSecrets={emptysecret}

# leer las NOTES de instalacion para configurarlo 
NAME: aks-blogdemo
LAST DEPLOYED: Sun Mar 22 21:08:17 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

To access your WordPress site from outside the cluster follow the steps below:
.... detalles de los pasos de acceso, login y configuracion a Wordpress

```

Ahora inspeccionamos los despliegues de nuestro WordPress y veremos que son distintos:
```
$ helm list
NAME                    NAMESPACE       REVISION        UPDATE         STATUS     CHART                APP VERSION
aks-blogdemo            default         1               2020-06-12 UTC deployed   wordpress-9.3.10     5.4.1
wordpress-1591957704    default         1               2020-06-12 UTC deployed   wordpress-9.0.3      5.3.2

#manifiesto del chart que hemos instalado
$ helm get manifest wordpress-1591957704
... yaml del deployment, service, etc.

#inspeccionar el chart que hemos instalado
$ helm inspect values azure-marketplace/wordpress
... explicaciones detalladas del creador del chart con todo lo que podemos cambiar.

$ kubectl get pods -w
NAME                                      READY   STATUS    RESTARTS   AGE
aks-blogdemo-mariadb-0                    0/1     Pending   0          16m
aks-blogdemo-wordpress-68bcc9676b-dk9xq   0/1     Pending   0          16m
wordpress-1591957704-766456c86f-dlvb4     1/1     Running   0          16m
wordpress-1591957704-mariadb-0            1/1     Running   0          16m
otros pods...
```

Ahora consultamos el estatus del servicio para obtener la IP publica de nuestro WordPress:

```
# comando para ver todos los services
$ kubectl get service -w

# comando para solo ver wordpress
$ kubectl get svc --namespace default -w aks-blogdemo-wordpress'

NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
aks-blogdemo-mariadb     ClusterIP      10.0.69.22     <none>          3306/TCP                     5m
aks-blogdemo-wordpress   LoadBalancer   10.0.254.234   51.xxx.xx.xxx   80:32487/TCP,443:31012/TCP   5m
otros services...
```

Una vez tenemos la EXTERNAL-IP, abrimos un navegador y veremos Wordpress ya funcionando, con el mensaje Welcome to WordPress!!

### Paso3: Borrando WordPress y MariaDB con Helm

En este punto podemos consultar el trabajo y eliminar WordPress si es necesario con los comandos siguientes:

```
# Desinstalando Wordpress completamente
$ helm uninstall aks-blogdemo
release "aks-blogdemo" uninstalled

$ helm uninstall wordpress-1591957704
release "wordpress-1591957704" uninstalled

$ helm list
...comprobar que no aparecen
```

### Paso4: Probando Helm con otras aplicaciones del Hub

Visto lo facil que es deplegar Aplicaciones en AKS con Helm podemos hacer otras pruebas como:

```
$ kubectl create namespace apache
$ helm install bitnami/apache --version 7.3.17 --namespace apache --generate-name 
... leer notas de la configuracion

$ kubectl create namespace joomla
$ helm install bitnami/joomla --namespace joomla --generate-name
... leer notas de la configuracion

$ kubectl create namespace drupal
$ helm install bitnami/drupal --namespace drupal --generate-name
... leer notas de la configuracion

$ kubectl create namespace sonarqube
$ helm install stable/sonarqube --namespace sonarqube --generate-name
... leer notas de la configuracion - DEPRECATED

#obtener lista de charts que hemos instalado
$ helm list -n apache
  ... listado
  
$ helm list --all-namespaces
  ... listado
  
#manifiesto del chart que hemos instalado
$ helm get manifest nombre-del-chart
```

### Bonus: Otros comandos interesantes de Helm

Tambien podemos crear nuestros propios Chart de forma muy sencilla::

```
# crear una plantilla base para nuestros propios chart
$ helm create prueba-appweb

# comprobar como queda el manifiesto generado
$ helm template .

# instalar directamente el chart desde el codigo-fuente
$ helm install .
```


Por último, no olvidar borrar los recursos creados y el cluster de AKS de pruebas para no incurrir en gastos innecesarios una vez finalizado el laboratorio.
...

Happy Lab!!

