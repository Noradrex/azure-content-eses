<properties
	pageTitle="Introducción al Centro de IoT de Azure para Node.js | Microsoft Azure"
	description="Tutorial de introducción al Centro de IoT de Azure con Node.js Utilice Centro de IoT de Azure y Node.js con los SDK de IoT de Microsoft Azure para implementar una solución de Internet de las cosas."
	services="iot-hub"
	documentationCenter="nodejs"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.devlang="javascript"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="03/22/2016"
     ms.author="dobett"/>

# Introducción al Centro de IoT de Azure para Node.js

[AZURE.INCLUDE [iot-hub-selector-get-started](../../includes/iot-hub-selector-get-started.md)]

## Introducción

El Centro de IoT de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos de Internet de las cosas (IoT) y un back-end de soluciones. Uno de los mayores desafíos que plantean los proyectos de IoT es cómo conectar dispositivos al back-end de la solución de manera segura y confiable. Para abordar este desafío, el Centro de IoT:

- Ofrece una mensajería confiable de gran escala de dispositivo a nube y de nube a dispositivo.
- Habilita las comunicaciones seguras con las credenciales de seguridad de cada dispositivo y el control de acceso.
- Incluye bibliotecas de dispositivos para las plataformas y los lenguajes más populares.

En este tutorial se muestra cómo realizar las siguientes acciones:

- Usar el Portal de Azure para crear un Centro de IoT.
- Crear una identidad de dispositivo en el Centro de IoT.
- Crear un dispositivo simulado que envía la telemetría al back-end en la nube.

Al final de este tutorial tendrá tres aplicaciones de consola de Node.js:

* **CreateDeviceIdentity.js**, que crea una identidad de dispositivo y una clave de seguridad asociada para conectar el dispositivo simulado.
* **ReadDeviceToCloudMessages.js**, que muestra los datos de telemetría enviados por el dispositivo simulado.
* **SimulatedDevice.js**, que se conecta con su Centro de IoT con la identidad del dispositivo creada anteriormente y envía un mensaje de telemetría cada segundo mediante el protocolo AMQPS.

> [AZURE.NOTE] El artículo [SDK de Centro de IoT][lnk-hub-sdks] proporciona información acerca de los SDK que puede usar para crear dos aplicaciones para ejecutarse en dispositivos y en el back-end de la solución.

Para completar este tutorial, necesitará lo siguiente:

+ Node.js versión 0.12.x o posteriores. <br/> [Preparación del entorno de desarrollo][lnk-dev-setup] describe cómo instalar Node.js para este tutorial en Windows o Linux.

+ Una cuenta de Azure activa. <br/>En caso de no tener ninguna, puede crear una cuenta de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure][lnk-free-trial].

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

Como paso final, haga clic en **Configuración** en la hoja Centro de IoT y luego en **Mensajes** en la hoja **Configuración**. En la hoja **Mensajes**, anote los valores de **Nombre compatible con Centro de eventos** y **Punto de conexión compatible de Centro de eventos**. Necesitará estos valores al crear la aplicación **read-d2c-messages**.

![][6]

Ahora que ha creado un Centro de IoT y que tiene el nombre de host del Centro de IoT, la cadena de conexión del Centro de IoT, el nombre compatible con el Centro de eventos y los valores de punto de conexión compatibles con el Centro de eventos, es preciso que complete el resto del tutorial.

## Creación de una identidad de dispositivo

En esta sección, creará una aplicación de consola de Node.js que crea una nueva identidad de dispositivo en el registro de identidades de su Centro de IoT. No se puede conectar un dispositivo al Centro de IoT a menos que tenga una entrada en el registro de identidades de dispositivo. Consulte la sección **Registro de identidad de dispositivos** de la [Guía para desarrolladores del Centro de IoT][lnk-devguide-identity] para obtener más información. Al ejecutar esta aplicación de consola, genera y una clave y un identificador de dispositivo únicos con el que el dispositivo puede identificarse cuando envía al Centro de IoT mensajes de dispositivo a la nube.

1. Cree una nueva carpeta vacía denominada **createdeviceidentity**. En la carpeta **createdeviceidentity**, cree un nuevo archivo package.json con el siguiente comando en el símbolo del sistema. Acepte todos los valores predeterminados:

    ```
    npm init
    ```

2. En el símbolo del sistema en la carpeta **createdeviceidentity**, ejecute el siguiente comando para instalar el paquete **azure-iothub**:

    ```
    npm install azure-iothub --save
    ```

3. Con un editor de texto, cree un nuevo archivo **CreateDeviceIdentity.js** en la carpeta **createdeviceidentity**.

4. Agregue la siguiente instrucción `require` al principio del archivo **CreateDeviceIdentity.js**:

    ```
    'use strict';
    
    var iothub = require('azure-iothub');
    ```

5. Agregue el siguiente código al archivo **CreateDeviceIdentity.js**; para ello, sustituya el valor de marcador de posición por la cadena de conexión del Centro de IoT que creó en la sección anterior:

    ```
    var connectionString = '{iothub connection string}';
    
    var registry = iothub.Registry.fromConnectionString(connectionString);
    ```

6. Agregue el siguiente código para crear una nueva definición de dispositivo en el registro de identidades de dispositivo en el Centro de IoT. Este código crea un nuevo dispositivo si el identificador de dispositivo no existe en el registro o, de lo contrario, devuelve la clave del dispositivo existente:

    ```
    var device = new iothub.Device(null);
    device.deviceId = 'myFirstDevice';
    registry.create(device, function(err, deviceInfo, res) {
      if (err) {
        registry.get(device.deviceId, printDeviceInfo);
      }
      if (deviceInfo) {
        printDeviceInfo(err, deviceInfo, res)
      }
    });

    function printDeviceInfo(err, deviceInfo, res) {
      if (deviceInfo) {
        console.log('Device id: ' + deviceInfo.deviceId);
        console.log('Device key: ' + deviceInfo.authentication.SymmetricKey.primaryKey);
      }
    }
    ```

7. Guarde y cierre el archivo **CreateDeviceIdentity.js**.

8. Para ejecutar la aplicación **createdeviceidentity**, ejecute el siguiente comando en el símbolo del sistema en la carpeta createdeviceidentity:

    ```
    node CreateDeviceIdentity.js 
    ```

9. Tome nota del **identificador de dispositivo** y de la **clave de dispositivo**. Los necesitará más adelante cuando cree una aplicación que se conecta a Centro de IoT como un dispositivo.

> [AZURE.NOTE] El registro de identidades del Centro de IoT solo almacena las identidades de dispositivo para permitir el acceso seguro al centro. Almacena las claves y los identificadores de dispositivo para usarlos como credenciales de seguridad, y un indicador de habilitado o deshabilitado que permite deshabilitar el acceso a un dispositivo individual. Si la aplicación necesita almacenar otros metadatos específicos del dispositivo, debe usar un almacén específico de la aplicación. Consulte [Guía para desarrolladores del Centro de IoT][lnk-devguide-identity] para más información.

## Recepción de mensajes de dispositivo a nube

En esta sección, creará una aplicación de consola de Node.js que lee los mensajes de dispositivo a nube del Centro de IoT. El Centro de IoT expone un punto de conexión compatible con [Centros de eventos ][lnk-event-hubs-overview] para poder leer los mensajes de dispositivo a nube. Para simplificar las cosas, este tutorial crea un lector básico que no es apto para una implementación de alta capacidad de procesamiento. El [Tutorial: procesamiento de mensajes de dispositivo a la nube del Centro de IoT][lnk-process-d2c-tutorial] muestra cómo procesar mensajes de dispositivo a la nube a escala. En el tutorial [Introducción a los Centros de eventos][lnk-eventhubs-tutorial] se proporciona información adicional acerca de cómo procesar los mensajes desde los Centros de eventos. Dicha información se puede aplicar a los puntos de conexión compatibles con Centros de eventos de Centro de IoT.

> [AZURE.NOTE] El punto de conexión compatible con los Centros de eventos para leer mensajes de dispositivo a la nube siempre usa el protocolo AMQPS.

1. Cree una nueva carpeta vacía denominada **readdevicetocloudmessages**. En la carpeta **readdevicetocloudmessages**, cree un nuevo archivo package.json con el siguiente comando en el símbolo del sistema. Acepte todos los valores predeterminados:

    ```
    npm init
    ```

2. En el símbolo del sistema, en la carpeta **readdevicetocloudmessages**, ejecute el siguiente comando para instalar los paquetes **amqp10** y **bluebird**:

    ```
    npm install amqp10 bluebird --save
    ```

3. Con un editor de texto, cree un nuevo archivo **ReadDeviceToCloudMessages.js** en la carpeta **readdevicetocloudmessages**.

4. Agregue las siguientes instrucciones `require` al principio del archivo **ReadDeviceToCloudMessages.js**:

    ```
    'use strict';

    var AMQPClient = require('amqp10').Client;
    var Policy = require('amqp10').Policy;
    var translator = require('amqp10').translator;
    var Promise = require('bluebird');
    ```

5. Agregue las siguientes declaraciones de variables, reemplazando los marcadores de posición por los valores que anotó anteriormente. El valor del marcador de posición **{your event hub-compatible namespace}** proviene del campo de **Punto de conexión compatible con Centro de eventos** en el portal y adopta la forma siguiente: **namespace.servicebus.windows.net** (sin el prefijo **sb://*).

    ```
    var protocol = 'amqps';
    var eventHubHost = '{your event hub-compatible namespace}';
    var sasName = 'iothubowner';
    var sasKey = '{your iot hub key}';
    var eventHubName = '{your event hub-compatible name}';
    var numPartitions = 2;
    ```

    > [AZURE.NOTE] Este código supone que usted creó el Centro de IoT en el nivel de F1 (gratis). Un Centro de IoT gratis tiene dos particiones denominadas "0" y "1". Si creó el Centro de IoT con otro plan de tarifa, debe ajustar el código para crear una clase **MessageReceiver** para cada partición.

6. Agregue la siguiente definición de filtro. Esta aplicación utiliza un filtro cuando crea el receptor para que este solo lea los mensajes enviados al Centro de IoT después de que el receptor comience a ejecutarse. Esto es útil en un entorno de prueba, porque puede ver el conjunto actual de mensajes, pero en un entorno de producción el código debe asegurarse de que se procesan todos los mensajes. Consulte el [Tutorial: procesamiento de mensajes de dispositivo a la nube del Centro de IoT][lnk-process-d2c-tutorial] para más información.

    ```
    var filterOffset = new Date().getTime();
    var filterOption;
    if (filterOffset) {
      filterOption = {
      attach: { source: { filter: {
      'apache.org:selector-filter:string': translator(
        ['described', ['symbol', 'apache.org:selector-filter:string'], ['string', "amqp.annotation.x-opt-enqueuedtimeutc > " + filterOffset + ""]])
        } } }
      };
    }
    ```

7. Agregue el código siguiente para crear la dirección de recepción y un cliente AMQP:

    ```
    var uri = protocol + '://' + encodeURIComponent(sasName) + ':' + encodeURIComponent(sasKey) + '@' + eventHubHost;
    var recvAddr = eventHubName + '/ConsumerGroups/$default/Partitions/';
    
    var client = new AMQPClient(Policy.EventHub);
    ```

8. Agregue las siguientes dos funciones que imprimen la salida en la consola:

    ```
    var messageHandler = function (partitionId, message) {
      console.log('Received(' + partitionId + '): ', message.body);
    };
    
    var errorHandler = function(partitionId, err) {
      console.warn('** Receive error: ', err);
    };
    ```

9. Agregue la siguiente función que actúa como un receptor de una partición determinada mediante el filtro:

    ```
    var createPartitionReceiver = function(partitionId, receiveAddress, filterOption) {
      return client.createReceiver(receiveAddress, filterOption)
        .then(function (receiver) {
          console.log('Listening on partition: ' + partitionId);
          receiver.on('message', messageHandler.bind(null, partitionId));
          receiver.on('errorReceived', errorHandler.bind(null, partitionId));
        });
    };
    ```

10. Agregue el código siguiente para conectarse al punto de conexión compatible con el Centro de eventos e inicie los receptores:

    ```
    client.connect(uri)
      .then(function () {
        var partitions = [];
        for (var i = 0; i < numPartitions; ++i) {
          partitions.push(createPartitionReceiver(i, recvAddr + i, filterOption));
        }
        return Promise.all(partitions);
    })
    .error(function (e) {
        console.warn('Connection error: ', e);
    });
    ```

11. Guarde y cierre el archivo **ReadDeviceToCloudMessages.js**.

## Creación de una aplicación de dispositivo simulado

En esta sección, creará una aplicación de consola de Node.js que simula un dispositivo que envía mensajes de dispositivo a nube a un Centro de IoT.

1. Cree una nueva carpeta vacía denominada **simulateddevice**. En la carpeta **simulateddevice**, cree un nuevo archivo package.json con el siguiente comando en el símbolo del sistema. Acepte todos los valores predeterminados:

    ```
    npm init
    ```

2. En el símbolo del sistema en la carpeta **simulateddevice**, ejecute el siguiente comando para instalar el paquete **azure-iot-device-amqp**:

    ```
    npm install azure-iot-device-amqp --save
    ```

3. Con un editor de texto, cree un nuevo archivo **SimulatedDevice.js** en la carpeta **simulateddevice**.

4. Agregue las siguientes instrucciones `require` al principio del archivo **SimulatedDevice.js**:

    ```
    'use strict';

    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
    var Message = require('azure-iot-device').Message;
    ```

5. Agregue una variable **connectionString** y utilícela para crear un cliente de dispositivo. Reemplace **{youriothubname}** por el nombre del Centro de IoT y **{yourdeviceid}** y **{yourdevicekey}** por los valores del dispositivo generados en la sección *Creación de una identidad de dispositivo*:

    ```
    var connectionString = 'HostName={youriothubname}.azure-devices.net;DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
    
    var client = clientFromConnectionString(connectionString);
    ```

6. Agregue la siguiente función para mostrar la salida de la aplicación:

    ```
    function printResultFor(op) {
      return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ' status: ' + res.constructor.name);
      };
    }
    ```

7. Cree una devolución de llamada y utilice la función **setInterval** para enviar un nuevo mensaje a su Centro de IoT cada segundo:

    ```
    var connectCallback = function (err) {
      if (err) {
        console.log('Could not connect: ' + err);
      } else {
        console.log('Client connected');

        // Create a message and send it to the IoT Hub every second
        setInterval(function(){
            var windSpeed = 10 + (Math.random() * 4);
            var data = JSON.stringify({ deviceId: 'mydevice', windSpeed: windSpeed });
            var message = new Message(data);
            console.log("Sending message: " + message.getData());
            client.sendEvent(message, printResultFor('send'));
        }, 2000);
      }
    };
    ```

8. Abra la conexión al su Centro de IoT y empiece a enviar mensajes:

    ```
    client.open(connectCallback);
    ```

9. Guarde y cierre el archivo **SimulatedDevice.js**.

> [AZURE.NOTE] Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo de MSDN [Control de errores transitorios][lnk-transient-faults].


## Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En un símbolo del sistema, en la carpeta **readdevicetocloudmessages**, ejecute el siguiente comando para empezar la supervisión del Centro de IoT:

    ```
    node ReadDeviceToCloudMessages.js 
    ```

    ![][7]

2. En un símbolo del sistema, en la carpeta **simulateddevice**, ejecute el siguiente comando para empezar el envío de datos de telemetría al Centro de IoT:

    ```
    node SimulatedDevice.js
    ```

    ![][8]

3. El icono **Uso** del [Portal de Azure][lnk-portal] muestra el número de mensajes enviados al centro:

    ![][43]

## Pasos siguientes

En este tutorial, configuró un nuevo Centro de IoT en el portal y después creó una identidad de dispositivo en el registro de identidades del centro. Esta identidad de dispositivo se usó para habilitar la aplicación del dispositivo simulado para enviar al centro los mensajes de dispositivo a la nube y creó otra aplicación que muestra los mensajes recibidos por el centro. Puede seguir explorando las características del Centro de IoT y otros escenarios en los tutoriales siguientes:

- [Envío de mensajes de nube a dispositivo con el Centro de IoT][lnk-c2d-tutorial] muestra cómo enviar mensajes a dispositivos y procesar los comentarios de entrega generados por el Centro de IoT.
- [Procesamiento de mensajes de dispositivo a la nube][lnk-process-d2c-tutorial] muestra cómo procesar de forma confiable la telemetría y los mensajes interactivos procedentes de los dispositivos.
- [Cómo cargar archivos desde dispositivos a la nube ][lnk-upload-tutorial] describe un patrón que usa mensajes de nube a dispositivo para facilitar la carga de archivos desde los dispositivos.

<!-- Images. -->
[6]: ./media/iot-hub-node-node-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-node-node-getstarted/runapp1.png
[8]: ./media/iot-hub-node-node-getstarted/runapp2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png

<!-- Links -->
[lnk-transient-faults]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx

[lnk-eventhubs-tutorial]: ../event-hubs/event-hubs-csharp-ephcs-getstarted.md
[lnk-devguide-identity]: iot-hub-devguide.md#identityregistry
[lnk-event-hubs-overview]: ../event-hubs/event-hubs-overview.md

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/node/device/doc/devbox_setup.md
[lnk-c2d-tutorial]: iot-hub-csharp-csharp-c2d.md
[lnk-process-d2c-tutorial]: iot-hub-csharp-csharp-process-d2c.md
[lnk-upload-tutorial]: iot-hub-csharp-csharp-file-upload.md

[lnk-hub-sdks]: iot-hub-sdks-summary.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal]: https://portal.azure.com/

<!---HONumber=AcomDC_0511_2016-->