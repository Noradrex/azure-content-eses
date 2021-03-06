Para los pasos de esta tarea, se utiliza una red virtual basada en los valores siguientes. En esta lista también se enumeran nombres y valores de configuración adicionales. No se utiliza esta lista directamente en ninguno de los pasos, aunque se agregan variables basadas en los valores que aparecen en ella. Puede copiar la lista para utilizarla como referencia y reemplazar los valores por los suyos propios.

Lista de referencia de configuración:
	
- Nombre de red virtual = "TestVNet"
- Espacio de direcciones de red virtual = 192.168.0.0/16
- Grupo de recursos: "TestRG"
- Nombre de subred 1 = "FrontEnd" 
- Espacio de direcciones de subred 1 = "192.168.0.0/16"
- Nombre de subred de puerta de enlace: "GatewaySubnet" (siempre debe asignar a las subredes de puerta de enlace el nombre *GatewaySubnet*).
- Espacio de direcciones de subred de puerta de enlace = "192.168.200.0/26"
- Región = "East US"
- Nombre de puerta de enlace = "GW"
- Nombre de IP de puerta de enlace = "GWIP"
- Nombre de configuración de IP de puerta de enlace = "gwipconf"
-  Tipo = "ExpressRoute" (este tipo es obligatorio para una configuración de ExpressRoute).
- Nombre de IP pública de puerta de enlace = "gwpip"


## Adición de una puerta de enlace

1. Conéctese a su suscripción de Azure. 

		Login-AzureRmAccount
		Get-AzureRmSubscription 
		Select-AzureRmSubscription -SubscriptionName "Name of subscription"

2. Declare las variables para este ejercicio. En este ejemplo, se usarán las variables de la siguiente muestra. No olvide editarlas para reflejar la configuración que desea utilizar.
		
		$RG = "TestRG"
		$Location = "East US"
		$GWName = "GW"
		$GWIPName = "GWIP"
		$GWIPconfName = "gwipconf"
		$VNetName = "TestVNet"

3. Almacene el objeto de red virtual como una variable.

		$vnet = Get-AzureRmVirtualNetwork -Name $VNetName -ResourceGroupName $RG

4. Agregue una subred de puerta de enlace a su red virtual. A la subred de puerta de enlace debe asignarle el nombre "GatewaySubnet". Se recomienda que cree una puerta de enlace con un valor /27 o mayor (/26, /25, etc.).
			
		Add-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet -AddressPrefix 192.168.200.0/26

5. Establezca la configuración.

			Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

6. Almacene la subred de puerta de enlace como una variable.

		$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet

7. Pida una dirección IP pública. La dirección IP se solicita antes de crear la puerta de enlace. No puede especificar la dirección IP que desea usar; se asigna una dinámicamente. Utilizará esta dirección IP en la siguiente sección de configuración. El valor de AllocationMethod debe ser Dynamic.

		$pip = New-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName $RG -Location $Location -AllocationMethod Dynamic
		$ipconf = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip

8. Cree la configuración para su puerta de enlace. La configuración de puerta de enlace define la subred y la dirección IP pública. En este paso, se especifica la configuración que se utilizará al crear la puerta de enlace, pero no se crea realmente el objeto de puerta de enlace. Use el ejemplo siguiente para crear la configuración de la puerta de enlace.

		$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name $GWIPconfName -SubnetId $subnet.Id -PublicIpAddressId $pip.Id 

9. Cree la puerta de enlace. En este paso, el elemento **-GatewayType** resulta especialmente importante. Debe utilizar el valor **ExpressRoute**. Tenga en cuenta que, después de ejecutar estos cmdlets, la puerta de enlace puede tardar 20 minutos o más en crearse.

		New-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG -Location $Location -IpConfigurations $ipconf -GatewayType Expressroute -GatewaySku Standard

## Comprobación de la creación de la puerta de enlace

Utilice el siguiente comando para comprobar si se ha creado la puerta de enlace.

	Get-AzureRmVirtualNetworkGateway -ResourceGroupName $RG

## Cambio del tamaño de una puerta de enlace

Hay tres [SKU de puerta de enlace](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md). Puede utilizar el siguiente comando para cambiar en cualquier momento la SKU de puerta de enlace.

	$gw = Get-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG
	Resize-AzureRmVirtualNetworkGateway -VirtualNetworkGateway $gw -GatewaySku HighPerformance

## Eliminación de una puerta de enlace

Utilice el siguiente comando para quitar una puerta de enlace.

	Remove-AzureRmVirtualNetworkGateway -Name $GWName -ResourceGroupName $RG  

<!---HONumber=AcomDC_0413_2016-->