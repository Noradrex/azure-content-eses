<properties 
	pageTitle="Análisis de aplicaciones web para ASP.NET con Application Insights" 
	description="Análisis de rendimiento, disponibilidad y uso para el sitio web de ASP.NET, hospedado localmente o en Azure." 
	services="application-insights" 
    documentationCenter=".net"
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="application-insights" 
	ms.workload="tbd" 
	ms.tgt_pltfrm="ibiza" 
	ms.devlang="na" 
	ms.topic="get-started-article" 
	ms.date="05/12/2016" 
	ms.author="awills"/>


# Configurar Application Insights para ASP.NET


[AZURE.INCLUDE [app-insights-selector-get-started-dotnet](../../includes/app-insights-selector-get-started-dotnet.md)]

El SDK de Application Insights envía un análisis telemétrico desde su aplicación web activa al Portal de Azure, donde puede iniciar sesión y ver gráficos que representan el rendimiento y uso de la aplicación.

![Gráficos de supervisión de rendimiento de ejemplo](./media/app-insights-asp-net/10-perf.png)

También podrá investigar y correlacionar solicitudes específicas, excepciones y eventos de registro. Puede usar la API para agregar la telemetría para supervisar el rendimiento y el uso en detalle.

#### Antes de comenzar

Necesita:

* Una suscripción a [Microsoft Azure](http://azure.com). Si su equipo u organización tiene una suscripción a Azure, el propietario puede agregarle a esta con su [cuenta Microsoft](http://live.com).
* Visual Studio 2013, actualización 3 o superior.


## <a name="ide"></a> Adición de Application Insights a su proyecto de Visual Studio

#### Si se trata de un proyecto nuevo:

Cuando cree un proyecto nuevo en Visual Studio, asegúrese de seleccionar Application Insights.


![Creación de un proyecto ASP.NET](./media/app-insights-asp-net/appinsights-01-vsnewp1.png)

Seleccione una cuenta con un inicio de sesión de Azure. Puede que reciba una invitación a volver a escribir sus credenciales. (O, si no inicia sesión, se agregará el código del SDK, y puede configurarlo más adelante).


#### ....o si se trata de un proyecto existente

Haga clic con el botón derecho en el proyecto del Explorador de soluciones y elija **Agregar Application Insights** o **Configurar Application Insights**.

![Elija Agregar Application Insights](./media/app-insights-asp-net/appinsights-03-addExisting.png)


#### Opciones de configuración

Si esta es su primera vez, se le invitará a que inicie sesión o se registre en Microsoft Azure.

Si esta aplicación forma parte de una aplicación mayor, es posible que quiera usar **Configurar valor** para colocarla en el mismo grupo de recursos que los demás componentes.


####<a name="land"></a> ¿Qué hizo "Agregar Application Insights"?

El comando realizó estos pasos (que podría [hacer manualmente](app-insights-asp-net-manual.md) en su lugar si lo prefiere):

1. Agrega el paquete NuGet del SDK web de Application Insights al proyecto. Para verlo en Visual Studio, haga clic con el botón secundario en el proyecto y elija Administrar paquetes de NuGet.
2. Crea un recurso de Application Insights en el [portal de Azure][portal]. Es donde verá los datos. Recupera la *clave de instrumentación*, que identifica el recurso.
3. Inserta la clave de instrumentación en `ApplicationInsights.config`, de modo que el SDK puede enviar datos de telemetría al portal.

Si no inicia sesión en Azure inicialmente, el SDK se instalará sin necesidad de conectarlo a un recurso. Podrá ver y buscar la telemetría de Application Insights en la ventana de búsqueda de Visual Studio mientras realiza la depuración. Puede llevar a cabo los demás pasos más adelante.

## <a name="run"></a> Depure el proyecto

Ejecute la aplicación con F5 y pruébela. Abra varias páginas para generar telemetría.

En Visual Studio, verá un recuento de los eventos que se han registrado.

![En Visual Studio, el botón de Application Insights se muestra durante la depuración.](./media/app-insights-asp-net/appinsights-09eventcount.png)

Haga clic en este botón para abrir la búsqueda de diagnóstico.



## Depuración de telemetría

### Concentrador de diagnósticos

El concentrador de diagnósticos (en Visual Studio 2015 o posterior) muestra los datos de telemetría del servidor de Application Insights cuando se generan. Esto funciona incluso si solo ha optado por instalar el SDK, sin conectarlo a un recurso del Portal de Azure.

![Abra la ventana Herramientas de diagnóstico e inspeccione los eventos de Application Insights.](./media/app-insights-asp-net/31.png)


### Búsqueda de diagnóstico

La ventana de búsqueda muestra los eventos que se han registrado. (Si inició la sesión en Azure al configurar Application Insights, podrá buscar los mismos eventos en el portal.)

![Haga clic con el botón derecho en el proyecto y seleccione Application Insights, Búsqueda.](./media/app-insights-asp-net/34.png)

La búsqueda de texto sin formato funciona en todos los campos de los eventos. Por ejemplo, buscar parte de la dirección URL de una página; o el valor de una propiedad, como la ciudad del cliente; o palabras específicas en un registro de seguimiento.

También puede abrir la pestaña de elementos relacionados para ayudar a diagnosticar las solicitudes con errores o las excepciones.


![](./media/app-insights-asp-net/41.png)



### Excepciones

Si ha [configurado la supervisión de excepciones](app-insights-asp-net-exceptions.md), los informes de excepciones se mostrarán en la ventana de búsqueda.

Haga clic en una excepción para obtener un seguimiento de la pila. Si el código de la aplicación es abierto en Visual Studio, puede hacer clic para recorrer el seguimiento de la pila hasta dar con la línea correspondiente del código.




### Supervisión local



(Desde Visual Studio 2015 Update 2) Si no ha configurado el SDK para enviar datos de telemetría al portal de Application Insights (para que no haya ninguna clave de instrumentación en ApplicationInsights.config), la ventana diagnóstico mostrará los datos de telemetría de la sesión de depuración más reciente.

Esto es conveniente si ya ha publicado una versión anterior de la aplicación. No quiere que los datos de telemetría de las sesiones de depuración se mezclen con los datos de telemetría en el portal de Application Insights de la aplicación publicada.

También es útil si tiene algunos [datos de telemetría personalizados](app-insights-api-custom-events-metrics.md) que desea depurar antes de enviarlos al portal.


* *En primer lugar, configuré totalmente Application Insights para enviar datos de telemetría al portal. Pero ahora me gustaría ver los datos de telemetría solo en Visual Studio.*

 * En la configuración de la ventana de búsqueda, hay una opción para buscar diagnósticos locales, incluso si la aplicación envía datos de telemetría al portal.
 * Para detener el envío de datos de telemetría al portal, comente la línea `<instrumentationkey>...` de ApplicationInsights.config. Cuando esté listo para enviar de nuevo datos de telemetría al portal, quite los comentarios.



## <a name="monitor"></a> Visualización de los datos de telemetría en el portal de Application Insights

El portal de Application Insights es donde verá los datos de telemetría una vez que se publique la aplicación y durante la depuración querrá comprobar que envía los datos correctamente.

Abra el recurso de Application Insights en el [Portal de Azure][portal].

![Haga clic con el botón secundario en el proyecto y abra el Portal de Azure](./media/app-insights-asp-net/appinsights-04-openPortal.png)

Si no inició sesión en Azure cuando agregó Application Ingsights a esta aplicación, hágalo ahora. Seleccione **Configurar Application Insights**. Eso le permitirá seguir viendo la telemetría desde la aplicación activa después de haberla implementado. La telemetría aparecerá en el portal de Application Insights.

### Transmisión en directo

Para obtener una vista rápida de los datos de telemetría cuando esté depurando o justo después de una implementación, use Transmisión en directo.

![En la hoja de información general, haga clic en Transmisión en directo.](./media/app-insights-asp-net/45.png)


Transmisión en directo se ha diseñado para que pueda comprobar si la aplicación funciona correctamente después de una implementación.

Muestra solamente los datos de los últimos minutos y no conserva ninguno.

Requiere la versión 2.1.0-beta1 o posterior del SDK.



### Búsqueda: eventos individuales

Abra Búsqueda para investigar solicitudes individuales y sus eventos asociados.

![En la hoja de búsqueda, busque nombres de página u otras propiedades.](./media/app-insights-asp-net/21-search.png)

[Más información sobre la búsqueda](app-insights-diagnostic-search.md)

* *¿No hay eventos asociados?* Configure [excepciones de servidor](app-insights-asp-net-exceptions.md) y [dependencias](app-insights-asp-net-dependencies.md).


### Métricas: datos agregados

Busque datos agregados en los gráficos de Información general. Al principio, solo aparecerán uno o dos puntos. Por ejemplo:

![Haga clic en las distintas opciones para obtener más datos](./media/app-insights-asp-net/12-first-perf.png)

Haga clic en cualquier gráfico para ver métricas más detalladas. [Más información acerca de las métricas][perf]

* *¿No hay datos de usuario o página?* - [Agregar datos de usuario y página](app-insights-web-track-usage.md)

### Analytics: Lenguaje de consulta eficaz

A medida que acumula más datos, puede ejecutar consultas tanto para agregar datos como para buscar instancias individuales. [Analytics]() es una herramienta eficaz para comprender el rendimiento y el uso, así como para el diagnóstico.

![Ejemplo de Analytics](./media/app-insights-asp-net/025.png)


## ¿No hay datos?

* En Visual Studio, asegúrese de que la aplicación está enviando datos de telemetría. Debería ver los seguimientos en la ventana de resultados y en el concentrador de diagnósticos.
* Asegúrese de que está viendo lo correcto en Azure. Inicie sesión en el [portal de Azure](https://portal.azure.com), haga clic en "Examinar" >, "Application Insights" y, a continuación, seleccione la aplicación.
* Use la aplicación y abra varias páginas para generar telemetría.
* Abra la hoja [Buscar][diagnostic] para ver los eventos individuales. A veces, los eventos tardan un poco en llegar a través de la canalización de métricas.
* Espere unos segundos y haga clic en Actualizar.
* Vea [Solución de problemas][qna].



## Publicación de la aplicación

Implemente ahora la aplicación y observe cómo se acumulan los datos.

Use [Transmisión en directo](#live-stream) para supervisar los primeros minutos de una implementación o reimplementación para ver si la aplicación funciona correctamente. En particular, al reemplazar una versión anterior, desea saber si el rendimiento ha mejorado. Si hay un problema, quizá desee volver a la versión anterior.


#### ¿Tiene problemas el servidor de compilación?

Consulte [este apartado de la solución de problemas](app-insights-asp-net-troubleshoot-no-data.md#NuGetBuild).

> [AZURE.NOTE] Si la aplicación genera muchos datos de telemetría (y está usando la versión 2.0.0-beta3, o una posterior, del SDK de ASP.NET), el módulo de muestreo adaptable reducirá automáticamente el volumen que se envía al portal mediante el envío de solamente una fracción representativa de los eventos. Sin embargo, los eventos relacionados con la misma solicitud se seleccionarán o se anulará su selección como grupo, por lo que puede navegar entre ellos. [Más información sobre el muestreo](app-insights-sampling.md).



## Pasos siguientes

- [Datos de página y usuario](../article/application-insights/app-insights-javascript.md#selector1)
- [Excepciones](../article/application-insights/app-insights-asp-net-exceptions.md#selector1)
- [Dependencias](../article/application-insights/app-insights-asp-net-dependencies.md#selector1)
- [Disponibilidad](../article/application-insights/app-insights-monitor-web-app-availability.md#selector1)





## Para actualizar a futuras versiones del SDK

Para actualizar a una [nueva versión del SDK](app-insights-release-notes-dotnet.md), vuelva a abrir el Administrador de paquetes de NuGet y filtre los paquetes instalados. Seleccione Microsoft.ApplicationInsights.Web y elija Actualizar.

Si ha realizado personalizaciones en ApplicationInsights.config, guarde una copia del mismo antes de actualizar y después combine los cambios en la nueva versión.



## <a name="video"></a>Vídeo

> [AZURE.VIDEO getting-started-with-application-insights]


<!--Link references-->

[api]: app-insights-api-custom-events-metrics.md
[apikey]: app-insights-api-custom-events-metrics.md#ikey
[availability]: app-insights-monitor-web-app-availability.md
[azure]: ../insights-perf-analytics.md
[client]: app-insights-javascript.md
[detect]: app-insights-detect-triage-diagnose.md
[diagnostic]: app-insights-diagnostic-search.md
[knowUsers]: app-insights-overview-usage.md
[metrics]: app-insights-metrics-explorer.md
[netlogs]: app-insights-asp-net-trace-logs.md
[perf]: app-insights-web-monitor-performance.md
[portal]: http://portal.azure.com/
[qna]: app-insights-troubleshoot-faq.md
[redfield]: app-insights-monitor-performance-live-website-now.md
[roles]: app-insights-resources-roles-access-control.md
[start]: app-insights-overview.md

 

<!---HONumber=AcomDC_0525_2016-->