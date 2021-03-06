<properties
   pageTitle="Configuración de un host de Docker con VirtualBox | Microsoft Azure"
   description="Instrucciones paso a paso para configurar una instancia de Docker predeterminada con la máquina de Docker y VirtualBox"
   services="visual-studio-online"
   documentationCenter="na"
   authors="TomArcher"
   manager="douge"
   editor="" />
<tags
   ms.service="multiple"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="multiple"
   ms.date="03/27/2016"
   ms.author="tarcher" />

# Configuración de un host de Docker con VirtualBox

## Información general
Este artículo le guiará por el proceso de configuración de una instancia de Docker predeterminada con la máquina de Docker y VirtualBox. Si utiliza [Docker para Windows beta](http://beta.docker.com/), esta configuración no es necesaria.

## Requisitos previos
Es necesario instalar las siguientes herramientas.

- [Docker Toolbox](https://www.docker.com/products/overview#/docker_toolbox)

## Configuración del cliente de Docker con Windows PowerShell

Para configurar a un cliente de Docker, simplemente abra Windows PowerShell y realice los pasos siguientes:

1. Cree una instancia de host de docker predeterminada.

    ```PowerShell
    docker-machine create --driver virtualbox default
    ```
 
1. Compruebe que la instancia predeterminada está configurada y en ejecución. (Verá que se ejecuta una instancia denominada 'predeterminada'.

		docker-machine ls 
		
	![][0]
 
1. Establezca el host actual como predeterminado y configure el shell.

        docker-machine env default | Invoke-Expression

1. Vea los contenedores de Docker activos. La lista debe estar vacía.

		docker ps

	![][1]
 
> [AZURE.NOTE] Cada vez que reinicie el equipo de desarrollo, deberá reiniciar el host de Docker local. Para ello, emita el siguiente comando en un símbolo del sistema: `docker-machine start default`.

[0]: ./media/vs-azure-tools-docker-setup/docker-machine-ls.png
[1]: ./media/vs-azure-tools-docker-setup/docker-ps.png

<!---HONumber=AcomDC_0330_2016-->