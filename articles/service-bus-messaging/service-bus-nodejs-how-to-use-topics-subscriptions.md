---
title: Uso de temas de Service Bus con Node.js | Microsoft Docs
description: Aprenda a usar los temas y las suscripciones de Service Bus en Azure desde una aplicación Node.js.
services: service-bus
documentationcenter: nodejs
author: sethmanheim
manager: timlt
editor: ''

ms.service: service-bus
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 10/04/2016
ms.author: sethm

---
# <a name="how-to-use-service-bus-topics-and-subscriptions"></a>Uso de temas y suscripciones de Service Bus
[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

En esta guía se describe cómo usar los temas y las suscripciones de Service Bus desde aplicaciones Node.js. Entre los escenarios tratados se incluyen la **creación de temas y suscripciones**, la **creación de filtros de suscripción**, **el envío de mensajes** a un tema, **la recepción de mensajes de una suscripción** y la **eliminación de temas y suscripciones**. Para más información sobre los temas y las suscripciones, consulte la sección [Pasos siguientes](#next-steps).

[!INCLUDE [howto-service-bus-topics](../../includes/howto-service-bus-topics.md)]

## <a name="create-a-node.js-application"></a>Creación de una aplicación Node.js
Cree una aplicación Node.js vacía. Para obtener instrucciones sobre cómo crear una aplicación Node.js, vea [Creación e implementación de una aplicación Node.js en un sitio web de Azure], [Servicio en la nube Node.js][](con Windows PowerShell.md) o Sitio web con WebMatrix.

## <a name="configure-your-application-to-use-service-bus"></a>Configuración de la aplicación para usar el Bus de servicio
Para usar el Bus de servicio, descargue el paquete Node.js de Azure. Este paquete incluye un conjunto de bibliotecas que se comunican con los servicios REST del Bus de servicio.

### <a name="use-node-package-manager-(npm)-to-obtain-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para obtener el paquete
1. Utilice una interfaz de línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix) y vaya a la carpeta donde ha creado la aplicación de ejemplo.
2. Escriba **npm install azure** en la ventana de comandos. Esto debería devolver la salida siguiente:
   
   ```
       azure@0.7.5 node_modules\azure
   ├── dateformat@1.0.2-1.2.3
   ├── xmlbuilder@0.4.2
   ├── node-uuid@1.2.0
   ├── mime@1.2.9
   ├── underscore@1.4.4
   ├── validator@1.1.1
   ├── tunnel@0.0.2
   ├── wns@0.5.3
   ├── xml2js@0.2.7 (sax@0.5.2)
   └── request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)
   ```
3. Puede ejecutar manualmente el comando **ls** para comprobar si se ha creado la carpeta **node\_modules**. Dentro de dicha carpeta, busque el paquete **azure**, que contiene las bibliotecas necesarias para acceder a los temas de Service Bus.

### <a name="import-the-module"></a>Importación del módulo
Utilizando el Bloc de notas u otro editor de texto, agregue el código siguiente en la parte superior del archivo **server.js** de la aplicación:

```
var azure = require('azure');
```

### <a name="set-up-a-service-bus-connection"></a>Configuración de una conexión del Bus de servicio
El módulo Azure lee las variables de entorno AZURE\_SERVICEBUS\_NAMESPACE and AZURE\_SERVICEBUS\_ACCESS\_KEY para obtener la información necesaria para conectarse al espacio de nombres. Si no se configuran estas variables de entorno, debe especificar la información de la cuenta al llamar a **createServiceBusService**.

Para ver un ejemplo de cómo configurar las variables de entorno en un archivo de configuración para un servicio en la nube de Azure, consulte [Servicio de nube de Node.js con Storage][].

Para ver un ejemplo de configuración de las variables de entorno en el [Portal de Azure clásico][Portal de Azure clásico] para un sitio web de Azure, consulte [Aplicación web Node.js con Storage][].

## <a name="create-a-topic"></a>de un tema
El objeto **ServiceBusService** le permite trabajar con temas. El siguiente código crea un objeto **ServiceBusService**. Agréguelo cerca de la parte superior del archivo **server.js** , tras la instrucción para importar el módulo azure:

```
var serviceBusService = azure.createServiceBusService();
```

Al llamar a **createTopicIfNotExists** en el objeto **ServiceBusService**, se devuelve el tema especificado (si existe) o se crea un nuevo tema con el nombre especificado. El código siguiente utiliza **createTopicIfNotExists** para crear un tema llamado "MyTopic" o conectarse al tema con ese nombre:

```
serviceBusService.createTopicIfNotExists('MyTopic',function(error){
    if(!error){
        // Topic was created or exists
        console.log('topic created or exists.');
    }
});
```

**createServiceBusService** también admite opciones adicionales, lo que permite invalidar la configuración predeterminada de los temas, como el período de vida de los mensajes o el tamaño máximo de los temas. En el siguiente ejemplo se establece el tamaño máximo de los temas en 5 GB y el valor del período de vida en 1 minuto:

```
var topicOptions = {
        MaxSizeInMegabytes: '5120',
        DefaultMessageTimeToLive: 'PT1M'
    };

serviceBusService.createTopicIfNotExists('MyTopic', topicOptions, function(error){
    if(!error){
        // topic was created or exists
    }
});
```

### <a name="filters"></a>Filtros
Las operaciones de filtrado opcionales pueden aplicarse a las tareas realizadas utilizando **ServiceBusService**. Las operaciones de filtrado pueden incluir registros, reintentos automáticos, etc. Los filtros son objetos que implementan un método con la firma:

```
function handle (requestOptions, next)
```

Después de realizar el preprocesamiento en las opciones de solicitud, el método llama a `next` y pasa una devolución de llamada con la signatura siguiente:

```
function (returnObject, finalCallback, next)
```

En esta devolución de llamada y después de procesar **returnObject** (la respuesta de la solicitud al servidor), la devolución de llamada tiene que invocar a next, si existe, para continuar procesando otros filtros, o bien simplemente invocar a **finalCallback** para finalizar la invocación del servicio.

Se incluyen dos filtros que implementan la lógica de reintento con el SDK de Azure para Node.js: **ExponentialRetryPolicyFilter** y **LinearRetryPolicyFilter**. Con el siguiente código se crea un objeto **ServiceBusService** que utiliza el filtro **ExponentialRetryPolicyFilter**:

    var retryOperations = new azure.ExponentialRetryPolicyFilter();
    var serviceBusService = azure.createServiceBusService().withFilter(retryOperations);

## <a name="create-subscriptions"></a>Creación de suscripciones
Las suscripciones a temas también se crean con el objeto **ServiceBusService**. A las suscripciones se les asigna un nombre y pueden tener un filtro opcional que restrinja el conjunto de mensajes que se entregan a su cola virtual.

> [!NOTE]
> Las suscripciones son permanentes y seguirán existiendo hasta que se eliminen, o se elimine el tema al que están asociadas. Si una aplicación contiene lógica para crear una suscripción, en primer lugar debe comprobar si esta ya existe, para lo que se utiliza el método **getSubscription**.
> 
> 

### <a name="create-a-subscription-with-the-default-(matchall)-filter"></a>Creación de una suscripción con el filtro predeterminado (MatchAll)
El filtro predeterminado **MatchAll** se usa en caso de que no se haya especificado ninguno al crear una suscripción. Al usar el filtro **MatchAll**, todos los mensajes publicados en el tema se colocan en la cola virtual de la suscripción. En el ejemplo siguiente se crea una suscripción llamada "AllMessages" que usa el filtro predeterminado **MatchAll**.

```
serviceBusService.createSubscription('MyTopic','AllMessages',function(error){
    if(!error){
        // subscription created
    }
});
```

### <a name="create-subscriptions-with-filters"></a>Creación de suscripciones con filtros
También se pueden crear filtros que permitan especificar qué mensajes enviados a un tema deben aparecer dentro de una suscripción de tema determinada.

El tipo de filtro más flexible compatible con las suscripciones es **SqlFilter**, que implementa un subconjunto de SQL92. Los filtros de SQL operan en las propiedades de los mensajes que se publican en el tema. Para obtener más información acerca de las expresiones que se pueden usar con un filtro de SQL, revise la sintaxis de [SqlFilter.SqlExpression][SqlFilter.SqlExpression].

Es posible agregar filtros a una suscripción mediante el método **createRule** del objeto **ServiceBusService**. Este método permite agregar nuevos filtros a una suscripción existente.

> [!NOTE]
> Debido a que el filtro predeterminado se aplica automáticamente a todas las suscripciones nuevas, debe eliminar primero el filtro predeterminado, si no quiere que **MatchAll** invalide los demás filtros que especifique. Puede eliminar el filtro predeterminado utilizando el método **deleteRule** del objeto **ServiceBusService**.
> 
> 

En el ejemplo siguiente, se crea una suscripción denominada `HighMessages` con un objeto **SqlFilter** que solo selecciona los mensajes con una propiedad **messagenumber** personalizada cuyo valor sea mayor que 3:

```
serviceBusService.createSubscription('MyTopic', 'HighMessages', function (error){
    if(!error){
        // subscription created
        rule.create();
    }
});
var rule={
    deleteDefault: function(){
        serviceBusService.deleteRule('MyTopic',
            'HighMessages', 
            azure.Constants.ServiceBusConstants.DEFAULT_RULE_NAME, 
            rule.handleError);
    },
    create: function(){
        var ruleOptions = {
            sqlExpressionFilter: 'messagenumber > 3'
        };
        rule.deleteDefault();
        serviceBusService.createRule('MyTopic', 
            'HighMessages', 
            'HighMessageFilter', 
            ruleOptions, 
            rule.handleError);
    },
    handleError: function(error){
        if(error){
            console.log(error)
        }
    }
}
```

Del mismo modo, en el ejemplo que aparece a continuación, se crea una suscripción denominada `LowMessages` con un filtro **SqlFilter** que solo selecciona los mensajes cuyo valor de la propiedad **messagenumber** sea menor o igual a 3:

```
serviceBusService.createSubscription('MyTopic', 'LowMessages', function (error){
    if(!error){
        // subscription created
        rule.create();
    }
});
var rule={
    deleteDefault: function(){
        serviceBusService.deleteRule('MyTopic',
            'LowMessages', 
            azure.Constants.ServiceBusConstants.DEFAULT_RULE_NAME, 
            rule.handleError);
    },
    create: function(){
        var ruleOptions = {
            sqlExpressionFilter: 'messagenumber <= 3'
        };
        rule.deleteDefault();
        serviceBusService.createRule('MyTopic', 
            'LowMessages', 
            'LowMessageFilter', 
            ruleOptions, 
            rule.handleError);
    },
    handleError: function(error){
        if(error){
            console.log(error)
        }
    }
}
```

Cuando ahora se envía un mensaje a `MyTopic`, siempre se entregará a los destinatarios que tengan la suscripción al tema `AllMessages`, mientras que se entrega de forma selectiva a los que tengan suscripciones a los temas `HighMessages` y `LowMessages` (según el contenido del mensaje).

## <a name="how-to-send-messages-to-a-topic"></a>Envío de mensajes a un tema
Para enviar un mensaje a un tema de Service Bus, la aplicación debe utilizar el método **sendTopicMessage** del objeto **ServiceBusService**.
Los mensajes enviados a los temas de Service Bus son objetos **BrokeredMessage**.
Los objetos **BrokeredMessage** cuentan con un conjunto de propiedades estándar (como **Label** y **TimeToLive**), un diccionario que se usa para mantener las propiedades personalizadas específicas de la aplicación y un conjunto de datos de cadenas. Una aplicación puede establecer el cuerpo del mensaje pasando un valor de cadena al método **sendTopicMessage**, con lo que las propiedades estándar requeridas adquieren valores predeterminados.

En el siguiente ejemplo se demuestra cómo enviar cinco mensajes de prueba a “MyTopic”. Observe que el valor de la propiedad **messagenumber** de cada mensaje varía en función de la iteración del bucle (lo que determinará qué suscripciones lo reciben):

```
var message = {
    body: '',
    customProperties: {
        messagenumber: 0
    }
}

for (i = 0;i < 5;i++) {
    message.customProperties.messagenumber=i;
    message.body='This is Message #'+i;
    serviceBusService.sendTopicMessage(topic, message, function(error) {
      if (error) {
        console.log(error);
      }
    });
}
```

El tamaño máximo de mensaje que admiten los temas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que contiene un tema, pero hay un tope para el tamaño total de los mensajes contenidos en un tema. El tamaño de los temas se define en el momento de la creación (el límite máximo es de 5 GB).

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
Los mensajes se reciben de una suscripción mediante el método **receiveSubscriptionMessage** del objeto **ServiceBusService**. De manera predeterminada, los mensajes se eliminan de la suscripción cuando se leen; sin embargo, se pueden leer y bloquear sin eliminarlos de la suscripción. Para ello, es preciso establecer el parámetro opcional **isPeekLock** en **true**.

El funcionamiento predeterminado por el que los mensajes se eliminan tras leerlos como parte del proceso de recepción es el modelo más sencillo y el que mejor funciona en aquellas situaciones en las que una aplicación puede tolerar que no se procese un mensaje en caso de error. Para entenderlo mejor, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Como el Bus de servicio habrá marcado el mensaje como consumido, cuando la aplicación se reinicie y empiece a consumir mensajes de nuevo, habrá perdido el mensaje que se consumió antes del bloqueo.

Si el parámetro **isPeekLock** está establecido en **true**, el proceso de recepción se convierte en una operación en dos fases que hace posible admitir aplicaciones que no toleran la pérdida de mensajes. Cuando el Bus de servicio recibe una solicitud, busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación.
Una vez que la aplicación termina de procesar el mensaje (o lo almacena de forma fiable para su futuro procesamiento), completa la segunda fase del proceso de recepción llamando al método **deleteMessage** y facilitando el mensaje que se va a eliminar a modo de parámetro. El método **deleteMessage** marcará el mensaje como consumido y lo quitará de la suscripción.

En el ejemplo que aparece a continuación, se indica cómo se pueden recibir y procesar los mensajes usando **receiveSubscriptionMessage**. En el ejemplo, primero se recibe un mensaje y se elimina de la suscripción "LowMessages" y, luego, se recibe un mensaje de la suscripción "HighMessages" con el parámetro **isPeekLock** establecido en true. A continuación, se elimina el mensaje utilizando **deleteMessage**:

```
serviceBusService.receiveSubscriptionMessage('MyTopic', 'LowMessages', function(error, receivedMessage){
    if(!error){
        // Message received and deleted
        console.log(receivedMessage);
    }
});
serviceBusService.receiveSubscriptionMessage('MyTopic', 'HighMessages', { isPeekLock: true }, function(error, lockedMessage){
    if(!error){
        // Message received and locked
        console.log(lockedMessage);
        serviceBusService.deleteMessage(lockedMessage, function (deleteError){
            if(!deleteError){
                // Message deleted
                console.log('message has been deleted.');
            }
        }
    }
});
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Actuación ante errores de la aplicación y mensajes que no se pueden leer
El Bus de servicio proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si por cualquier motivo una aplicación de recepción es incapaz de procesar el mensaje, entonces puede llamar al método **unlockMessage** del objeto **ServiceBusService**. Esto hará que Service Bus desbloquee el mensaje de la suscripción y esté disponible para que pueda volver a recibirse, ya sea por la misma aplicación que lo consume o por otra.

También hay otro tiempo de expiración asociado a un mensaje bloqueado en la suscripción y, si la aplicación no puede procesar el mensaje antes de que expire el tiempo de espera del bloqueo (por ejemplo, si la aplicación se bloquea), Service Bus desbloquea el mensaje automáticamente y hace que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación sufra un error después de procesar el mensaje y antes de llamar al método **deleteMessage**, entonces el mensaje se volverá a entregar a la aplicación cuando esta se reinicie. Habitualmente se denomina **Al menos un procesamiento**, es decir, cada mensaje se procesará al menos una vez; aunque en determinadas situaciones podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, entonces los desarrolladores de la aplicación deberían agregar lógica adicional a su aplicación para solucionar la entrega de mensajes duplicados. A menudo, esto se consigue usando la propiedad **MessageId** del mensaje, que permanecerá constante en todos los intentos de entrega.

## <a name="delete-topics-and-subscriptions"></a>Eliminación de temas y suscripciones
Los temas y las suscripciones son permanentes, por lo que deben eliminarse explícitamente a través del [Portal de Azure clásico][Portal de Azure clásico] o mediante programación.
En el ejemplo siguiente se muestra cómo eliminar el tema denominado `MyTopic`:

    serviceBusService.deleteTopic('MyTopic', function (error) {
        if (error) {
            console.log(error);
        }
    });

Al eliminar un tema también se eliminan todas las suscripciones que estén registradas con él. También se pueden eliminar las suscripciones de forma independiente. El ejemplo siguiente muestra cómo eliminar una suscripción denominada `HighMessages` del tema `MyTopic`:

    serviceBusService.deleteSubscription('MyTopic', 'HighMessages', function (error) {
        if(error) {
            console.log(error);
        }
    });

## <a name="next-steps"></a>Pasos siguientes
Ahora que conoce los fundamentos de los temas del Bus de servicio, siga estos vínculos para obtener más información.

* Vea [Colas, temas y suscripciones][Colas, temas y suscripciones].
* Referencia de API para [SqlFilter][SqlFilter].
* Visite el repositorio del [SDK de Azure para Node][SDK de Azure para Node] en GitHub.

[SDK de Azure para Node]: https://github.com/Azure/azure-sdk-for-node
[Portal de Azure clásico]: https://manage.windowsazure.com
[SqlFilter.SqlExpression]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx
[Colas, temas y suscripciones]: service-bus-queues-topics-subscriptions.md
[SqlFilter]: http://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.sqlfilter.aspx
[Servicio en la nube de Node.js]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Creación e implementación de una aplicación Node.js en un sitio web de Azure]: ../app-service-web/web-sites-nodejs-develop-deploy-mac.md
[Servicio en la nube de Node.js con Storage]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Aplicación web de Node.js con Storage]: ../cloud-services/storage-nodejs-use-table-storage-cloud-service-app.md




<!--HONumber=Oct16_HO2-->


