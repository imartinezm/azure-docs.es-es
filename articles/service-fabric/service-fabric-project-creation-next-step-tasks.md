---
title: Pasos siguientes de la creación del proyecto de Service Fabric | Microsoft Docs
description: Este artículo contiene vínculos a un conjunto de tareas básicas de desarrollo para Service Fabric
services: service-fabric
documentationcenter: .net
author: seanmck
manager: timlt
editor: ''

ms.service: service-fabric
ms.devlang: dotNet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/08/2016
ms.author: seanmck

---
# Su aplicación de Service Fabric y próximos pasos
La aplicación Azure Service Fabric se ha creado. En este artículo se describe la preparación del proyecto y algunos posibles pasos siguientes.

## Su aplicación
Cada nueva aplicación incluye un proyecto de aplicación. Puede haber uno o dos proyectos adicionales según el tipo de servicio elegido.

### El proyecto de aplicación
El proyecto de aplicación consiste en:

* Un conjunto de referencias a los servicios que conforman la aplicación.
* Dos perfiles de publicación (local y en la nube) que puede usar para conservar las preferencias para trabajar con entornos diferentes, tales como las preferencias relacionadas con un punto de conexión de clúster y si va a realizar implementaciones de actualización de forma predeterminada.
* Dos archivos de parámetro de aplicación (local y en la nube) que puede usar para mantener configuraciones de aplicaciones específicas del entorno, como el número de particiones para crear un servicio.
* Un script de implementación que puede usar para implementar la aplicación desde la línea de comandos o como parte de una canalización de integración e implementación continua automatizada.
* El manifiesto de aplicación que describe la aplicación. Puede encontrar el manifiesto en la carpeta ApplicationPackageRoot.

### Servicio sin estado
Cuando se agrega un nuevo servicio sin estado, Visual Studio agrega un proyecto de servicio a la solución que incluye un tipo que se hereda de `StatelessService`. El servicio incrementa una variable local en un contador.

### Servicio con estado
Cuando se agrega un nuevo servicio con estado, Visual Studio agrega un proyecto de servicio a la solución que incluye un tipo que se hereda de `StatefulService`. El servicio incrementa un contador en su método `RunAsync` y almacena el resultado en una clase `ReliableDictionary`.

### Servicio de actor
Cuando se agrega una nueva instancia de Reliable Actors, Visual Studio agrega dos proyectos a la solución: un proyecto de actor y un proyecto de interfaz.

El proyecto de actor proporciona métodos para configurar y obtener el valor de un contador que se conserva de forma confiable dentro de estado del actor. El proyecto de interfaz proporciona una interfaz que pueden usar otros servicios para invocar al actor.

### API web sin estado
El proyecto de API web sin estado proporciona un servicio web básico que puede usar para abrir la aplicación en clientes externos. Para más información sobre cómo ver el proyecto estructurado, consulte [Introducción a los servicios de la API web de Microsoft Azure Service Fabric con autohospedaje OWIN](service-fabric-reliable-services-communication-webapi.md).

### ASP.NET Core
El SDK de Service Fabric proporciona el mismo conjunto de plantillas de ASP.NET Core que las disponibles para los proyectos independientes de ASP.NET Core: una vacía, una de [API web][aspnet-webapi] y una de [aplicación web][aspnet-webapp].

## Pasos siguientes
### Creación de un clúster de Azure
El SDK de Service Fabric proporciona un clúster local para desarrollo y pruebas. Para crear un clúster en Azure, consulte [Creación de un clúster de Service Fabric en el Portal de Azure][create-cluster-in-portal].

### Pruebe a implementar en Azure de forma gratuita con clústeres de Party Cluster.
Si quiere probar la implementación y administración de aplicaciones de Azure sin necesidad de configurar sus propios clústeres, puede utilizar la versión de evaluación del [Servicio Party Cluster](http://aka.ms/tryservicefabric).

### Publicación de su aplicación en Azure
Puede publicar su aplicación directamente desde Visual Studio a un clúster de Azure. Para obtener información sobre cómo hacerlo, consulte [Publicación de la aplicación en Azure][publish-app-to-azure].

### Uso del explorador de Service Fabric para visualizar el clúster
El Explorador de Service Fabric ofrece una forma sencilla de visualizar el clúster, como las aplicaciones implementadas y el diseño físico. Para más información, consulte [Visualización del clúster mediante Service Fabric Explorer][visualize-with-sfx].

### Control de versiones y actualización de los servicios
Service Fabric permite el control de versiones independiente y la actualización de servicios independientes en una aplicación. Para más información, consulte [Control de versiones y actualización de los servicios][app-upgrade-tutorial].

### Configuración de la integración continua con Visual Studio Team Services
Para obtener información sobre cómo puede configurar un proceso de integración continua para su aplicación de Service Fabric, consulte [Configuración de la integración continua con Visual Studio Team Services][ci-with-vso].

<!-- Links -->
[add-web-frontend]: service-fabric-add-a-web-frontend.md
[create-cluster-in-portal]: service-fabric-cluster-creation-via-portal.md
[publish-app-to-azure]: service-fabric-publish-app-remote-cluster.md
[visualize-with-sfx]: service-fabric-visualizing-your-cluster.md
[ci-with-vso]: service-fabric-set-up-continuous-integration.md
[reliable-services-webapi]: service-fabric-reliable-services-communication-webapi.md
[app-upgrade-tutorial]: service-fabric-application-upgrade-tutorial.md
[aspnet-webapi]: https://docs.asp.net/en/latest/tutorials/first-web-api.html
[aspnet-webapp]: https://docs.asp.net/en/latest/tutorials/first-mvc-app/index.html

<!---HONumber=AcomDC_0713_2016-->