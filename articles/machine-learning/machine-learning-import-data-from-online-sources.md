---
title: Importación de datos al Estudio de aprendizaje automático desde orígenes de datos en línea | Microsoft Docs
description: Cómo importar datos de entrenamiento al Estudio de aprendizaje automático de Azure desde varios orígenes en línea.
keywords: importar datos, formato de datos, tipos de datos, orígenes de datos, datos de entrenamiento
services: machine-learning
documentationcenter: ''
author: bradsev
manager: jhubbard
editor: cgronlun

ms.service: machine-learning
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/19/2016
ms.author: bradsev;garye

---
# Importación de datos al Estudio de aprendizaje automático de Azure desde varios orígenes de datos en línea con el módulo Importar datos
En este artículo se describe la compatibilidad con la importación de datos en línea desde varios orígenes y la información necesaria para mover los datos desde estos orígenes a un experimento de Aprendizaje automático de Azure.

> [!NOTE]
> En este artículo se proporciona información general sobre el módulo [Importar datos][import-data]. Para obtener más información sobre los tipos de datos a los que puede acceder, los formatos, los parámetros y las respuestas a las preguntas más comunes, consulte el tema de referencia del módulo [Importar datos][import-data].
> 
> 

<!-- -->

[!INCLUDE [import-data-into-aml-studio-selector](../../includes/machine-learning-import-data-into-aml-studio.md)]

## Introducción
Puede acceder a los datos del Estudio de aprendizaje automático de Azure desde cualquiera de los orígenes de datos en línea mientras su experimento se ejecuta con el módulo [Importar datos][import-data]\:

* Una dirección URL web con HTTP
* Hadoop con HiveQL
* Almacenamiento de blobs de Azure
* Tabla de Azure
* Base de datos SQL de Azure o SQL Server en máquina virtual de Azure
* La base de datos de SQL Server local
* Un proveedor de fuentes de datos, actualmente OData

El flujo de trabajo para llevar a cabo experimentos en el Estudio de aprendizaje automático de Azure consta de componentes para arrastrar y colocar en el lienzo. Para acceder a orígenes de datos en línea, agregue el módulo [Importar datos][import-data] al experimento, seleccione **Origen de datos** y, después, especifique los parámetros necesarios para acceder a los datos. Los orígenes de datos en línea admitidos se muestran en la tabla siguiente. Esta tabla también resume los formatos de archivo admitidos y los parámetros usados para tener acceso a los datos.

Tenga en cuenta que, como a estos datos de entrenamiento se tiene acceso mientras se está ejecutando el experimento, solo están disponibles en ese experimento. En comparación, los datos que estén almacenados en módulos de conjunto de datos se encontrarán disponibles para cualquier experimento que se realice en su área de trabajo.

> [!IMPORTANT]
> En estos momentos, los módulos [Importar datos][import-data] y [Exportar datos][export-data] solo pueden leer y escribir datos del almacenamiento de Azure que se creó utilizando el modelo de implementación clásico. Es decir, todavía no se admite el nuevo tipo de cuenta de Almacenamiento de blobs de Azure que ofrece un nivel de acceso frecuente o esporádico al almacenamiento.
> 
> Por lo general, las cuentas de almacenamiento de Azure que podría haber creado antes de que esta opción de servicio estuviera disponible no deberían verse afectadas. Si tiene que crear una cuenta nueva, seleccione **Clásico** en Modelo de implementación, o bien utilice Azure Resource Manager. En **Tipo de cuenta**, seleccione **Uso general** en lugar de **Almacenamiento de blobs**.
> 
> Para obtener más información, consulte [Capas de almacenamiento de acceso frecuente y acceso esporádico](../storage/storage-blob-storage-tiers.md).
> 
> 

## Orígenes de datos en línea admitidos
El módulo **Importar datos** de Aprendizaje automático Azure admite los siguientes orígenes de datos:

| Origen de datos | Description | Parámetros |
| --- | --- | --- |
| Dirección URL web a través de HTTP |Lee datos en valores separados por comas (CSV), valores separados por tabulaciones (TSV), formato de archivo de atributo-relación (ARFF) y formatos de máquinas de vectores de soporte (SVM-light), desde cualquier dirección URL web que use HTTP |<b>URL</b>: especifica el nombre completo del archivo, incluidos la dirección URL del sitio y el nombre de archivo, con cualquier extensión. <br/><br/><b>Formato de datos</b>: especifica uno de los formatos de datos compatibles: CSV, TSV, ARFF o SVM-light. Si los datos tienen una fila de encabezado, esta se usa para asignar nombres de columna. |
| Hadoop/HDFS |Lee datos de almacenamiento distribuido de Hadoop. Especifique los datos que desee mediante HiveQL, un lenguaje de consulta similar a SQL. HiveQL también se puede usar para agregar datos y realizar un filtrado de datos antes de agregarlos al Estudio de aprendizaje automático. |<b>Consulta de base de datos de Hive</b>: especifica la consulta de Hive que se usa para generar los datos.<br/><br/><b>URI del servidor de HCatalog </b>: especifica el nombre del clúster con el formato *&lt;nombre del clúster&gt;.azurehdinsight.net.*<br/><br/><b>Nombre de cuenta de usuario de Hadoop</b>: especifica el nombre de cuenta de usuario de Hadoop que se usa para aprovisionar el clúster.<br/><br/><b>Contraseña de cuenta de usuario de Hadoop</b>: especifica las credenciales que se usan durante el aprovisionamiento del clúster. Para obtener más información, consulte [Creación de clústeres de Hadoop en HDInsight](../hdinsight/hdinsight-provision-clusters.md).<br/><br/><b>Ubicación de los datos de salida</b>: especifica si los datos se almacenan en un sistema de archivos distribuido de Hadoop (HDFS) o en Azure. <br/><ul>Si almacena datos de salida en HDFS, especifique el URI del servidor HDFS. (Asegúrese de usar el nombre del clúster de HDInsight sin el prefijo HTTPS://). <br/><br/>Si almacena los datos de salida en Azure, debe especificar el nombre de cuenta del Almacenamiento de Azure, la clave de acceso del Almacenamiento y el nombre del contenedor del Almacenamiento.</ul> |
| Base de datos SQL |Lee los datos almacenados en una base de datos SQL Azure o en una base de datos SQL Server que se ejecuta en una máquina virtual de Azure. |<b>Nombre del servidor de base de datos</b>: especifica el nombre del servidor donde se ejecuta la base de datos.<br/><ul>En el caso de una base de datos SQL Azure, escriba el nombre del servidor que se genera. Normalmente tiene el formato *&lt;identificador\_generado&gt;.database.windows.net.* <br/><br/>En el caso de un servidor SQL que se hospeda en una máquina virtual de Azure, escriba *tcp:&lt;Nombre de DNS de máquina virtual&gt;, 1433*</ul><br/><b>Nombre de la base de datos</b>: especifica el nombre de la base de datos en el servidor. <br/><br/><b>Nombre de cuenta de usuario del servidor</b>: especifica un nombre de usuario de una cuenta que tiene permisos de acceso para la base de datos. <br/><br/><b>Contraseña de cuenta de usuario del servidor</b>: especifica la contraseña para la cuenta de usuario.<br/><br/><b>Aceptar cualquier certificado de servidor</b>: use esta opción (menos segura) si desea no revisar el certificado del sitio antes de leer los datos.<br/><br/><b>Consulta de base de datos</b>: escriba una instrucción SQL que describa los datos que desea leer. |
| SQL Database local |Lee los datos que se almacenan en una instancia de SQL Database local. |<b>Puerta de enlace de datos</b>: especifica el nombre de la instancia de Data Management Gateway instalada en un equipo donde puede tener acceso a la base de datos de SQL Server. Para obtener información acerca de cómo configurar la puerta de enlace, vea [Análisis avanzados con Azure Machine Learning con datos de una base de datos de SQL Server local](machine-learning-use-data-from-an-on-premises-sql-server.md).<br/><br/><b>Nombre del servidor de base de datos</b>: especifica el nombre del servidor en el que se ejecuta la base de datos.<br/><br/><b>Nombre de la base de datos</b>: especifica el nombre de la base de datos en el servidor. <br/><br/><b>Nombre de cuenta de usuario del servidor</b>: especifica un nombre de usuario de una cuenta que tiene permisos de acceso para la base de datos. <br/><br/><b>Nombre de usuario y contraseña</b>: haga clic en <b>Especificar valores</b> y escriba sus credenciales de la base de datos. Puede usar la autenticación integrada de Windows o la autenticación de SQL Server según la configuración de SQL Server local.<br/><br/><b>Consulta de base de datos</b>: escriba una instrucción SQL que describa los datos que desea leer. |
| Tabla de Azure |Lee los datos del servicio Tabla en el Almacenamiento de Azure.<br/><br/>Si se leen grandes cantidades de datos con poca frecuencia, use el servicio Tabla de Azure. Proporciona una solución de almacenamiento flexible, no relacional (No SQL), escalable a gran escala, económica y de alta disponibilidad. |Las opciones del módulo **Importar datos** cambian en función de si se accede a información pública o a una cuenta de almacenamiento privada que requiere credenciales de inicio de sesión. Esto viene determinado por el <b>Tipo de autenticación</b> que puede tener el valor "PublicOrSAS" o "Cuenta", cada uno de los cuales tiene su propio conjunto de parámetros. <br/><br/><b>URI público o de Firma de acceso compartido (SAS)</b>: los parámetros son:<br/><br/><ul><b>URI de tabla</b>: especifica la dirección URL pública o de SAS para la tabla.<br/><br/><b>Filas para buscar nombres de propiedad</b>: los valores son <i>TopN</i> para examinar el número de filas especificado o <i>ScanAll</i> para obtener todas las filas de la tabla. <br/><br/>Si los datos son homogéneos y predecibles, se recomienda que seleccione *TopN* y escriba un número para N. Para las tablas grandes, esto puede dar lugar a tiempos de lectura más rápidos.<br/><br/>Si los datos están estructurados con conjuntos de propiedades que varían en función de la profundidad y la posición de la tabla, elija la opción *ScanAll* para examinar todas las filas. Esto garantiza la integridad de la propiedad resultante y la conversión de metadatos.<br/><br/></ul><b>Cuenta de almacenamiento privado</b>: los parámetros son: <br/><br/><ul><b>Nombre de cuenta</b>: especifica el nombre de la cuenta que contiene la tabla que se va a leer.<br/><br/><b>Clave de cuenta</b>: indica la clave de almacenamiento asociada a la cuenta.<br/><br/><b>Nombre de tabla</b> : especifica el nombre de la tabla que contiene los datos que se van a leer.<br/><br/><b>Filas para buscar nombres de propiedad</b>: los valores son <i>TopN</i> para examinar el número de filas especificado o <i>ScanAll</i> para obtener todas las filas de la tabla.<br/><br/>Si los datos son homogéneos y predecibles, se recomienda que seleccione *TopN* y escriba un número para N. Para las tablas grandes, esto puede dar lugar a tiempos de lectura más rápidos.<br/><br/>Si los datos están estructurados con conjuntos de propiedades que varían en función de la profundidad y la posición de la tabla, elija la opción *ScanAll* para examinar todas las filas. Esto garantiza la integridad de la propiedad resultante y la conversión de metadatos.<br/><br/> |
| Almacenamiento de blobs de Azure |Lee los datos almacenados en el servicio BLOB en el Almacenamiento de Azure, incluidos imágenes, texto no estructurado o datos binarios.<br/><br/>Puede usar el servicio BLOB para exponer datos públicamente o para almacenar datos de aplicación de forma privada. Puede tener acceso a los datos desde cualquier lugar a través de conexiones HTTP o HTTPS. |Las opciones del módulo **Importar datos** cambian en función de si se accede a información pública o a una cuenta de almacenamiento privada que requiere credenciales de inicio de sesión. Esto viene determinado por el <b>Tipo de autenticación</b> que puede tener un valor "PublicOrSAS" o "Cuenta".<br/><br/><b>URI público o de Firma de acceso compartido (SAS)</b>: los parámetros son:<br/><br/><ul><b>URI</b>: especifica la dirección URL pública o de SAS para el blob de almacenamiento.<br/><br/><b>Formato de archivo</b>: especifica el formato de los datos en el servicio BLOB. Los formatos compatibles son CSV, TSV y ARFF.<br/><br/></ul><b>Cuenta de almacenamiento privada</b>: los parámetros son: <br/><br/><ul><b>Nombre de cuenta</b>: especifica el nombre de la cuenta que contiene el blob que desea leer.<br/><br/><b>Clave de cuenta</b>: especifica la clave de almacenamiento asociada a la cuenta.<br/><br/><b>Ruta de acceso al contenedor, directorio o blob</b>: especifica el nombre del blob que contiene los datos que se van a leer.<br/><br/><b>Formato de archivo de blob</b>: especifica el formato de los datos en el servicio BLOB. Los formatos de datos admitidos son CSV, TSV, ARFF, CSV con una codificación especificada y Excel. <br/><br/><ul>Si el formato es CSV o TSV, asegúrese de indicar si el archivo contiene una fila de encabezado.<br/><br/>Puede usar la opción Excel para leer datos de libros de Excel. En la opción <i>Formato de datos de Excel</i>, indique si los datos están en un intervalo de hojas de cálculo de Excel o en una tabla de Excel. En la opción <i>Hoja de Excel o tabla insertada</i>, especifique el nombre de la hoja o tabla de la que desea leer.</ul><br/> |
| Proveedor de fuente de distribución de datos |Lee datos de un proveedor de fuente admitido. Actualmente se admite únicamente el formato Open Data Protocol (OData). |<b>Tipo de contenido de datos</b>: especifica el formato OData.<br/><br/><b>Dirección URL de origen</b>: especifica la dirección URL completa para la fuente de datos. <br/>Por ejemplo, la siguiente dirección URL lee de la base de datos de ejemplo Northwind: http://services.odata.org/northwind/northwind.svc/ |

<!-- Module References -->
[import-data]: https://msdn.microsoft.com/library/azure/4e1b0fe6-aded-4b3f-a36f-39b8862b9004/
[export-data]: https://msdn.microsoft.com/library/azure/7A391181-B6A7-4AD4-B82D-E419C0D6522C/

<!---HONumber=AcomDC_0921_2016-->