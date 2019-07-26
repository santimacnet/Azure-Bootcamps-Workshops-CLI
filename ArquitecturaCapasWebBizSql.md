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

La plantilla crea dos máquinas virtuales (VM), una base de datos de Azure SQL y los recursos necesarios para admitir estos recursos, como discos, NIC y redes virtuales. También implementa el código para ejecutar la aplicación en cada nivel. 

La red virtual tiene 2 subredes, una para presentación y otra para el nivel aplicación, con un límite de seguridad para cada uno.

Se han aplicado etiquetas (tags) a los recursos como parte de la implementación para reflejar el nivel que admite el recurso (tier:presentation, tier:application, tier:data). 

Una vez finalizado ir al sitio web y poner valores validos: pizza, tacos, sushi.




