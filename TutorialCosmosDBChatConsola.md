**PRACTICA AZURE-COSMOS DB CON MONGO **
-------------------------------------------------------

Consola: [az login] o [https://shell.azure.com]
Ejemplo: https://github.com/MicrosoftDocs/mslearn-handle-transient-errors-in-your-app

### Configurar entorno Azure para crear la base de datos

$ COSMOS_DB_NAME=cosmos-db-$RANDOM

$ COSMOS_DB_ENDPOINT=$(az cosmosdb create \
  --resource-group practica-acfb5bbc \
  --name $COSMOS_DB_NAME \
  --kind MongoDB \
  --query documentEndpoint \
  --output tsv)
  
$ az cosmosdb database create --name $COSMOS_DB_NAME --resource-group practica-acfb5bbc --db-name chat-app
  
  
### Obtener la cadena conexion de Mongo

$ az cosmosdb list-connection-strings \
  --resource-group practica-acfb5bbc \
  --name $COSMOS_DB_NAME  | sed -n -e '4 p' | sed -E -e 's/.*mongo(.*)true.*/mongo\1true/'
  
### Clonar repo de la aplicación de chat y ejecutarla con NetCore

cd ~
git clone https://github.com/MicrosoftDocs/mslearn-handle-transient-errors-in-your-app.git
cd ~/mslearn-handle-transient-errors-in-your-app/csharp/chatapp/

nano Program.cs //Poner cadena de conexion con mongo en el codigo fuente

dotnet build
dotnet run

Para probar la aplicacion utilizar los comandos que se muestran en consola.
Introducir varios mensajes para guardarlos en base de datos
En Azure buscar cosmosdb y abrir colecciones para ver los datos registrados.




