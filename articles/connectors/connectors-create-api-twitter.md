<properties
	pageTitle="Introducción a la API de Twitter | Microsoft Azure"
	description="Información general del conector de Twitter con parámetros de la API de REST"
	services=""
	documentationCenter="" 
	authors="MandiOhlinger"
	manager="erikre"
	editor=""
	tags="connectors"/>

<tags
   ms.service="multiple"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na" 
   ms.date="05/12/2016"
   ms.author="mandia"/>


# Introducción al conector de Twitter
Conectarse a Twitter para publicar un tweet, obtener la escala de tiempo de un usuario y mucho más. El conector de Twitter puede usarse desde:

- Aplicaciones lógicas 
- PowerApps

> [AZURE.SELECTOR]
- [Aplicaciones lógicas](../articles/connectors/connectors-create-api-twitter.md)
- [PowerApps Enterprise](../articles/power-apps/powerapps-create-api-twitter.md)

&nbsp;

>[AZURE.NOTE] Esta versión del artículo se aplica a la versión de esquema 2015-08-01-preview de las aplicaciones lógicas.

Con Twitter, puede:

- Compilar el flujo de negocio en función de los datos que obtiene de Twitter. 
- Usar desencadenadores cuando haya un nuevo tweet.
- Usar acciones para registrar un tweet, buscar tweets y mucho más. Estas acciones obtienen una respuesta y luego dejan el resultado a disposición de otras acciones. Por ejemplo, cuando aparezca un nuevo tweet, puede publicarlo en Facebook.
- Agregue el conector de Twitter a PowerApps Enterprise. Así, los usuarios pueden utilizar este conector en sus aplicaciones. 

Si desea información sobre cómo agregar un conector en PowerApps Enterprise, acuda a [Registro de una API administrada por Microsoft o una API administrada por TI](../power-apps/powerapps-register-from-available-apis.md).

Para agregar una operación en aplicaciones lógicas, consulte [Creación de una nueva aplicación lógica mediante la conexión de servicios de SaaS](../app-service-logic/app-service-logic-create-a-logic-app.md).


## Desencadenadores y acciones
Twitter incluye los desencadenadores y las acciones siguientes.

Desencadenador | Acciones
--- | ---
<ul><li>Cuando aparezca un nuevo tweet</li></ul>| <ul><li>Publicar un nuevo tweet</li><li>Cuando aparezca un nuevo tweet</li><li>Obtener cronología de inicio</li><li>Obtener usuario</li><li>Obtener cronología de usuario</li><li>Buscar tweet</li><li>Obtener seguidores</li><li>Obtener mis seguidores</li><li>Obtener a quienes se sigue</li><li>Obtener a quienes sigo</li></ul>

Todos los conectores admiten datos en formato JSON y XML.


## Creación de la conexión a Twitter

Al agregar este conector a las aplicaciones lógicas, debe autorizar a estas para que se conecten a su instancia de Twitter.

1. Inicie sesión en su cuenta de Twitter.
2. Seleccione **Autorizar** y permita que las aplicaciones lógicas se conecten y utilicen su cuenta de Twitter. 

>[AZURE.INCLUDE [Pasos para crear una conexión a Twitter](../../includes/connectors-create-api-twitter.md)]

>[AZURE.TIP] Puede usar esta misma conexión de Twitter en otras aplicaciones lógicas.


## Referencia de la API de REST de Swagger
Se aplica a la versión: 1.0.

### Publicar un nuevo tweet 
Publica un tweet. ```POST: /posttweet```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|tweetText|cadena|no|query|Ninguna|Texto que se va a publicar|
|body| cadena (binario) |no|body|Ninguna|Elementos multimedia que se van a publicar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Cuando aparezca un nuevo tweet 
Activa un flujo de trabajo cuando se publica un nuevo tweet que coincide con su consulta de búsqueda. ```GET: /onnewtweet```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|searchQuery|cadena|yes|query|Ninguna|Texto de consulta (puede usar los operadores de consulta de Twitter admitidos: http://www.twitter.com/search)|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|202|Accepted|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener escala de tiempo de inicio 
Recupera los tweets y retweets más recientes enviados por mí y mis seguidores. ```GET: /hometimeline```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|maxResults|integer|no|query|20|Número máximo de tweets que recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener usuario 
Recupera los detalles sobre el usuario especificado (ejemplo: nombre de usuario, descripción, número de seguidores, etc.) ```GET: /user```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|userName|cadena|yes|query|Ninguna|Controlador de usuario de Twitter|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener biografía de usuario 
Recupera una colección de los tweets más recientes publicados por el usuario especificado. ```GET: /usertimeline```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|userName|cadena|yes|query|Ninguna|Identificador de Twitter|
|maxResults|integer|no|query|20|Número máximo de tweets que recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Buscar tweet 
Recupera una colección de tweets relevantes que coincidan con una consulta especificada. ```GET: /searchtweets```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|searchQuery|cadena|yes|query|Ninguna|Texto de consulta (puede usar los operadores de consulta de Twitter admitidos: http://www.twitter.com/search)|
|maxResults|integer|no|query|20|Número máximo de tweets que recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener seguidores 
Recupera los usuarios que siguen al usuario especificado. ```GET: /followers```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|userName|cadena|yes|query|Ninguna|Controlador de usuario de Twitter|
|maxResults|integer|no|query|20|Número máximo de usuarios para recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener mi seguidores 
Recupera los usuarios que me siguen a mí. ```GET: /myfollowers```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|maxResults|integer|no|query|20|Número máximo de usuarios para recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener a quienes se sigue 
Recupera los usuarios a quienes sigue el usuario especificado. ```GET: /friends```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|userName|cadena|yes|query|Ninguna|Controlador de usuario de Twitter|
|maxResults|integer|no|query|20|Número máximo de usuarios para recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


### Obtener a quienes sigo 
Recupera los usuarios a quienes sigo. ```GET: /myfriends```

| Nombre| Tipo de datos|Obligatorio|Ubicado en|Valor predeterminado|Descripción|
| ---|---|---|---|---|---|
|maxResults|integer|no|query|20|Número máximo de usuarios para recuperar|

#### Respuesta
|Nombre|Descripción|
|---|---|
|200|OK|
|400|Bad Request|
|401|No autorizado|
|403|Prohibido|
|404|No encontrado|
|500|Error interno del servidor. Error desconocido|
|default|Error en la operación.|


## Definiciones de objeto

### TweetModel: representación de objeto tweet

| Nombre de propiedad | Tipo de datos | Obligatorio |
|---|---| --- | 
|TweetText|cadena|yes|
|TweetId|cadena|no|
|CreatedAt|cadena|no|
|RetweetCount|integer|yes|
|TweetedBy|cadena|yes|
|MediaUrls|array|no|

### UserDetailsModel: representación de los detalles de un usuario de Twitter

|Nombre de propiedad | Tipo de datos | Obligatorio |
|---|---|---|
|FullName|cadena|yes|
|Ubicación|cadena|yes|
|Id|integer|no|
|UserName|cadena|yes|
|FollowersCount|integer|no|
|Descripción|cadena|yes|
|StatusesCount|integer|no|
|FriendsCount|integer|no|

### TweetResponseModel: modelo que representa el tweet publicado

| Nombre | Tipo de datos | Obligatorio |
|---|---|---|
|TweetId|cadena|yes|

### TriggerBatchResponse[TweetModel]

|Nombre de propiedad | Tipo de datos |Obligatorio |
|---|---|---|
|value|array|no|


## Pasos siguientes

[Crear una aplicación lógica](../app-service-logic/app-service-logic-create-a-logic-app.md).

Volver a la [lista de API](apis-list.md).

<!--References-->

[6]: ./media/connectors-create-api-twitter/twitter-apps-page.png
[7]: ./media/connectors-create-api-twitter/twitter-app-create.png

<!---HONumber=AcomDC_0518_2016-->