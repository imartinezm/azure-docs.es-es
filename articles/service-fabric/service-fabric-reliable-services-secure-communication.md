---
title: Ayuda para garantizar la comunicación de servicios de Service Fabric | Microsoft Docs
description: Información general sobre cómo ayudar a proteger la comunicación de los servicios de confianza que se ejecutan en el clúster de Azure Service Fabric.
services: service-fabric
documentationcenter: .net
author: suchiagicha
manager: timlt
editor: vturecek

ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 07/06/2016
ms.author: suchiagicha

---
# Ayuda para garantizar la comunicación de los servicios de Azure Service Fabric
La seguridad es uno de los aspectos más importantes de la comunicación. El marco de trabajo de la aplicación de Reliable Services proporciona una serie de pilas de comunicación creadas previamente y herramientas que puede utilizar para mejorar la seguridad. En este artículo hablaremos sobre cómo mejorar la seguridad cuando se utiliza la pila de comunicación de Windows Communication Foundation (WCF) y de comunicación remota para los servicios.

## Ayuda para garantizar un servicio cuando usa la comunicación remota para los servicios
Vamos a usar un [ejemplo](service-fabric-reliable-services-communication-remoting.md) existente que explica cómo configurar la comunicación remota para servicios de confianza Para ayudar a garantizar un servicio cuando usa la comunicación remota para los servicios, siga estos pasos:

1. Cree una interfaz,`IHelloWorldStateful`, que defina los métodos que estarán disponibles para la llamada a procedimiento remoto en su servicio. El servicio usará `FabricTransportServiceRemotingListener`, que se declara en el espacio de nombres `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime`. Se trata de una implementación de `ICommunicationListener` que ofrece capacidades de comunicación remota.
   
    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }
   
    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }
   
        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. Agregue la configuración de escucha y las credenciales de seguridad.
   
    Asegúrese de que el certificado que desea utilizar para ayudar a proteger la comunicación de servicio esté instalado en todos los nodos del clúster. Hay dos formas de proporcionar la configuración de escucha y las credenciales de seguridad:
   
   1. Proporcionarlas directamente en el código de servicio:
      
       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportListenerSettings listenerSettings = new FabricTransportListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }
      
       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           return x509Credentials;
       }
       ```
   2. Proporcionarlas utilizando un [paquete de configuración](service-fabric-application-model.md):
      
       Agregue una sección `TransportSettings` en el archivo settings.xml.
      
       ```xml
       <!--Section name should always end with "TransportSettings".-->
       <!--Here we are using a prefix "HelloWorldStateful".-->
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```
      
       En este caso, el método `CreateServiceReplicaListeners` tendrá este aspecto.
      
       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportListenerSettings.LoadFrom("HelloWorldStateful")))
           };
       }
       ```
      
        Si agrega una sección `TransportSettings` en el archivo settings.xml sin ningún prefijo, `FabricTransportListenerSettings` cargará toda la configuración de esta sección de forma predeterminada.
      
        ```xml
        <!--"TransportSettings" section without any prefix.-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        En este caso, el método `CreateServiceReplicaListeners` tendrá este aspecto.
      
        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. Al invocar métodos en un servicio protegido con una pila de comunicación remota, en lugar de utilizar la clase `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` para crear un proxy de servicio, utilice `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`. Pase `FabricTransportSettings`, que contiene `SecurityCredentials`.
   
    ```csharp
   
    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
   
    FabricTransportSettings transportSettings = new FabricTransportSettings
    {
        SecurityCredentials = x509Credentials,
    };
   
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));
   
    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));
   
    string message = await client.GetHelloWorld();
   
    ```
   
    Si el código de cliente se ejecuta como parte de un servicio, puede cargar la clase `FabricTransportSettings` desde el archivo settings.xml. Cree una sección TransportSettings similar al código de servicio, tal y como se mostró anteriormente. Realice los siguientes cambios en el código de cliente:
   
    ```csharp
   
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportSettings.LoadFrom("TransportSettingsPrefix")));
   
    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));
   
    string message = await client.GetHelloWorld();
   
    ```
   
    Si el cliente no se está ejecutando como parte de un servicio, puede crear un archivo client\_name.settings.xml en la misma ubicación donde se encuentra client\_name.exe. A continuación, cree una sección TransportSettings en ese archivo.
   
    De forma similar al servicio, si agrega una sección `TransportSettings` sin ningún prefijo en el archivo settings.xml/client\_name.settings.xml del cliente, `FabricTransportSettings` cargará toda la configuración desde esta sección de forma predeterminada.
   
    En ese caso, el código anterior se simplifica más aún:
   
    ```csharp
   
    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));
   
    string message = await client.GetHelloWorld();
   
    ```

## Ayuda para garantizar un servicio cuando se utiliza una pila de comunicación basada en WCF
Vamos a usar un [ejemplo](service-fabric-reliable-services-communication-wcf.md) existente que explica cómo configurar una pila de comunicación basada en WCF para servicios de confianza. Para ayudar a garantizar un servicio cuando se utiliza una pila de comunicación basada en WCF, siga estos pasos:

1. Para el servicio, debe ayudar a proteger el agente de escucha de comunicación WCF (`WcfCommunicationListener`) que cree. Para ello, modifique el método `CreateServiceReplicaListeners`.
   
    ```csharp
    protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
    {
        return new[]
        {
            new ServiceReplicaListener(
                this.CreateWcfCommunicationListener)
        };
    }
   
    private WcfCommunicationListener<ICalculator> CreateWcfCommunicationListener(StatefulServiceContext context)
    {
       var wcfCommunicationListener = new WcfCommunicationListener<ICalculator>(
            serviceContext:context,
            wcfServiceObject:this,
            // For this example, we will be using NetTcpBinding.
            listenerBinding: GetNetTcpBinding(),
            endpointResourceName:"WcfServiceEndpoint");
   
        // Add certificate details in the ServiceHost credentials.
        wcfCommunicationListener.ServiceHost.Credentials.ServiceCertificate.SetCertificate(
            StoreLocation.LocalMachine,
            StoreName.My,
            X509FindType.FindByThumbprint,
            "9DC906B169DC4FAFFD1697AC781E806790749D2F");
        return wcfCommunicationListener;
    }
   
    private static NetTcpBinding GetNetTcpBinding()
    {
        NetTcpBinding b = new NetTcpBinding(SecurityMode.TransportWithMessageCredential);
        b.Security.Message.ClientCredentialType = MessageCredentialType.Certificate;
        return b;
    }
    ```
2. En el cliente, la clase `WcfCommunicationClient` que hemos creado en el [ejemplo](service-fabric-reliable-services-communication-wcf.md) anterior no se modifica. Sin embargo, tendremos que reemplazar el método `CreateClientAsync` de `WcfCommunicationClientFactory`:
   
    ```csharp
    public class SecureWcfCommunicationClientFactory<TServiceContract> : WcfCommunicationClientFactory<TServiceContract> where TServiceContract : class
    {
        private readonly Binding clientBinding;
        private readonly object callbackObject;
        public SecureWcfCommunicationClientFactory(
            Binding clientBinding,
            IEnumerable<IExceptionHandler> exceptionHandlers = null,
            IServicePartitionResolver servicePartitionResolver = null,
            string traceId = null,
            object callback = null)
            : base(clientBinding, exceptionHandlers, servicePartitionResolver,traceId,callback)
        {
            this.clientBinding = clientBinding;
            this.callbackObject = callback;
        }
   
        protected override Task<WcfCommunicationClient<TServiceContract>> CreateClientAsync(string endpoint, CancellationToken cancellationToken)
        {
            var endpointAddress = new EndpointAddress(new Uri(endpoint));
            ChannelFactory<TServiceContract> channelFactory;
            if (this.callbackObject != null)
            {
                channelFactory = new DuplexChannelFactory<TServiceContract>(
                this.callbackObject,
                this.clientBinding,
                endpointAddress);
            }
            else
            {
                channelFactory = new ChannelFactory<TServiceContract>(this.clientBinding, endpointAddress);
            }
            // Add certificate details to the ChannelFactory credentials.
            // These credentials will be used by the clients created by
            // SecureWcfCommunicationClientFactory.  
            channelFactory.Credentials.ClientCertificate.SetCertificate(
                StoreLocation.LocalMachine,
                StoreName.My,
                X509FindType.FindByThumbprint,
                "9DC906B169DC4FAFFD1697AC781E806790749D2F");
            var channel = channelFactory.CreateChannel();
            var clientChannel = ((IClientChannel)channel);
            clientChannel.OperationTimeout = this.clientBinding.ReceiveTimeout;
            return Task.FromResult(this.CreateWcfCommunicationClient(channel));
        }
    }
    ```
   
    Use `SecureWcfCommunicationClientFactory` para crear un cliente de comunicación WCF (`WcfCommunicationClient`). Use al cliente para invocar métodos de servicio.
   
    ```csharp
    IServicePartitionResolver partitionResolver = ServicePartitionResolver.GetDefault();
   
    var wcfClientFactory = new SecureWcfCommunicationClientFactory<ICalculator>(clientBinding: GetNetTcpBinding(), servicePartitionResolver: partitionResolver);
   
    var calculatorServiceCommunicationClient =  new WcfCommunicationClient(
        wcfClientFactory,
        ServiceUri,
        ServicePartitionKey.Singleton);
   
    var result = calculatorServiceCommunicationClient.InvokeWithRetryAsync(
        client => client.Channel.Add(2, 3)).Result;
    ```

## Pasos siguientes
* [Web API con OWIN en Reliable Services](service-fabric-reliable-services-communication-webapi.md)

<!---HONumber=AcomDC_0706_2016-->