<properties
	pageTitle="Provisión de acceso remoto seguro a aplicaciones locales"
	description="Explica cómo utilizar el proxy de la aplicación de Azure AD para proporcionar acceso remoto seguro a sus aplicaciones locales."
	services="active-directory"
	documentationCenter=""
	authors="kgremban"
	manager="stevenpo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/12/2016"
	ms.author="kgremban"/>

# Provisión de acceso remoto seguro a aplicaciones locales

> [AZURE.NOTE] Proxy de aplicación es una característica que solo está disponible si actualizó a la edición Premium o Basic de Azure Active Directory. Para obtener más información, consulte [Ediciones de Azure Active Directory](active-directory-editions.md).

Quiere ofrecer acceso a los usuarios remotos que tienen todo tipo de dispositivos: administrados y no administrados; tabletas, smartphones y equipos portátiles. Pero ofrecer un acceso seguro a un sinfín de recursos puede ser una tarea compleja. En los últimos años, los proxy inversos se convirtieron en un medio popular de proporcionar acceso remoto seguro, pero era necesario que estos se encontrasen detrás de firewalls que eran difíciles de proteger y cuya alta disposición era compleja.

## Acceso remoto seguro en la nube
En un entorno de nube moderno, llevamos el acceso remoto al siguiente nivel mediante el proxy de aplicación en Azure Active Directory (AD). El proxy de la aplicación es una característica de Azure AD que se proporciona como un servicio, lo que significa que es fácil de implementar y usar. Se integra con Azure AD, la misma plataforma de identidad utilizada por Office 365.

## ¿Qué es el proxy de la aplicación de Azure Active Directory?
El proxy de la aplicación proporciona un inicio de sesión único y acceso remoto seguro para las aplicaciones web hospedadas de forma local, como los sitios de SharePoint y Outlook Web Access. Ahora es posible tener acceso a las aplicaciones web locales del mismo modo que a las aplicaciones SaaS en Azure Active Directory, sin necesidad de una red privada virtual o de cambios en la infraestructura de red. El Proxy de aplicaciones le permite publicar aplicaciones y que los empleados puedan iniciar sesión en sus aplicaciones desde casa o desde sus propios dispositivos, y autenticarse a través de este proxy basado en la nube.

## ¿Cómo funciona?

El proxy de aplicación funciona con tres pasos básicos. Primero, los conectores se implementan en la red local. Después, el conector se conecta al servicio en la nube. Finalmente, el conector y el servicio en la nube enrutan el tráfico de usuario a las aplicaciones.

 ![Diagrama del proxy de aplicación de Azure AD](./media/active-directory-appssoaccess-whatis/azureappproxxy.png)

1. El usuario obtiene acceso a la aplicación a través del proxy de aplicación y se le dirige a la página de inicio de sesión de Azure AD para que se autentique.
2. Después de iniciar sesión correctamente, se genera un token y se envía al usuario.
3. El usuario envía el token al proxy de aplicación, que recupera el nombre principal de usuario (UPN) y el nombre de entidad de seguridad (SPN) del token y dirige la solicitud al conector.
4. En nombre del usuario, el conector solicita un vale Kerberos que se puede usar para la autenticación interna (Windows). Esto se conoce como delegación limitada de Kerberos.
5. Se recupera un vale Kerberos de Active Directory.
6. El vale se envía al servidor de aplicaciones y se comprueba.
7. La respuesta se envía a través del proxy de aplicación al usuario.

### Habilitación del acceso
El proxy de aplicación funciona mediante la instalación de un pequeño servicio de Windows Server denominado conector dentro de la red. El conector no requiere abrir puertos de entrada y no tiene que poner nada en la red perimetral. Si tiene mucho tráfico a las aplicaciones, puede agregar más conectores, y el servicio se encargará de mantener el equilibrio de carga. Los conectores no tienen estado y extraen todo el contenido de la nube según sea necesario.

Cuando un usuario tiene acceso a las aplicaciones de forma remota, desde cualquier dispositivo, se autentica por parte de Azure Active Directory y obtiene acceso a la aplicación.

### Inicio de sesión único
El proxy de la aplicación de Azure AD proporciona el inicio de sesión único para las aplicaciones que usa la autenticación integrada de Windows (IWA) o aplicaciones para notificaciones. Si la aplicación utiliza IWA, el proxy de aplicación suplanta al usuario mediante la delegación limitada de Kerberos para proporcionar el inicio de sesión único (SSO). Si tiene una aplicación para notificaciones que confía en Azure Active Directory, el inicio de sesión único se logra porque el usuario ya se ha autenticado con Azure AD.

## Primeros pasos
Asegúrese de que tiene una suscripción de nivel Básico o Premium de Azure AD y un directorio de Azure AD del cual es usted administrador global. También necesita licencias de nivel Básico o Premium de Azure AD para el administrador de directorios y los usuarios que tienen acceso a las aplicaciones. Consulte [Ediciones de Azure Active Directory](active-directory-editions.md) para más información.

### Introducción a la habilitación del acceso remoto a aplicaciones locales
La configuración del proxy de la aplicación se realiza en dos pasos:

1. [Habilitar el proxy de la aplicación y configurar el conector](active-directory-application-proxy-enable.md)  
2. [Publicar aplicaciones](active-directory-application-proxy-publish.md). Use el asistente rápido y sencillo para que sus aplicaciones locales se publiquen y sean accesibles de manera remota.

## Pasos siguientes
Hay mucho más que puede hacer con el proxy de la aplicación:

- [Publicar aplicaciones mediante su propio nombre de dominio](active-directory-application-proxy-custom-domains.md)
- [Habilitar el inicio de sesión único](active-directory-application-proxy-sso-using-kcd.md)
- [Trabajar con las aplicaciones para notificaciones](active-directory-application-proxy-claims-aware-apps.md)
- [Habilitar el acceso condicional](active-directory-application-proxy-conditional-access.md)


### Obtenga más información acerca del proxy de la aplicación
- [Eche un vistazo para ver nuestra ayuda en línea](active-directory-application-proxy-enable.md)
- [Consulte el blog del proxy de la aplicación](http://blogs.technet.com/b/applicationproxyblog/)
- [Vea nuestros vídeos de Channel 9](http://channel9.msdn.com/events/Ignite/2015/BRK3864)

## Recursos adicionales
- [Índice de artículos sobre la administración de aplicaciones en Azure Active Directory](active-directory-apps-index.md)
- [Registro en Azure como una organización](sign-up-organization.md)
- [Identidad de Azure](fundamentals-identity.md)

<!---HONumber=AcomDC_0518_2016-->