---
title: Estrategias Efectivas de Respaldo para Azure Tables
date: 2023-11-22 18:22:00 -0500
categories: [Azure, Architecture]
tags: [azure, architecture, paas, disaster recovery, high availability, backup, azure tables, azure storage]
image:
  path: /assets/img/posts/2023-11-22-Azure-Tables-Backup.webp
---

## Abstract:
En el dinámico mundo de los servicios en la nube, donde la seguridad de los datos y los respaldos son de suma importancia, Azure Tables en la plataforma en la nube Microsoft Azure presenta un desafío único. Este post técnico profundiza en las estrategias prácticas y eficientes de respaldo para Azure Tables, destacando diversas herramientas y métodos para proteger sus datos.

## Estrategias de respaldo para Azure Tables
Azure Tables, robusto y escalable para almacenar datos estructurados NoSQL, carece de una funcionalidad de respaldo integrada específica. Esto plantea un desafío significativo para los usuarios que necesitan asegurar sus datos contra posibles incidentes o pérdidas de datos. Aunque el componente "Copy Data" de Azure Data Factory puede copiar tablas a otro servicio de almacenamiento Azure o incluso a un servicio fuera de Azure, podría ser excesivo para necesidades de respaldo simples debido a sus amplias características y costos adicionales potenciales.

Un enfoque efectivo implica el uso de Container Apps y Azure Functions para activar respaldos programados. Estos pueden recuperar datos de Azure Tables y almacenarlos en Azure Blob Storage u otros servicios. Además, más allá del conocido SDK, existen otras herramientas como los cmdlets de PowerShell, comandos CLI y `AzCopy`, todos utilizando la API de Azure.

`AzCopy` emerge como una herramienta particularmente efectiva para realizar operaciones rápidamente en Azure Blob Storage utilizando líneas de comando simples. Permite procesos automatizados a través de comandos shell en lenguajes como C#. En un contexto C#, estos comandos shell se pueden ejecutar usando [`Process.Start`](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.start?view=net-8.0). Este método permite la construcción dinámica de comandos `AzCopy` necesarios para copiar una tabla a un blob.

Después de exportar los datos al almacenamiento blob, podría establecer una [estrategia automatizada de respaldo de almacenamiento blob](https://learn.microsoft.com/en-us/azure/backup/blob-backup-overview).

Ejemplo de comando `AzCopy` para respaldar una tabla:

```PowerShell
AzCopy /Source:https://myaccount.table.core.windows.net/myTable/ /Dest:https://myaccount.blob.core.windows.net/mycontainer/ /SourceKey:key1 /DestKey:key2
```

Restaurando datos con `AzCopy`:

``` PowerShell
AzCopy /Source:https://myaccount.blob.core.windows.net/mycontainer /Dest:https://myaccount.table.core.windows.net/mytable /SourceKey:key1 /DestKey:key2 /Manifest:"myaccount_mytable_20140103T112020.manifest" /EntityOperation:"InsertOrReplace" 
```

Este mecanismo es tan flexible que incluso puede dividir las tablas para ajustarlas a un tamaño de archivo exportado específico. Además, puede aprovechar la concurrencia para exportar las tablas en paralelo basándose en los `PKs` mientras configura el número de hilos concurrentes usando la opción `/NC`. Recomiendo que ajuste el paralelismo al número de núcleos que tenga en la unidad de procesamiento que utilice para hacer el respaldo.

`AzCopy` ahora también admite archivos de parámetros (response files) para una configuración más fácil y flexible.

Puedes encontrar todos los detalles para la implementación de estas recomendaciones [aqui](https://learn.microsoft.com/en-us/previous-versions/azure/storage/storage-use-azcopy#export-data-from-table-storage)

Alternativamente, una solución más convencional implica mezclar el [`SDK Azure.Data.Tables`](https://www.nuget.org/packages/Azure.Data.Tables) con la biblioteca `Data Movement` para una carga eficiente al [`Blob Storage`](https://github.com/Azure/azure-storage-net-data-movement). Este enfoque ofrece ventajas como la compresión de archivos y no necesitar el ejecutable de `AzCopy` en la distribución de la aplicación. Comparativamente, `AzCopy` simplifica el proceso de exportación en una sola línea de comando, mientras que usar el SDK requiere codificación manual. `AzCopy` también se puede usar para descargar tablas a un disco local para su compresión antes de enviarlas a Blob Storage, lo que puede ahorrar espacio de almacenamiento.

Otra opción es migrar su solución de `Azure Tables` a `CosmosDB`. Proporciona opciones de respaldo automáticas y el código existente para administrar sus tablas funcionará igual con `CosmosDB`; pero conlleva costos más elevados debido a sus características adicionales.



