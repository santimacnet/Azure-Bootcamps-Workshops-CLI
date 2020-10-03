**PRACTICA IDENTIDADES ADMINISTRADAS EN AZURE**
------------------------------------------------------------------

- Tutorial para charlas y formación sobre Gestionar Identidades y Azure KeyVault.
- MSLEARN: https://docs.microsoft.com/es-es/learn/modules/authenticate-apps-with-managed-identities/5-exercise-configure-managed-identity-for-vm
   
###Configuración de una identidad administrada asignada por el sistema para una máquina virtual de Azure

Crear **Azure Key Vault** para almacenar los datos privados de la empresa. 
```
# configurar  grupo-recursos correcto para ejercicio
export VMNAME=prodserver
export KVNAME=furniture-secrets$RANDOM

az keyvault create --name $KVNAME \
    --resource-group learn-xxxxx \
    --default-action Allow \
    --location $(az resource list --output tsv --query [0].location) \
    --sku standard    
```


Crear **Virtual Machine** para hospedar una aplicación web y guardar dirección IP pública en una variable de entorno.
```
# configurar grupo-recursos correcto para ejercicio
export publicIP=$(az vm create \
    --name $VMNAME \
    --resource-group learn-xxxxx \
    --image UbuntuLTS \
    --generate-ssh-keys \
    --output tsv \
    --query "publicIpAddress")
```


Asignar identidad administrada asignada por el sistema a la máquina virtual.
```
# configurar grupo-recursos correcto para ejercicio
az vm identity assign \
  --name $VMNAME \
  --resource-group learn-be5e742c-bf03-4628-897a-da17c76e140c

# Este comando debe devolver una respuesta en la que se muestre la identidad administrada. 
{
    "systemAssignedIdentity": "a78ddd60-183b-4e27-9f0d-c11a11c417d8",
    "userAssignedIdentities": {}
}
```

Uso del almacén de claves para almacenar un secreto
```
# Agregue la cadena de conexión al almacén de claves.
az keyvault secret set \
  --vault-name $KVNAME \
  --name DBCredentials \
  --value "Server=tcp:prodserverSQL.database.windows.net,1433;Database=myDataBase;User ID=mylogin@myserver;Password=examplePassword;Trusted_Connection=False;Encrypt=True;"

# Anotar nombre del almacén de claves.
echo $KVNAME
```

###Configuración de la máquina virtual para la aplicación de seguimiento de inventario de la empresa

Completar los siguentes pasos desde Azure CLI
```
# Contectar pos SSH para acceder a la máquina virtual y escribir "yes"
ssh $publicIP

# instalar .NET Core en la máquina virtual
wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-3.1

# Descargue el código fuente para la aplicación de ejemplo de este módulo.
git clone https://github.com/MicrosoftDocs/mslearn-authenticate-apps-with-managed-identities identity

# Finalizar la sesión de SSH
exit
```


