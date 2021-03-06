---
title: Conexión y comunicación con los servicios de Service Fabric de Azure | Microsoft Docs
description: Aprenda a resolver, conectar y comunicar mediante servicios de Service Fabric.
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: msfussell

ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 07/05/2016
ms.author: vturecek

---
# Conexión y comunicación con servicios en Service Fabric
En Service Fabric, un servicio se ejecuta en algún lugar en un clúster de Service Fabric que normalmente se distribuye entre varias máquinas virtuales. Se puede mover de un lugar a otro, ya sea por indicación del propietario del servicio o automáticamente mediante Service Fabric. Los servicios no están enlazados estáticamente a un equipo o una dirección determinados.

Una aplicación de Service Fabric se compone, por lo general, de muchos servicios diferentes, donde cada uno de ellos realiza una tarea especializada. Estos servicios pueden comunicarse entre sí para formar una función completa, como representar diferentes partes de una aplicación web. También hay aplicaciones cliente que se conectan a los servicios y se comunican con ellos. Este documento describe cómo establecer la comunicación con los servicios de Service Fabric y entre ellos.

## Traiga su propio protocolo
Service Fabric le ayuda a administrar el ciclo de vida de los servicios, pero no toma ninguna decisión sobre qué hacen sus servicios. Esto incluye la comunicación. Cuando Service Fabric abre un servicio, este tiene la oportunidad de configurar un punto de conexión para las solicitudes entrantes mediante cualquier protocolo o pila de comunicación que desee. El servicio escuchará en una dirección **IP:port** normal mediante cualquier esquema de direccionamiento, como un URI. Varias instancias de servicio o réplicas pueden compartir un proceso de host, en cuyo caso deberán usar puertos diferentes o utilizar un mecanismo de uso compartido de puertos, como el controlador del kernel http.sys en Windows. En ambos casos, cada instancia de servicio o réplica de un proceso de host debe ser direccionable de forma exclusiva.

![puntos de conexión de servicio][1]

## Detección y resolución de servicios
En un sistema distribuido, los servicios se pueden mover de una máquina a otra con el tiempo. Esto puede ocurrir por diversos motivos, incluidos el equilibrio, las actualizaciones, las conmutaciones por error o el escalado de recursos. Esto significa que el punto de conexión del servicio experimenta cambios a medida que el servicio se mueve a nodos con direcciones IP diferentes y se puede abrir en puertos diferentes si el servicio utiliza un puerto seleccionado de forma dinámica.

![Distribución de servicios][7]

Service Fabric proporciona un servicio de detección y resolución denominado servicio de nomenclatura. El servicio de nomenclatura mantiene una tabla que asigna instancias de servicio con nombre a las direcciones del punto de conexión a las que escuchan. Todas las instancias de servicio con nombre en Service Fabric tienen nombres únicos como URI como, por ejemplo, `"fabric:/MyApplication/MyService"`. El nombre del servicio no cambia durante el ciclo de vida del mismo, solo las direcciones de punto de conexión pueden cambiar cuando se mueve el servicio. Esto es análogo a los sitios web que tienen direcciones URL constantes, pero donde la dirección IP puede cambiar. Y de forma similar a un DNS en la web, que resuelve direcciones URL del sitio web en direcciones IP, Service Fabric tiene un registrador que asigna nombres de servicio a la dirección del punto de conexión.

![puntos de conexión de servicio][2]

La resolución y conexión a servicios implica que los pasos siguientes se ejecuten en un bucle:

* **Resolver**: Obtenga el punto de conexión que ha publicado un servicio mediante el servicio de nomenclatura.
* **Conectar**: Conéctese al servicio a través de cualquier protocolo que se utilice en ese punto de conexión.
* **Reintentar**: Un intento de conexión puede producir un error por diversos motivos como, por ejemplo, cuando el servicio se ha movido desde la última vez que se resolvió la dirección del punto de conexión. En ese caso, se deben probar de nuevo los pasos de conexión y resolución anteriores, y este ciclo se repetirá hasta que la conexión sea correcta.

## Conexiones desde clientes externos
Los servicios que se conectan entre sí dentro de un clúster pueden acceder, por lo general, directamente a los puntos de conexión de otros servicios ya que los nodos de un clúster se encuentran normalmente en la misma red local. Sin embargo, en algunos entornos, un clúster puede estar detrás de un equilibrador de carga que enruta el tráfico de entrada externo a través de un conjunto limitado de puertos. En estos casos, los servicios todavía pueden comunicarse entre sí y resolver direcciones mediante el servicio de nomenclatura, pero se deben realizar pasos adicionales para permitir que los clientes externos se conecten a los servicios.

## Service Fabric en Azure
Un clúster de Service Fabric en Azure se coloca detrás de un equilibrador de carga de Azure. Todo el tráfico externo al clúster debe pasar a través del equilibrador de carga. El equilibrador de carga reenviará automáticamente el tráfico entrante de un puerto determinado a un *nodo* aleatorio que tenga el mismo puerto abierto. El equilibrador de carga solo detecta los puertos abiertos en los *nodos*, no detecta puertos abiertos por otros *servicios* individuales.

![Topología de equilibrador de carga de Azure y Service Fabric][3]

Por ejemplo, para poder aceptar tráfico externo en el puerto **80**, se deben configurar las siguientes condiciones:

1. Escribir un servicio que escuche en el puerto 80. Configure el puerto 80 en el archivo ServiceManifest.xml del servicio y abra un agente de escucha en el mismo, por ejemplo, un servidor web autohospedado.
   
    ```xml
    <Resources>
        <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>
    ```
    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...
   
            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint = 
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");
   
                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";
   
                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();
   
                string uriPublished = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
   
                return Task.FromResult(this.publishUri);
            }
   
            ...
        }
   
        class WebService : StatelessService
        {
            ...
   
            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] {new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }
   
            ...
        }
    ```
2. Crear un clúster de Service Fabric en Azure y especificar el puerto **80** como un puerto de punto de conexión personalizado para el tipo de nodo que va a hospedar el servicio. Si tiene más de un tipo de nodo, puede configurar una *restricción de colocación* en el servicio para asegurarse de que solo se puede ejecutar en el tipo de nodo que tiene el puerto de punto de conexión personalizado abierto.
   
    ![Apertura de un puerto en un tipo de nodo][4]
3. Una vez creado el clúster, configure el equilibrador de carga de Azure en el grupo de recursos del clúster para reenviar el tráfico hacia el puerto 80. Al crear un clúster mediante el Portal de Azure, este se configura automáticamente para cada puerto de punto de conexión personalizado que se ha configurado.
   
    ![Reenvío del tráfico en el equilibrador de carga de Azure][5]
4. El equilibrador de carga de Azure utiliza un sondeo para determinar si envía o no tráfico a un nodo concreto. El sondeo comprueba periódicamente un punto de conexión de cada nodo para determinar si este está respondiendo o no. Si el sondeo no recibe una respuesta después de un número determinado de veces, el equilibrador de carga deja de enviar tráfico a dicho nodo. Al crear un clúster mediante el Portal de Azure, se configura automáticamente un sondeo para cada puerto de punto de conexión personalizado que se ha configurado.
   
    ![Reenvío del tráfico en el equilibrador de carga de Azure][8]

Es importante recordar que el equilibrador de carga de Azure y el sondeo solo detectan los *nodos* y no los *servicios* que se ejecutan en dichos nodos. El equilibrador de carga de Azure siempre enviará tráfico a los nodos que respondan al sondeo, por lo que se debe garantizar que haya servicios disponibles en los nodos que puedan responder al sondeo.

## Opciones de API de comunicación incorporadas
El marco de Reliable Services incluye varias opciones de comunicación pregeneradas. La decisión de cuál funcionará mejor para usted depende de la elección del modelo de programación, el marco de comunicación y el lenguaje de programación en que se crean los servicios.

* **No hay ningún protocolo específico:** si no tiene una opción concreta de marco de comunicación, pero desea crear algo que se pueda ejecutar rápidamente, entonces la opción idónea para usted es la [comunicación remota de servicio](service-fabric-reliable-services-communication-remoting.md), que permite llamadas a procedimientos remotos fuertemente tipados para Reliable Services y Reliable Actors. Esta es la manera más sencilla y rápida de empezar con la comunicación del servicio. La comunicación remota de servicio controla la resolución de direcciones, la conexión, los reintentos y el control de errores del servicio. Tenga en cuenta que la comunicación remota de servicio solo está disponible en aplicaciones de C#.
* **HTTP**: para una comunicación independiente del idioma, HTTP proporciona una opción estándar con las herramientas y los servidores HTTP disponibles en muchos idiomas diferentes, compatibles todos ellos con Service Fabric. Los servicios pueden utilizar cualquier pila HTTP disponible, incluida la [ASP.NET Web API](service-fabric-reliable-services-communication-webapi.md). Los clientes escritos en C# pueden aprovechar las [clases `ICommunicationClient` y `ServicePartitionClient` ](service-fabric-reliable-services-communication.md) para la resolución del servicio, las conexiones HTTP y los bucles de reintento.
* **WCF**: si tiene código existente que usa WCF como su marco de comunicación, puede usar el `WcfCommunicationListener` para el lado servidor, y las clases `WcfCommunicationClient` y `ServicePartitionClient` para el cliente. Para obtener más información, consulte este artículo sobre la [implementación basada en WCF de la pila de comunicación](service-fabric-reliable-services-communication-wcf.md).

## Uso de protocolos personalizados y otros marcos de comunicación
Los servicios pueden usar cualquier protocolo o marco de comunicación, ya sea un protocolo binario personalizado sobre sockets de TCP o eventos de streaming a través de [Centros de eventos de Azure](https://azure.microsoft.com/services/event-hubs/) o [Centro de IoT de Azure](https://azure.microsoft.com/services/iot-hub/). Service Fabric proporciona las API de comunicación a las que puede conectar su pila de comunicación, al tiempo que se le quita todo el trabajo de detección y conexión. Consulte el artículo sobre el [Modelo de comunicación de Reliable Services](service-fabric-reliable-services-communication.md) para obtener más detalles.

## Pasos siguientes
Obtenga más información sobre los conceptos y las API disponibles en el [modelo de comunicación de Reliable Services](service-fabric-reliable-services-communication.md) y, después, empiece a trabajar rápidamente con [comunicación remota de servicio](service-fabric-reliable-services-communication-remoting.md) o profundice para más información sobre cómo escribir un agente de escucha de comunicación mediante la [API web con autohospedaje OWIN](service-fabric-reliable-services-communication-webapi.md).

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png

<!---HONumber=AcomDC_0713_2016-->