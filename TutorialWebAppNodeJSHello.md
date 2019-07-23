**TUTORIAL NODE-AZURE - PRACTICA PARA MEETUPS**

Podemos usar [az login] o [https://shell.azure.com]
---------------------------------------------------

https://docs.microsoft.com/es-es/azure/app-service/app-service-web-get-started-nodejs

### Configurar entorno Azure para Node
$ az group create --name WebAppsNode-recursos --location northeurope

$ az appservice plan create --name WebAppsNodePlan --resource-group WebAppsNode-recursos --sku FREE

$ az webapp create --resource-group WebAppsNode-recursos --plan WebAppsNodePlan --name appwebnode01

$ az webapp config appsettings set --resource-group WebAppsNode-recursos --name appwebnode01 --settings WEBSITE_NODE_DEFAULT_VERSION=10.14.1

Visitar la web creada (sin aplicacion): https://appwebnode01.azurewebsites.net

### Deploy del proyecto HelloNode con Kudu y ZipDeployUI
- Comprimir proyecto desde los archicos en ZIP
- visitar Kudu: https://appwebnode01.scm.azurewebsites.net/ZipDeployUI
- Arrastrar el archivo ZIP sobre el entorno Kudu y se desplegará automaticamente.
- Visitar la web creada: https://appwebnode01.azurewebsites.net
- Veremos ahora el mensaje "Hello World" que esta en index.js
- Podemos editar index.js directamente en Kudu y cambiar el mensaje
