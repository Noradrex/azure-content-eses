<properties 
   pageTitle="Acerca de los dispositivos VPN para las conexiones de puerta de enlace de VPN de sitio a sitio para redes virtuales de Azure | Microsoft Azure"
   description="Obtenga más información sobre los dispositivos VPN y los parámetros de IPsec para conexiones de puerta de enlace de VPN de sitio a sitio. Se pueden utilizar conexiones de sitio a sitio para las configuraciones híbridas. Este artículo contiene vínculos a instrucciones de configuración y ejemplos para dispositivos de puerta de enlace de VPN."
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
  tags="azure-resource-manager, azure-service-management"/>
<tags 
   ms.service="vpn-gateway"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/29/2016"
   ms.author="cherylmc" />

# Acerca de los dispositivos VPN para las conexiones de puerta de enlace de VPN de sitio a sitio

Se requiere un dispositivo VPN para configurar una conexión VPN de sitio a sitio. Las conexiones de sitio a sitio pueden usarse para crear una solución híbrida o siempre que desee una conexión segura entre su red local y la red virtual. Este artículo trata sobre los dispositivos VPN compatibles y los parámetros de configuración. Tenga en cuenta que al configurar una conexión de sitio a sitio, una dirección IP IPv4 pública es necesaria para el dispositivo VPN.

Si el dispositivo no aparece en la tabla de dispositivos VPN validados, consulte la sección de dispositivos VPN no validados de este artículo. Es posible que el dispositivo funcione aún con Azure. Para obtener soporte con los dispositivos VPN, póngase en contacto con el fabricante.

**Elementos que hay que tener en cuenta para consultar las tablas:**

- Ha habido un cambio de terminología del enrutamiento estático y dinámico. Es probable que realice la ejecución en ambos términos. No hay ningún cambio de funcionalidad; solo cambian los nombres.
	- Enrutamiento estático = basada en directivas
	- Enrutamiento dinámico = basada en enrutamiento 
- Las especificaciones de puerta de enlace de VPN de alto rendimiento y de puerta de enlace de VPN basada en enrutamiento son las mismas, a menos que se indique lo contrario. Por ejemplo, los dispositivos VPN validados que son compatibles con las puertas de enlace de VPN basadas en enrutamiento también serán compatibles con la nueva puerta de enlace de VPN de alto rendimiento de Azure. 


## Dispositivos VPN validados 

Hemos validado un conjunto de dispositivos VPN estándar en colaboración con proveedores de dispositivos. Todos los dispositivos de las familias de dispositivos incluidos en la lista siguiente deben funcionar con las puertas de enlace de VPN de Azure. Consulte el artículo sobre las [puertas de enlace de VPN](vpn-gateway-about-vpngateways.md) para verificar el tipo de puerta de enlace que debe crear para la solución que desea configurar.

Con el fin de configurar el dispositivo VPN, consulte los vínculos correspondientes a la familia de dispositivos apropiada.



| **Proveedor** | **Familia de dispositivos** | **Versión mínima de sistema operativo** | **Basada en directivas** | **Basada en enrutamiento** |
|---------------------------------|----------------------------------------------------------|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Allied Telesis | Enrutadores VPN de la serie AR | 2\.9.2 | Próximamente | No compatible |
| Barracuda Networks, Inc. | Barracuda NextGen Firewall de la serie F | Basada en directivas: 5.4.3, basadas en enrutamiento: 6.2.0 | [Instrucciones de configuración](https://techlib.barracuda.com/NGF/AzurePolicyBasedVPNGW) | [Instrucciones de configuración](https://techlib.barracuda.com/NGF/AzureRouteBasedVPNGW) |
| Barracuda Networks, Inc. | Barracuda NextGen Firewall de la serie X | Barracuda Firewall 6.5 | [Barracuda Firewall](https://techlib.barracuda.com/BFW/ConfigAzureVPNGateway) | No compatible |
| Brocade | Vyatta 5400 vRouter | Virtual Router 6.6R3 GA | [Instrucciones de configuración](http://www1.brocade.com/downloads/documents/html_product_manuals/vyatta/vyatta_5400_manual/wwhelp/wwhimpl/js/html/wwhelp.htm#href=VPN_Site-to-Site%20IPsec%20VPN/Preface.1.1.html) | No compatible |
| Punto de comprobación | Puerta de enlace de seguridad | R75.40, R75.40VS | [Instrucciones de configuración](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) | [Instrucciones de configuración](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk101275) |
| Cisco | ASA | 8\.3 | [Ejemplos de Cisco](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASA) | No compatible |
| Cisco | ASR | IOS 15.1 (basada en directivas), IOS 15.2 (basada en enrutamiento) | [Ejemplos de Cisco](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) | [Ejemplos de Cisco](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ASR) |
| Cisco | ISR | IOS 15.0 (basada en directiva), IOS 15.1 (basada en enrutamiento) | [Ejemplos de Cisco](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) | [Ejemplos de Cisco](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Cisco/Current/ISR) |
| Citrix | Dispositivo CloudBridge MPX o dispositivo virtual VPX | N/D | [Instrucciones de integración](https://www.citrix.com/welcome.html?resource=%2Fdownloads%2Fcloudbridge%2Fbetas-and-tech-previews%2Fcloudbridge-azure-integration) | No compatible |
| Dell SonicWALL | Serie TZ, serie NSA, serie SuperMassive, serie NSA E-Class | SonicOS 5.8.x, [SonicOS 5.9.x](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide/supported-platforms?ParentProduct=850), [SonicOS 6.x](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide/supported-platforms?ParentProduct=646) | [Instrucciones: SonicOS 6.2](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646) [Instrucciones: SonicOS 5.9](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) | [Instrucciones: SonicOS 6.2](http://documents.software.dell.com/sonicos/6.2/microsoft-azure-configuration-guide?ParentProduct=646) [Instrucciones: SonicOS 5.9](http://documents.software.dell.com/sonicos/5.9/microsoft-azure-configuration-guide?ParentProduct=850) |
| F5 | Serie BIG-IP | N/D | [Instrucciones de configuración](https://devcentral.f5.com/articles/connecting-to-windows-azure-with-the-big-ip) | No compatible |
| Fortinet | FortiGate | FortiOS 5.0.7 | [Instrucciones de configuración](http://docs.fortinet.com/fortigate/admin-guides) | [Instrucciones de configuración](http://docs.fortinet.com/fortigate/admin-guides) |
| Internet Initiative Japan (IIJ) | Serie SEIL | SEIL/X 4.60, SEIL/B1 4.60, SEIL/x86 3.20 | [Instrucciones de configuración](http://www.iij.ad.jp/biz/seil/ConfigAzureSEILVPN.pdf) | No compatible |
| Juniper | SRX | JunOS 10.2 (basada en directivas), JunOS 11.4 (basada en enrutamiento) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SRX) |
| Juniper | Serie J | JunOS 10.4r9 (basada en directivas), JunOS 11.4 (basada en enrutamiento) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/JSeries) |
| Juniper | ISG | ScreenOS 6.3 (basada en directivas y basada en enrutamiento) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/ISG) |
| Juniper | SSG | ScreenOS 6.2 (basada en directiva y basada en enrutamiento) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) | [Ejemplos de Juniper](https://github.com/Azure/Azure-vpn-config-samples/tree/master/Juniper/Current/SSG) |
| Microsoft | Servicio de acceso remoto y enrutamiento | Windows Server 2012 | No compatible | [Ejemplos de Microsoft](http://go.microsoft.com/fwlink/p/?LinkId=717761) |
| Open Systems AG | Mission Control Security Gateway | N/D | [Guía de instalación](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) | [Guía de instalación](https://www.open.ch/_pdf/Azure/AzureVPNSetup_Installation_Guide.pdf) |
| Openswan | Openswan | 2\.6.32 | (Próximamente) | No compatible |
| Palo Alto Networks | Todos los dispositivos que ejecutan PAN-OS | PAN-OS 6.1.5 o posterior (basado en directivas), PAN-OS 7.0.5 o posterior (basado en enrutamiento) | [Instrucciones de configuración](https://live.paloaltonetworks.com/t5/Configuration-Articles/How-to-Configure-VPN-Tunnel-Between-a-Palo-Alto-Networks/ta-p/59065) | [Instrucciones de configuración](https://live.paloaltonetworks.com/t5/Integration-Articles/Configuring-IKEv2-VPN-for-Microsoft-Azure-Environment/ta-p/60340) |
| WatchGuard | Todo | Fireware XTM v11.x | [Instrucciones de configuración](http://customers.watchguard.com/articles/Article/Configure-a-VPN-connection-to-a-Windows-Azure-virtual-network/) | No compatible |


## Dispositivos VPN no validados

Si el dispositivo no aparece en la tabla de dispositivos VPN validados (arriba), es posible que todavía funcione con una conexión de sitio a sitio. Verifique que el dispositivo VPN cumple los requisitos mínimos descritos en la sección Requisitos de las puertas de enlace del artículo [Acerca de las puertas de enlace de VPN](vpn-gateway-about-vpngateways.md#gateway-requirements). Los dispositivos que cumplen los requisitos mínimos deberían funcionar correctamente también con las puertas de enlace de VPN. Póngase en contacto con el fabricante del dispositivo para obtener instrucciones adicionales de soporte técnico y configuración.


## Editar los ejemplos de configuración de dispositivos

Después de descargar el ejemplo de configuración del dispositivo VPN proporcionado, deberá reemplazar algunos de los valores para reflejar la configuración de su entorno.

**Para editar una muestra:**

1. Abra el ejemplo con el Bloc de notas. 
1. Busque y reemplace todas las cadenas de <*texto*> por los valores que pertenezcan al entorno. Asegúrese de incluir < and >. Cuando se especifica un nombre, el nombre que seleccione debe ser único. Si un comando no funciona, consulte la documentación del fabricante del dispositivo.

| **Texto de ejemplo** | **Cambiar a** |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------|
| &lt;RP_OnPremisesNetwork&gt; | Nombre elegido para este objeto. Ejemplo: miRedLocal |
| &lt;RP_AzureNetwork&gt; | Nombre elegido para este objeto. Ejemplo: miRedAzure |
| &lt;RP_AccessList&gt; | Nombre elegido para este objeto. Ejemplo: miListaAccesoAzure |
| &lt;RP_IPSecTransformSet&gt; | Nombre elegido para este objeto. Ejemplo: miConjuntoTransIPSec |
| &lt;RP_IPSecCryptoMap&gt; | Nombre elegido para este objeto. Ejemplo: miAsignCifradoIPSec |
| &lt;SP_AzureNetworkIpRange&gt; | Especifique el rango. Ejemplo: 192.168.0.0 |
| &lt;SP_AzureNetworkSubnetMask&gt; | Especifique la máscara de subred. Ejemplo: 255.255.0.0 |
| &lt;SP_OnPremisesNetworkIpRange&gt; | Especifique el rango local. Ejemplo: 10.2.1.0 |
| &lt;SP_OnPremisesNetworkSubnetMask&gt; | Especifique la máscara de subred local. Ejemplo: 255.255.255.0 |
| &lt;SP_AzureGatewayIpAddress&gt; | Esta información es específica de la red virtual y se encuentra en el Portal de administración como **Dirección IP de puerta de enlace**. |
| &lt;SP_PresharedKey&gt; | Esta información es específica de la red virtual y se encuentra en el Portal de administración, en Administrar clave. |



## Parámetros de IPsec

>[AZURE.NOTE] Aunque los valores enumerados a continuación son compatibles con la Puerta de enlace de VPN de Azure, actualmente no hay ninguna manera de especificar o seleccionar una combinación específica en la Puerta de enlace de VPN de Azure. Debe especificar las restricciones en el dispositivo VPN local.

### Configuración de la fase 1 de IKE

| **Propiedad** | **Basada en directivas** | **Puerta de enlace de VPN de alto rendimiento o estándar y basada en enrutamiento** |
|----------------------------------------------------|--------------------------------|------------------------------------------------------------------|
| Versión de IKE | IKEv1 | IKEv2 |
| Grupo Diffie-Hellman | Grupo 2 (1024 bits) | Grupo 2 (1024 bits) |
| Método de autenticación | Clave previamente compartida | Clave previamente compartida |
| Algoritmos de cifrado | AES256 AES128 3DES | AES256 3DES |
| Algoritmo hash | SHA1(SHA128) | SHA1(SHA128), SHA2(SHA256) |
| Vida útil (tiempo) de la asociación de seguridad (SA) de la fase 1 | 28\.800 segundos | 10\.800 segundos |


### Configuración de la fase 2 de IKE

| **Propiedad** | **Basada en directivas** | **Puerta de enlace de VPN de alto rendimiento o estándar y basada en enrutamiento** |
|--------------------------------------------------------------------------|------------------------------------------------|--------------------------------------------------------------------|
| Versión de IKE | IKEv1 | IKEv2 |
| Algoritmo hash | SHA1(SHA128) | SHA1(SHA128) |
| Vida útil (tiempo) de la asociación de seguridad (SA) de la fase 2 | 3\.600 segundos | 3\.600 segundos |
| Vida útil (rendimiento) de la asociación de seguridad (SA) de la fase 2 | 102.400.000 KB | - |
| Cifrado y ofertas de autenticación de SA de IPsec (por orden de preferencia) | 1. ESP-AES256 2. ESP-AES128 3. ESP-3DES 4. No disponible | Consulte *Ofertas de asociación de seguridad (SA) con IPsec de puerta de enlace basada en enrutamiento* (a continuación) |
| Confidencialidad directa total (PFS) | No | Sí (DH Grupo1, 2, 5, 14, 24) |
| Detección de pares no alcanzados (DPD) | No se admite | Se admite |

### Ofertas de asociación de seguridad (SA) con IPsec de puerta de enlace basada en enrutamiento

En la tabla encontrará una lista de las ofertas de autenticación y cifrado de SA de IPsec. Las ofertas se enumeran en el orden de preferencia en el que se presentan o se aceptan.

| **Ofertas de autenticación y cifrado de SA de IPsec** | **Puerta de enlace de Azure como iniciador** | **Puerta de enlace de Azure como respondedor** |
|---------------------------------------------------|--------------------------------------------------------------|--------------------------------------------------------------|
| 1 | ESP AES_256 SHA | ESP AES_128 SHA |
| 2 | ESP AES_128 SHA | ESP 3_DES MD5 |
| 3 | ESP 3_DES MD5 | ESP 3_DES SHA |
| 4 | ESP 3_DES SHA | AH SHA1 con ESP AES_128 con HMAC nulo |
| 5 | AH SHA1 con ESP AES_256 con HMAC nulo | AH SHA1 con ESP 3_DES con HMAC nulo |
| 6 | AH SHA1 con ESP AES_128 con HMAC nulo | AH MD5 con ESP 3_DES con HMAC nulo, sin vida útil propuesta |
| 7 | AH SHA1 con ESP 3_DES con HMAC nulo | AH SHA1 con ESP 3_DES SHA1, sin vida útil |
| 8 | AH MD5 con ESP 3_DES con HMAC nulo, sin vida útil propuesta | AH MD5 con ESP 3_DES MD5, sin vida útil |
| 9 | AH SHA1 con ESP 3_DES SHA1, sin vida útil | ESP DES MD5 |
| 10 | AH MD5 con ESP 3_DES MD5, sin vida útil | ESP DES SHA1, sin vida útil |
| 11 | ESP DES MD5 | AH SHA1 con ESP DES HMAC nulo, sin vida útil propuesta |
| 12 | ESP DES SHA1, sin vida útil | AH MD5 con ESP DES HMAC nulo, sin vida útil propuesta |
| 13 | AH SHA1 con ESP DES HMAC nulo, sin vida útil propuesta | AH SHA1 con ESP DES SHA1, sin vida útil |
| 14 | AH MD5 con ESP DES HMAC nulo, sin vida útil propuesta | AH MD5 con ESP DES MD5, sin vida útil |
| 15 | AH SHA1 con ESP DES SHA1, sin vida útil | ESP SHA, sin vida útil |
| 16 | AH MD5 con ESP DES MD5, sin vida útil | ESP MD5, sin vida útil |
| 17 | - | AH SHA, sin vida útil |
| 18 | - | AH MD5, sin vida útil |


- Puede especificar el cifrado IPsec ESP NULL con puertas de enlace de VPN de alto rendimiento y basadas en enrutamiento. El cifrado basado en null no proporciona protección de datos en tránsito, solo se debe usar al máximo rendimiento y es necesaria la mínima latencia. Los clientes pueden optar por usar estos escenarios de comunicación de red virtual a red virtual, o el momento de aplicación del cifrado en otra parte de la solución.

- Para conectividad entre locales a través de Internet, use la configuración de la puerta de enlace de VPN de Azure predeterminada con los algoritmos de cifrado y hash de las tablas anteriores para garantizar la seguridad de su comunicación crítica.

<!---HONumber=AcomDC_0504_2016-->