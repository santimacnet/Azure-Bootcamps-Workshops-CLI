###### TIPS BASICOS PARA WORKSHOPS
Abrir una shell de Azure para ejecutar los comandos



### Configurar entorno Azure
```
$ az configure 
$ az configure --defaults location=westeurope
$ az configure --defaults group=rg-default-recursos
```

### Configurar suscripcion correcta
```
$ az account list
$ az account set --subscription ****-****-***-***
```

### Listar contenido ACR tanto Docker-images como Helm-Charts
```
$ az acr repository list -n acrhubdemolab -o table
$ az acr helm  list -n acrhubdemolab -o table
```

###### AZURE DEVOPS - TIPS PARA COMMITS Y PIPELINES CI-CD 

AzureDevOps: Updated azure-pipeline-CI - Tag Build Number
AzureDevOps: pipeline: tag: '$(Build.BuildNumber)'
AzureDevOps: commit: Tag sources - on success: $(build.buildNumber)

YAML schema: https://aka.ms/yaml

Git - bajar cambios desde repo-remoto (azdevops-github-otros): git pull origin master
Git - subir cambios hacia repo-remoto (azdevops-github-otros): git push origin master

Readme- poner el status Badge markdown coger url-imagen desde Azure DevOps
