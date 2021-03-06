---
title: "Conexión a SQL Database - SQL Server Management Studio | Microsoft Docs"
description: "Aprenda a conectarse a Base de datos SQL en Azure con SQL Server Management Studio (SSMS). Después, ejecute una consulta de ejemplo con Transact-SQL (T-SQL)."
metacanonical: 
keywords: "conexión a base de datos sql,sql server management studio"
services: sql-database
documentationcenter: 
author: stevestein
manager: jhubbard
editor: 
ms.assetid: 7cd2a114-c13c-4ace-9088-97bd9d68de12
ms.service: sql-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/17/2016
ms.author: sstein;carlrab
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 0eb25eb76c6c6c2446ac0b2b07c65975c3719db0


---
# <a name="connect-to-sql-database-with-sql-server-management-studio-and-execute-a-sample-tsql-query"></a>Conexión a Base de datos SQL con SQL Server Management Studio y ejecución de una consulta T-SQL de ejemplo
> [!div class="op_single_selector"]
> * [Visual Studio](sql-database-connect-query.md)
> * [SSMS](sql-database-connect-query-ssms.md)
> * [Excel](sql-database-connect-excel.md)
> 
> 

Este artículo explica cómo conectarse a una Base de datos SQL de Azure con SQL Server Management Studio (SSMS). Después de conectarse correctamente, ejecutamos una consulta de Transact-SQL (T-SQL) simple para comprobar la comunicación con la base de datos.

[!INCLUDE [SSMS Install](../../includes/sql-server-management-studio-install.md)]

[!INCLUDE [SSMS Connect](../../includes/sql-database-sql-server-management-studio-connect-server-principal.md)]

## <a name="run-sample-queries"></a>Ejecutar consultas de ejemplo
Después de conectarse al servidor, puede conectarse a una base de datos y ejecutar una consulta de ejemplo. Si no está familiarizado con la escritura de consultas, consulte [Escribir instrucciones Transact-SQL](https://msdn.microsoft.com/library/ms365303.aspx).

1. En el **Explorador de objetos**, vaya a una base de datos del servidor como la base de datos de ejemplo **AdventureWorks**.
2. Haga clic con el botón derecho en la base de datos y seleccione **Nueva consulta**:
   
    ![Nueva consulta. Conexión a servidor de Base de datos SQL: SQL Server Management Studio](./media/sql-database-connect-query-ssms/4-run-query.png)
3. En la ventana de consulta, copie y pegue lo siguiente:
   
        SELECT
        CustomerId
        ,Title
        ,FirstName
        ,LastName
        ,CompanyName
        FROM SalesLT.Customer;
4. Haga clic en el botón **Ejecutar** :
   
    ![Correcta. Conexión a servidor de Base de datos SQL: SQL Server Management Studio](./media/sql-database-connect-query-ssms/5-success.png)

## <a name="next-steps"></a>Pasos siguientes
Puede usar instrucciones T-SQL para crear y administrar bases de datos en Azure casi de la misma manera que con SQL Server. Si está familiarizado con el uso de T-SQL con SQL Server, consulte [Información sobre Transact-SQL de Base de datos SQL de Azure](sql-database-transact-sql-information.md) para obtener un resumen de las diferencias.

Si no está familiarizado con T-SQL, consulte [Tutorial: Escribir instrucciones Transact-SQL](https://msdn.microsoft.com/library/ms365303.aspx) y [Referencia de Transact-SQL (motor de base de datos)](https://msdn.microsoft.com/library/bb510741.aspx).

Para comenzar con la creación de usuarios de base de datos y administradores de usuarios de base de datos, consulte [Introducción a la seguridad de Base de datos SQL de Azure](sql-database-get-started-security.md)

Para más información acerca de SSMS, consulte [Usar SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx).




<!--HONumber=Nov16_HO2-->


