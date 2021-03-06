---
title: "Introducción a la creación de un equilibrador de carga orientado a Internet en un modelo de implementación clásica con la CLI de Azure | Microsoft Docs"
description: "Obtener información sobre cómo crear un equilibrador de carga orientado a Internet en el modelo de implementación clásica con la CLI de Azure"
services: load-balancer
documentationcenter: na
author: sdwheeler
manager: carmonm
editor: 
tags: azure-service-management
ms.assetid: e433a824-4a8a-44d2-8765-a74f52d4e584
ms.service: load-balancer
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/09/2016
ms.author: sewhee
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: e8a346c1b2d430eceb4aa1b8bc94fbbe89394556


---
# <a name="get-started-creating-an-internet-facing-load-balancer-classic-in-the-azure-cli"></a>Introducción a la creación de un equilibrador de carga orientado a Internet (clásico) en la CLI de Azure
[!INCLUDE [load-balancer-get-started-internet-classic-selectors-include.md](../../includes/load-balancer-get-started-internet-classic-selectors-include.md)]

[!INCLUDE [load-balancer-get-started-internet-intro-include.md](../../includes/load-balancer-get-started-internet-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

Este artículo trata sobre el modelo de implementación clásico. También puede [obtener información sobre cómo crear un equilibrador de carga orientado a Internet con el Administrador de recursos de Azure](load-balancer-get-started-internet-arm-ps.md).

[!INCLUDE [load-balancer-get-started-internet-scenario-include.md](../../includes/load-balancer-get-started-internet-scenario-include.md)]

## <a name="step-by-step-creating-an-internet-facing-load-balancer-using-cli"></a>Creación paso a paso de un equilibrador de carga orientado a Internet con la CLI
Esta guía muestra cómo crear un equilibrador de carga de Internet basado en el escenario anterior.

1. Si nunca ha usado la CLI de Azure, consulte [Instalación y configuración de la CLI de Azure](../xplat-cli-install.md) y siga las instrucciones hasta el punto donde deba seleccionar su cuenta y suscripción de Azure.
2. Ejecute el comando **azure config mode** para cambiar al modo clásico, como se muestra a continuación.
   
        azure config mode asm
   
    Resultado esperado:
   
        info:    New mode is asm

## <a name="create-endpoint-and-load-balancer-set"></a>Crear punto de conexión y conjunto de equilibrador de carga
En esta situación se supone que se han creado las máquinas virtuales "web1" y "web2".
Esta guía creará un conjunto de equilibrador de carga mediante el puerto 80 como puerto público y el puerto 80 como puerto local. También se configura un puerto de sondeo en el puerto 80 y se llama al conjunto de equilibrador de carga "lbset".

### <a name="step-1"></a>Paso 1
Crear el primer punto de conexión y conjunto de equilibrador de carga mediante `azure network vm endpoint create` para la máquina virtual "web1"

    azure vm endpoint create web1 80 -k 80 -o tcp -t 80 -b lbset

Parámetros usados:

**-k**: puerto de la máquina virtual local<br>
**-o**: protocolo<BR>
**-t**: puerto de sondeo<BR>
**-b**: nombre de equilibrador de carga<BR>

## <a name="step-2"></a>Paso 2
Agregar una segunda máquina virtual "web2" al conjunto de equilibrador de carga

    azure vm endpoint create web2 80 -k 80 -o tcp -t 80 -b lbset

## <a name="step-3"></a>Paso 3
Comprobar la configuración del equilibrador de carga mediante `azure vm show`

    azure vm show web1

El resultado será:

    data:    DNSName "contoso.cloudapp.net"
    data:    Location "East US"
    data:    VMName "web1"
    data:    IPAddress "10.0.0.5"
    data:    InstanceStatus "ReadyRole"
    data:    InstanceSize "Standard_D1"
    data:    Image "a699494373c04fc0bc8f2bb1389d6106__Windows-Server-2012-R2-2015
    6-en.us-127GB.vhd"
    data:    OSDisk hostCaching "ReadWrite"
    data:    OSDisk name "joaoma-1-web1-0-201509251804250879"
    data:    OSDisk mediaLink "https://XXXXXXXXXXXXXXX.blob.core.windows.
    /vhds/joaomatest-web1-2015-09-25.vhd"
    data:    OSDisk sourceImageName "a699494373c04fc0bc8f2bb1389d6106__Windows-Se
    r-2012-R2-20150916-en.us-127GB.vhd"
    data:    OSDisk operatingSystem "Windows"
    data:    OSDisk iOType "Standard"
    data:    ReservedIPName ""
    data:    VirtualIPAddresses 0 address "XXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 name "XXXXXXXXXXXXXXXXXXXX"
    data:    VirtualIPAddresses 0 isDnsProgrammed true
    data:    Network Endpoints 0 loadBalancedEndpointSetName "lbset"
    data:    Network Endpoints 0 localPort 80
    data:    Network Endpoints 0 name "tcp-80-80"
    data:    Network Endpoints 0 port 80
    data:    Network Endpoints 0 loadBalancerProbe port 80
    data:    Network Endpoints 0 loadBalancerProbe protocol "tcp"
    data:    Network Endpoints 0 loadBalancerProbe intervalInSeconds 15
    data:    Network Endpoints 0 loadBalancerProbe timeoutInSeconds 31
    data:    Network Endpoints 0 protocol "tcp"
    data:    Network Endpoints 0 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 0 enableDirectServerReturn false
    data:    Network Endpoints 1 localPort 5986
    data:    Network Endpoints 1 name "PowerShell"
    data:    Network Endpoints 1 port 5986
    data:    Network Endpoints 1 protocol "tcp"
    data:    Network Endpoints 1 virtualIPAddress "XXXXXXXXXXXX"
    data:    Network Endpoints 1 enableDirectServerReturn false
    data:    Network Endpoints 2 localPort 3389
    data:    Network Endpoints 2 name "Remote Desktop"
    data:    Network Endpoints 2 port 58081
    info:    vm show command OK

## <a name="create-a-remote-desktop-endpoint-for-a-virtual-machine"></a>Crear un punto de conexión de escritorio remoto para una máquina virtual
Puede crear un punto de conexión de escritorio remoto para reenviar el tráfico de red desde un puerto público a un puerto local para una máquina virtual específica mediante `azure vm endpoint create`.

    azure vm endpoint create web1 54580 -k 3389


## <a name="remove-virtual-machine-from-load-balancer"></a>Quitar máquina virtual del equilibrador de carga
Tendrá que eliminar el punto de conexión asociado al conjunto de equilibrador de carga desde la máquina virtual. Una vez eliminado el punto de conexión, la máquina virtual deja de pertenecer al conjunto de equilibrador de carga.

 En el ejemplo anterior, puede quitar el punto de conexión creado para la máquina virtual "web1" del equilibrador de carga "lbset" mediante el comando `azure vm endpoint delete`.

    azure vm endpoint delete web1 tcp-80-80


> [!NOTE]
> Puede explorar más opciones para administrar los puntos de conexión con el comando `azure vm endpoint --help`.
> 
> 

## <a name="next-steps"></a>Pasos siguientes
[Introducción a la configuración de un equilibrador de carga interno](load-balancer-get-started-ilb-arm-ps.md)

[Configuración de un modo de distribución del equilibrador de carga](load-balancer-distribution-mode.md)

[Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)




<!--HONumber=Nov16_HO2-->


