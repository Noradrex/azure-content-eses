<properties
	pageTitle="Administración de clústeres de Hadoop en HDInsight con el SDK de .NET | Microsoft Azure"
	description="Aprenda a realizar tareas administrativas para clústeres de Hadoop en HDInsight mediante el SDK de .NET en HDInsight."
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.workload="big-data"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/02/2016"
	ms.author="jgao"/>

# Administración de clústeres de Hadoop en HDInsight con el SDK de .NET

[AZURE.INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Más información sobre cómo administrar los clústeres de HDInsight con el [SDK de .NET de HDInsight](https://msdn.microsoft.com/library/mt271028.aspx).


**Requisitos previos**

Antes de empezar este artículo, debe tener lo siguiente:

- **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/).


##Conexión a HDInsight de Azure

Necesitará los siguientes paquetes de Nuget:

	Install-Package Microsoft.Azure.Common.Authentication -Pre
	Install-Package Microsoft.Azure.Management.ResourceManager -Pre
	Install-Package Microsoft.Azure.Management.HDInsight

El ejemplo de código siguiente muestra cómo conectarse a Azure antes de poder administrar clústeres de HDInsight con su suscripción de Azure.

	using System;
	using System.Security;
	using Microsoft.Azure;
	using Microsoft.Azure.Common.Authentication;
	using Microsoft.Azure.Common.Authentication.Factories;
	using Microsoft.Azure.Common.Authentication.Models;
	using Microsoft.Azure.Management.HDInsight;
	using Microsoft.Azure.Management.HDInsight.Models;
	using Microsoft.Azure.Management.ResourceManager;

	namespace HDInsightManagement
	{
		class Program
		{
			private static HDInsightManagementClient _hdiManagementClient;
			private static Guid SubscriptionId = new Guid("<Your Azure Subscription ID>");

			static void Main(string[] args)
			{
				var tokenCreds = GetTokenCloudCredentials();
				var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
				
				var svcClientCreds = new TokenCredentials(tokenCreds.Token); 
				var resourceManagementClient = new ResourceManagementClient(svcClientCreds);
				var rpResult = resourceManagementClient.Providers.Register("Microsoft.HDInsight");

				_hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);

				// insert code here

				System.Console.WriteLine("Press ENTER to continue");
				System.Console.ReadLine();
			}

			public static TokenCloudCredentials GetTokenCloudCredentials(string username = null, SecureString password = null)
			{
				var authFactory = new AuthenticationFactory();

				var account = new AzureAccount { Type = AzureAccount.AccountType.User };

				if (username != null && password != null)
					account.Id = username;

				var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];

				var accessToken =
					authFactory.Authenticate(account, env, AuthenticationFactory.CommonAdTenant, password, ShowDialog.Auto)
						.AccessToken;

				return new TokenCloudCredentials(accessToken);
			}

			public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(TokenCloudCredentials creds, Guid subId)
			{
				return new TokenCloudCredentials(subId.ToString(), creds.Token);
			}
		}
	}

Al ejecutar este programa, verá un mensaje. Si no desea ver el mensaje, consulte [Crear aplicaciones .NET para HDInsight de autenticación no interactiva](hdinsight-create-non-interactive-authentication-dotnet-applications.md).

##Creación de clústeres

Consulte [Crear clústeres basados en Linux en HDInsight con el SDK de .NET](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md).

##Enumeración de clústeres

El fragmento de código siguiente enumera los clústeres y algunas propiedades:

    var results = _hdiManagementClient.Clusters.List();
    foreach (var name in results.Clusters) {
        Console.WriteLine("Cluster Name: " + name.Name);
        Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
        Console.WriteLine("\t Cluster location: " + name.Location);
        Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
    }

##Eliminación de clústeres

Use el siguiente fragmento de código para eliminar un clúster de forma sincrónica o asincrónica:

    _hdiManagementClient.Clusters.Delete("<Resource Group Name>", "<Cluster Name>");
    _hdiManagementClient.Clusters.DeleteAsync("<Resource Group Name>", "<Cluster Name>");
            
##Escalado de clústeres
La característica de escalado de clústeres permite cambiar la cantidad de nodos de trabajo que usa un clúster que se ejecuta en HDInsight de Azure sin necesidad de volver a crear el clúster.

>[AZURE.NOTE] Solo son compatibles los clústeres con la versión 3.1.3 de HDInsight, o superior. Si no está seguro de la versión del clúster, puede comprobar la página de propiedades. Vea [Enumeración y visualización de clústeres](hdinsight-administer-use-portal-linux.md#list-and-show-clusters).

A continuación se muestra el efecto que tiene cambiar la cantidad de nodos de datos de cada tipo de clúster compatible con HDInsight:

- Hadoop

	Puede aumentar sin ningún problema la cantidad de nodos de trabajo en un clúster de Hadoop que se encuentre en ejecución, sin que afecte a ningún trabajo pendiente o en ejecución. También se pueden enviar trabajos nuevos mientras la operación está en curso. Los errores que puedan surgir en una operación de escalado se enfrentan oportunamente, por lo que el clúster siempre queda en estado funcional.

	Cuando se realiza la reducción vertical de un clúster de Hadoop al disminuir la cantidad de nodos de datos, se reinician algunos de los servicios del clúster. Esto provoca que todos los trabajos pendientes y en ejecución fallen al completarse la operación de escalado. Sin embargo, puede volver a enviar los trabajos una vez finalizada la operación.

- HBase

	Puede agregar nodos sin problemas al clúster de HBase mientras se encuentra en ejecución, así como eliminarlos. Los servidores regionales se equilibran automáticamente en unos pocos minutos tras completar la operación de escalado. Sin embargo, puede equilibrar manualmente los servidores regionales iniciando sesión en el nodo principal del clúster y ejecutando los comandos siguientes desde una ventana del símbolo del sistema:

		>pushd %HBASE_HOME%\bin
		>hbase shell
		>balancer

- Storm

	Puede agregar o quitar sin problemas nodos de datos de su clúster de Storm mientras se encuentra en ejecución. Sin embargo, después de finalizar correctamente la operación de escalado, deberá volver a equilibrar la topología.

	Esto se puede realizar de dos formas:

	* La interfaz de usuario web de Storm
	* La herramienta de la interfaz de línea de comandos (CLI)

	Consulte la [documentación de Apache Storm](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) para obtener más detalles.

	La interfaz de usuario web de Storm se encuentra disponible en el clúster de HDInsight:

	![reequilibrio de escalado de storm de hdinsight](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.storm.rebalance.png)

	El siguiente es un ejemplo de cómo usar el comando CLI para volver a equilibrar la topología de Storm:

		## Reconfigure the topology "mytopology" to use 5 worker processes,
		## the spout "blue-spout" to use 3 executors, and
		## the bolt "yellow-bolt" to use 10 executors

		$ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

El fragmento de código siguiente muestra cómo cambiar el tamaño de un clúster de forma sincrónica o asincrónica:

    _hdiManagementClient.Clusters.Resize("<Resource Group Name>", "<Cluster Name>", <New Size>);   
    _hdiManagementClient.Clusters.ResizeAsync("<Resource Group Name>", "<Cluster Name>", <New Size>);   
	

##Concesión o revocación del acceso

Los clústeres de HDInsight tienen los siguientes servicios web HTTP (todos estos servicios tienen extremos RESTful):

- ODBC
- JDBC
- Ambari
- Oozie
- Templeton


De manera predeterminada, estos servicios se conceden para el acceso. Puede revocar/conceder el acceso. Para revocar:

	var httpParams = new HttpSettingsParameters
	{
		HttpUserEnabled = false,
		HttpUsername = "admin",
		HttpPassword = "*******",
	};
	_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);

Para conceder:

	var httpParams = new HttpSettingsParameters
	{
		HttpUserEnabled = enable,
		HttpUsername = "admin",
		HttpPassword = "*******",
	};
	_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);


>[AZURE.NOTE] Al conceder/revocar el acceso, restablecerá el nombre de usuario y la contraseña del clúster.

Esto también se puede hacer a través del Portal. Vea [Administración de HDInsight mediante el Portal de Azure][hdinsight-admin-portal].

##Actualización de las credenciales de usuario HTTP

Es el mismo procedimiento que [Concesión o revocación del acceso HTTP](#grant/revoke-access). Si al clúster se le ha concedido el acceso HTTP, primero debe revocarlo. Y después conceda el acceso con nuevas credenciales de usuario HTTP.


##Búsqueda de la cuenta de almacenamiento predeterminada

El siguiente fragmento de código muestra cómo obtener el nombre y la clave de la cuenta de almacenamiento predeterminada para un clúster.

	var results = _hdiManagementClient.Clusters.GetClusterConfigurations(<Resource Group Name>, <Cluster Name>, "core-site");
	foreach (var key in results.Configuration.Keys)
	{
	    Console.WriteLine(String.Format("{0} => {1}", key, results.Configuration[key]));
	}


##Envío de trabajos

**Para enviar trabajos de MapReduce**

Consulte [Ejecución de ejemplos de Hadoop en HDInsight](hdinsight-hadoop-run-samples-linux.md).

**Para enviar trabajos de Hive**

Consulte [Ejecución de consultas de Hive mediante el SDK de .NET de HDInsight](hdinsight-hadoop-use-hive-dotnet-sdk.md).

**Para enviar trabajos de Pigs**

Consulte [Ejecución de trabajos de Pig con el SDK de .NET para Hadoop en HDInsight](hdinsight-hadoop-use-pig-dotnet-sdk.md).

**Para enviar trabajos de Sqoop**

Consulte [Uso de Sqoop con HDInsight](hdinsight-hadoop-use-sqoop-dotnet-sdk.md).

**Para enviar trabajos de Oozie**

Vea [Uso de Oozie con Hadoop para definir y ejecutar un flujo de trabajo en HDInsight](hdinsight-use-oozie-linux-mac.md).

##Carga de archivos de datos al almacenamiento de blobs de Azure
Consulte [Carga de datos en HDInsight][hdinsight-upload-data].


## Otras referencias
* [Documentación de referencia del SDK de HDInsight](https://msdn.microsoft.com/library/mt271028.aspx)
* [Administración de HDInsight mediante el Portal de Azure][hdinsight-admin-portal]
* [Administración de HDInsight con la interfaz de línea de comandos][hdinsight-admin-cli]
* [Creación de clústeres de HDInsight][hdinsight-provision]
* [Carga de datos en HDInsight][hdinsight-upload-data]
* [Introducción a HDInsight de Azure][hdinsight-get-started]


[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]: hdinsight-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-provision-clusters.md
[hdinsight-provision-custom-options]: hdinsight-provision-clusters.md#configuration
[hdinsight-submit-jobs]: hdinsight-submit-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-portal-linux.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-mapreduce]: hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md

<!---HONumber=AcomDC_0511_2016-->