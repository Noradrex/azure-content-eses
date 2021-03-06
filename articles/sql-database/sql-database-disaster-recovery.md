<properties 
   pageTitle="Recuperación ante desastres de Base de datos SQL" 
   description="Obtenga información acerca de cómo recuperar una base de datos tras un error o una interrupción de un centro de datos regional con las funcionalidades de replicación geográfica activa y restauración geográfica de Base de datos SQL de Azure." 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-management" 
   ms.date="05/10/2016"
   ms.author="elfish"/>

# Restauración de una base de datos SQL de Azure o una conmutación por error en una secundaria

Base de datos SQL de Azure ofrece las siguientes capacidades para recuperarse de un corte en el suministro eléctrico:

- [Replicación geográfica activa](sql-database-geo-replication-overview.md)
- [Restauración geográfica](sql-database-geo-restore.md)

Para obtener información acerca de cómo prepararse ante desastres y sobre cuándo se debe recuperar la base de datos, visite la página [Diseño para la continuidad del negocio](sql-database-business-continuity-design.md).

## Cuándo iniciar la recuperación

La operación de recuperación repercute en la aplicación. Este proceso requiere cambiar la cadena de conexión SQL y puede provocar la pérdida de datos permanente. Por lo tanto, debe realizarse solo cuando la duración de la interrupción pueda durar más que el RTO de su aplicación. Cuando se implementa la aplicación en producción, deberá supervisar con frecuencia el estado de la aplicación. Para ello, use los siguientes puntos de datos para garantizar la recuperación:

1.	Error de conectividad permanente de la capa de aplicación en la base de datos.
2.	El Portal de Azure muestra una alerta acerca de una incidencia en una región con un gran impacto.
3.	El servidor de base de datos de SQL Azure está marcado como degradado. 

En función de la tolerancia de la aplicación al tiempo de inactividad y de la posible responsabilidad civil, puede considerar las siguientes opciones de recuperación.

## Espera para la recuperación del servicio

Los equipos de Azure trabajan diligentemente para restaurar la disponibilidad del servicio lo antes posible, pero dependiendo de la causa principal pueden tardar horas, o incluso días. Si la aplicación puede tolerar un tiempo de inactividad considerable, puede esperar hasta que se complete la recuperación. En este caso, no se requieren acciones por su parte. El estado actual del servicio se puede ver en el [panel de estado de los servicios de Azure](https://azure.microsoft.com/status/). Después de la recuperación de la región se restaurará la disponibilidad de su aplicación.

## Conmutación por error de la base de datos secundaria de replicación geográfica

Si el tiempo de inactividad de la aplicación puede dar lugar a responsabilidades civiles, debería utilizar bases de datos con replicación geográfica en la aplicación. Esto permitirá que la aplicación restaure rápidamente la disponibilidad en una región diferente en caso de interrupción. Información acerca de la [configuración de la replicación geográfica](sql-database-geo-replication-portal.md).

Para restaurar la disponibilidad de las bases de datos, es preciso iniciar la conmutación por error en la secundaria con replicación geográfica mediante uno de los métodos admitidos.


Utilice una de las siguientes guías para realizar la conmutación por error en una base de datos secundaria con replicación geográfica:

- [Configuración de replicación geográfica para Base de datos SQL de Azure con el Portal de Azure](sql-database-geo-replication-portal.md)
- [Configuración de la replicación geográfica para Base de datos SQL de Azure con PowerShell](sql-database-geo-replication-powershell.md)
- [Configuración de la replicación geográfica para una base de datos SQL de Azure con Transact-SQL](sql-database-geo-replication-transact-sql.md) 



## Recuperación mediante la restauración geográfica

Si el tiempo de inactividad de la aplicación no da lugar a responsabilidades civiles, puede usar la restauración geográfica como método para recuperar las bases de datos de la aplicación. Dicho método crea una copia de la base de datos a partir de la última copia de seguridad con redundancia geográfica.

Utilice una de las siguientes guías para restaurar geográficamente una base de datos en una región nueva:

- [Geo-Restore an Azure SQL Database from a geo-redundant backup using the Azure Portal](sql-database-geo-restore-portal.md) (Restauración geográfica de una Base de datos SQL de Azure a partir de una copia de seguridad con redundancia geográfica con el Portal de Azure)
- [Restore an Azure SQL Database from a geo-redundant backup using PowerShell](sql-database-geo-restore-powershell.md) (Restauración de una Base de datos SQL de Azure a una región nueva con Powershell) 


## Configuración de la base de datos después de realizar la recuperación

Si se usa la conmutación por error con replicación geográfica de las opciones de restauración geográfica para recuperar la aplicación tras una interrupción, es preciso asegurarse de que la conectividad con las bases de datos nuevas está configurada correctamente para que se pueda reanudar la función normal de la aplicación. Esta es una lista de comprobación de las tareas necesarias para que la producción de la base de datos recuperada esté lista.

### Actualización de cadenas de conexión

Dado que la base de datos recuperada residirá en otro servidor, es preciso que actualice la cadena de conexión de la aplicación para que apunte a dicho servidor.

Para más información acerca de cómo cambiar las cadenas de conexión, consulte [Connections to Azure SQL Database: Central Recommendations](sql-database-connect-central-recommendations.md) (Conexión a Base de datos SQL de Azure: prácticas recomendadas y directrices de diseño).

### Configuración de las reglas del firewall

Es preciso que se asegure de que las reglas del firewall configuradas tanto en el servidor como en la base de datos coinciden con las configuradas en el servidor principal y en la base de datos principal. Para obtener más información, consulte [Configuración del firewall (Base de datos SQL de Azure)](sql-database-configure-firewall-settings.md).


### Configuración de inicios de sesión de y de usuarios de la base de datos

Es preciso asegurarse de que todos los inicios de sesión que usa la aplicación existen en el servidor que hospeda la base de datos recuperada. Para más información, consulte Administración de la seguridad durante la recuperación ante desastres. Para más información, consulte [Configuración de seguridad para Replicación geográfica activa o estándar](sql-database-geo-replication-security-config.md)

>[AZURE.NOTE] Si utiliza la opción de restauración geográfica para recuperarse de una interrupción, debe configurar las reglas del firewall del servidor y los inicios de sesión durante la exploración de DR para asegurarse de que el servidor principal sigue estando disponible para recuperar su configuración. Dado que la restauración geográfica usa las copias de seguridad de la base de datos es posible que la configuración del nivel de servidor no esté disponible durante la interrupción. Después de la exploración puede quitar las bases de datos restauradas, pero mantenga el servidor y su configuración listos para el proceso de recuperación. Para más información acerca de las exploraciones de DR, consulte [Obtención de detalles de la recuperación ante desastres](sql-database-disaster-recovery-drills.md).

### Configuración de alertas de telemetría

Debe asegurarse de que la configuración actual de la regla de alertas está actualizada para asignarla a la base de datos recuperada y al otro servidor.

Para obtener más información sobre las reglas de alerta de las bases de datos, consulte [Recepción de notificaciones de alerta](../azure-portal/insights-receive-alert-notifications.md) y [Estado del servicio de seguimiento](../azure-portal/insights-service-health.md).

### Habilitar auditoría

Si se requiere una auditoría para tener acceso a una base de datos, será preciso habilitar Auditoría tras la recuperación de la base de datos. Un buen indicador de que se requiere una auditoría es que las aplicaciones cliente usen cadenas de conexión seguras en un patrón de *.database.secure.windows.net. Para obtener más información, consulte [Introducción a la auditoría de la Base de datos SQL](sql-database-auditing-get-started.md).




## Recursos adicionales


- [Información general: continuidad del negocio en la nube y recuperación ante desastres con la Base de datos SQL](sql-database-business-continuity.md)
- [Overview: SQL Database Point-in-Time Restore](sql-database-point-in-time-restore.md) (Información general: Restauración a un momento dado de Base de datos SQL)
- [Restauración geográfica](sql-database-geo-restore.md)
- [Replicación geográfica activa](sql-database-geo-replication-overview.md)
- [Diseño de aplicaciones para la recuperación ante desastres en la nube](sql-database-designing-cloud-solutions-for-disaster-recovery.md)
- [Finalización de una base de datos SQL de Azure recuperada](sql-database-recovered-finalize.md)
- [Configuración de seguridad para Replicación geográfica activa o estándar](sql-database-geo-replication-security-config.md)
- [P+F de BCDR de Base de datos SQL](sql-database-bcdr-faq.md)

<!---HONumber=AcomDC_0511_2016-->