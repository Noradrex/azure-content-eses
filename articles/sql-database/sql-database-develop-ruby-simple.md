<properties
	pageTitle="Conexión a Base de datos SQL mediante Ruby | Microsoft Azure"
	description="Muestra de código Ruby que puede ejecutar en Windows para conectarse a Base de datos SQL de Azure."
	services="sql-database"
	documentationCenter=""
	authors="ajlam"
	manager="jhubbard"
	editor=""/>


<tags
	ms.service="sql-database"
	ms.workload="sql-database"
	ms.tgt_pltfrm="na"
	ms.devlang="ruby"
	ms.topic="article"
	ms.date="04/07/2016"
	ms.author="andrela"/>


# Conexión a Base de datos SQL mediante Ruby 

[AZURE.INCLUDE [sql-database-develop-includes-selector-language-platform-depth](../../includes/sql-database-develop-includes-selector-language-platform-depth.md)]

En este tema se muestra cómo conectarse a una base de datos SQL de Azure y consultarla mediante Ruby. Puede ejecutar esta muestra desde las plataformas Windows, Ubuntu Linux o Mac.

## Paso 1: Configuración del entorno de desarrollo

[Requisitos previos para usar el controlador de Ruby TinyTDS para SQL Server](https://msdn.microsoft.com/library/mt711041.aspx)

## Paso 2: Creación de una base de datos SQL

Vea la [página de introducción](sql-database-get-started.md) para aprender a crear una base de datos de ejemplo. Es importante seguir las directrices para crear una **plantilla de base de datos de AdventureWorks**. Los ejemplos que se muestran a continuación solo funcionan con el **esquema de AdventureWorks**.

## Paso 3: Obtención de detalles de la conexión

[AZURE.INCLUDE [sql-database-include-connection-string-details-20-portalshots](../../includes/sql-database-include-connection-string-details-20-portalshots.md)]

## Paso 4: ejecución de código de muestra

[Proof of Concept connecting to SQL using Ruby](http://msdn.microsoft.com/library/mt715797.aspx) (Prueba de concepto que se conecta a SQL con Ruby)

<!---HONumber=AcomDC_0518_2016-->