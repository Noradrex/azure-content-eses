<properties
   pageTitle="Preguntas más frecuentes sobre el Centro de seguridad de Azure | Microsoft Azure"
   description="Estas preguntas más frecuentes responden a las preguntas sobre el Centro de seguridad de Azure."
   services="security-center"
   documentationCenter="na"
   authors="TerryLanfear"
   manager="StevenPo"
   editor=""/>

<tags
   ms.service="security-center"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/02/2016"
   ms.author="terrylan"/>

# Preguntas más frecuentes sobre el Centro de seguridad de Azure

Estas preguntas más frecuentes responden a preguntas sobre el Centro de seguridad de Azure, un servicio que le ayuda a evitar, detectar y responder a amenazar con mayor visibilidad y control sobre la seguridad de los recursos de Microsoft Azure.

> [AZURE.NOTE] La información de este documento se aplica a la versión preliminar del Centro de seguridad de Azure.

## Preguntas generales

### ¿Qué es el Centro de seguridad de Azure?
El Centro de seguridad de Azure ayuda a evitar, detectar y responder a amenazas con más visibilidad y control de la seguridad en sus recursos de Azure. Proporciona administración de directivas y supervisión de la seguridad integrada en las suscripciones, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con un amplio ecosistema de soluciones de seguridad.

### ¿Cómo puedo obtener el Centro de seguridad de Azure?
El Centro de seguridad de Azure se habilita con la suscripción de Microsoft Azure y es posible tener acceso a él desde el [Portal de Azure](https://azure.microsoft.com/features/azure-portal/). ([Inicie sesión en el portal](https://portal.azure.com), seleccione **Examinar** y desplácese al **Centro de seguridad**). Puede consultar algunas recomendaciones de seguridad en el panel inmediatamente. Esto se debe a que el servicio puede evaluar el estado de seguridad de algunos controles según su configuración en Azure. Para habilitar el conjunto total de funcionalidades de supervisión, recomendaciones y alertas de seguridad, deberá [habilitar la colección de datos](#data-collection).

## Facturación

### ¿Cómo funciona la facturación para el Centro de seguridad de Azure?
Vea [Precios del Centro de seguridad de Azure](https://azure.microsoft.com/pricing/details/security-center/) para información.

## Colección de datos

### ¿Cómo habilito la colección de datos?<a name=data-collection></a>
Puede habilitar la colección de datos de las suscripciones de Azure en la directiva de seguridad. Para habilitar la colección de datos, [inicie sesión en el Portal de Azure](https://portal.azure.com), seleccione **Examinar**, **Centro de seguridad** y, luego, **Directiva de seguridad**. Establezca **Colección de datos** en **Habilitada** y configure las cuentas de almacenamiento donde desea colectar los datos (consulte la pregunta "[¿Dónde se almacenan los datos?](#where-is-my-data-stored)"). Una vez habilitada la **colección de datos**, colecta automáticamente información de eventos y configuración de seguridad de todas las máquinas virtuales compatibles de la suscripción.

> [AZURE.NOTE] Las directivas de seguridad se pueden establecer en el nivel de suscripción y el nivel de grupo de recursos de Azure, pero la configuración de la recopilación de datos tiene lugar solo en el nivel de suscripción.

### ¿Qué sucede cuando habilito la colección de datos?
La colección de datos se habilita a través de la extensión Supervisión de seguridad de Azure y el Agente de supervisión de Azure. La extensión Supervisión de seguridad de Azure busca diversos eventos y configuraciones de seguridad importantes y los envía a [Seguimiento de eventos para Windows](https://msdn.microsoft.com/library/windows/desktop/bb968803.aspx) (ETW). Además, el sistema operativo crea las entradas del registro de eventos. El Agente de supervisión de Azure lee las entradas de los registros de eventos y los seguimientos de ETW y los copia en la cuenta de almacenamiento para su análisis. Esta es la cuenta de almacenamiento que configuró en la directiva de seguridad. Para obtener más información sobre la cuenta de almacenamiento, consulte "[¿Dónde se almacenan los datos?](#where-is-my-data-stored)".

### ¿El Agente de supervisión o la extensión Supervisión de seguridad afecta el rendimiento de los servidores?
El agente y la extensión utilizan una cantidad simbólica de los recursos del sistema y tienen un impacto menor en el rendimiento.

### ¿Cómo puedo revertir la habilitación de la colección de datos si ya no la deseo?
Puede deshabilitar la **colección de datos** de una suscripción en la directiva de seguridad. ([Inicie sesión en el Portal de Azure](https://portal.azure.com), seleccione **Examinar**, **Centro de seguridad** y, luego, **Directiva de seguridad**). Cuando se selecciona una suscripción, se abre una hoja nueva que le brinda la opción de deshabilitar la colección de datos. Seleccione la opción **Eliminar agentes** en la barra de herramientas superior para quitar los agentes de las máquinas virtuales existentes.

> [AZURE.NOTE] Las directivas de seguridad se pueden establecer en el nivel de suscripción y el nivel de grupo de recursos de Azure, pero debe seleccionar una suscripción para desactivar la colección de datos.

### ¿Dónde se almacenan los datos?<a name=where-is-my-data-stored></a>
Para cada región en la que disponga de máquinas virtuales en funcionamiento, elija la cuenta de almacenamiento en la que se almacenan los datos recopilados de esas máquinas virtuales. Esto facilita el mantenimiento de los datos en la misma zona geográfica para fines de privacidad y soberanía de datos. Elija la cuenta de almacenamiento para una suscripción en la directiva de seguridad. ([Inicie sesión en el Portal de Azure](https://portal.azure.com), seleccione **Examinar**, **Centro de seguridad** y, luego, **Directiva de seguridad**). Cuando hace clic en una suscripción, se abre una hoja nueva. Seleccione **Elegir cuentas de almacenamiento** para seleccionar una región. Los datos colectados se aíslan lógicamente de los datos de otros clientes por motivos de seguridad.

> [AZURE.NOTE] Las directivas de seguridad se pueden establecer en el nivel de suscripción y el nivel de grupo de recursos de Azure, pero la selección de una región para la cuenta de almacenamiento tiene lugar solo en el nivel de suscripción.

Para obtener más información sobre las cuentas de almacenamiento y el almacenamiento de Azure, consulte [Documentación de almacenamiento](https://azure.microsoft.com/documentation/services/storage/) y [Acerca de las cuentas de almacenamiento de Azure](../storage/storage-create-storage-account.md).

## Uso del Centro de seguridad de Azure

### ¿Qué es una directiva de seguridad?
Una directiva de seguridad define el conjunto de controles recomendados para los recursos en la suscripción o grupo de recursos especificados. En el Centro de seguridad de Azure, se definen las directivas para sus suscripciones y grupos de recursos de Azure de acuerdo con los requisitos de seguridad de la empresa y el tipo de aplicaciones o la confidencialidad de los datos de cada suscripción.

Por ejemplo, es posible que los recursos usados para el desarrollo o las pruebas tengan distintos requisitos de seguridad que los de aquellos que se emplean para aplicaciones de producción. Del mismo modo, es posible que las aplicaciones con datos regulados como información de identificación personal (PII) requieran un mayor nivel de seguridad. Las directivas de seguridad habilitadas en el Centro de seguridad de Azure generarán la supervisión y recomendaciones de seguridad. Para obtener más información sobre las directivas de seguridad, consulte [Supervisión del estado de seguridad en el Centro de seguridad de Azure](security-center-monitoring.md).

> [AZURE.NOTE] En caso de un conflicto entre la directiva del nivel de suscripción y la directiva del nivel de grupo de recursos, la del nivel de grupo de recursos tiene prioridad.

### ¿Quién puede modificar una directiva de seguridad?
Las directivas de seguridad se configuran para cada suscripción o grupo de recursos. Para modificar una directiva de seguridad en el nivel de suscripción o el nivel de grupo de recursos, debe ser el propietario de la suscripción o un colaborador de esta.

Para obtener información sobre cómo configurar una directiva de seguridad, consulte [Establecimiento de directivas de seguridad en el Centro de seguridad de Azure](security-center-policies.md).

### ¿Qué es una recomendación de seguridad?
El Centro de seguridad de Azure analiza el estado de seguridad de los recursos de Azure. Las recomendaciones se crean una vez que se identifican las posibles vulnerabilidades de seguridad. Las recomendaciones le guían en el proceso de configurar el control necesario. Algunos ejemplos son:

- Aprovisionamiento de antimalware para ayudar a identificar y eliminar el software malintencionado.
- Configuración de reglas y [grupos de seguridad de red](../virtual-network/virtual-networks-nsg.md) para controlar el tráfico a las máquinas virtuales.
- Aprovisionamiento de un firewall de aplicaciones web para ayudar a defenderse contra ataques dirigidos a las aplicaciones web.
- Implementación de actualizaciones del sistema que faltan.
- Resolución de las configuraciones de sistema operativo que no coinciden con las líneas base recomendadas.

Aquí solo se muestran las recomendaciones habilitadas en las directivas de seguridad.

### ¿Cómo puedo ver el estado de seguridad actual de los recursos de Azure?
Un icono **Estado de los recursos** en la hoja **Centro de seguridad** muestra la posición de seguridad general del entorno, desglosada por máquinas virtuales, aplicaciones web y otros recursos. Cada recurso tiene un indicador que muestra si se identificó alguna posible vulnerabilidad de seguridad. Si hace clic en el icono Estado de los recursos, aparecen los recursos y se identifica dónde se requiere poner atención o los problemas que puedan existir.

### ¿Qué desencadena una alerta de seguridad?
El Centro de seguridad de Azure recopila, analiza y combina automáticamente los datos de registro de los recursos de Azure, la red y soluciones de asociados como firewalls y antimalware. Cuando se detecten amenazas, se creará una alerta de seguridad. Como ejemplos se incluye la detección de:

- Máquinas virtuales en peligro que se comunican con direcciones IP malintencionadas conocidas.
- Malware avanzado detectado mediante la generación de informes de errores de Windows.
- Ataques por fuerza bruta contra máquinas virtuales.
- Alertas de seguridad de soluciones de seguridad integradas de socio, como antimalware o Firewall de aplicaciones web.

### ¿Qué diferencia hay entre las amenazas detectadas y advertidas por el Centro de respuestas de seguridad de Microsoft y las comunicadas por Azure Security Center?
El Centro de respuestas de seguridad de Microsoft (MSRC) lleva a cabo una selecta supervisión de seguridad de la red e infraestructura de Azure y recibe información sobre amenazas y quejas sobre abusos de terceros. Cuando MSRC se da cuenta de que un usuario ilegítimo o no autorizado ha tenido acceso a los datos del cliente o de que el uso de Azure por parte del cliente no cumple las condiciones de uso aceptable, un administrador de incidentes de seguridad notifica al cliente. La notificación suele consistir en el envío de un correo electrónico a los contactos de seguridad especificados en Azure Security Center o el propietario de la suscripción de Azure en caso de no especificarse un contacto de seguridad.

Azure Security Center es un servicio de Azure que no deja de supervisar el entorno de Azure del cliente y aplica análisis para detectar de forma automática una amplia variedad de actividades potencialmente malintencionadas. Estas detecciones aparecen como alertas de seguridad en el panel de Azure Security Center. En el futuro, la notificación por correo electrónico de alertas de seguridad se enviará también a los contactos de seguridad.

### ¿Cómo se administran los permisos en el Centro de seguridad de Azure?
El Centro de seguridad de Azure admite el acceso basado en rol. Para obtener más información sobre el control de acceso basado en rol (RBAC) de Azure, consulte [Control de acceso basado en roles de Azure Active Directory](../active-directory/role-based-access-control-configure.md).

Cuando un usuario abre el Centro de seguridad de Azure, solo se verán las recomendaciones de alertas relacionadas con los recursos a los que el usuario tiene acceso. Esto significa que el usuario verá solo los elementos relacionados con los recursos donde el usuario tiene asignado el rol de Propietario, Colaborador o Lector a la suscripción o el grupo de recursos al que pertenece un recurso.

Para editar una directiva de seguridad, debe ser propietario o colaborador de la suscripción.

### ¿Qué tipos de máquinas virtuales serán compatibles?
Se admiten las máquinas virtuales creadas mediante los [modelos de implementación clásico y de Resource Manager](../azure-classic-rm.md), incluidas las máquinas virtuales que forman parte de los clústeres de Azure Service Fabric.

Las recomendaciones de lista de control de acceso se aplican actualmente a las máquinas virtuales (clásicas). Los grupos de seguridad de red solo se aplican actualmente a las máquinas virtuales (Administrador de recursos).

### ¿Las máquinas virtuales de Linux son compatibles?
El Centro de seguridad de Azure ofrece supervisión de línea base para máquinas virtuales de Linux (solo Ubuntu versiones 12.04, 14.04, 14.10 y 15.04). En el futuro, habrá disponible una supervisión del estado de seguridad y colección de datos/análisis adicionales, además de compatibilidad para otras distribuciones de Linux.

<!---HONumber=AcomDC_0504_2016-->