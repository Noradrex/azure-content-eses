<properties
	pageTitle="Introducción al almacenamiento de colas y los servicios conectados de Visual Studio (ASP.NET 5) | Microsoft Azure"
	description="Cómo empezar a usar el almacenamiento en cola de Azure en un proyecto de ASP.NET en Visual Studio"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.workload="web"
	ms.tgt_pltfrm="vs-getting-started"
	ms.devlang="na"
	ms.topic="article"
	ms.date="05/08/2016"
	ms.author="tarcher"/>

# Introducción al almacenamiento de colas y a los servicios conectados de Visual Studio (ASP.NET 5)

##Información general

En este artículo se describe cómo empezar a usar Almacenamiento en cola de Azure en Visual Studio después de crear a una cuenta de almacenamiento de Azure en un proyecto de ASP.NET 5 con el cuadro de diálogo **Agregar servicios conectados** de Visual Studio, o después de hacer referencia a una. La operación **Agregar servicios conectados** instala los paquetes de NuGet adecuados para tener acceso al almacenamiento de Azure en el proyecto y agrega la cadena de conexión de la cuenta de almacenamiento a los archivos de configuración del proyecto.

El almacenamiento de cola de Azure es un servicio para almacenar grandes cantidades de mensajes a los que puede obtenerse acceso desde cualquier lugar del mundo a través de llamadas autenticadas con HTTP o HTTPS. Un único mensaje en cola puede tener un tamaño de hasta 64 kilobytes (KB) y una cola puede contener millones de mensajes, hasta el límite de capacidad total de una cuenta de almacenamiento.

Para comenzar, necesita crear una cola de Azure en la cuenta de almacenamiento. Le mostraremos cómo crear una cola en el código. También le mostraremos cómo realizar operaciones básicas de cola, como agregar, modificar, leer y quitar mensajes de cola. Los ejemplos están escritos en código C# y utilizan la biblioteca del cliente de almacenamiento de Azure para .NET. Para obtener más información acerca de ASP.NET, consulte [ASP.NET](http://www.asp.net).

**NOTA:** algunas de las API que realizan llamadas al almacenamiento de Azure en ASP.NET 5 son asincrónicas. Vea [Programación asincrónica con Async y Await](http://msdn.microsoft.com/library/hh191443.aspx) para más información. El código siguiente supone que se están utilizando métodos de programación asincrónica.

- Vea [Introducción al Almacenamiento en cola de Azure mediante .NET](storage-dotnet-how-to-use-queues.md) para más información sobre la manipulación de colas mediante programación.
- Vea [Documentación sobre Almacenamiento](https://azure.microsoft.com/documentation/services/storage/) para información general sobre Almacenamiento de Azure.
- Vea [Documentación sobre Servicios en la nube](https://azure.microsoft.com/documentation/services/cloud-services/) para información general sobre los servicios en la nube de Azure.
- Vea [ASP.NET](http://www.asp.net) para más información sobre la programación de aplicaciones ASP.NET.





##Acceso a colas en el código

Para obtener acceso a las colas en los proyectos de ASP.NET 5, debe incluir los elementos siguientes en los archivos de origen C# que tengan acceso al almacenamiento de colas de Azure.

1. Asegúrese de que las declaraciones del espacio de nombres de la parte superior del archivo de C# incluyen estas instrucciones **using**.

		using Microsoft.Framework.Configuration;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Queue;
		using System.Threading.Tasks;
		using LogLevel = Microsoft.Framework.Logging.LogLevel;

2. Obtenga un objeto **CloudStorageAccount** que represente la información de su cuenta de almacenamiento. Use el código siguiente para obtener la cadena de conexión de almacenamiento y la información de la cuenta de almacenamiento de la configuración del servicio de Azure.

		 CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		   CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

3. Obtenga un objeto **CloudTableClient** para hacer referencia a los objetos de cola en la cuenta de almacenamiento.

	    // Create the CloudQueueClient object for the storage account.
    	CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

4. Obtenga un objeto **CloudQueue** para hacer referencia a una cola específica.

    	// Get a reference to the CloudQueue named "messageQueue"
	    CloudQueue messageQueue = queueClient.GetQueueReference("messageQueue");


**NOTA:** use todo el código anterior delante del código que aparece en las muestras siguientes.

###Creación de una cola en código

Para crear la cola de Azure en el código, solo tiene que agregar una llamada a **CreateIfNotExistsAsync**.

	// Create the CloudQueue if it does not exist.
	await messageQueue.CreateIfNotExistsAsync();

##un mensaje a una cola

Para insertar un mensaje en una cola existente, cree un nuevo objeto **CloudQueueMessage** y luego llame al método **AddMessageAsync**.

Se puede crear un objeto **CloudQueueMessage** a partir de una cadena (en formato UTF-8) o de una matriz de bytes.

Este es un ejemplo que inserta el mensaje "Hello, World".

	// Create a message and add it to the queue.
	CloudQueueMessage message = new CloudQueueMessage("Hello, World");
	await messageQueue.AddMessageAsync(message);

##Leer un mensaje de una cola

Puede inspeccionar el mensaje situado al principio de una cola sin quitarlo de ella, llamando al método **PeekMessageAsync**.

	// Peek the next message in the queue. 
	CloudQueueMessage peekedMessage = await messageQueue.PeekMessageAsync();


##Leer y eliminar un mensaje de una cola

Su código puede quitar un mensaje de una cola en dos pasos.
1. Llame a **GetMessageAsync** para obtener el mensaje siguiente de una cola. Un mensaje devuelto por **GetMessageAsync** se hace invisible a cualquier otro código de lectura de mensajes de esta cola. De forma predeterminada, este mensaje permanece invisible durante 30 segundos.
2.	Para terminar de quitar el mensaje de la cola, llame a **DeleteMessageAsync**.

Este proceso extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código siguiente llama a **DeleteMessageAsync** justo después de procesar el mensaje.

	// Get the next message in the queue.
	CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();

	// Process the message in less than 30 seconds.

    // Then delete the message.
	await messageQueue.DeleteMessageAsync(retrievedMessage);

## las opciones adicionales de los mensajes quitados de la cola

Hay dos formas de personalizar la recuperación de mensajes de una cola. En primer lugar, puede obtener un lote de mensajes (hasta 32). En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje. El siguiente ejemplo de código utiliza el método **GetMessages** para obtener 20 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle **foreach**. También establece el tiempo de espera de la invisibilidad en 5 minutos para cada mensaje. Tenga en cuenta que los 5 minutos empiezan a contar para todos los mensajes al mismo tiempo, por lo que, después de pasar los 5 minutos desde la llamada a **GetMessages**, todos los mensajes que no se han eliminado volverán a estar visibles.

    // Retrieve 20 messages at a time, keeping those messages invisible for 5 minutes, 
    //   delete each message after processing.

    foreach (CloudQueueMessage message in messageQueue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.
        queue.DeleteMessage(message);
    }

## la longitud de la cola

Puede obtener una estimación del número de mensajes existentes en una cola. El método **FetchAttributes** solicita al servicio de cola la recuperación de los atributos de la cola, incluido el número de mensajes. La propiedad **ApproximateMethodCount** devuelve el último valor recuperado por el método **FetchAttributes**, sin llamar al servicio de cola.

	// Fetch the queue attributes.
	messageQueue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = messageQueue.ApproximateMessageCount;

	// Display the number of messages.
	Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## Uso del patrón Async-Await con API de cola comunes

En este ejemplo se muestra cómo usar el patrón Async-Await con API de cola comunes. El ejemplo llama a la versión asincrónica de cada uno de los métodos determinados. Esto se puede ver con el postfijo de Async de cada método. Cuando se usa un método asincrónico, el patrón Async-Await suspende la ejecución local hasta que se complete la llamada. Este comportamiento permite que el subproceso actual realice otro trabajo que ayuda a evitar cuellos de botella en el rendimiento y mejora la capacidad de respuesta general de la aplicación. Para más información sobre el uso del patrón Async-Await en. NET, vea [Async y Await (C# y Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx).

    // Create a message to add to the queue.
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Async enqueue the message.
    await messageQueue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message.
    CloudQueueMessage retrievedMessage = await messageQueue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Async delete the message.
    await messageQueue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");
## Eliminación de una cola

Para eliminar una cola y todos los mensajes contenidos en ella, llame al método **Delete** en el objeto de cola.

    // Delete the queue.
    messageQueue.Delete();


##Pasos siguientes

[AZURE.INCLUDE [vs-storage-dotnet-queues-next-steps](../../includes/vs-storage-dotnet-queues-next-steps.md)]

<!---HONumber=AcomDC_0511_2016-->