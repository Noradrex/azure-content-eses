<properties 
    pageTitle="Administración de un grupo de bases de datos elásticas (PowerShell) | Microsoft Azure" 
    description="Obtenga más información sobre cómo usar PowerShell para administrar un grupo de bases de datos elásticas."  
	services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="data-management" 
    ms.date="05/10/2016"
    ms.author="sidneyh"/>

# Supervisión, administración y ajuste de tamaño de un grupo de bases de datos elásticas con PowerShell 

> [AZURE.SELECTOR]
- [Portal de Azure](sql-database-elastic-pool-manage-portal.md)
- [PowerShell](sql-database-elastic-pool-manage-powershell.md)
- [C#](sql-database-elastic-pool-manage-csharp.md)
- [T-SQL](sql-database-elastic-pool-manage-tsql.md)

Administre un [grupo de bases de datos elásticas](sql-database-elastic-pool.md) mediante cmdlets de PowerShell.

Para ver los códigos de error comunes, consulte [Códigos de error para las aplicaciones cliente de la Base de datos SQL: error de conexión de base de datos y otros problemas](sql-database-develop-error-messages.md).

Los valores de los grupos se pueden encontrar en el artículo de [límites de almacenamiento y de eDTU](sql-database-elastic-pool#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases).

## Requisitos previos

* Azure PowerShell 1.0 o posterior. Para obtener información detallada, vea [Instalación y configuración de Azure PowerShell](../powershell-install-configure.md).
* Los grupos de bases de datos elásticas solo están disponibles en los servidores de la versión 12 de Base de datos SQL de Azure. Si tiene un servidor de la versión 11 de Base de datos SQL, [use PowerShell para actualizarlo a la versión 12 y crear un grupo](sql-database-upgrade-server-portal.md) en un solo paso.


## Movimiento de una base de datos a un grupo elástico

Puede mover una base de datos dentro o fuera de un grupo con el cmdlet [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/library/azure/mt619433.aspx).

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## Cambio de la configuración de rendimiento de un grupo

Cuando el rendimiento se ve afectado, puede cambiar la configuración del grupo para adaptarse al crecimiento. Utilice el cmdlet [Set-AzureRmSqlElasticPool](https://msdn.microsoft.com/library/azure/mt603511.aspx). Establezca el parámetro -Dtu en las eDTU por grupo. Consulte los posibles valores en el artículo de [límites de almacenamiento y de eDTU](sql-database-elastic-pool#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases).

    Set-AzureRmSqlElasticPool –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 


## Obtención del estado de las operaciones de los grupos

El proceso de crear grupos puede llevar tiempo. Puede realizar un seguimiento del estado de las operaciones del grupo, como la creación y las actualizaciones, mediante el cmdlet [Get-AzureRmSqlElasticPoolActivity](https://msdn.microsoft.com/library/azure/mt603812.aspx).

	Get-AzureRmSqlElasticPoolActivity –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” 


## Obtención del estado entrada o salida de un grupo de una base de datos elástica

El proceso de mover bases de datos puede llevar tiempo. Realice un seguimiento del estado de movimiento mediante el cmdlet [Get-AzureRmSqlDatabaseActivity](https://msdn.microsoft.com/library/azure/mt603687.aspx).

	Get-AzureRmSqlDatabaseActivity -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## Obtención de datos de uso de recursos para un grupo

Métricas que se pueden recuperar como porcentaje del límite del grupo de recursos:


| Nombre de métrica | Descripción |
| :-- | :-- |
| cpu\_percent | Uso de proceso medio en porcentaje del límite del grupo. |
| physical\_data\_read\_percent | Uso de E/S medio en porcentaje basado en el límite del grupo. |
| log\_write\_percent | Uso de recursos de escritura medio en porcentaje del límite del grupo. | 
| DTU\_consumption\_percent | Uso de eDTU medio en porcentaje del límite de eDTU del grupo. | 
| storage\_percent | Uso de almacenamiento medio en porcentaje del límite de almacenamiento del grupo. |  
| workers\_percent | Cantidad máxima de trabajos simultáneos (solicitudes) en porcentaje basado en el límite del grupo. |  
| sessions\_percent | Cantidad máxima de sesiones simultáneas en porcentaje basado en el límite del grupo. | 
| eDTU\_limit | Configuración de cantidad máxima de DTU de grupos elásticos actual para este grupo elástico durante este intervalo. |
| storage\_limit | Configuración de límite máximo de almacenamiento de grupos elásticos actual para este grupo elástico en megabytes durante este intervalo. |
| eDTU\_used | Cantidad media de eDTU que usa el grupo en este intervalo. |
| storage\_used | Cantidad media de almacenamiento en bytes que usa el grupo en este intervalo. |

**Métricas de períodos de retención y granularidad:**

* Los datos se devolverán con una granularidad de 5 minutos.  
* La retención de datos es de 14 días.  

Este cmdlet y la API limita el número de filas que se pueden recuperar en una llamada a 1000 filas (aproximadamente 3 días de datos con una granularidad de 5 minutos). Sin embargo, este comando se puede llamar varias veces con distintos intervalos de tiempo de inicio y fin para recuperar más datos.

Para recuperar las métricas, siga estos pasos:

	$metrics = (Get-AzureRmMetric -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015")  


## Obtención de datos de uso de recursos de una base de datos elástica

Estas API son las mismas que las actuales (versión 12) que se usan para supervisar el uso de recursos de una base de datos independiente, excepto por la diferencia semántica siguiente.

Para esta API, las métricas recuperadas se expresan como un porcentaje de la capacidad por cantidad máxima de eDTU (o una equivalente a la métrica subyacente como CPU, E/S etc.) establecido para ese grupo. Por ejemplo, el 50 % de uso de cualquiera de estas métricas indica que el consumo de recursos específicos es del 50 % del límite de capacidad por cada base de datos de dicho recurso del grupo principal.

Para recuperar las métricas, siga estos pasos:

    $metrics = (Get-AzureRmMetric -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

## Recopilación y supervisión de los datos de uso de recursos entre varios grupos de una suscripción

Si hay un gran número de bases de datos en una suscripción, es complicado supervisar cada grupo elástico por separado. En su lugar, se pueden combinar las consultas de T-SQL y los cmdlets de PowerShell de Base de datos SQL para recopilar datos de uso de recursos de varios grupos y sus bases de datos para la supervisión y el análisis del uso de recursos. Puede encontrar una [implementación de ejemplo](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools) de un conjunto similar de scripts de Powershell en el repositorio de ejemplos de GitHub SQL Server junto con documentación sobre lo que hace y cómo se utiliza.

Para utilizar esta implementación de ejemplo siga los pasos enumerados a continuación.


1. Descargue los [scripts y la documentación](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools):
2. Modifique los scripts para su entorno. Especifique uno o más servidores en los que se alojan los grupos elásticos.
3. Especifique una base de datos de telemetría donde se puedan almacenar las métricas recopiladas. 
4. Personalice el script para especificar el tiempo de ejecución de los scripts.

En un nivel alto, los scripts realizan lo siguiente:

*	Enumera todos los servidores de una determinada suscripción de Azure (o una lista de servidores especificada).
*	Ejecuta un trabajo en segundo plano para cada servidor. El trabajo se ejecuta en un bucle a intervalos regulares y recopila los datos de telemetría de todos los grupos en el servidor. A continuación, carga los datos recopilados en la base de datos de telemetría especificada.
*	Enumera una lista de bases de datos en cada grupo para recopilar los datos de uso de recursos de base de datos. A continuación, carga los datos recopilados en la base de datos de telemetría.

Para supervisar el estado de los grupos elásticos y de las bases de datos en tales grupos, se pueden analizar las métricas recopiladas en la base de datos de telemetría. El script también instala una función de valores de tabla (TVF) predefinida en la base de datos de telemetría para ayudar a agregar las métricas para un período de tiempo especificado. Por ejemplo, los resultados de la función TVF pueden utilizarse para mostrar "los N grupos elásticos que presentan el máximo uso de eDTU en un período de tiempo determinado". Opcionalmente, puede utilizar herramientas de análisis como Excel o Power BI para consultar y analizar los datos recopilados.

## Ejemplo: recuperación de métricas de consumo de recursos de un grupo y de sus bases de datos

En este ejemplo se recuperan las métricas de consumo de un grupo elástico determinado y de todas sus bases de datos. Se da formato a los datos recopilados y se escriben en un archivo .csv. El archivo puede consultarse con Excel.

	$subscriptionId = '<Azure subscription id>'	      # Azure subscription ID
	$resourceGroupName = '<resource group name>'             # Resource Group
	$serverName = <server name>                              # server name
	$poolName = <elastic pool name>                          # pool name
		
	# Login to Azure account and select the subscription.
	Login-AzureRmAccount
	Set-AzureRmContext -SubscriptionId $subscriptionId
	
	# Get resource usage metrics for an elastic pool for the specified time interval.
	$startTime = '4/27/2016 00:00:00'  # start time in UTC
	$endTime = '4/27/2016 01:00:00'    # end time in UTC
	
	# Construct the pool resource ID and retrive pool metrics at 5 minute granularity.
	$poolResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/elasticPools/' + $poolName
	$poolMetrics = (Get-AzureRmMetric -ResourceId $poolResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime -EndTime $endTime) 
	
	# Get the list of databases in this pool.
	$dbList = Get-AzureRmSqlElasticPoolDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -ElasticPoolName $poolName
	
	# Get resource usage metrics for a database in an elastic database for the specified time interval.
	$dbMetrics = @()
	foreach ($db in $dbList)
	{
	    $dbResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/databases/' + $db.DatabaseName
	    $dbMetrics = $dbMetrics + (Get-AzureRmMetric -ResourceId $dbResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime -EndTime $endTime)
	}
	
	#Optionally you can format the metrics and output as .csv file using the following script block.
	$command = {
    param($metricList, $outputFile)

    # Format metrics into a table.
    $table = @()
    foreach($metric in $metricList) { 
      foreach($metricValue in $metric.MetricValues) {
        $sx = New-Object PSObject -Property @{
            Timestamp = $metricValue.Timestamp.ToString()
            MetricName = $metric.Name; 
            Average = $metricValue.Average;
            ResourceID = $metric.ResourceId 
          }
          $table = $table += $sx
      }
    }
    
    # Output the metrics into a .csv file.
    write-output $table | Export-csv -Path $outputFile -Append -NoTypeInformation
	}
	
	# Format and output pool metrics
	Invoke-Command -ScriptBlock $command -ArgumentList $poolMetrics,c:\temp\poolmetrics.csv
	
	# Format and output database metrics
	Invoke-Command -ScriptBlock $command -ArgumentList $dbMetrics,c:\temp\dbmetrics.csv



## Latencia de las operaciones de grupos elásticos

- El cambio del número mínimo de eDTU por base de datos o del máximo de eDTU por base de datos suele completarse en cinco minutos o menos.
- El cambio de eDTU por grupo depende de la cantidad total de espacio que usen todas las bases de datos del grupo. Los cambios tienen un duración media de 90 minutos o menos por cada 100 GB. Por ejemplo, si el espacio total que usan todas las bases de datos del grupo es de 200 GB, la latencia esperada para cambiar las eDTU de grupo por grupo es de 3 horas o menos.

## Migración de la versión 11 de los servidores a la 12

Los cmdlets de PowerShell se pueden usar para iniciar, detener o supervisar una actualización de la Base de datos SQL de Azure V12 a partir de la versión V11, o desde cualquier otra versión anterior a la V12.

- [Actualización a la base de datos SQL V12 con PowerShell](sql-database-upgrade-server-powershell.md)

Para obtener documentación de referencia acerca de estos cmdlets de Powershell, consulte:


- [Get-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603582.aspx)
- [Start-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt619403.aspx)
- [Stop-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/library/azure/mt603589.aspx)


El cmdlet Stop- significa cancelar, no pausar. La única forma de reanudar una actualización es volver a iniciarla desde el principio. El cmdlet Stop- limpia y libera todos los recursos adecuados.

## Pasos siguientes

- [Creación de trabajos elásticos](sql-database-elastic-jobs-overview.md): los trabajos elásticos permiten ejecutar scripts de T-SQL en cualquier cantidad de bases de datos del grupo.
- Consulte [Información general de las características de bases de datos elásticas](sql-database-elastic-scale-introduction.md): use herramientas de bases de datos elásticas para realizar un escalado horizontal, mover los datos, realizar consultas o crear transacciones.

<!---HONumber=AcomDC_0511_2016-->