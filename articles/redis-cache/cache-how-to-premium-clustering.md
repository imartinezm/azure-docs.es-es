---
title: Cómo configurar la agrupación en clústeres de Redis para una Caché en Redis de Azure Premium | Microsoft Docs
description: Obtener información sobre cómo crear y administrar la agrupación en clústeres de para sus instancias de Caché en Redis de Azure de nivel Premium
services: redis-cache
documentationcenter: ''
author: steved0x
manager: douge
editor: ''

ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: na
ms.topic: article
ms.date: 09/15/2016
ms.author: sdanie

---
# Cómo configurar la agrupación en clústeres de Redis para una Caché en Redis de Azure Premium
Azure Redis Cache tiene diferentes ofertas de caché que proporcionan flexibilidad en la elección del tamaño y las características de la caché, incluido el nuevo nivel Premium.

El nivel premium de Azure Redis Cache incluye características como la agrupación en clústeres, la persistencia y la compatibilidad de red virtual. En este artículo se describe cómo configurar la agrupación en clústeres en una instancia de Caché en Redis de Azure premium.

Para obtener información sobre otras características de caché Premium, consulte [Introducción al nivel Premium de Azure Redis Cache](cache-premium-tier-intro.md).

## ¿Qué es Clúster Redis?
Caché en Redis de Azure ofrece clúster de Redis como [implementado en Redis](http://redis.io/topics/cluster-tutorial). Con el Clúster de Redis, obtendrá las siguientes ventajas.

* La capacidad de dividir automáticamente el conjunto de datos entre varios nodos.
* La capacidad de continuar las operaciones cuando un subconjunto de los nodos está teniendo errores o no se puede comunicar con el resto del clúster.
* Mayor rendimiento: el rendimiento aumenta de manera lineal a medida que aumenta el número de particiones.
* Mayor tamaño de memoria: aumenta de manera lineal a medida que aumenta el número de particiones.

Consulte el artículo [P+F de Azure Redis Cache](cache-faq.md#what-redis-cache-offering-and-size-should-i-use) para más información sobre el tamaño, el rendimiento y el ancho de banda de las memorias caché de nivel premium.

En Azure, el clúster de Redis se ofrece como un modelo de principal/réplica donde cada partición tiene un par de principal/réplica con la replicación donde la replicación se administra mediante el servicio de la Caché en Redis de Azure.

## Agrupación en clústeres
La agrupación en clústeres se habilita en la hoja **Nueva caché en Redis** durante la creación de la memoria caché.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

La agrupación en clústeres se configura en la hoja **Clúster de Redis**.

![Agrupación en clústeres][redis-cache-clustering]

Puede tener hasta 10 particiones en el clúster. Haga clic en **Habilitado** y deslice el control deslizante o escriba un número comprendido entre 1 y 10 para **Número de particiones** y haga clic en **Aceptar**.

Cada partición es un par de caché principal/réplica administrado por Azure y el tamaño total de la memoria caché se calcula multiplicando el número de particiones por el tamaño de la memoria caché seleccionado en el plan de tarifa.

![Agrupación en clústeres][redis-cache-clustering-selected]

Cuando se haya creado la memoria caché, conéctese a ella y úsela simplemente como una memoria caché no agrupada y Redis distribuirá los datos a través de las particiones de memoria caché. Si el diagnóstico está [habilitado](cache-how-to-monitor.md#enable-cache-diagnostics), las métricas se capturan por separado para cada partición y pueden [verse](cache-how-to-monitor.md) en la hoja Caché en Redis.

Para obtener el código de ejemplo sobre el trabajo con clústeres con el cliente StackExchange.Redis, consulte la parte [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) del ejemplo [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

<a name="cluster-size"></a>

## Cambio del tamaño del clúster en una caché premium en ejecución
Para cambiar el tamaño del clúster de una caché premium en ejecución con la agrupación en clústeres habilitada, haga clic en **(VISTA PREVIA) Tamaño del clúster en Redis** desde la hoja **Configuración**.

> [!NOTE]
> Tenga en cuenta que, a pesar de que el nivel Premium de Caché en Redis de Azure se publicó con disponibilidad general, la característica Tamaño del clúster en Redis está actualmente en la versión preliminar.
> 
> 

![Tamaño del Clúster en Redis][redis-cache-redis-cluster-size]

Para cambiar el tamaño del clúster, use el control deslizante o escriba un número entre 1 y 10 en el cuadro de texto **Número de particiones** y haga clic en **Aceptar** para guardar.

## P+F de agrupación en clústeres
La lista siguiente contiene las respuestas a las preguntas más frecuentes sobre la agrupación en clústeres de Caché en Redis de Azure.

* [¿Es necesario realizar algún cambio en mi aplicación cliente para usar la agrupación en clústeres?](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
* [¿Cómo se distribuyen las claves en un clúster?](#how-are-keys-distributed-in-a-cluster)
* [¿Cuál es el mayor tamaño de caché que puedo crear?](#what-is-the-largest-cache-size-i-can-create)
* [¿Todos los clientes de Redis admiten la agrupación en clústeres?](#do-all-redis-clients-support-clustering)
* [¿Cómo me conecto a mi memoria caché cuando la agrupación en clústeres esté habilitada?](#how-do-i-connect-to-my-cache-when-clustering-is-enabled)
* [¿Puedo conectarme directamente a las particiones individuales de mi memoria caché?](#can-i-directly-connect-to-the-individual-shards-of-my-cache)
* [¿Puedo configurar la agrupación en clústeres para una memoria caché creada anteriormente?](#can-i-configure-clustering-for-a-previously-created-cache)
* [¿Puedo configurar la agrupación en clústeres para una caché básica o estándar?](#can-i-configure-clustering-for-a-basic-or-standard-cache)
* [¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?.](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)
* [Estoy recibiendo excepciones MOVE al usar StackExchange.Redis y agrupaciones en clústeres, ¿qué debo hacer?](#i-am-getting-move-exceptions-whes-ESing-stackexchangeredis-and-clustering-what-should-i-do)

### ¿Es necesario realizar algún cambio en mi aplicación cliente para usar la agrupación en clústeres?
* Cuando la agrupación en clústeres está habilitada, solo está disponible la base de datos 0. Si la aplicación cliente usa varias bases de datos e intenta leer o escribir en una base de datos distinta de 0, se produce la siguiente excepción. `Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`
  
  Para obtener más información, consulte [Redis Cluster Specification - Implemented subset](http://redis.io/topics/cluster-spec#implemented-subset) (Especificación de clúster en Redis: subconjunto implementado).
* Si usa [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/), debe usar la versión 1.0.481 o posterior. Se conecta a la memoria caché con los mismos [puntos de conexión, puertos y claves](cache-configure.md#properties) que usa al conectarse a una memoria caché que no tenga la agrupación en clústeres habilitada. La única diferencia es que se deben realizar todas las lecturas y escrituras en la base de datos 0.
  
  * Otros clientes pueden tener requisitos diferentes. Vea [¿Todos los clientes de Redis admiten la agrupación en clústeres?](#do-all-redis-clients-support-clustering)
* Si la aplicación usa varias operaciones de claves por lotes en un solo comando, todas las claves deben estar ubicadas en la misma partición. Para lograr esto, consulte [¿Cómo se distribuyen las claves en un clúster?](#how-are-keys-distributed-in-a-cluster)
* Si está usando el proveedor de estado de sesión de ASP.NET de Redis, debe usar 2.0.1 o posterior. Consulte [¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)

### ¿Cómo se distribuyen las claves en un clúster?
Según la documentación del [modelo de distribución de claves](http://redis.io/topics/cluster-spec#keys-distribution-model) de Redis, el espacio de claves se divide en 16 384 espacios. Cada clave tiene un hash y se asigna a una de estas ranuras, que se distribuyen por todos los nodos del clúster. Puede configurar qué parte de la clave está con hash para asegurarse de que varias claves se encuentran en la misma partición con etiquetas hash.

* Claves con una etiqueta hash: si cualquier parte de la clave está encerrada entre `{` y `}`, se aplica un algoritmo hash únicamente a esa parte de la clave para determinar la ranura hash de una clave. Por ejemplo, las siguientes tres claves se encontrarían en la misma partición: `{key}1`, `{key}2` y `{key}3` ya que solo la parte `key` del nombre está con hash. Para obtener una lista completa de especificaciones de etiquetas hash de claves, consulte [Etiquetas hash de claves](http://redis.io/topics/cluster-spec#keys-hash-tags).
* Claves sin etiquetas hash: se usa todo el nombre de la clave para aplicar el hash. Esto produce una distribución estadísticamente uniforme entre las particiones de la memoria caché.

Para optimizar el rendimiento, se recomienda distribuir las claves de manera uniforme. Si usa claves con etiquetas hash, es responsabilidad de la aplicación asegurarse de que las claves se distribuyan de manera uniforme.

Para obtener más información, consulte [Modelo de distribución de claves](http://redis.io/topics/cluster-spec#keys-distribution-model), [Particionamiento de datos de clúster Redis](http://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding) y [Etiquetas hash de claves](http://redis.io/topics/cluster-spec#keys-hash-tags).

Para obtener el código de ejemplo sobre el trabajo con agrupación en clústeres y la ubicación de claves en la misma partición con el cliente StackExchange.Redis, consulte la parte [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) del ejemplo [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

### ¿Cuál es el mayor tamaño de caché que puedo crear?
El tamaño máximo de caché premium es 53 GB. Puede crear hasta 10 particiones con un tamaño máximo de 530 GB. Si necesita un tamaño mayor, puede [solicitar más](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase). Para obtener más información, consulte [Precios de Caché en Redis de Azure](https://azure.microsoft.com/pricing/details/cache/).

### ¿Todos los clientes de Redis admiten la agrupación en clústeres?
En este momento no todos los clientes admiten la agrupación en clústeres de Redis. StackExchange.Redis es uno de los que los admiten. Para obtener más información sobre otros clientes, consulte la sección [Jugar con el clúster](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster) del [Tutorial de clúster de Redis](http://redis.io/topics/cluster-tutorial).

> [!NOTE]
> Si está usando StackExchange.Redis como su cliente, asegúrese de que está usando la versión más reciente de [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) 1.0.481 o posterior para que la agrupación en clústeres funcione correctamente. Si tiene problemas con las excepciones move, consulte la explicación sobre [excepciones move](#move-exceptions) para obtener más información.
> 
> 

### ¿Cómo me conecto a mi memoria caché cuando la agrupación en clústeres esté habilitada?
Puede conectarse a su memoria caché con los mismos [extremos, puertos y claves](cache-configure.md#properties) que usa al conectarse a una memoria caché que no tenga la agrupación en clústeres habilitada. Redis administra la agrupación en clústeres en el back-end para que no tenga que administrarla desde el cliente.

### ¿Puedo conectarme directamente a las particiones individuales de mi memoria caché?
Esto no se admite oficialmente. Dicho esto, cada partición consta de un par de caché principal/réplica que se conoce colectivamente como una instancia de caché. Puede conectarse a estas instancias de caché mediante la utilidad redis-cli en la rama [inestable](http://redis.io/download) del repositorio de Redis en GitHub. Esta versión implementa compatibilidad básica cuando se inicia con el conmutador `-c`. Para obtener más información, consulte [Jugar con el clúster](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster) en [http://redis.io](http://redis.io) en el [Tutorial del clúster de Redis](http://redis.io/topics/cluster-tutorial).

Cuando no sea ssl, use los siguientes comandos.

    Redis-cli.exe –h <<cachename>> -p 13000 (to connect to instance 0)
    Redis-cli.exe –h <<cachename>> -p 13001 (to connect to instance 1)
    Redis-cli.exe –h <<cachename>> -p 13002 (to connect to instance 2)
    ...
    Redis-cli.exe –h <<cachename>> -p 1300N (to connect to instance N)

Para ssl, reemplace `1300N` por `1500N`.

### ¿Puedo configurar la agrupación en clústeres para una memoria caché creada anteriormente?
En este momento solo puede habilitar la agrupación en clústeres cuando cree una memoria caché. También puede cambiar el tamaño de clúster una vez creada la memoria caché, pero no puede agregar la agrupación en clústeres a una caché premium o quitar la agrupación en clústeres de una memoria caché premium una vez creada la memoria caché. Una memoria caché premium que tiene habilitada la agrupación en clústeres y solo una partición es distinta de una memoria caché premium del mismo tamaño, sin agrupación en clústeres.

### ¿Puedo configurar la agrupación en clústeres para una caché básica o estándar?
La agrupación en clústeres solo está disponible para las memorias cachés premium.

### ¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?.
* **Proveedor de caché de salida de Redis**: no se requieren cambios.
* **Proveedor de estado de sesión de Redis**: para usar la agrupación en clústeres, debe usar [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 2.0.1 o superior o se iniciará una excepción. Este es un cambio importante. Para obtener más información, consulte [Detalles sobre cambios importantes de la versión 2.0.0](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details).

<a name="move-exceptions"></a>

### Estoy recibiendo excepciones MOVE al usar StackExchange.Redis y agrupaciones en clústeres, ¿qué debo hacer?
Si utiliza StackExchange.Redis y recibe `MOVE` excepciones al emplear agrupaciones en clústeres, asegúrese de que está utilizando la versión [1\.1.603 de StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) o posterior. Para obtener instrucciones sobre cómo configurar las aplicaciones .NET para usar StackExchange.Redis, consulte [Configuración de los clientes de caché](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

## Pasos siguientes
Obtenga información acerca de cómo usar más características de la memoria caché del nivel Premium.

* [Introducción al nivel Premium de Caché en Redis de Azure](cache-premium-tier-intro.md)

<!-- IMAGES -->

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png

<!---HONumber=AcomDC_0921_2016-->