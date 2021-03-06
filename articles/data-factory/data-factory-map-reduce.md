---
title: Invocar programa MapReduce desde la factoría de datos de Azure
description: Obtenga información sobre cómo procesar datos mediante la ejecución de programas MapReduce en un clúster de HDInsight de Azure desde una factoría de datos de Azure.
services: data-factory
documentationcenter: ''
author: spelluru
manager: jhubbard
editor: monicar

ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/12/2016
ms.author: spelluru

---
# Invocar programas MapReduce desde la factoría de datos de Azure
La actividad MapReduce de HDInsight en una [canalización](data-factory-create-pipelines.md) de Factoría de datos ejecuta programas de MapReduce en [su propio](data-factory-compute-linked-services.md#azure-hdinsight-linked-service) clúster de HDInsight o en uno basado en Windows/Linux [a petición](data-factory-compute-linked-services.md#azure-hdinsight-on-demand-linked-service). Este artículo se basa en el artículo sobre [actividades de transformación de datos](data-factory-data-transformation-activities.md), que presenta información general de la transformación de datos y las actividades de transformación admitidas.

## Introducción
Una canalización en una factoría de datos de Azure procesa los datos de los servicios de almacenamiento vinculados mediante el uso de servicios de proceso vinculados. Contiene una secuencia de actividades donde cada actividad realiza una operación de procesamiento específica. En este artículo se describe el uso de la actividad MapReduce de HDInsight.

Consulte [Pig](data-factory-pig-activity.md) y [Hive](data-factory-hive-activity.md) para obtener detalles sobre la ejecución de scripts de Pig/Hive en un clúster de HDInsight basado en Windows/Linux desde una canalización mediante actividades Pig y Hive de HDInsight.

## JSON para la actividad MapReduce de HDInsight
En la definición de JSON para la actividad de HDInsight:

1. Establezca el **tipo** de la **actividad** en **HDInsight**.
2. Especifique el nombre de la clase para la propiedad **className**.
3. Especifique la ruta de acceso al archivo JAR incluyendo el nombre de archivo de la propiedad **jarFilePath**.
4. Especifique el servicio vinculado que hace referencia al almacenamiento de blobs de Azure que contiene el archivo JAR de la propiedad **jarLinkedService**.
5. Especifique los argumentos para el programa de MapReduce en la sección **argumentos**. En tiempo de ejecución, verá unos argumentos adicionales (por ejemplo, mapreduce.job.tags) desde el marco de trabajo MapReduce. Para diferenciar sus argumentos con los argumentos de MapReduce, considere el uso tanto de opción como valor como argumentos tal como se muestra en el siguiente ejemplo (-s, --input, --output etc., son opciones seguidas inmediatamente por sus valores).
   
        {
            "name": "MahoutMapReduceSamplePipeline",
            "properties": {
                "description": "Sample Pipeline to Run a Mahout Custom Map Reduce Jar. This job calcuates an Item Similarity Matrix to determine the similarity between 2 items",
                "activities": [
                    {
                        "type": "HDInsightMapReduce",
                        "typeProperties": {
                            "className": "org.apache.mahout.cf.taste.hadoop.similarity.item.ItemSimilarityJob",
                            "jarFilePath": "adfsamples/Mahout/jars/mahout-examples-0.9.0.2.2.7.1-34.jar",
                            "jarLinkedService": "StorageLinkedService",
                            "arguments": [
                                "-s",
                                "SIMILARITY_LOGLIKELIHOOD",
                                "--input",
                                "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/input",
                                "--output",
                                "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/output/",
                                "--maxSimilaritiesPerItem",
                                "500",
                                "--tempDir",
                                "wasb://adfsamples@spestore.blob.core.windows.net/Mahout/temp/mahout"
                            ]
                        },
                        "inputs": [
                            {
                                "name": "MahoutInput"
                            }
                        ],
                        "outputs": [
                            {
                                "name": "MahoutOutput"
                            }
                        ],
                        "policy": {
                            "timeout": "01:00:00",
                            "concurrency": 1,
                            "retry": 3
                        },
                        "scheduler": {
                            "frequency": "Hour",
                            "interval": 1
                        },
                        "name": "MahoutActivity",
                        "description": "Custom Map Reduce to generate Mahout result",
                        "linkedServiceName": "HDInsightLinkedService"
                    }
                ],
                "start": "2014-01-03T00:00:00Z",
                "end": "2014-01-04T00:00:00Z",
                "isPaused": false,
                "hubName": "mrfactory_hub",
                "pipelineMode": "Scheduled"
            }
        }

Puede usar la actividad MapReduce de HDInsight para ejecutar cualquier archivo jar de MapReduce en un clúster de HDInsight. En la siguiente definición de JSON de ejemplo de una canalización, la actividad de HDInsight se configura para ejecutar un archivo JAR de Mahout.

## Ejemplo en GitHub
Puede descargar un ejemplo para usar la actividad MapReduce de HDInsight desde: [Ejemplos de factoría de datos en GitHub](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/JSON/MapReduce_Activity_Sample).

## Ejecutar el programa de recuento de palabras
La canalización de este ejemplo ejecuta el programa de asignación/reducción del recuento de palabras en el clúster de HDInsight de Azure.

### Servicios vinculados
En primer lugar, cree un servicio vinculado para vincular el almacenamiento de Azure usado por el clúster de HDInsight de Azure con la Factoría de datos de Azure. Si copia/pega el código siguiente, no olvide reemplazar el **nombre** y la **clave de la cuenta** por el nombre y la clave de su Almacenamiento de Azure.

#### Servicio vinculado de Almacenamiento de Azure
    {
        "name": "StorageLinkedService",
        "properties": {
            "type": "AzureStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
            }
        }
    }

#### Servicio vinculado de HDInsight de Azure
A continuación, cree un servicio vinculado para vincular el clúster de HDInsight de Azure con la Factoría de datos de Azure. Si copia/pegar el código siguiente, reemplace el **nombre del clúster de HDInsight** por el nombre de su clúster de HDInsight y cambie los valores de nombre de usuario y contraseña.

    {
        "name": "HDInsightLinkedService",
        "properties": {
            "type": "HDInsight",
            "typeProperties": {
                "clusterUri": "https://<HDInsight cluster name>.azurehdinsight.net",
                "userName": "admin",
                "password": "**********",
                "linkedServiceName": "StorageLinkedService"
            }
        }
    }

### Conjuntos de datos
#### Conjunto de datos de salida
La canalización de este ejemplo no toma ninguna entrada. Deberá especificar un conjunto de datos de salida para la actividad MapReduce de HDInsight. Este conjunto de datos es simplemente un conjunto de datos ficticio que es necesario para la programación de la canalización.

    {
        "name": "MROutput",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "StorageLinkedService",
            "typeProperties": {
                "fileName": "WordCountOutput1.txt",
                "folderPath": "example/data/",
                "format": {
                    "type": "TextFormat",
                    "columnDelimiter": ","
                }
            },
            "availability": {
                "frequency": "Day",
                "interval": 1
            }
        }
    }

### Canalización
La canalización de este ejemplo tiene solo una actividad de tipo: HDInsightMapReduce. Algunas de las propiedades importantes en JSON son:

| Propiedad | Notas |
|:--- |:--- |
| type |El tipo debe establecerse en **HDInsightMapReduce**. |
| className |El nombre de la clase es: **wordcount** |
| jarFilePath |Ruta de acceso al archivo .jar que contiene la clase anterior. Si copia/pega el código siguiente, no olvide cambiar el nombre del clúster. |
| jarLinkedService |Servicio vinculado al Almacenamiento de Azure que contiene el archivo jar. Este servicio vinculado hace referencia al almacenamiento asociado al clúster de HDInsight. |
| argumentos |El programa de recuento de palabras toma dos argumentos, una entrada y una salida. El archivo de entrada es el archivo davinci.txt. |
| frecuencia/intervalo |Los valores de estas propiedades coinciden con el conjunto de datos de salida. |
| linkedServiceName |hace referencia al servicio vinculado a HDInsight creado anteriormente. |

    {
        "name": "MRSamplePipeline",
        "properties": {
            "description": "Sample Pipeline to Run the Word Count Program",
            "activities": [
                {
                    "type": "HDInsightMapReduce",
                    "typeProperties": {
                        "className": "wordcount",
                        "jarFilePath": "<HDInsight cluster name>/example/jars/hadoop-examples.jar",
                        "jarLinkedService": "StorageLinkedService",
                        "arguments": [
                            "/example/data/gutenberg/davinci.txt",
                            "/example/data/WordCountOutput1"
                        ]
                    },
                    "outputs": [
                        {
                            "name": "MROutput"
                        }
                    ],
                    "policy": {
                        "timeout": "01:00:00",
                        "concurrency": 1,
                        "retry": 3
                    },
                    "scheduler": {
                        "frequency": "Day",
                        "interval": 1
                    },
                    "name": "MRActivity",
                    "linkedServiceName": "HDInsightLinkedService"
                }
            ],
            "start": "2014-01-03T00:00:00Z",
            "end": "2014-01-04T00:00:00Z"
        }
    }

## Ejecutar programas Spark
Puede usar la actividad MapReduce para ejecutar programas Spark en su clúster de HDInsight Spark. Consulte [Invoke Spark programs from Azure Data Factory](data-factory-spark.md) (Invocar programas Spark desde Data Factory de Azure) para obtener información detallada.

[developer-reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[cmdlet-reference]: http://go.microsoft.com/fwlink/?LinkId=517456


[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[adfgetstartedmonitoring]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md#monitor-pipelines

[Developer Reference]: http://go.microsoft.com/fwlink/?LinkId=516908
[Azure Portal]: http://portal.azure.com

## Otras referencias
* [Actividad de Hive](data-factory-hive-activity.md)
* [Actividad de Pig](data-factory-pig-activity.md)
* [Actividad de streaming de Hadoop](data-factory-hadoop-streaming-activity.md)
* [Invocar programas Spark](data-factory-spark.md)
* [Invocar scripts de R](https://github.com/Azure/Azure-DataFactory/tree/master/Samples/RunRScriptUsingADFSample)

<!---HONumber=AcomDC_0914_2016-->