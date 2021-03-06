<properties
	pageTitle="Administración del Almacén de claves de Azure mediante Automatización de Azure | Microsoft Azure"
	description="Obtenga información acerca de cómo puede usarse el servicio de Automatización de Azure para administrar el Almacén de claves de Azure."
	services="Key-Vault, automation"
	documentationCenter=""
	authors="csand-msft"
	manager="eamono"
	editor=""/>

<tags
	ms.service="key-vault"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/18/2016"
	ms.author="csand"/>



#Administración del Almacén de claves de Azure mediante Automatización de Azure

Esta guía le ofrece el servicio Automatización de Azure y cómo se puede usar para simplificar la administración de claves y secretos en el Almacén de claves de Azure.

## ¿Qué es Automatización de Azure?

[Automatización de Azure](https://azure.microsoft.com/services/automation/) es un servicio de Azure para simplificar la administración en la nube mediante la automatización de procesos y la configuración de estado deseado. Mediante Automatización de Azure, se pueden automatizar las tareas de ejecución prolongada, manuales, propensas a errores y que se repiten para aumentar la confiabilidad, la eficiencia y el valioso tiempo para su organización.

Automatización de Azure proporciona un motor de ejecución de flujo de trabajo altamente confiable y de alta disponibilidad que realiza la escalación para satisfacer sus necesidades. En Automatización de Azure, los sistemas de terceros pueden interrumpir los procesos manualmente o en intervalos programados para que las tareas se realicen justo cuando sea necesario.

Reduzca la sobrecarga operativa y libere al personal de TI/DevOps para concentrarse en el trabajo que proporciona valor al negocio trasladando las tareas de administración en la nube para que se ejecuten automáticamente mediante Automatización de Azure.


## ¿Cómo puede ayudar Automatización de Azure a administrar el Almacén de claves de Azure?

El Almacén de claves se puede administrar en Automatización de Azure mediante los [cmdlets del Almacén de claves de AzureRM](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) y los [cmdlets del Almacén de claves clásico de Azure](https://msdn.microsoft.com/library/azure/dn868052.aspx). El módulo de Azure para administrar el Almacén de claves clásico está disponible automáticamente en Automatización de Azure, mientras que el [módulo AzureRM-KeyVault](https://www.powershellgallery.com/packages/AzureRM.KeyVault/1.1.4) se puede importar a Automatización de Azure para poder realizar muchas de las tareas de administración de Almacén de claves dentro del servicio. También puede emparejar estos cmdlets en Automatización de Azure con los cmdlets para otros servicios de Azure, para automatizar tareas complejas a través de los servicios de Azure y sistemas de terceros.

Con los cmdlets del Almacén de claves de Azure puede realizar estas tareas, entre otras: crear y configurar un Almacén de claves, crear o importar una clave, crear o actualizar un secreto, actualizar los atributos de una clave, obtener una clave o un secreto y eliminar una clave o un secreto.

Estos son algunos ejemplos de cómo usar PowerShell para administrar el Almacén de claves:
* [Azure Key Vault - Step by Step](https://blogs.technet.microsoft.com/kv/2015/06/02/azure-key-vault-step-by-step/) (Almacén de claves de Azure: paso a paso)
* [Setting Up and Configuring an Azure Key Vault](https://www.simple-talk.com/cloud/platform-as-a-service/setting-up-and-configuring-an-azure-key-vault/) (Instalar y configurar un Almacén de claves de Azure)


## Pasos siguientes

Ahora que ha aprendido los aspectos básicos de Automatización de Azure y cómo se puede usar para administrar el Almacén de claves de Azure, siga estos vínculos para obtener más información acerca de Automatización de Azure.

* Consulte el [Tutorial de introducción](../automation/automation-first-runbook-graphical.md) de Automatización de Azure.
* Consulte los [scripts de PowerShell del Almacén de claves de Azure](https://gallery.technet.microsoft.com/scriptcenter/Azure-Key-Vault-Powershell-1349b091).

<!---HONumber=AcomDC_0518_2016-->