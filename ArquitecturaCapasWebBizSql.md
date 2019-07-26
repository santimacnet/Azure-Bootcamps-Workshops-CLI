**PRACTICA AZURE CON ARQUITECTURA WEB EN CAPAS Y SQL SERVER**
-------------------------------------------------------------- 

- Consola: [az login] o [https://shell.azure.com]
- Ejemplo: https://raw.githubusercontent.com/MicrosoftDocs/mslearn-n-tier-architecture/master
- Documentacion: https://docs.microsoft.com/es-es/learn/modules/n-tier-architecture/3-deploy-n-tier-architecture

### Crear entorno Azure con template de recursos

La creacion de recursos tardará varios minutos y el password se generá de forma aleatoria en este ejemplo.

```
$ az group create --name ArquitecturaWebCapas-recursos --location northeurope

$ az group deployment create \
    --resource-group ArquitecturaWebCapas-recursos \
    --template-uri  https://raw.githubusercontent.com/MicrosoftDocs/mslearn-n-tier-architecture/master/Deployment/azuredeploy.json \
    --parameters password="$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c 32)"

$ az group deployment show \
    --name azuredeploy \
    --resource-group ArquitecturaWebCapas-recursos \
    --query properties.outputs.webSiteUrl \
    --output table 
```

La plantilla crea una red virtual con 2 subredes, 2 máquinas virtuales (VM), una base de datos de Azure SQL y los recursos necesarios para admitir estos recursos, como discos, NIC y redes virtuales con un límite de seguridad entre ellos, lo entenderemos mejor con este diagrama de la pagina de Microsoft

![](https://docs.microsoft.com/es-es/learn/modules/n-tier-architecture/media/3-n-tier-deployment.svg)

También se hace deploy de las aplicaciones en cada nivel, ya sea para VM o SQL Server. 
Una vez finalizado ir al sitio web y poner valores validos: pizza, tacos, sushi.

### Revisar la infraestructura creada en Azure

En la creacion de la infraestrucutra, se han aplicado etiquetas (tags) a los recursos como parte de la implementación para reflejar el nivel que admite el recurso (tier:presentation, tier:application, tier:data). 

La creacion ya esta lista y podemos verla desde el portal de Azure o bien desde comandos AZ consultando los recursos por etiquetas.

```
$ az resource list --tag tier=presentation --output table

$ az resource list --tag tier=application --output table

$ az resource list --tag tier=data --output table
```

Con estos comandos se muestran  todos los recursos para cada una de las capas del proyecto.








