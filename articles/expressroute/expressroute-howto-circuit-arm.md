<properties
   pageTitle="Creación y modificación de un circuito ExpressRoute mediante Resource Manager y PowerShell | Microsoft Azure"
   description="Este artículo describe cómo crear, aprovisionar, comprobar, actualizar, eliminar y desaprovisionar un circuito ExpressRoute."
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/15/2016"
   ms.author="ganesr"/>


# Creación y modificación de un circuito ExpressRoute

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](expressroute-howto-circuit-portal-resource-manager.md)
[PowerShell - Resource Manager](expressroute-howto-circuit-arm.md)
[PowerShell - Classic](expressroute-howto-circuit-classic.md)


En este artículo se describe cómo crear un circuito Azure ExpressRoute mediante los cmdlets de Windows PowerShell y el modelo de implementación de Azure Resource Manager. En este artículo también se muestra cómo comprobar el estado del circuito, actualizarlo, o eliminarlo y desaprovisionarlo.

**Información acerca de los modelos de implementación de Azure**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## Antes de empezar


- Obtenga la versión más reciente de los módulos de Azure PowerShell (como mínimo, la versión 1.0). Para obtener instrucciones detalladas sobre cómo configurar el equipo para usar los módulos de Azure PowerShell, siga las instrucciones de la página [Cómo instalar y configurar Azure PowerShell](../powershell-install-configure.md).

- Revise los [Requisitos previos y lista de comprobación de ExpressRoute](expressroute-prerequisites.md) y los [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md) antes de comenzar la configuración.

## Creación y aprovisionamiento de un circuito ExpressRoute

### 1\. Iniciar sesión en la cuenta de Azure y seleccione la suscripción

Para empezar la configuración, inicie sesión en la cuenta de Azure. Para obtener más información sobre PowerShell, consulte [Uso de Azure PowerShell con Azure Resource Manager](../powershell-azure-resource-manager.md). Use los siguientes ejemplos para conectarse:

	Login-AzureRmAccount

Compruebe las suscripciones para la cuenta:

	Get-AzureRmSubscription

Seleccione la suscripción para la que desea crear un circuito ExpressRoute:

	Select-AzureRmSubscription -SubscriptionId "<subscription ID>"

### 2\. Obtención de la lista de proveedores, ubicaciones y anchos de banda admitidos

Para crear un circuito ExpressRoute, necesita la lista de proveedores de conectividad, ubicaciones y opciones de ancho de banda admitidas.

El cmdlet de PowerShell `Get-AzureRmExpressRouteServiceProvider` devuelve esta información, que se usará en pasos posteriores:

	Get-AzureRmExpressRouteServiceProvider

Compruebe si aparece su proveedor de conectividad. Tome nota de la siguiente información, porque la necesitará más adelante cuando cree un circuito:

- Nombre

- PeeringLocations

- BandwidthsOffered

Ahora está listo para crear un circuito ExpressRoute.

### 3\. Creación de un circuito ExpressRoute

Si todavía no tiene un grupo de recursos, debe crear uno antes de crear ExpressRoute. Para ello, ejecute el siguiente comando:


	New-AzureRmResourceGroup -Name "ExpressRouteResourceGroup" -Location "West US"


En el ejemplo siguiente se muestra cómo crear un circuito ExpressRoute de 200 Mbps a través de Equinix en Silicon Valley. Si usa otro proveedor y otra configuración, sustituya esa información al realizar la solicitud. A continuación se muestra un ejemplo de solicitud para una nueva clave de servicio:

	New-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "West US" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Equinix" -PeeringLocation "Silicon Valley" -BandwidthInMbps 200

Asegúrese de que especifica el nivel y la familia correctos de SKU.

- El nivel de SKU determina si está habilitado un complemento estándar o premium de ExpressRoute. Puede especificar *Estándar* para obtener la SKU estándar o *Premium* si quiere el complemento Premium.

- La familia de SKU determina el tipo de facturación. Puede seleccionar *Metereddata* para el plan de datos limitado y *Unlimiteddata* para el plan de datos ilimitado. Tenga en cuenta que puede cambiar el tipo de facturación de *Metereddata* a *Unlimiteddata*, pero no se puede cambiar el tipo de *Unlimiteddata* a *Metereddata*.


>[AZURE.IMPORTANT] El circuito ExpressRoute se facturará a partir del momento en que se emita una clave de servicio. Asegúrese de realizar esta operación cuando el proveedor de conectividad esté listo para aprovisionar el circuito.

La respuesta contiene la clave del servicio. Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:


	get-help New-AzureRmExpressRouteCircuit -detailed


### 4\. Lista de todos los circuitos ExpressRoute

Para obtener una lista de todos los circuitos ExpressRoute que haya creado, ejecute el comando `Get-AzureRmExpressRouteCircuit`:


	Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

La respuesta será similar al ejemplo siguiente:


	Name                             : ExpressRouteARMCircuit
	ResourceGroupName                : ExpressRouteResourceGroup
	Location                         : westus
	Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
	Etag                             : W/"################################"
	ProvisioningState                : Succeeded
	Sku                              : {
	                                     "Name": "Standard_MeteredData",
	                                     "Tier": "Standard",
	                                     "Family": "MeteredData"
	   		                           }
	CircuitProvisioningState          : Enabled
	ServiceProviderProvisioningState  : NotProvisioned
	ServiceProviderNotes              :
	ServiceProviderProperties         : {
	                                      "ServiceProviderName": "Equinix",
	                                      "PeeringLocation": "Silicon Valley",
	                                      "BandwidthInMbps": 200
	                                    }
	ServiceKey                        : **************************************
	Peerings                          : []

Esta información se puede recuperar en cualquier momento con el cmdlet `Get-AzureRmExpressRouteCircuit`. Si se realiza la llamada sin parámetros, se obtendrá una lista de todos los circuitos. La clave de servicio se mostrará en el campo *ServiceKey*:


	Get-AzureRmExpressRouteCircuit


La respuesta será similar al ejemplo siguiente:


	Name                             : ExpressRouteARMCircuit
	ResourceGroupName                : ExpressRouteResourceGroup
	Location                         : westus
	Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
	Etag                             : W/"################################"
	ProvisioningState                : Succeeded
	Sku                              : {
	                                     "Name": "Standard_MeteredData",
	                                     "Tier": "Standard",
	                                     "Family": "MeteredData"
	   		                           }
	CircuitProvisioningState         : Enabled
	ServiceProviderProvisioningState : NotProvisioned
	ServiceProviderNotes             :
	ServiceProviderProperties        : {
	                                     "ServiceProviderName": "Equinix",
	                                     "PeeringLocation": "Silicon Valley",
	                                     "BandwidthInMbps": 200
	   		                           }
	ServiceKey                       : **************************************
	Peerings                         : []


Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:


	get-help Get-AzureRmExpressRouteCircuit -detailed

### 5\. Envío de la clave de servicio al proveedor de conectividad para el aprovisionamiento

*ServiceProviderProvisioningState* da información sobre el estado actual del aprovisionamiento en el lado del proveedor de servicios. "Status" proporciona el estado relativo al lado de Microsoft. Para más información sobre los estados de aprovisionamiento del circuito, consulte el artículo [Flujos de trabajo de ExpressRoute para aprovisionamiento de circuitos y estados de circuitos de ExpressRoute](expressroute-workflows.md#expressroute-circuit-provisioning-states).

Cuando se crea un nuevo circuito ExpressRoute, dicho circuito estará en el siguiente estado:


	ServiceProviderProvisioningState : NotProvisioned
	CircuitProvisioningState         : Enabled



El circuito cambiará al estado siguiente cuando el proveedor de conectividad se encuentre en el proceso de habilitarlo:

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

Para poder usar un circuito ExpressRoute, dicho circuito tiene que estar en el siguiente estado.

	ServiceProviderProvisioningState : Provisioned
	CircuitProvisioningState         : Enabled

### 6\. Comprobación periódica del estado y la condición de la clave del circuito

La comprobación del estado y la condición de la clave de circuito le informa cuando el proveedor ha habilitado el circuito. Después de configurar el circuito, *ServiceProviderProvisioningState* aparece como *Provisioned*, tal como se muestra en el ejemplo siguiente:


	Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"


La respuesta será similar al ejemplo siguiente:


	Name                             : ExpressRouteARMCircuit
	ResourceGroupName                : ExpressRouteResourceGroup
	Location                         : westus
	Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
	Etag                             : W/"################################"
	ProvisioningState                : Succeeded
	Sku                              : {
	                                     "Name": "Standard_MeteredData",
	                                     "Tier": "Standard",
	                                     "Family": "MeteredData"
	                                   }
	CircuitProvisioningState         : Enabled
	ServiceProviderProvisioningState : Provisioned
	ServiceProviderNotes             :
	ServiceProviderProperties        : {
	                                     "ServiceProviderName": "Equinix",
	                                     "PeeringLocation": "Silicon Valley",
	                                     "BandwidthInMbps": 200
	   	                            }
	ServiceKey                       : **************************************
	Peerings                         : []

### 7\. Creación de la configuración de enrutamiento

Consulte [Creación y modificación del enrutamiento de un circuito ExpressRoute mediante PowerShell](expressroute-howto-routing-arm.md) para ver las instrucciones paso a paso.


>[AZURE.IMPORTANT] Estas instrucciones se aplican solo a los circuitos creados con proveedores de servicios que ofrecen servicios de conectividad de nivel 2. Si usa un proveedor de servicios que ofrece servicios administrados de nivel 3 (normalmente VPN IP, como MPLS), el mismo proveedor de conectividad configurará y administrará el enrutamiento.

### 8\. Vinculación de una red virtual a un circuito ExpressRoute

A continuación, vincule una red virtual a su circuito ExpressRoute. Consulte el artículo [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-arm.md) al trabajar con el modelo de implementación de Resource Manager.

## Obtención del estado de un circuito ExpressRoute

Esta información se puede recuperar en cualquier momento con el cmdlet `Get-AzureRmExpressRouteCircuit`. Si se realiza la llamada sin parámetros, se obtendrá una lista de todos los circuitos.

	Get-AzureRmExpressRouteCircuit


La respuesta será similar al siguiente ejemplo:


	Name                             : ExpressRouteARMCircuit
	ResourceGroupName                : ExpressRouteResourceGroup
	Location                         : westus
	Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
	Etag                             : W/"################################"
	ProvisioningState                : Succeeded
	Sku                              : {
	                                     "Name": "Standard_MeteredData",
	                                     "Tier": "Standard",
	                                     "Family": "MeteredData"
	                                   }
	CircuitProvisioningState         : Enabled
	ServiceProviderProvisioningState : Provisioned
	ServiceProviderNotes             :
	ServiceProviderProperties        : {
	   		                             "ServiceProviderName": "Equinix",
	   		                             "PeeringLocation": "Silicon Valley",
	   		                             "BandwidthInMbps": 200
	   		                           }
	ServiceKey                       : **************************************
	Peerings                         : []


Se puede obtener información sobre un circuito ExpressRoute específico si se pasa el nombre del grupo de recursos y el nombre del circuito como parámetro a la llamada:


	Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"


La respuesta será similar al ejemplo siguiente:


	Name                             : ExpressRouteARMCircuit
	ResourceGroupName                : ExpressRouteResourceGroup
	Location                         : westus
	Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
	Etag                             : W/"################################"
	ProvisioningState                : Succeeded
	Sku                              : {
	                                     "Name": "Standard_MeteredData",
	   		                             "Tier": "Standard",
	   		                             "Family": "MeteredData"
	   		                           }
	CircuitProvisioningState         : Enabled
	ServiceProviderProvisioningState : Provisioned
	ServiceProviderNotes             :
	ServiceProviderProperties        : {
	                                     "ServiceProviderName": "Equinix",
	                                     "PeeringLocation": "Silicon Valley",
	                                     "BandwidthInMbps": 200
	   		                           }
	ServiceKey                       : **************************************
	Peerings                         : []


Puede obtener una descripción detallada de todos los parámetros ejecutando lo siguiente:

	get-help get-azurededicatedcircuit -detailed


## <a name="modify"></a>Modificación de un circuito ExpressRoute

Puede modificar determinadas propiedades de un circuito ExpressRoute sin afectar a la conectividad.

Puede hacer lo siguiente sin experimentar tiempo de inactividad:

- Habilitar o deshabilitar el complemento ExpressRoute Premium en su circuito ExpressRoute.
- Aumentar el ancho de banda de su circuito ExpressRoute. Tenga en cuenta que no se admite la degradación del ancho de banda de un circuito.
- Cambio del plan de medición de datos limitados a datos ilimitados. Tenga en cuenta que no es posible cambiar el plan de medición de datos ilimitados a datos limitados.
-  Puede habilitar y deshabilitar *Allow Classic Operations* (Permitir operaciones clásicas).

Consulte la página [P+F de ExpressRoute](expressroute-faqs.md) para más información sobre los límites y las limitaciones.

### Para habilitar el complemento ExpressRoute Premium

Puede habilitar el complemento ExpressRoute Premium para el circuito existente mediante el siguiente fragmento de PowerShell:

	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.Sku.Tier = "Premium"
	$ckt.sku.Name = "Premium_MeteredData"

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt


El circuito tendrá ahora las características del complemento ExpressRoute Premium habilitadas. Tenga en cuenta que la facturación de la capacidad del complemento Premium comienza en cuanto el comando se ejecuta correctamente.

### Para deshabilitar el complemento ExpressRoute Premium

>[AZURE.IMPORTANT] Esta operación puede producir un error si usa recursos que son más grandes de lo que está permitido para el circuito estándar.

Tenga en cuenta lo siguiente:

- Debe asegurarse de que el número de redes virtuales vinculadas al circuito es inferior a 10 antes de realizar la degradación de Premium a Estándar. Si no lo hace, se producirá un error en la solicitud de actualización y se le facturará con las tarifas de nivel Premium.

- Tiene que desvincular todas las redes virtuales en otras regiones geopolíticas. Si no lo hace, se producirá un error en la solicitud de actualización y se le facturará con las tarifas de nivel Premium.

- La tabla de enrutamiento tiene que tener menos de 4.000 rutas para el emparejamiento entre pares privados. Si el tamaño de la tabla de ruta sobrepasa las 4.000 rutas, la sesión BGP se anulará y no se volverá a habilitar hasta que el número de prefijos anunciados esté por debajo de 4.000.

Puede deshabilitar el complemento ExpressRoute Premium en el circuito existente mediante el siguiente cmdlet de PowerShell:


	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.Sku.Tier = "Standard"
	$ckt.sku.Name = "Standard_MeteredData"

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt


### Para actualizar el ancho de banda del circuito ExpressRoute

Consulte la página [P+F de ExpressRoute](expressroute-faqs.md) para conocer las opciones de ancho de banda compatibles con su proveedor. Puede elegir cualquier tamaño mayor que el tamaño de su circuito existente.

>[AZURE.IMPORTANT] No podrá reducir el ancho de banda de un circuito ExpressRoute sin interrupciones. Para degradar un ancho de banda, es necesario desaprovisionar el circuito ExpressRoute y luego volver a aprovisionar un nuevo circuito ExpressRoute.

Cuando haya decidido el tamaño que necesita, use el comando siguiente para cambiar el tamaño del circuito:


	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt


El circuito se cambiará de tamaño en el lado de Microsoft. Así pues, debe ponerse en contacto con su proveedor de conectividad para actualizar las configuraciones de su parte para que coincidan con este cambio. Después de realizar esta notificación, se le empezará a facturar por la opción de ancho de banda actualizada.


### Para pasar la SKU de limitada a ilimitada

Puede cambiar la SKU de un circuito ExpressRoute mediante el siguiente fragmento de código de PowerShell:

	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.Sku.Family = "UnlimitedData"
	$ckt.sku.Name = "Premium_UnlimitedData"

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

### Para controlar el acceso a los entornos de implementación clásica y del Resource Manager  

Revise las instrucciones que se ofrecen en [Transición de los circuitos ExpressRoute desde el modelo de implementación clásica al modelo de implementación de Resource Manager](expressroute-howto-move-arm.md).


## Eliminación y la cancelación de un circuito ExpressRoute

Tenga en cuenta lo siguiente:

- Tiene que desvincular todas las redes virtuales del circuito ExpressRoute. Si se produce un error en esta operación, compruebe si hay alguna red virtual vinculada al circuito.

- Si el estado de aprovisionamiento del proveedor de servicios del circuito ExpressRoute está habilitado, el estado cambiará de habilitado a *deshabilitando*. Tiene que cooperar con su proveedor de servicios para desaprovisionar el circuito en su lado. Se le continuará reservando recursos y facturándole por ello hasta que el proveedor de servicios complete el desaprovisionamiento del circuito y nos lo notifique.

- Si el proveedor de servicios ha desaprovisionado el circuito (el estado de aprovisionamiento del proveedor de servicios está establecido en *no aprovisionado*) antes de ejecutar el cmdlet anterior, cancelaremos el aprovisionamiento del circuito y dejaremos de facturarle.

Puede eliminar el circuito ExpressRout con la ejecución del siguiente comando:

	Remove-AzureRmExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"



## Pasos siguientes

Después de crear el circuito, asegúrese de hacer lo siguiente:

- [Crear y modificar el enrutamiento para el circuito ExpressRoute](expressroute-howto-routing-arm.md)
- [Vincular la red virtual a su circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)

<!---HONumber=AcomDC_0518_2016-->