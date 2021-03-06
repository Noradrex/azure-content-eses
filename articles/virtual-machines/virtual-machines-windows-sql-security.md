<properties
	pageTitle="Consideraciones de seguridad para SQL Server en Azure | Microsoft Azure"
	description="Este tema hace referencia a los recursos creados con el modelo de implementación clásica y proporciona instrucciones generales para la seguridad de SQL Server que se ejecuta en una máquina virtual de Azure."
	services="virtual-machines-windows"
	documentationCenter="na"
	authors="rothja"
	manager="jhubbard"
   editor=""    
   tags="azure-service-management"/>
<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="03/23/2016"
	ms.author="jroth" />

# Consideraciones de seguridad para SQL Server en Máquinas virtuales de Azure

Este tema incluye las direcciones de seguridad generales que ayudan a establecer el acceso seguro a instancias de SQL Server en una máquina virtual de Azure. Sin embargo, para garantizar una mejor protección para las instancias de base de datos de SQL Server en Azure, es recomendable que implemente los procedimientos de seguridad locales tradicionales además de los procedimientos recomendados de seguridad de Azure.

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]Modelo del Administrador de recursos.


Para obtener más información sobre los procedimientos de seguridad de SQL Server, consulte [Procedimientos recomendados de seguridad en SQL Server 2008 R2: tareas operativas y administrativas](http://download.microsoft.com/download/1/2/A/12ABE102-4427-4335-B989-5DA579A4D29D/SQL_Server_2008_R2_Security_Best_Practice_Whitepaper.docx)

Azure cumple diversos reglamentos y estándares del sector que permiten compilar una solución compatible con SQL Server que se ejecuta en una máquina virtual. Para obtener información acerca del cumplimiento normativo de Azure, consulte [Centro de confianza de Azure](https://azure.microsoft.com/support/trust-center/).

La siguiente es una lista de recomendaciones de seguridad que deben tenerse en cuenta al configurar y conectarse a la instancia de SQL Server en una máquina virtual de Azure.

## Consideraciones para administrar cuentas:

- Cree una cuenta de administrador local única que no se llame **Administrador**.

- Use contraseñas seguras complejas para todas sus cuentas. Para más información sobre cómo crear una contraseña segura, consulte el artículo [Sugerencias para crear una contraseña segura](http://windows.microsoft.com/es-ES/windows-vista/Tips-for-creating-a-strong-password).

- De forma predeterminada, Azure selecciona la autenticación de Windows durante la instalación de la máquina virtual de SQL Server. Por lo tanto, el inicio de sesión de **SA** está deshabilitado y el programa de instalación asigna una contraseña. Se recomienda no usar ni habilitar el inicio de sesión de **SA**. Las siguientes son estrategias alternativas si se desea un inicio de sesión de SQL:
	- Crear una cuenta SQL que sea miembro de sysadmin.
	- Si tiene que usar un inicio de sesión de **SA**, habilite el inicio de sesión, cámbiele el nombre y asígnele una nueva contraseña.
	- Ambas opciones mencionadas requieren cambiar el modo de autenticación al **modo de autenticación de Windows y SQL Server**. Para obtener más información, consulte [Cambiar el modo de autenticación del servidor](https://msdn.microsoft.com/library/ms188670.aspx).

## Consideraciones para proteger las conexiones con la máquina virtual de Azure:

- Considere la posibilidad de usar [Red virtual de Azure](../virtual-network/virtual-networks-overview.md) para administrar las máquinas virtuales en lugar de los puertos públicos RDP.

- Use un [grupo de seguridad de red](../virtual-network/virtual-networks-nsg.md) (NSG) para permitir o denegar el tráfico de red a la máquina virtual. Si desea usar un grupo de seguridad de red y ya tiene un extremo del ACL, quite primero el extremo del ACL. Para obtener información acerca de cómo hacerlo, consulte [Administración de listas de control de acceso (ACL) para extremos mediante PowerShell](../virtual-network/virtual-networks-acl-powershell.md).

- Si está utilizando puntos de conexión, quite de la máquina virtual todos los puntos de conexión que no use. Para obtener instrucciones sobre el uso de ACL con puntos de conexión, consulte [Administrar la ACL en un punto de conexión](../virtual-network/virtual-machines-windows-classic-setup-endpoints.md#manage-the-acl-on-an-endpoint).

- Habilite una opción de conexión cifrada para una instancia de motor de base de datos de SQL Server en máquinas virtuales de Azure. Configure la instancia de SQL Server con un certificado firmado. Para obtener más información, consulte [Habilitar conexiones cifradas en el motor de base de datos](https://msdn.microsoft.com/library/ms191192.aspx) y [Sintaxis de cadena de conexión](https://msdn.microsoft.com/library/ms254500.aspx).

- Si el acceso a las máquinas virtuales debe realizarse solo desde una red específica, use Firewall de Windows para restringir el acceso a ciertas direcciones IP o subredes de red.

## Pasos siguientes

Si también está interesado en los procedimientos recomendados de rendimiento, consulte [Procedimientos recomendados para SQL Server en máquinas virtuales de Azure](virtual-machines-windows-sql-performance.md).

Para ver otros temas sobre la ejecución de SQL Server en máquinas virtuales de Azure, consulte [Información general sobre SQL Server en máquinas virtuales de Azure](virtual-machines-windows-sql-server-iaas-overview.md).

<!---HONumber=AcomDC_0413_2016-->