<properties
	pageTitle="Versión preliminar de Servicios de dominio de Azure Active Directory: administración de DNS en dominios administrados | Microsoft Azure"
	description="Administración de DNS en dominios administrados con Servicios de dominio de Azure Active Directory"
	services="active-directory-ds"
	documentationCenter=""
	authors="mahesh-unnikrishnan"
	manager="stevenpo"
	editor="curtand"/>

<tags
	ms.service="active-directory-ds"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/27/2016"
	ms.author="maheshu"/>

# Administración de DNS en un dominio administrado con Servicios de dominio de Azure AD
Servicios de dominio de Azure Active Directory incluye un servidor DNS (resolución de nombres de dominio) que proporciona una resolución DNS para el dominio administrado. En ocasiones, puede ser necesario configurar DNS en el dominio administrado con el objetivo de crear registros DNS de máquinas que no están unidas al dominio, las direcciones IP virtuales de los equilibradores de carga o los reenviadores DNS externos. Por este motivo, se concede a los usuarios que pertenecen al grupo "Administradores del controlador de dominio de AAD" privilegios de administración de DNS en el dominio administrado.


## Antes de empezar
Para realizar las tareas enumeradas en este artículo, necesita:

1. Una **suscripción de Azure** válida.

2. Un **directorio de Azure AD**: sincronizado con un directorio local o solo en la nube.

3. Los **Servicios de dominio de Azure AD** deben estar habilitado en el directorio de Azure AD. Si no lo ha hecho, siga todas las tareas descritas en [Servicios de dominio de Azure AD (vista previa): introducción](./active-directory-ds-getting-started.md).

4. Una **máquina virtual unida a un dominio** , desde la que administrará el dominio administrado de los Servicios de dominio de Azure AD. Si no tiene una máquina virtual, siga todas las tareas descritas en el artículo [Unión de una máquina virtual Windows Server a un dominio administrado](./active-directory-ds-admin-guide-join-windows-vm.md).

5. Necesitará las credenciales de una **cuenta de usuario que pertenezca al grupo "Administradores del controlador de dominio de AAD"** en el directorio con el fin de administrar DNS en el dominio administrado.

<br>

## Tarea 1: Aprovisionamiento de una máquina virtual unida a un dominio para administrar de forma remota DNS en el dominio administrado
Los dominios administrados con Servicios de dominio de Azure AD pueden administrarse de forma remota mediante las conocidas herramientas administrativas de Active Directory, como el Centro de administración de Active Directory (ADAC) o AD PowerShell. De forma similar, DNS para el dominio administrado se puede administrar remotamente con las herramientas de administración del servidor DNS.

Los administradores de su directorio de Azure AD no tienen privilegios para conectarse a controladores de dominio en el dominio administrado a través del Escritorio remoto. Por lo tanto, los miembros del grupo "Administradores del controlador de dominio de AAD" pueden administrar DNS para dominios administrados de forma remota mediante las herramientas del servidor DNS desde un equipo cliente/Windows Server que se ha unido al dominio administrado. Las herramientas del servidor DNS pueden instalarse como parte de la característica opcional Herramientas de administración remota del servidor (RSAT) en Windows Server y equipos cliente unidos al dominio administrado.

El primer paso consiste en configurar una máquina virtual Windows Server que esté unida al dominio administrado. Para ver instrucciones sobre cómo hacerlo, consulte el artículo [Unión de una máquina virtual de Windows Server a un dominio administrado](active-directory-ds-admin-guide-join-windows-vm.md).


## Tarea 2: Instalación de las herramientas del servidor DNS en la máquina virtual
Realice los pasos siguientes para instalar las herramientas de administración de DNS en la máquina virtual unida al dominio. Para obtener más información sobre cómo [instalar y usar Herramientas de administración remota del servidor](https://technet.microsoft.com/library/hh831501.aspx), consulte TechNet.

1. Vaya al nodo **Máquinas virtuales** del Portal de Azure clásico. Seleccione la máquina virtual que acaba de crear y haga clic en la opción **Conectar** de la barra de comandos situada en la parte inferior de la ventana.

    ![Conexión a máquina virtual de Windows](./media/active-directory-domain-services-admin-guide/connect-windows-vm.png)

2. El portal clásico le solicitará que abra o guarde un archivo .rdp, que se utiliza para conectarse a la máquina virtual. Haga clic en el archivo .rdp cuando haya terminado de descargarse.

3. En el aviso de inicio de sesión, utilice las credenciales de un usuario que pertenezca al grupo "Administradores del controlador de dominio de AAD". Por ejemplo, en nuestro caso "bob@domainservicespreview.onmicrosoft.com".

4. En la pantalla Inicio, abra **Administrador del servidor**. Haga clic en **Agregar roles y características** en el panel central de la ventana Administrador del servidor.

    ![Inicio del Administrador del servidor en la máquina virtual](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager.png)

5. En la página **Antes de comenzar** del **Asistente para agregar roles y características**, haga clic en **Siguiente**.

    ![Página Antes de comenzar](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-begin.png)

6. En la página **Tipo de instalación**, deje la opción **Instalación basada en características o en roles** activada y haga clic en **Siguiente**.

	![Página Tipo de instalación](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-type.png)

7. En la página **Selección del servidor**, seleccione la máquina virtual actual del grupo de servidores y haga clic en **Siguiente**.

	![Página Selección de servidor](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-server.png)

8. En la página **Roles de servidor**, haga clic en **Siguiente**. Se omitirá esta página ya que no se van a instalar roles en el servidor.

9. En la página **Características**, haga clic para expandir el nodo **Herramientas de administración remota del servidor** y, después, haga clic para expandir el nodo **Herramientas de administración de roles**. Seleccione la característica **Herramientas del servidor DNS** en la lista de herramientas de administración de roles, tal y como se muestra a continuación.

	![Página Características](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-dns-tools.png)

10. En la página **Confirmación**, haga clic en **Instalar** para instalar la característica de herramientas del servidor DNS en la máquina virtual. Cuando finalice correctamente la instalación de la característica, haga clic en **Cerrar** para salir del **Asistente para agregar roles y características**.

	![Página de confirmación](./media/active-directory-domain-services-admin-guide/install-rsat-server-manager-add-roles-dns-confirmation.png)


## Tarea 3: Inicio de la consola de administración de DNS para administrar DNS
Ahora que la característica Herramientas del servidor DNS está instalada en la máquina virtual unida al dominio, puede usar las herramientas de DNS para administrar DNS en el dominio administrado.

> [AZURE.NOTE] Tendrá que ser miembro del grupo "Administradores del controlador de dominio de AAD" para administrar DNS en el dominio administrado.

1. En la pantalla Inicio, haga clic en **Herramientas administrativas**. Debería ver la consola **DNS** instalada en la máquina virtual.

	![Herramientas administrativas: consola DNS](./media/active-directory-domain-services-admin-guide/install-rsat-dns-tools-installed.png)

2. Haga clic en **DNS** para iniciar la consola de administración de DNS.

3. En el cuadro de diálogo **Conectar a servidor DNS**, haga clic en la opción **El siguiente equipo** y escriba el nombre de dominio DNS del dominio administrado (por ejemplo, "contoso100.com").

    ![Consola DNS: conexión al dominio](./media/active-directory-domain-services-admin-guide/dns-console-connect-to-domain.png)

4. La consola DNS se conecta al dominio administrado. Debería ver una vista similar a la siguiente:

    ![Consola DNS: administración del dominio](./media/active-directory-domain-services-admin-guide/dns-console-managed-domain.png)

5. Ahora puede usar la consola DNS para agregar entradas de DNS para los equipos de la red virtual en la que ha habilitado Servicios de dominio de AAD.

> [AZURE.WARNING] Tenga mucho cuidado al administrar DNS para el dominio administrado con las herramientas de administración de DNS. Asegúrese de **no eliminar ni modificar los registros DNS integrados que usan los Servicios de dominio en el dominio**. Esto incluye registros DNS de dominio, registros de servidor de nombres y otros registros usados para la ubicación del controlador de dominio. Si modifica estos registros, se interrumpirán los servicios de dominio en la red virtual.


Para obtener más información sobre la administración de DNS, consulte el artículo [Herramientas de DNS](https://technet.microsoft.com/library/cc753579.aspx) de TechNet.


## Contenido relacionado

- [Servicios de dominio de Azure AD (vista previa): introducción](./active-directory-ds-getting-started.md)

- [Administer an Azure AD Domain Services managed domain (Administración de un dominio administrado con Servicios de dominio de Azure AD)](active-directory-ds-admin-guide-administer-domain.md)

- [Unión de una máquina virtual de Windows Server a un dominio administrado](active-directory-ds-admin-guide-join-windows-vm.md)

- [Herramientas de DNS](https://technet.microsoft.com/library/cc753579.aspx)

<!---HONumber=AcomDC_0504_2016-->