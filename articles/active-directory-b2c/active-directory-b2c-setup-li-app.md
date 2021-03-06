---
title: 'Azure Active Directory B2C: configuración de LinkedIn | Microsoft Docs'
description: Proporcionar a los consumidores registro e inicio de sesión con cuentas de LinkedIn en las aplicaciones protegidas por Azure Active Directory B2C
services: active-directory-b2c
documentationcenter: ''
author: swkrish
manager: msmbaldwin
editor: bryanla

ms.service: active-directory-b2c
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2016
ms.author: swkrish

---
# Azure Active Directory B2C: provisión de registro e inicio de sesión a los consumidores con cuentas de LinkedIn
## Creación de una aplicación de LinkedIn
Para usar LinkedIn como proveedor de identidades en Azure Active Directory (Azure AD) B2C, primero debe crear una aplicación de LinkedIn y suministrarle los parámetros correctos. Necesita una cuenta de LinkedIn para ello. Si no tiene una, puede obtenerla en [https://www.linkedin.com/](https://www.linkedin.com/).

1. Vaya al [sitio web para desarrolladores de LinkedIn](https://www.developer.linkedin.com/) e inicie sesión con las credenciales de su cuenta de LinkedIn.
2. Haga clic en **My Apps** (Mis aplicaciones) en la barra de menús superior y, luego, en **Create Application** (Crear aplicación).
   
    ![LinkedIn - nueva aplicación](./media/active-directory-b2c-setup-li-app/linkedin-new-app.png)
3. En el formulario **Create a New Application** (Crea una nueva aplicación), rellene la información relevante (**empresa**, **nombre**, **descripción**, **URL del logo de la aplicación**, **uso de la aplicación**, **URL del sitio web**, **correo electrónico de empresa** y **teléfono de empresa**).
4. Acepte las **condiciones de uso de API de LinkedIn** y haga clic en **Submit** (Enviar).
   
    ![LinkedIn - registrar aplicación](./media/active-directory-b2c-setup-li-app/linkedin-register-app.png)
5. Copie los valores **Client ID** (Id. de cliente) y **Client Secret** (Secreto de cliente). [Los encontrará en **Authentication Keys** (Claves de autenticación)]. Necesitará ambos para configurar LinkedIn como proveedor de identidades de su inquilino.
   
   > [!NOTE]
   > El **secreto de cliente** es una credencial de seguridad importante.
   > 
   > 
6. Escriba `https://login.microsoftonline.com/te/{tenant}/oauth2/authresp` en el campo **Authorized Redirect URLs** (Direcciones URL de redirección autorizadas) (en **OAuth 2.0**). Reemplace **{tenant}** por el nombre de su inquilino (por ejemplo, contoso.onmicrosoft.com). Haga clic en **Add** (Agregar) y después en **Update** (Actualizar). El valor de **{tenant}** distingue mayúsculas de minúsculas.
   
    ![LinkedIn - configurar aplicación](./media/active-directory-b2c-setup-li-app/linkedin-setup.png)

## Configuración de LinkedIn como proveedor de identidades del inquilino
1. Siga estos pasos para [desplazarse hasta la hoja de características B2C](active-directory-b2c-app-registration.md#navigate-to-the-b2c-features-blade) en el Portal de Azure.
2. En la hoja de características B2C, haga clic en **Proveedores de identidades**.
3. Haga clic en **+Agregar** en la parte superior de la hoja.
4. Proporcione un **Nombre** descriptivo para la configuración del proveedor de identidades. Por ejemplo "LI".
5. Haga clic en el **Identity provider type** (Tipo de proveedor de identidades), seleccione **LinkedIn** y haga clic en **Aceptar**.
6. Haga clic en **Set up this identity provider** (Configurar este proveedor de identidades) y escriba el identificador de cliente y el secreto de cliente de la aplicación de LinkedIn que creó anteriormente.
7. Haga clic en **Aceptar** y después en **Crear** para guardar la configuración de LinkedIn.

<!---HONumber=AcomDC_0727_2016-->