---
title: Información general de caché local del Servicio de aplicaciones de Azure | Microsoft Docs
description: En este artículo se describe cómo habilitar, cambiar el tamaño y consultar el estado de la característica de caché local del Servicio de aplicaciones de Azure
services: app-service
documentationcenter: app-service
author: SyntaxC4
manager: yochayk
editor: ''
tags: optional
keywords: ''

ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/04/2016
ms.author: cfowler

---
# Información general de caché local del Servicio de aplicaciones de Azure
El contenido de una aplicación web de Azure se almacena en Almacenamiento de Azure y surge de manera duradera como un recurso compartido de contenido. Este diseño está pensado para trabajar con una amplia gama de aplicaciones y tiene los siguientes atributos:

* El contenido se comparte entre varias instancias de máquina virtual de la aplicación web.
* El contenido es duradero y se puede modificar mediante la ejecución de las aplicaciones web.
* Los archivos de registro y los archivos de datos de diagnóstico están disponibles bajo la misma carpeta de contenido compartida.
* La publicación de nuevo contenido directamente actualiza la carpeta de contenido Puede ver el mismo contenido a través del sitio web de SCM y la aplicación web en ejecución (normalmente, algunas tecnologías como ASP.NET, llevan a cabo un reinicio de la aplicación web en algunos cambios de archivo para obtener el contenido más reciente).

Si bien muchas aplicaciones web usan una o la totalidad de estas características, algunas aplicaciones web solo necesitan un almacén de contenido de solo lectura de gran rendimiento desde donde se puedan ejecutar con alta disponibilidad. Estas aplicaciones se pueden beneficiar de una instancia de VM de una caché local específica.

La característica de caché local del Servicio de aplicaciones de Azure proporciona una vista de rol web del contenido. Este contenido es una caché de escritura temporal del contenido de almacenamiento que se crea de forma asincrónica durante el inicio del sitio. Cuando la memoria caché está preparada, el sitio cambia para ejecutarse con el contenido almacenado en caché. Las aplicaciones web que se ejecutan en la caché local tienen los siguientes beneficios:

* Son inmunes a las latencias que se producen cuando tienen acceso al contenido en Almacenamiento de Azure.
* Son inmunes a las actualizaciones planeadas o a los tiempos de inactividad no planeados y cualquier otra interrupción con Almacenamiento de Azure que se produzca en los servidores que atienden el recurso compartido del contenido.
* Tienen menos reinicios de aplicación debido a cambios en el recurso compartido de almacenamiento.

## Cómo la caché local cambia el comportamiento de Servicio de aplicaciones
* La caché local es una copia de las carpetas /site y /siteextensions de la aplicación web. Se crea en la instancia de VM local en el inicio de la aplicación web. El tamaño de la caché local por aplicación web se limita a 300 MB de forma predeterminada, pero se puede aumentar hasta 1 GB.
* La caché local es de lectura y escritura. Sin embargo, se descartará cualquier modificación cuando la aplicación web mueva máquinas virtuales o se reinicie. No debe usar la caché local para las aplicaciones que almacenan datos críticos en el almacén de contenido.
* Las aplicaciones web pueden continuar escribiendo archivos de registro y datos de diagnóstico como lo hacen actualmente. Sin embargo, los datos y archivos de registro se almacenan localmente en la VM. Luego, se copian periódicamente en el almacén de contenido compartido. La copia en el almacén de contenido compartido se realiza lo mejor posible y las reescrituras podrían perderse debido a un bloqueo repentino de una instancia de VM.
* Hay un cambio en la estructura de carpetas de las carpetas LogFiles y Data para las aplicaciones web que usan la memoria caché local. Ahora hay subcarpetas en las carpetas LogFiles y Data de almacenamiento que siguen el patrón de nomenclatura de "identificador único" + marca de tiempo. Cada una de las subcarpetas corresponde a una instancia de VM en donde la aplicación web se está ejecutando o se ha ejecutado.  
* La publicación de cambios en la aplicación web a través de cualquiera de los mecanismos de publicación se realizará en el almacén de contenido compartido. Esto es así por diseño, ya que queremos que el contenido publicado sea duradero. Para actualizar la caché local de la aplicación web, debe reiniciarse. ¿Le parece un paso excesivo? Para que el ciclo de vida avance sin problemas, consulte la información que aparece más adelante en este artículo.
* D:\\Home apuntará a la caché local. D:\\Local continuará apuntando al almacenamiento específico de VM temporal.
* La vista de contenido predeterminado del sitio de SCM seguirá siendo la del almacén de contenido compartido.

## Habilitación de la caché local en Servicio de aplicaciones
Para configurar la caché local, se usa una combinación de configuraciones de aplicación reservadas. Puede configurar estos ajustes de la aplicación con los métodos siguientes:

* [Portal de Azure](#Configure-Local-Cache-Portal)
* [Administrador de recursos de Azure](#Configure-Local-Cache-ARM)

### Configuración de la caché local mediante el Portal de Azure
<a name="Configure-Local-Cache-Portal"></a>

Use esta configuración de aplicación para habilitar la caché local en función de la aplicación web: `WEBSITE_LOCAL_CACHE_OPTION` = `Always`

![Configuración de aplicación del Portal de Azure: Caché local](media/app-service-local-cache/app-service-local-cache-configure-portal.png)

### Configuración de la caché local mediante Azure Resource Manager
<a name="Configure-Local-Cache-ARM"></a>

```
...

{
    "apiVersion": "2015-08-01",
    "type": "config",
    "name": "appsettings",
    "dependsOn": [
        "[resourceId('Microsoft.Web/sites/', variables('siteName'))]"
    ],
    "properties": {
        "WEBSITE_LOCAL_CACHE_OPTION": "Always",
        "WEBSITE_LOCAL_CACHE_SIZEINMB": "300"
    }
}

...
```

## Cambio de la configuración de tamaño en la caché local
El tamaño predeterminado de la caché local es **300 MB**. Esto incluye las carpetas /site y /siteextensions que se copian desde el almacén de contenido así como en cualquier registro y carpeta de datos creados localmente. Para aumentar este límites, use la configuración de aplicación `WEBSITE_LOCAL_CACHE_SIZEINMB`. Puede aumentar el tamaño hasta **1 GB** (1000 MB) por aplicación web.

## Prácticas recomendadas para usar la caché local del Servicio de aplicaciones
Se recomienda que use la caché local conjuntamente con la característica [Entornos de ensayo](../app-service-web/web-sites-staged-publishing.md).

* Agregue la configuración de aplicación *temporal* `WEBSITE_LOCAL_CACHE_OPTION` con el valor `Always` a la ranura de **producción**. Si usa `WEBSITE_LOCAL_CACHE_SIZEINMB`, agréguela también como una configuración temporal a su ranura de producción.
* Cree una ranura de **ensayo** y haga la publicación en su ranura de ensayo. Habitualmente, no define la ranura de ensayo para usar la caché local para habilitar un ciclo de vida de compilación, implementación y prueba sin problemas si obtiene los beneficios de la caché local para la ranura de producción.
* Pruebe el sitio en la ranura de ensayo.  
* Una vez que esté preparado, emita una [operación de intercambio](../app-service-web/web-sites-staged-publishing.md#to-swap-deployment-slots) entre las ranuras de ensayo y producción.  
* La configuración temporal incluye un nombre y se adhiere a una ranura. Por tanto, cuando la ranura de ensayo se intercambia por producción, hereda la configuración de aplicación de la caché local. La ranura de producción recién intercambiada se ejecutará en la caché local después de unos minutos y se calentará como parte del calentamiento de la ranura después del intercambio. Por tanto, cuando se complete el intercambio de ranura, la ranura de producción del usuario se ejecutará en la caché local.

## Preguntas más frecuentes
### ¿Cómo puedo saber si la caché local se aplica a mi aplicación web?
Si la aplicación web necesita un almacén de contenido confiable y de alto rendimiento y no usa el almacén de contenido para escribir datos críticos en tiempo de ejecución y tiene un tamaño menor que 1 GB, la respuesta es Sí. Para obtener el tamaño total de las carpetas /site y /siteextensions, puede utilizar la extensión del sitio "Uso de disco de Aplicaciones web de Azure".

### ¿Cómo puedo saber si mi sitio ha cambiado para usar caché local?
Si usa la característica de caché local con entornos de ensayo, la operación de intercambio no se completará hasta que la memoria caché local se haya calentado. Para comprobar si el sitio se ejecuta en caché local, puede comprobar la variable de entorno del proceso de trabajo `WEBSITE_LOCALCACHE_READY`. Siga las instrucciones de la página de la [variable de entorno del proceso de trabajo](https://github.com/projectkudu/kudu/wiki/Process-Threads-list-and-minidump-gcdump-diagsession#process-environment-variable) para tener acceso a la variable de entorno del proceso de trabajo en varias instancias.

### Acabo de publicar nuevos cambios, pero mi aplicación web no parece tenerlos. ¿Por qué?
Si la aplicación web usa caché local, debe reiniciar su sitio para obtener los cambios más recientes. ¿No desea publicar los cambios en un sitio de producción? Consulte las opciones de ranura en la sección anterior sobre prácticas recomendadas.

### ¿Dónde están mis registros?
Con la caché local, los registros y las carpetas de datos tienen un aspecto diferente. Sin embargo, la estructura de las subcarpetas permanece igual, excepto en que están anidadas en una subcarpeta con el formato "identificador de VM único" + marca de tiempo.

### Tengo habilitada la caché local, pero mi aplicación web todavía se reinicia. ¿Por qué ocurre esto? Pensé que la memoria caché local que iba a ayudar con los reinicios de aplicación frecuentes.
La caché local ayuda a evitar que la aplicación web relacionada con el almacenamiento se reinicie. Sin embargo, la aplicación web todavía puede sufrir reinicios durante actualizaciones de infraestructura de la VM planeadas. Los reinicios totales de aplicación que experimenta con la caché local habilitada deben ser menos.

<!---HONumber=AcomDC_0330_2016-->