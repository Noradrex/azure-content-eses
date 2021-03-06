<properties
   pageTitle="Visualización del mantenimiento agregado de entidades de Azure Service Fabric | Microsoft Azure"
   description="Se describe cómo consultar, ver y evaluar el mantenimiento agregado de entidades de Azure Service Fabric, a través de consultas de mantenimiento y consultas generales."
   services="service-fabric"
   documentationCenter=".net"
   authors="oanapl"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="04/25/2016"
   ms.author="oanapl"/>

# Vista de los informes de estado de Service Fabric
Azure Service Fabric presenta un [modelo de mantenimiento](service-fabric-health-introduction.md) formado por entidades de mantenimiento en las que componentes y guardianes del sistema pueden notificar las condiciones locales que están supervisando. El [almacén de estado](service-fabric-health-introduction.md#health-store) agrega todos los datos de mantenimiento para determinar si las entidades son correctas.

De fábrica, el clúster está rellenado con informes de estado enviados por los componentes del sistema. Lea más en [Uso de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md).

Service Fabric proporciona varias maneras de obtener el mantenimiento agregado de las entidades:

- [Explorador de Service Fabric](service-fabric-visualizing-your-cluster.md) y otras herramientas de visualización

- Consultas de mantenimiento (a través de PowerShell, la API o REST)

- Consultas generales que devuelven una lista de entidades con el mantenimiento como una de las propiedades (a través de Powerhsell, la API o REST)

Para demostrar estas opciones, vamos a usar un clúster local con cinco nodos. Junto a la aplicación **fabric:/System** (que está integrada), se implementan algunas otras aplicaciones. Una de ellas es **fabric:/WordCount**. Esta aplicación contiene un servicio con estado configurado con siete réplicas. Dado que solo hay cinco nodos, los componentes del sistema mostrarán una advertencia de que la partición está por debajo del recuento de destino.

```xml
<Service Name="WordCountService">
    <StatefulService ServiceTypeName="WordCountServiceType" TargetReplicaSetSize="7" MinReplicaSetSize="2">
      <UniformInt64Partition PartitionCount="1" LowKey="1" HighKey="26" />
    </StatefulService>
</Service>
```

## Mantenimiento del Explorador de Service Fabric
El Explorador de Service Fabric proporciona una vista visual del clúster. En la imagen siguiente, puede ver que:

- La aplicación **fabric:/WordCount** está en rojo (en error) porque tiene un evento de error notificado por **MyWatchdog** para la propiedad **Availability**.

- Uno de sus servicios, **fabric:/WordCount/WordCountService** está en amarillo (en advertencia). Como se describió anteriormente, el servicio está configurado con siete réplicas, y no se pueden colocar todas (ya que solo hay cinco nodos). Aunque no se muestra aquí, la partición de servicio está en amarillo debido al informe del sistema. La partición en amarillo desencadena el servicio en amarillo.

- El clúster está en rojo debido a la aplicación en rojo.

La evaluación usa las directivas predeterminadas del manifiesto de clúster y el manifiesto de aplicación. Son directivas estrictas y no toleran ningún error.

Vista del clúster con el Explorador de Service Fabric.

![Vista del clúster con el Explorador de Service Fabric.][1]

[1]: ./media/service-fabric-view-entities-aggregated-health/servicefabric-explorer-cluster-health.png


> [AZURE.NOTE] Obtenga más información sobre el [Explorador de Service Fabric](service-fabric-visualizing-your-cluster.md).

## Consultas de mantenimiento
Service Fabric expone las consultas de mantenimiento para cada uno de los [tipos de entidad](service-fabric-health-introduction.md#health-entities-and-hierarchy) admitidos. Se puede acceder e ellas mediante la API (los métodos se pueden encontrar en **FabricClient.HealthManager**), los cmdlets de PowerShell y REST. Estas consultas devuelven información completa de mantenimiento sobre la entidad, incluido el estado de mantenimiento agregado, los eventos de mantenimiento notificados en la entidad, los estados de mantenimiento de los elementos secundarios (si procede) y las evaluaciones de mantenimiento incorrecto cuando el mantenimiento de la entidad no es correcto.

> [AZURE.NOTE] Una entidad de mantenimiento se devuelve al usuario cuando se rellena completamente en el almacén de estado. La entidad debe estar activa (no eliminada) y tener un informe de sistema. Sus entidades primarias en la cadena de jerarquía deben tener también informes del sistema. Si no se cumple alguna de estas condiciones, las consultas de mantenimiento devuelven una excepción que muestra por qué no se devuelve la entidad.

Las consultas de mantenimiento requieren pasar el identificador de entidad, que depende del tipo de entidad. Las consultas aceptan parámetros de directivas de mantenimiento opcionales. Si no se especifican, se usan para la evaluación las [directivas de mantenimiento](service-fabric-health-introduction.md#health-policies) del manifiesto de aplicación o de clúster. También aceptan filtros para devolver solo los elementos secundarios o eventos parciales, los que respetan los filtros especificados.

> [AZURE.NOTE] Los filtros de salida se aplican en el servidor, por lo que se reduce el tamaño de la respuesta del mensaje. Recomendamos usar los filtros de salida para limitar los datos devueltos en lugar de aplicar filtros en el cliente.

El mantenimiento de una entidad contiene la siguiente información:

- El estado de mantenimiento agregado de la entidad. Esto lo calcula el almacén de estado en función de los informes de mantenimiento de entidades, los estados de mantenimiento de los elementos secundarios (si procede) y las directivas de mantenimiento. Lea más sobre la [evaluación del mantenimiento de entidades](service-fabric-health-introduction.md#entity-health-evaluation).  

- Eventos de mantenimiento de la entidad.

- La colección de los estados de mantenimiento de todos los elementos secundarios de las entidades que pueden tener elementos secundarios. Los estados de mantenimiento contienen identificadores de entidad y el estado de mantenimiento agregado. Para obtener el mantenimiento completo de un elemento secundario, llame al mantenimiento de consultas del tipo de entidad secundaria y pase el identificador de elementos secundarios.

- Si la entidad no es correcta, las evaluaciones de mantenimiento incorrecto apuntan al informe que desencadenó el estado de la entidad.

## Obtención del mantenimiento de clúster
Este proceso devuelve el mantenimiento de la entidad del clúster y contiene los estados de mantenimiento de aplicaciones y nodos (elementos secundarios del clúster). Entrada:

- [Opcional] La directiva de mantenimiento del clúster utilizada para evaluar los nodos y los eventos del clúster.

- [Opcional] La asignación de directivas de mantenimiento de aplicaciones, con las directivas de mantenimiento usadas para invalidar las directivas de manifiesto de aplicación.

- [Opcional] Filtros para eventos, nodos y aplicaciones que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente los errores o advertencias y errores). Tenga en cuenta que todos los eventos, nodos y aplicaciones se utilizan para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento del clúster, cree una instancia de `FabricClient` y llame al método [GetClusterHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthasync.aspx) en su instancia de **HealthManager**.

Lo siguiente obtiene el mantenimiento del clúster:

```csharp
ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync();
```

A continuación se obtiene el mantenimiento del clúster mediante una directiva de mantenimiento personalizada del clúster y filtros para nodos y aplicaciones. Tenga en cuenta que esta operación crea la clase [ClusterHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.clusterhealthquerydescription.aspx), que contiene todos los datos de entrada.

```csharp
var policy = new ClusterHealthPolicy()
{
    MaxPercentUnhealthyNodes = 20
};
var nodesFilter = new NodeHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error | HealthStateFilter.Warning
};
var applicationsFilter = new ApplicationHealthStatesFilter()
{
    HealthStateFilterValue = HealthStateFilter.Error
};
var queryDescription = new ClusterHealthQueryDescription()
{
    HealthPolicy = policy,
    ApplicationsFilter = applicationsFilter,
    NodesFilter = nodesFilter,
};

ClusterHealth clusterHealth = await fabricClient.HealthManager.GetClusterHealthAsync(queryDescription);
```

### PowerShell
El cmdlet para obtener el mantenimiento del clúster es [Get-ServiceFabricClusterHealth](https://msdn.microsoft.com/library/mt125850.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

El estado del clúster es igual a cinco nodos, la aplicación del sistema y fabric:/WordCount se configuran como antes.

El siguiente cmdlet obtiene el mantenimiento del clúster con directivas de mantenimiento predeterminadas. El estado de mantenimiento agregado está en advertencia, porque la aplicación fabric:/WordCount está en advertencia. Observe cómo las evaluaciones de mantenimiento incorrecto muestran detalles de las condiciones que desencadenaron el mantenimiento agregado.

```xml
PS C:\> Get-ServiceFabricClusterHealth

AggregatedHealthState   : Warning
UnhealthyEvaluations    :
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Warning'.

                              Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                  Unhealthy event: SourceId='System.PLB',
                          Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          ConsiderWarningAsError=false.


NodeHealthStates        :
                          NodeName              : _Node_2
                          AggregatedHealthState : Ok

                          NodeName              : _Node_0
                          AggregatedHealthState : Ok

                          NodeName              : _Node_1
                          AggregatedHealthState : Ok

                          NodeName              : _Node_3
                          AggregatedHealthState : Ok

                          NodeName              : _Node_4
                          AggregatedHealthState : Ok

ApplicationHealthStates :
                          ApplicationName       : fabric:/System
                          AggregatedHealthState : Ok

                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Warning

HealthEvents            : None
```

El siguiente cmdlet de PowerShell obtiene el mantenimiento del clúster mediante una directiva de aplicación personalizada. Filtra los resultados para obtener solo los nodos y las aplicaciones con error o advertencia. Como resultado, no se devolverá ningún nodo ya que todos son correctos. Solo la aplicación fabric:/WordCount respeta el filtro de aplicaciones. Puesto que la directiva personalizada especifica que hay que considerar que las advertencias son errores en la aplicación fabric:/WordCount, la aplicación se evalúa como en error y lo mismo el clúster.

```powershell
PS c:> $appHealthPolicy = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicy
$appHealthPolicy.ConsiderWarningAsError = $true
$appHealthPolicyMap = New-Object -TypeName System.Fabric.Health.ApplicationHealthPolicyMap
$appUri1 = New-Object -TypeName System.Uri -ArgumentList "fabric:/WordCount"
$appHealthPolicyMap.Add($appUri1, $appHealthPolicy)
Get-ServiceFabricClusterHealth -ApplicationHealthPolicyMap $appHealthPolicyMap -ApplicationsFilter "Warning,Error" -NodesFilter "Warning,Error"


AggregatedHealthState   : Error
UnhealthyEvaluations    :
                          Unhealthy applications: 100% (1/1), MaxPercentUnhealthyApplications=0%.

                          Unhealthy application: ApplicationName='fabric:/WordCount', AggregatedHealthState='Error'.

                              Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                              Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                  Unhealthy event: SourceId='System.PLB',
                          Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                          ConsiderWarningAsError=true.


NodeHealthStates        : None
ApplicationHealthStates :
                          ApplicationName       : fabric:/WordCount
                          AggregatedHealthState : Error

HealthEvents            : None

```

## Obtención del mantenimiento de nodo
Este proceso devuelve el mantenimiento de una entidad del nodo y contiene los eventos de mantenimiento notificados en el nodo. Entrada:

- [Obligatorio] El nombre de nodo que identifica al nodo.

- [Opcional] La configuración de la directiva de mantenimiento del clúster usada para evaluar el mantenimiento.

- [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento del nodo mediante la API, cree una instancia de `FabricClient` y llame al método [GetNodeHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getnodehealthasync.aspx) en su instancia de HealthManager.

A continuación se obtiene el mantenimiento del nodo para el nombre de nodo especificado:

```csharp
NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(nodeName);
```

A continuación se recibe el mantenimiento del nodo para el nombre de nodo especificado y se pasa un filtro de eventos y la directiva personalizada mediante [NodeHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.nodehealthquerydescription.aspx):

```csharp
var queryDescription = new NodeHealthQueryDescription(nodeName)
{
    HealthPolicy = new ClusterHealthPolicy() {  ConsiderWarningAsError = true },
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.Warning },
};

NodeHealth nodeHealth = await fabricClient.HealthManager.GetNodeHealthAsync(queryDescription);
```

### PowerShell
El cmdlet para obtener el mantenimiento del nodo es [Get-ServiceFabricNodeHealth](https://msdn.microsoft.com/library/mt125937.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx). El siguiente cmdlet obtiene el mantenimiento del nodo mediante directivas de mantenimiento predeterminadas:

```powershell
PS C:\> Get-ServiceFabricNodeHealth _Node_1


NodeName              : _Node_1
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 6
                        SentAt                : 3/22/2016 7:47:56 PM
                        ReceivedAt            : 3/22/2016 7:48:19 PM
                        TTL                   : Infinite
                        Description           : Fabric node is up.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:48:19 PM, LastWarning = 1/1/0001 12:00:00 AM
```

El siguiente cmdlet obtiene el mantenimiento de todos los nodos del clúster:

```powershell
PS C:\> Get-ServiceFabricNode | Get-ServiceFabricNodeHealth | select NodeName, AggregatedHealthState | ft -AutoSize

NodeName AggregatedHealthState
-------- ---------------------
_Node_2                     Ok
_Node_0                     Ok
_Node_1                     Ok
_Node_3                     Ok
_Node_4                     Ok
```

## Obtención del mantenimiento de la aplicación
Este proceso devuelve el mantenimiento de una entidad de aplicación. Contiene los estados de mantenimiento de la aplicación implementada y de los elementos secundarios del servicio. Entrada:

- [Obligatorio] El nombre de la aplicación (URI) que identifica la aplicación.

- [Opcional] La directiva de mantenimiento de aplicación usada para invalidar las directivas de manifiesto de aplicación.

- [Opcional] Filtros para eventos, servicios y aplicaciones implementadas que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos, servicios y aplicaciones implementadas para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento de la aplicación, cree una instancia de `FabricClient` y llame al método [GetApplicationHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getapplicationhealthasync.aspx) en su instancia de HealthManager.

A continuación se obtiene el mantenimiento de la aplicación para el nombre de aplicación especificado (URI):

```csharp
ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(applicationName);
```

A continuación se recibe el mantenimiento de la aplicación para el nombre de aplicación especificado (URI), con filtros y directivas personalizadas especificados mediante [ApplicationHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.applicationhealthquerydescription.aspx).

```csharp
HealthStateFilter warningAndErrors = HealthStateFilter.Error | HealthStateFilter.Warning;
var serviceTypePolicy = new ServiceTypeHealthPolicy()
{
    MaxPercentUnhealthyPartitionsPerService = 0,
    MaxPercentUnhealthyReplicasPerPartition = 5,
    MaxPercentUnhealthyServices = 0,
};
var policy = new ApplicationHealthPolicy()
{
    ConsiderWarningAsError = false,
    DefaultServiceTypeHealthPolicy = serviceTypePolicy,
    MaxPercentUnhealthyDeployedApplications = 0,
};

var queryDescription = new ApplicationHealthQueryDescription(applicationName)
{
    HealthPolicy = policy,
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = warningAndErrors },
    ServicesFilter = new ServiceHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
    DeployedApplicationsFilter = new DeployedApplicationHealthStatesFilter() { HealthStateFilterValue = warningAndErrors },
};

ApplicationHealth applicationHealth = await fabricClient.HealthManager.GetApplicationHealthAsync(queryDescription);
```

### PowerShell
El cmdlet para obtener el mantenimiento de la aplicación es [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

El siguiente cmdlet devuelve el mantenimiento de la aplicación **fabric:/WordCount**:

```powershell
PS c:>
PS C:\WINDOWS\system32>  Get-ServiceFabricApplicationHealth fabric:/WordCount


ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Warning
UnhealthyEvaluations            :
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Warning'.

                                      Unhealthy event: SourceId='System.PLB',
                                  Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  ConsiderWarningAsError=false.

ServiceHealthStates             :
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Warning

                                  ServiceName           : fabric:/WordCount/WordCountWebService
                                  AggregatedHealthState : Ok

DeployedApplicationHealthStates :
                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_0
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_2
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_3
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_4
                                  AggregatedHealthState : Ok

                                  ApplicationName       : fabric:/WordCount
                                  NodeName              : _Node_1
                                  AggregatedHealthState : Ok

HealthEvents                    :
                                  SourceId              : System.CM
                                  Property              : State
                                  HealthState           : Ok
                                  SequenceNumber        : 360
                                  SentAt                : 3/22/2016 7:56:53 PM
                                  ReceivedAt            : 3/22/2016 7:56:53 PM
                                  TTL                   : Infinite
                                  Description           : Application has been created.
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 3/22/2016 7:56:53 PM, LastWarning = 1/1/0001 12:00:00 AM

                                  SourceId              : MyWatchdog
                                  Property              : Availability
                                  HealthState           : Ok
                                  SequenceNumber        : 131031545225930951
                                  SentAt                : 3/22/2016 9:08:42 PM
                                  ReceivedAt            : 3/22/2016 9:08:42 PM
                                  TTL                   : Infinite
                                  Description           : Availability checked successfully, latency ok
                                  RemoveWhenExpired     : False
                                  IsExpired             : False
                                  Transitions           : Error->Ok = 3/22/2016 8:55:39 PM, LastWarning = 1/1/0001 12:00:00 AM
```

El siguiente cmdlet de PowerShell pasa directivas personalizadas. También filtra los elementos secundarios y los eventos.

```powershell
PS C:\> Get-ServiceFabricApplicationHealth -ApplicationName fabric:/WordCount -ConsiderWarningAsError $true -ServicesFilter Error -EventsFilter Error -DeployedApplicationsFilter Error

ApplicationName                 : fabric:/WordCount
AggregatedHealthState           : Error
UnhealthyEvaluations            :
                                  Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy event: SourceId='System.PLB',
                                  Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                                  ConsiderWarningAsError=true.

ServiceHealthStates             :
                                  ServiceName           : fabric:/WordCount/WordCountService
                                  AggregatedHealthState : Error

DeployedApplicationHealthStates : None
HealthEvents                    : None
```

## Obtención del mantenimiento del servicio
Este proceso devuelve el mantenimiento de una entidad del servicio. Contiene los estados de mantenimiento de la partición. Entrada:

- [Obligatorio] El nombre del servicio (URI) que identifica el servicio.

- [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.

- [Opcional] Filtros para eventos y particiones que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos y particiones para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento del servicio mediante la API, cree una instancia de `FabricClient` y llame al método [GetServiceHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getservicehealthasync.aspx) en su instancia de HealthManager.

En el ejemplo siguiente se obtiene el mantenimiento de un servicio con el nombre de servicio especificado (URI):

```charp
ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(serviceName);
```

A continuación se obtiene el mantenimiento del servicio para el nombre de servicio especificado (URI), mediante la especificación de filtros y directivas personalizadas con la clase [ServiceHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.servicehealthquerydescription.aspx):

```csharp
var queryDescription = new ServiceHealthQueryDescription(serviceName)
{
    EventsFilter = new HealthEventsFilter() { HealthStateFilterValue = HealthStateFilter.All },
    PartitionsFilter = new PartitionHealthStatesFilter() { HealthStateFilterValue = HealthStateFilter.Error },
};

ServiceHealth serviceHealth = await fabricClient.HealthManager.GetServiceHealthAsync(queryDescription);
```

### PowerShell
El cmdlet para obtener el mantenimiento del servicio es [Get-ServiceFabricServiceHealth](https://msdn.microsoft.com/library/mt125984.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

El siguiente cmdlet obtiene el mantenimiento del servicio con directivas de mantenimiento predeterminadas.

```powershell
PS C:\> Get-ServiceFabricServiceHealth -ServiceName fabric:/WordCount/WordCountService


ServiceName           : fabric:/WordCount/WordCountService
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.PLB',
                        Property='ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b', HealthState='Warning',
                        ConsiderWarningAsError=false.

PartitionHealthStates :
                        PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        AggregatedHealthState : Warning

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 10
                        SentAt                : 3/22/2016 7:56:53 PM
                        ReceivedAt            : 3/22/2016 7:57:18 PM
                        TTL                   : Infinite
                        Description           : Service has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:18 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.PLB
                        Property              : ServiceReplicaUnplacedHealth_Secondary_a1f83a35-d6bf-4d39-b90d-28d15f39599b
                        HealthState           : Warning
                        SequenceNumber        : 131031547693687021
                        SentAt                : 3/22/2016 9:12:49 PM
                        ReceivedAt            : 3/22/2016 9:12:49 PM
                        TTL                   : 00:01:05
                        Description           : The Load Balancer was unable to find a placement for one or more of the Service's Replicas:
                        fabric:/WordCount/WordCountService Secondary Partition a1f83a35-d6bf-4d39-b90d-28d15f39599b could not be placed, possibly,
                        due to the following constraints and properties:  
                        Placement Constraint: N/A
                        Depended Service: N/A

                        Constraint Elimination Sequence:
                        ReplicaExclusionStatic eliminated 4 possible node(s) for placement -- 1/5 node(s) remain.
                        ReplicaExclusionDynamic eliminated 1 possible node(s) for placement -- 0/5 node(s) remain.

                        Nodes Eliminated By Constraints:

                        ReplicaExclusionStatic:
                        FaultDomain:fd:/0 NodeName:_Node_0 NodeType:NodeType0 UpgradeDomain:0 UpgradeDomain: ud:/0 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/1 NodeName:_Node_1 NodeType:NodeType1 UpgradeDomain:1 UpgradeDomain: ud:/1 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/3 NodeName:_Node_3 NodeType:NodeType3 UpgradeDomain:3 UpgradeDomain: ud:/3 Deactivation Intent/Status:
                        None/None
                        FaultDomain:fd:/4 NodeName:_Node_4 NodeType:NodeType4 UpgradeDomain:4 UpgradeDomain: ud:/4 Deactivation Intent/Status:
                        None/None

                        ReplicaExclusionDynamic:
                        FaultDomain:fd:/2 NodeName:_Node_2 NodeType:NodeType2 UpgradeDomain:2 UpgradeDomain: ud:/2 Deactivation Intent/Status:
                        None/None


                        RemoveWhenExpired     : True
                        IsExpired             : False
```

## Obtención del mantenimiento de partición
Este proceso devuelve el mantenimiento de una entidad de partición. Contiene los estados de mantenimiento de la réplica. Entrada:

- [Obligatorio] El identificador de partición (GUID) que identifica la partición.

- [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.

- [Opcional] Filtros de eventos y réplicas que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos y réplicas para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento de la partición mediante la API, cree una instancia de `FabricClient` y llame al método [GetPartitionHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getpartitionhealthasync.aspx) en su instancia de HealthManager. Para especificar parámetros opcionales, cree [PartitionHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.partitionhealthquerydescription.aspx).

```csharp
PartitionHealth partitionHealth = await fabricClient.HealthManager.GetPartitionHealthAsync(partitionId);
```

### PowerShell
El cmdlet para obtener el mantenimiento de la partición es [Get-ServiceFabricPartitionHealth](https://msdn.microsoft.com/library/mt125869.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

El siguiente cmdlet obtiene el mantenimiento de todas las particiones del servicio **fabric:/WordCount/WordCountService**:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricPartitionHealth


PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
AggregatedHealthState : Warning
UnhealthyEvaluations  :
                        Unhealthy event: SourceId='System.FM', Property='State', HealthState='Warning', ConsiderWarningAsError=false.

ReplicaHealthStates   :
                        ReplicaId             : 131031502143040223
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844060
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844059
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844061
                        AggregatedHealthState : Ok

                        ReplicaId             : 131031502346844058
                        AggregatedHealthState : Ok

HealthEvents          :
                        SourceId              : System.FM
                        Property              : State
                        HealthState           : Warning
                        SequenceNumber        : 76
                        SentAt                : 3/22/2016 7:57:26 PM
                        ReceivedAt            : 3/22/2016 7:57:48 PM
                        TTL                   : Infinite
                        Description           : Partition is below target replica or instance count.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Warning = 3/22/2016 7:57:48 PM, LastOk = 1/1/0001 12:00:00 AM
```

## Obtención del mantenimiento de réplica
Esta operación devuelve el mantenimiento de una réplica de servicio con estado o una instancia de servicio sin estado. Entrada:

- [Obligatorio] El identificador de partición (GUID) y el identificador de réplica que identifican a la réplica

- [Opcional] Los parámetros de la directiva de mantenimiento de aplicación usados para invalidar las directivas de manifiesto de aplicación.

- [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento de la réplica mediante la API, cree una instancia de `FabricClient` y llame al método [GetReplicaHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getreplicahealthasync.aspx) en su instancia de HealthManager. Para especificar parámetros avanzados, utilice [ReplicaHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.replicahealthquerydescription.aspx).

```csharp
ReplicaHealth replicaHealth = await fabricClient.HealthManager.GetReplicaHealthAsync(partitionId, replicaId);
```

### PowerShell
El cmdlet para obtener el mantenimiento de la réplica es [Get-ServiceFabricReplicaHealth](https://msdn.microsoft.com/library/mt125808.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

El cmdlet siguiente obtiene el mantenimiento de la réplica principal para todas las particiones del servicio:

```powershell
PS C:\> Get-ServiceFabricPartition fabric:/WordCount/WordCountService | Get-ServiceFabricReplica | where {$_.ReplicaRole -eq "Primary"} | Get-ServiceFabricReplicaHealth


PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
ReplicaId             : 131031502143040223
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.RA
                        Property              : State
                        HealthState           : Ok
                        SequenceNumber        : 131031502145556748
                        SentAt                : 3/22/2016 7:56:54 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : Replica has been created.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## Obtención del mantenimiento de la aplicación implementada
Este proceso devuelve el mantenimiento de una aplicación implementada en una entidad del nodo. Contiene los estados de mantenimiento del paquete de servicio implementado. Entrada:

- [Obligatorio] El nombre de la aplicación (URI) y el nombre del nodo (cadena) que identifican a la aplicación implementada

- [Opcional] La directiva de mantenimiento de aplicación usada para invalidar las directivas de manifiesto de aplicación.

- [Opcional] Filtros de eventos y paquetes de servicio implementados que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos y paquetes de servicio implementados para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento de una aplicación implementada en un nodo mediante la API, cree una instancia de `FabricClient` y llame al método [GetDeployedApplicationHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getdeployedapplicationhealthasync.aspx) en su instancia de HealthManager. Para especificar parámetros opcionales, utilice [DeployedApplicationHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.deployedapplicationhealthquerydescription.aspx).

```csharp
DeployedApplicationHealth health = await fabricClient.HealthManager.GetDeployedApplicationHealthAsync(
    new DeployedApplicationHealthQueryDescription(applicationName, nodeName));
```

### PowerShell
El cmdlet para obtener el mantenimiento de la aplicación implementada es [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/library/mt163523.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx). Para averiguar dónde se implementa una aplicación, ejecute [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx) y observe los elementos secundarios de la aplicación implementada.

El siguiente cmdlet obtiene el mantenimiento de la aplicación **fabric:/WordCount** implementada en **\_Node\_2**.

```powershell
PS C:\> Get-ServiceFabricDeployedApplicationHealth -ApplicationName fabric:/WordCount -NodeName _Node_2


ApplicationName                    : fabric:/WordCount
NodeName                           : _Node_2
AggregatedHealthState              : Ok
DeployedServicePackageHealthStates :
                                     ServiceManifestName   : WordCountServicePkg
                                     NodeName              : _Node_2
                                     AggregatedHealthState : Ok

                                     ServiceManifestName   : WordCountWebServicePkg
                                     NodeName              : _Node_2
                                     AggregatedHealthState : Ok

HealthEvents                       :
                                     SourceId              : System.Hosting
                                     Property              : Activation
                                     HealthState           : Ok
                                     SequenceNumber        : 131031502143710698
                                     SentAt                : 3/22/2016 7:56:54 PM
                                     ReceivedAt            : 3/22/2016 7:57:12 PM
                                     TTL                   : Infinite
                                     Description           : The application was activated successfully.
                                     RemoveWhenExpired     : False
                                     IsExpired             : False
                                     Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## Obtención del mantenimiento del paquete de servicio implementado
Este proceso devuelve el mantenimiento de una entidad de paquete de servicio implementada. Entrada:

- [Obligatorio] El nombre de la aplicación (URI), el nombre del nodo (cadena) y el nombre del manifiesto de servicio (cadena) que identifican al paquete de servicio implementado.

- [Opcional] La directiva de mantenimiento de aplicación usada para invalidar la directiva de manifiesto de aplicación.

- [Opcional] Filtros para eventos que especifican las entradas que son de interés y se deben devolver en el resultado (por ejemplo, únicamente errores o advertencias y errores). Tenga en cuenta que se utilizan todos los eventos para evaluar el mantenimiento agregado de la entidad, independientemente del filtro.

### API
Para obtener el mantenimiento de un paquete de servicio implementado mediante la API, cree una instancia de `FabricClient` y llame al método [GetDeployedServicePackageHealthAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getdeployedservicepackagehealthasync.aspx) en su instancia de HealthManager. Para especificar parámetros opcionales, utilice [DeployedServicePackageHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.deployedservicepackagehealthquerydescription.aspx).

```csharp
DeployedServicePackageHealth health = await fabricClient.HealthManager.GetDeployedServicePackageHealthAsync(
    new DeployedServicePackageHealthQueryDescription(applicationName, nodeName, serviceManifestName));
```

### PowerShell
El cmdlet para obtener el mantenimiento del paquete de servicio implementado es [Get-ServiceFabricDeployedServicePackageHealth](https://msdn.microsoft.com/library/mt163525.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx). Para averiguar dónde se implementa una aplicación, ejecute [Get-ServiceFabricApplicationHealth](https://msdn.microsoft.com/library/mt125976.aspx) y examine las aplicaciones implementadas. Para ver qué paquetes de servicio están en una aplicación, examine los elementos secundarios del paquete de servicio implementado en la salida de [Get-ServiceFabricDeployedApplicationHealth](https://msdn.microsoft.com/library/mt163523.aspx).

El siguiente cmdlet obtiene el mantenimiento del paquete de servicio **WordCountServicePkg** de la aplicación **fabric:/WordCount** implementada en **\_Node\_2**. La entidad tiene informes **System.Hosting** para la activación correcta del paquete de servicio y el punto de entrada, y para el registro del tipo de servicio correcto.

```powershell
PS C:\> Get-ServiceFabricDeployedApplication -ApplicationName fabric:/WordCount -NodeName _Node_2 | Get-ServiceFabricDeployedServicePackageHealth -ServiceManifestName WordCountServicePkg


ApplicationName       : fabric:/WordCount
ServiceManifestName   : WordCountServicePkg
NodeName              : _Node_2
AggregatedHealthState : Ok
HealthEvents          :
                        SourceId              : System.Hosting
                        Property              : Activation
                        HealthState           : Ok
                        SequenceNumber        : 131031502301306211
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServicePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : CodePackageActivation:Code:EntryPoint
                        HealthState           : Ok
                        SequenceNumber        : 131031502301568982
                        SentAt                : 3/22/2016 7:57:10 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The CodePackage was activated successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM

                        SourceId              : System.Hosting
                        Property              : ServiceTypeRegistration:WordCountServiceType
                        HealthState           : Ok
                        SequenceNumber        : 131031502314788519
                        SentAt                : 3/22/2016 7:57:11 PM
                        ReceivedAt            : 3/22/2016 7:57:12 PM
                        TTL                   : Infinite
                        Description           : The ServiceType was registered successfully.
                        RemoveWhenExpired     : False
                        IsExpired             : False
                        Transitions           : Error->Ok = 3/22/2016 7:57:12 PM, LastWarning = 1/1/0001 12:00:00 AM
```

## Consultas de fragmentos de mantenimiento
Las consultas de fragmentos de mantenimiento pueden devolver elementos secundarios de clúster de varios niveles (recursivamente), según los filtros de entrada. Admiten filtros avanzados que permiten una gran flexibilidad para expresar los elementos secundarios específicos que se devolverán, identificados por su identificador único u otro identificador de grupo o estado de mantenimiento. De forma predeterminada, no se incluye ningún elemento secundario, por oposición a los comandos de mantenimiento que siempre incluyen elementos secundario de primer nivel.

Las [consultas de mantenimiento](service-fabric-view-entities-aggregated-health.md#health-queries) solo devuelven elementos secundarios de primer nivel de la entidad especificada, según los filtros necesarios. Para obtener los elementos secundarios de los elementos secundarios, los usuarios deben llamar a API de mantenimiento adicionales para cada entidad que sea de interés. De igual forma, para obtener el mantenimiento de entidades específicas, los usuarios deben llamar a una API de mantenimiento para cada entidad deseada. El filtrado avanzado de consultas de fragmentos permite a los usuarios solicitar varios elementos de interés en una consulta, lo que reduce el tamaño del mensaje y el número de mensajes.

El valor de la consulta de fragmento es que los usuarios pueden obtener el estado de mantenimiento para más entidades del clúster (posiblemente todas las entidades del clúster que comienzan en la raíz necesaria) en una llamada. Puede expresar consultas de mantenimiento complejas, por ejemplo:

- Devolver solo las aplicaciones con error y, para esas aplicaciones, incluir todos los servicios con advertencias o errores. Para los servicios devueltos, incluir todas las particiones.

- Devolver solo el mantenimiento de cuatro aplicaciones, especificadas por sus nombres.

- Devolver solo el mantenimiento de aplicaciones de un tipo de aplicación deseado.

- Devolver todas las entidades implementadas en un nodo. Se devuelven todas las aplicaciones, todas las aplicaciones implementadas en el nodo especificado y todos los paquetes de servicio implementados en ese nodo.

- Devolver todas las réplicas con error. Se devuelven todas las aplicaciones, los servicios, las particiones y solo las réplicas con error.

- Devolver todas las aplicaciones. Para un servicio especificado, incluir todas las particiones.

Actualmente, la consulta de fragmentos de mantenimiento se expone solo para la entidad del clúster. Devuelve un fragmento de estado del clúster, que contiene:

- El estado de mantenimiento agregado del clúster.

- La lista de fragmentos de estado de mantenimiento de nodos que respetan los filtros de entrada.

- La lista de fragmentos de estado de mantenimiento de aplicaciones que respetan los filtros de entrada. Cada fragmento de estado de mantenimiento de aplicación contiene una lista de fragmentos con todos los servicios que respetan los filtros de entrada y una lista de fragmentos con todas las aplicaciones implementadas que respetan los filtros. Igual para los elementos secundarios de servicios y aplicaciones implementadas. De este modo, todas las entidades del clúster pueden devolverse potencialmente si se solicitan, de una manera jerárquica.

### Consulta de fragmentos de mantenimiento del clúster
Esta acción devuelve el mantenimiento de la entidad del clúster y contiene los fragmentos de estado de mantenimiento jerárquico de los elementos secundarios necesarios. Entrada:

- [Opcional] La directiva de mantenimiento del clúster utilizada para evaluar los nodos y los eventos del clúster.

- [Opcional] La asignación de directivas de mantenimiento de aplicaciones, con las directivas de mantenimiento usadas para invalidar las directivas de manifiesto de aplicación.

- [Opcional] Filtros para nodos y aplicaciones que especifican las entradas que son de interés y se deben devolver en el resultado. Los filtros son específicos de una entidad o grupo de entidades o son aplicables a todas las entidades de ese nivel. La lista de filtros puede contener un filtro general o filtros para identificadores específicos, para entidades específicas devueltas por la consulta. Si está vacía, no se devuelven los elementos secundarios de forma predeterminada. Más información sorbe los filtros en [NodeHealthStateFilter](https://msdn.microsoft.com/library/azure/system.fabric.health.nodehealthstatefilter.aspx) y [ApplicationHealthStateFilter](https://msdn.microsoft.com/library/azure/system.fabric.health.applicationhealthstatefilter.aspx). Los filtros de aplicación pueden especificar de forma recursiva filtros avanzados para elementos secundarios.

El resultado del fragmento incluye los elementos secundarios que respetan los filtros.

Actualmente, la consulta de fragmentos no devuelve evaluaciones de mantenimiento incorrecto o eventos de entidad. Estos pueden obtenerse mediante la consulta de mantenimiento del clúster existente.

### API
Para obtener el fragmento de mantenimiento del clúster, cree una instancia de `FabricClient` y llame al método [GetClusterHealthChunkAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.healthclient.getclusterhealthchunkasync.aspx) en su instancia de **HealthManager**. Puede pasar [ClusterHealthQueryDescription](https://msdn.microsoft.com/library/azure/system.fabric.description.clusterhealthchunkquerydescription.aspx) para describir directivas de mantenimiento y filtros avanzados.

A continuación se obtiene un fragmento de mantenimiento del clúster con filtros avanzados.

```csharp
var queryDescription = new ClusterHealthChunkQueryDescription();
queryDescription.ApplicationFilters.Add(new ApplicationHealthStateFilter()
    {
        // Return applications only if they are in error
        HealthStateFilter = HealthStateFilter.Error
    });

// Return all replicas
var wordCountServiceReplicaFilter = new ReplicaHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };

// Return all replicas and all partitions
var wordCountServicePartitionFilter = new PartitionHealthStateFilter()
    {
        HealthStateFilter = HealthStateFilter.All
    };
wordCountServicePartitionFilter.ReplicaFilters.Add(wordCountServiceReplicaFilter);

// For specific service, return all partitions and all replicas
var wordCountServiceFilter = new ServiceHealthStateFilter()
{
    ServiceNameFilter = new Uri("fabric:/WordCount/WordCountService"),
};
wordCountServiceFilter.PartitionFilters.Add(wordCountServicePartitionFilter);

// Application filter: for specific application, return no services except the ones of interest
var wordCountApplicationFilter = new ApplicationHealthStateFilter()
    {
        // Always return fabric:/WordCount application
        ApplicationNameFilter = new Uri("fabric:/WordCount"),
    };
wordCountApplicationFilter.ServiceFilters.Add(wordCountServiceFilter);

queryDescription.ApplicationFilters.Add(wordCountApplicationFilter);

var result = await fabricClient.HealthManager.GetClusterHealthChunkAsync(queryDescription);
```

### PowerShell
El cmdlet para obtener el mantenimiento del clúster es [Get-ServiceFabricClusterChunkHealth](https://msdn.microsoft.com/library/mt644772.aspx). Conéctese primero al clúster mediante el cmdlet [Connect-ServiceFabricCluster](https://msdn.microsoft.com/library/mt125938.aspx).

A continuación se obtienen los nodos solo si se encuentran en estado de error, a excepción de un nodo específico, que se debe devolver siempre.

```xml
PS C:\> $errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$nodeFilter1 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{HealthStateFilter=$errorFilter}
$nodeFilter2 = New-Object System.Fabric.Health.NodeHealthStateFilter -Property @{NodeNameFilter="_Node_1";HealthStateFilter=$allFilter}
# Create node filter list that will be passed in the cmdlet
$nodeFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.NodeHealthStateFilter]
$nodeFilters.Add($nodeFilter1)
$nodeFilters.Add($nodeFilter2)

Get-ServiceFabricClusterHealthChunk -NodeFilters $nodeFilters

HealthState                  : Error
NodeHealthStateChunks        :
                               TotalCount            : 1

                               NodeName              : _Node_1
                               HealthState           : Ok

ApplicationHealthStateChunks : None
```

El siguiente cmdlet obtiene fragmentos del clúster con filtros de aplicación.

```xml
$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

# All replicas
$replicaFilter = New-Object System.Fabric.Health.ReplicaHealthStateFilter -Property @{HealthStateFilter=$allFilter}

# All partitions
$partitionFilter = New-Object System.Fabric.Health.PartitionHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$partitionFilter.ReplicaFilters.Add($replicaFilter)

# For WordCountService, return all partitions and all replicas
$svcFilter1 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{ServiceNameFilter="fabric:/WordCount/WordCountService"}
$svcFilter1.PartitionFilters.Add($partitionFilter)

$svcFilter2 = New-Object System.Fabric.Health.ServiceHealthStateFilter -Property @{HealthStateFilter=$errorFilter}

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{ApplicationNameFilter="fabric:/WordCount"}
$appFilter.ServiceFilters.Add($svcFilter1)
$appFilter.ServiceFilters.Add($svcFilter2)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)

Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters

HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks :
                               TotalCount            : 1

                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               ServiceHealthStateChunks :
                                   TotalCount            : 1

                                   ServiceName           : fabric:/WordCount/WordCountService
                                   HealthState           : Error
                                   PartitionHealthStateChunks :
                                       TotalCount            : 1

                                       PartitionId           : a1f83a35-d6bf-4d39-b90d-28d15f39599b
                                       HealthState           : Error
                                       ReplicaHealthStateChunks :
                                           TotalCount            : 5

                                           ReplicaOrInstanceId   : 131031502143040223
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844060
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844059
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844061
                                           HealthState           : Ok

                                           ReplicaOrInstanceId   : 131031502346844058
                                           HealthState           : Error
```

El siguiente cmdlet devuelve todas las entidades implementadas en un nodo.

```xml
$errorFilter = [System.Fabric.Health.HealthStateFilter]::Error;
$allFilter = [System.Fabric.Health.HealthStateFilter]::All;

$dspFilter = New-Object System.Fabric.Health.DeployedServicePackageHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$daFilter =  New-Object System.Fabric.Health.DeployedApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter;NodeNameFilter="_Node_2"}
$daFilter.DeployedServicePackageFilters.Add($dspFilter)

$appFilter = New-Object System.Fabric.Health.ApplicationHealthStateFilter -Property @{HealthStateFilter=$allFilter}
$appFilter.DeployedApplicationFilters.Add($daFilter)

$appFilters = New-Object System.Collections.Generic.List[System.Fabric.Health.ApplicationHealthStateFilter]
$appFilters.Add($appFilter)
Get-ServiceFabricClusterHealthChunk -ApplicationFilters $appFilters


HealthState                  : Error
NodeHealthStateChunks        : None
ApplicationHealthStateChunks :
                               TotalCount            : 2

                               ApplicationName       : fabric:/System
                               HealthState           : Ok
                               DeployedApplicationHealthStateChunks :
                                   TotalCount            : 1

                                   NodeName              : _Node_2
                                   HealthState           : Ok
                                   DeployedServicePackageHealthStateChunks :
                                       TotalCount            : 1

                                       ServiceManifestName   : FAS
                                       HealthState           : Ok



                               ApplicationName       : fabric:/WordCount
                               ApplicationTypeName   : WordCount
                               HealthState           : Error
                               DeployedApplicationHealthStateChunks :
                                   TotalCount            : 1

                                   NodeName              : _Node_2
                                   HealthState           : Ok
                                   DeployedServicePackageHealthStateChunks :
                                       TotalCount            : 2

                                       ServiceManifestName   : WordCountServicePkg
                                       HealthState           : Ok

                                       ServiceManifestName   : WordCountWebServicePkg
                                       HealthState           : Ok
```

## Consultas generales
Las consultas generales devuelven una lista de entidades de Service Fabric de un tipo especificado. Se exponen mediante la API (por medio de los métodos de **FabricClient.QueryManager**), los cmdlets de PowerShell y REST. Estas consultas agregan subconsultas de varios componentes. Uno de ellos es el [almacén de estado](service-fabric-health-introduction.md#health-store), que rellena el estado de mantenimiento agregado para cada resultado de consulta.

> [AZURE.NOTE] Las consultas generales devuelven el estado de mantenimiento agregado de la entidad y no contienen los datos completos de mantenimiento. Si el mantenimiento de una entidad no es correcto, puede seguir con las consultas de mantenimiento para obtener toda su información de mantenimiento, como eventos, estados de mantenimiento de los elementos secundarios y evaluaciones de mantenimiento incorrecto.

Si las consultas generales devuelven un estado de mantenimiento desconocido para una entidad, es posible que el almacén de estado no tenga datos completos sobre la entidad. También es posible que una subconsulta al almacén de estado no fuera correcta (por ejemplo, se produjo un error de comunicación o se limitó el almacén de estado). Realice un seguimiento con una consulta de mantenimiento para la entidad. Si la subconsulta se encontró con errores transitorios, como problemas de red, esta consulta de seguimiento puede ser de ayuda. También le puede proporcionar más detalles del almacén de estado sobre por qué no se expone la entidad.

Las consultas que contienen **HealthState** para las entidades son las siguientes:

- Lista de nodos: devuelve los nodos de la lista del clúster (paginada).
  - API: [FabricClient.QueryClient.GetNodeListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getnodelistasync.aspx)
  - PowerShell: Get-ServiceFabricNode.
- Lista de aplicaciones: devuelve la lista de aplicaciones del clúster (paginada).
  - API: [FabricClient.QueryClient.GetApplicationListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getapplicationlistasync.aspx)
  - PowerShell: Get-ServiceFabricApplication.
- Lista de servicios: devuelve la lista de servicios de una aplicación (paginada).
  - API: [FabricClient.QueryClient.GetServiceListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getservicelistasync.aspx)
  - PowerShell: Get-ServiceFabricService.
- Lista de particiones: devuelve la lista de particiones de un servicio (paginada).
  - API: [FabricClient.QueryClient.GetPartitionListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getpartitionlistasync.aspx)
  - PowerShell: Get-ServiceFabricPartition.
- Lista de réplicas: devuelve la lista de réplicas de una partición (paginada).
  - API: [FabricClient.QueryClient.GetReplicaListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getreplicalistasync.aspx)
  - PowerShell: Get-ServiceFabricReplica.
- Lista de aplicaciones implementadas: devuelve la lista de aplicaciones implementadas en un nodo.
  - API: [FabricClient.QueryClient.GetDeployedApplicationListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getdeployedapplicationlistasync.aspx)
  - PowerShell: Get-ServiceFabricDeployedApplication.
- Lista de paquetes de servicio implementados: devuelve la lista de paquetes de servicio de una aplicación implementada.
  - API: [FabricClient.QueryClient.GetDeployedServicePackageListAsync](https://msdn.microsoft.com/library/azure/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync.aspx)
  - PowerShell: Get-ServiceFabricDeployedApplication.

> [AZURE.NOTE] Algunas de las consultas devuelven resultados paginados. La devolución de estas consultas es una lista que se deriva de [PagedList<T>](https://msdn.microsoft.com/library/azure/mt280056.aspx). Si los resultados no caben en un mensaje, solo se devuelve una página y se establece ContinuationToken para realizar el seguimiento de dónde se detuvo la enumeración. El usuario debe continuar llamando a la misma consulta y pasar el token de continuación de la consulta anterior para obtener los siguientes resultados.

### Ejemplos

Lo siguiente obtiene las aplicaciones cuyo mantenimiento es incorrecto en el clúster:

```csharp
var applications = fabricClient.QueryManager.GetApplicationListAsync().Result.Where(
  app => app.HealthState == HealthState.Error);
```

El siguiente cmdlet obtiene los detalles de la aplicación fabric:/WordCount. Observe que el estado de mantenimiento está en advertencia.

```powershell
PS C:\> Get-ServiceFabricApplication -ApplicationName fabric:/WordCount

ApplicationName        : fabric:/WordCount
ApplicationTypeName    : WordCount
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Warning
ApplicationParameters  : { "WordCountWebService_InstanceCount" = "1";
                         "_WFDebugParams_" = "[{"ServiceManifestName":"WordCountWebServicePkg","CodePackageName":"Code","EntryPointType":"Main","Debug
                         ExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {74f7e5d5-71a9-47e2-a8cd-1878ec4734f1} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"},{"ServiceManifestName":"WordCountServicePkg","CodeP
                         ackageName":"Code","EntryPointType":"Main","DebugExePath":"C:\\Program Files (x86)\\Microsoft Visual Studio
                         14.0\\Common7\\Packages\\Debugger\\VsDebugLaunchNotify.exe","DebugArguments":" {2ab462e6-e0d1-4fda-a844-972f561fe751} -p
                         [ProcessId] -tid [ThreadId]","EnvironmentBlock":"_NO_DEBUG_HEAP=1\u0000"}]" }
```

El siguiente cmdlet obtiene los servicios con un estado de mantenimiento de advertencia:

```powershell
PS C:\> Get-ServiceFabricApplication | Get-ServiceFabricService | where {$_.HealthState -eq "Warning"}


ServiceName            : fabric:/WordCount/WordCountService
ServiceKind            : Stateful
ServiceTypeName        : WordCountServiceType
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
HasPersistedState      : True
ServiceStatus          : Active
HealthState            : Warning
```

## Actualización de clústeres y aplicaciones
Durante una actualización supervisada del clúster y la aplicación, Service Fabric comprueba el mantenimiento para asegurarse de que todo está correcto. Si una entidad es incorrecta según se ha evaluado mediante las directivas de mantenimiento configuradas, la actualización aplica directivas específicas de actualización para determinar la próxima acción. La actualización se puede poner en pausa para permitir la interacción de los usuarios (por ejemplo, corregir las condiciones de error o cambiar las directivas), o se puede revertir automáticamente a la versión buena anterior.

Durante la actualización de un *clúster*, puede obtener el estado de actualización del clúster. Esto incluirá las evaluaciones de mantenimiento incorrecto, que señalan lo que está mal en el clúster. Si se revierte la actualización debido a problemas de mantenimiento, el estado de actualización conserva las razones del último mantenimiento incorrecto. De esta forma se dispone de información que puede ayudar a los administradores a investigar qué salió mal.

De igual forma, durante la actualización de una *aplicación*, las evaluaciones de mantenimiento incorrecto están contenidas en el estado de actualización de la aplicación.

A continuación se muestra el estado de actualización de la aplicación para una aplicación fabric/WordCount modificada. Un guardián ha informado de un error en una de sus réplicas. La actualización se revierte porque no se respetan las comprobaciones de mantenimiento.

```powershell
PS C:\> Get-ServiceFabricApplicationUpgrade fabric:/WordCount

ApplicationName               : fabric:/WordCount
ApplicationTypeName           : WordCount
TargetApplicationTypeVersion  : 1.0.0.0
ApplicationParameters         : {}
StartTimestampUtc             : 4/21/2015 5:23:26 PM
FailureTimestampUtc           : 4/21/2015 5:23:37 PM
FailureReason                 : HealthCheck
UpgradeState                  : RollingBackInProgress
UpgradeDuration               : 00:00:23
CurrentUpgradeDomainDuration  : 00:00:00
CurrentUpgradeDomainProgress  : UD1

                                NodeName            : _Node_1
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_2
                                UpgradePhase        : Upgrading

                                NodeName            : _Node_3
                                UpgradePhase        : PreUpgradeSafetyCheck
                                PendingSafetyChecks :
                                EnsurePartitionQuorum - PartitionId: 30db5be6-4e20-4698-8185-4bd7ca744020
NextUpgradeDomain             : UD2
UpgradeDomainsStatus          : { "UD1" = "Completed";
                                "UD2" = "Pending";
                                "UD3" = "Pending";
                                "UD4" = "Pending" }
UnhealthyEvaluations          :
                                Unhealthy services: 100% (1/1), ServiceType='WordCountServiceType', MaxPercentUnhealthyServices=0%.

                                  Unhealthy service: ServiceName='fabric:/WordCount/WordCountService', AggregatedHealthState='Error'.

                                      Unhealthy partitions: 100% (1/1), MaxPercentUnhealthyPartitionsPerService=0%.

                                      Unhealthy partition: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b', AggregatedHealthState='Error'.

                                          Unhealthy replicas: 20% (1/5), MaxPercentUnhealthyReplicasPerPartition=0%.

                                          Unhealthy replica: PartitionId='a1f83a35-d6bf-4d39-b90d-28d15f39599b',
                                  ReplicaOrInstanceId='131031502346844058', AggregatedHealthState='Error'.

                                              Error event: SourceId='DiskWatcher', Property='Disk'.

UpgradeKind                   : Rolling
RollingUpgradeMode            : UnmonitoredAuto
ForceRestart                  : False
UpgradeReplicaSetCheckTimeout : 00:15:00
```

Más información sobre la [actualización de aplicaciones de Service Fabric](service-fabric-application-upgrade.md).

## Uso de las evaluaciones de mantenimiento para solucionar problemas
Siempre que haya un problema con el clúster o una aplicación, consulte el mantenimiento del clúster o de la aplicación para analizar lo que pasa. Las evaluaciones de mantenimiento incorrecto proporcionarán detalles sobre lo que activó el estado incorrecto actual. Si lo necesita, puede explorar en profundidad las entidades secundarias incorrectas para identificar la causa principal.

> [AZURE.NOTE] Las evaluaciones de mantenimiento incorrecto muestran la razón principal de que la entidad se evaluara con el estado de mantenimiento actual. Puede haber muchos otros eventos que desencadenen este estado, pero no se reflejarán en las evaluaciones. Deberá explorar en profundidad las entidades de mantenimiento para averiguar todos los informes de mantenimiento incorrecto en el clúster.

## Pasos siguientes
[Utilización de informes de mantenimiento del sistema para solucionar problemas](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)

[Incorporación de informes de mantenimiento de Service Fabric personalizados](service-fabric-report-health.md)

[Supervisión y diagnóstico de los servicios localmente](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Actualización de la aplicación de Service Fabric](service-fabric-application-upgrade.md)

<!---HONumber=AcomDC_0518_2016-->