---
title: Soluciones de Azure para EL Internet de las cosas | Microsoft Docs
description: "Descripción general de IoT en Azure que incluye la arquitectura de una solución de ejemplo y cómo se relaciona con el Centro de IoT de Azure, los SDK de dispositivos y las soluciones preconfiguradas"
services: iot-hub
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: a859e379-dca7-42fa-bdf6-1125c86ad140
ms.service: iot-hub
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2016
ms.author: dobett
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 2dbf639abfa505eb329769bcc346efb5f1db443e


---
[!INCLUDE [iot-azure-and-iot](../../includes/iot-azure-and-iot.md)]

## <a name="next-steps"></a>Pasos siguientes
El Centro de IoT de Azure es un servicio de Azure que permite la comunicación bidireccional fiable y segura entre el back-end de la aplicación y millones de dispositivos. Habilita al back-end de aplicación para:

* Recibir telemetría a escala de los dispositivos.
* Distribuir datos de los dispositivos a un procesador de eventos de transmisión.
* Recibir cargas de archivos de los dispositivos.
* Enviar comandos de nube a dispositivo a dispositivos concretos.

Puede usar el Centro de IoT para implementar su propio back-end de soluciones. Además, el Centro de IoT incluye un registro de identidades de dispositivo que se usa para aprovisionar dispositivos, sus credenciales de seguridad y sus derechos para conectarse al Centro. Para más información acerca de IoT Hub, vea [¿Qué es IoT Hub?][lnk-iot-hub].

Para más información acerca de cómo puede administrar, configurar y actualizar sus dispositivos de forma remota mediante la administración de dispositivos basada en estándares de Azure IoT Hub, consulte [Introducción a la administración de dispositivos con IoT Hub][lnk-device-management].

Para implementar aplicaciones cliente que se ejecuten en una gran variedad de plataformas de hardware de dispositivos y sistemas operativos, puede usar los SDK de dispositivos IoT. Los SDK de dispositivos IoT incluyen bibliotecas que facilitan el envío de telemetría a un Centro de IoT y la recepción de comandos de nube a dispositivo. Al usar los SDK, puede elegir entre varios protocolos de red para comunicarse con IoT Hub. Para más información, consulte la [información acerca de los SDK de los dispositivos][lnk-device-sdks].

Para comenzar a escribir código y ejecutar algunos ejemplos, consulte el tutorial [Introducción a IoT Hub][lnk-getstarted].

También puede interesarle el [Conjunto de aplicaciones de IoT de Azure][lnk-iot-suite], que es una colección de soluciones preconfiguradas. El Conjunto de aplicaciones de IoT le permite comenzar rápidamente y escalar proyectos IoT en escenarios comunes de IoT como, por ejemplo, supervisión remota, administración de activos y mantenimiento predictivo.

[lnk-getstarted]: iot-hub-csharp-csharp-getstarted.md
[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-iot-hub]: iot-hub-what-is-iot-hub.md
[lnk-iot-suite]: https://azure.microsoft.com/documentation/suites/iot-suite/
[lnk-iotdev]: https://azure.microsoft.com/develop/iot/
[lnk-device-management]: iot-hub-device-management-overview.md



<!--HONumber=Nov16_HO2-->


