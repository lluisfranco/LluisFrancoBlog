---
title: "Enable CORS on Azure App Service"
subtitle: "Usando Azure Cloud Shell"
date: 2022-03-21T13:42:50+02:00
featuredImage: /images/posts/AzureCloudShellCover.png
fontawesome: "true"
categories: 
- Azure
tags:
- api
- cors
- tip
# draft: true
---
## Error accediendo a API en Azure (CORS not enabled)

Esta mañana estaba haciendo unas pruebas accediendo a una API (mia) que tengo en Azure.

Suelo utilizar esta API para hacer pruebas, y en esta ocasión estaba probando cuatro cosas de una app realizada en React. En el momento de hacer un fetch he visto que no podía acceder porque me tiraba un [típico error de CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors/CORSMissingAllowOrigin):

> Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at https://la-url-de-la-app

El problema es que como estos días estoy de baja, en este momento no tengo acceso al código fuente de la API. De modo que he probado de solucionarlo desde el propio portal de azure.

Lo que pasa es que al estar haciendo pruebas con esta aplicación de React, la ruta que debería autorizar para que pueda usar la API es la típica de:

http://localhost:puerto 

Y claro, la interfaz del portal de Azure no lo permite (o al menos a mi no me dejaba).

### Agregar la URL mediante Azure Bash

Bueno, al menos es posible solucionarlo con el comando usando la [Azure Cloud Shell](https://learn.microsoft.com/en-us/azure/cloud-shell/overview)

Si no tienes activada la shell para esta suscripción de Azure, la primera vez te pedirá crear una cuenta de Storage necesaria para su funcionamiento. El proceso es muy sencillo, y está descrito en la [sección de quickstart](https://learn.microsoft.com/en-us/azure/cloud-shell/quickstart?tabs=azurecli) de la shell.

En mi caso suelo utilizar Bash pero también puede usar Powershell si te sientes más cómodo.

El comando es el siguiente y necesitarás saber el nombre de tu Resource Group y del App Service (los tienes ambos en la página de tu app service).

Suponiendo que el nombre de tu Resource Group es **MyResourceGroup** y tu App Service es **ApiDePruebas** solo debes agregar la URL a la que quiers permitir el acceso a tu API, en este caso vamos a suponer que es **http://localhost:3000**

``` bash
az webapp cors add --resource-group 'MyResourceGroup' --name 'ApiDePruebas' --allowed-origins 'http://localhost:3000'
```

Una vez añadida la URL vais a poder acceder a ella desde la acplicación cliente sin problema :)

Happy coding!
