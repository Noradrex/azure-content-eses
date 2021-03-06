<properties
	pageTitle="Azure AD Connect Sync: referencia de funciones | Microsoft Azure"
	description="Referencia de expresiones declarativas de aprovisionamiento en la sincronización de Azure AD Connect"
	services="active-directory"
	documentationCenter=""
	authors="andkjell"
	manager="StevenPo"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/07/2016"
	ms.author="andkjell;markusvi"/>


# Azure AD Connect Sync: referencia de funciones


En Sincronización de Azure Active Directory, las funciones se usan para manipular un valor de atributo durante la sincronización. La sintaxis de las funciones se expresa con el siguiente formato: `<output type> FunctionName(<input type> <position name>, ..)`

Si una función está sobrecargada y acepta varias sintaxis, se enumeran todas las sintaxis válidas. Las funciones están fuertemente tipadas y comprueban que el tipo pasado coincide con el tipo documentado. Se produce un error si el tipo no coincide.

Los tipos se expresan con la siguiente sintaxis:

- **bin**: binario
- **bool**: booleano
- **dt**: fecha y hora UTC
- **enum**: enumeración de constantes conocidas
- **exp**: expresión, que se espera que evalúe en un booleano
- **mvbin**: binario de varios valores
- **mvstr**: referencia de varios valores
- **num**: numérico
- **ref**: referencia de valor único
- **str**: cadena de valor único
- **var**: una variante de (casi) cualquier otro tipo
- **void**: no devuelve un valor



## Referencia de funciones

----------
**Conversión:**

[CBool](#cbool) &nbsp;&nbsp;&nbsp;&nbsp; [CDate](#cdate) &nbsp;&nbsp;&nbsp;&nbsp; [CGuid](#cguid) &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromBase64](#convertfrombase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToBase64](#converttobase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertFromUTF8Hex](#convertfromutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToUTF8Hex](#converttoutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [CNum](#cnum) &nbsp;&nbsp;&nbsp;&nbsp; [CRef](#cref) &nbsp;&nbsp;&nbsp;&nbsp; [CStr](#cstr) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromGuid](#StringFromGuid) &nbsp;&nbsp;&nbsp;&nbsp; [StringFromSid](#stringfromsid)

**Fecha y hora:**

[DateAdd](#dateadd) &nbsp;&nbsp;&nbsp;&nbsp; [DateFromNum](#datefromnum) &nbsp;&nbsp;&nbsp;&nbsp; [FormatDateTime](#formatdatetime) &nbsp;&nbsp;&nbsp;&nbsp; [Now](#now) &nbsp;&nbsp;&nbsp;&nbsp; [NumFromDate](#numfromdate)

**Directorio**

[DNComponent](#dncomponent) &nbsp;&nbsp;&nbsp;&nbsp; [DNComponentRev](#dncomponentrev) &nbsp;&nbsp;&nbsp;&nbsp; [EscapeDNComponent](#escapedncomponent)

**Evaluación:**

[IsBitSet](#isbitset) &nbsp;&nbsp;&nbsp;&nbsp; [IsDate](#isdate) &nbsp;&nbsp;&nbsp;&nbsp; [IsEmpty](#isempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsGuid](#isguid) &nbsp;&nbsp;&nbsp;&nbsp; [IsNull](#isnull) &nbsp;&nbsp;&nbsp;&nbsp; [IsNullOrEmpty](#isnullorempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsNumeric](#isnumeric) &nbsp;&nbsp;&nbsp;&nbsp; [IsPresent](#ispresent) &nbsp;&nbsp;&nbsp;&nbsp; [IsString](#isstring)

**Matemático:**

[BitAnd](#bitand) &nbsp;&nbsp;&nbsp;&nbsp; [BitOr](#bitor) &nbsp;&nbsp;&nbsp;&nbsp; [RandomNum](#randomnum)

**Con varios valores**

[Contains](#contains) &nbsp;&nbsp;&nbsp;&nbsp; [Count](#count) &nbsp;&nbsp;&nbsp;&nbsp; [Item](#item) &nbsp;&nbsp;&nbsp;&nbsp; [ItemOrNull](#itemornull) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [RemoveDuplicates](#removeduplicates) &nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)

**Flujo del programa:**

[Error](#error) &nbsp;&nbsp;&nbsp;&nbsp; [IIF](#iif) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)


**Texto**

[GUID](#guid) &nbsp;&nbsp;&nbsp;&nbsp; [InStr](#instr) &nbsp;&nbsp;&nbsp;&nbsp; [InStrRev](#instrrev) &nbsp;&nbsp;&nbsp;&nbsp; [LCase](#lcase) &nbsp;&nbsp;&nbsp;&nbsp; [Left](#left) &nbsp;&nbsp;&nbsp;&nbsp; [Len](#len) &nbsp;&nbsp;&nbsp;&nbsp; [LTrim](#ltrim) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp;&nbsp;&nbsp; [PadLeft](#padleft) &nbsp;&nbsp;&nbsp;&nbsp; [PadRight](#padright) &nbsp;&nbsp;&nbsp;&nbsp; [PCase](#pcase) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [ReplaceChars](#replacechars) &nbsp;&nbsp;&nbsp;&nbsp; [Right](#right) &nbsp;&nbsp;&nbsp;&nbsp; [RTrim](rtrim) &nbsp;&nbsp;&nbsp;&nbsp; [Trim](#trim) &nbsp;&nbsp;&nbsp;&nbsp; [UCase](#ucase) &nbsp;&nbsp;&nbsp;&nbsp; [Word](#word)

----------
### BitAnd

**Descripción:** la función BitAnd establece los bits especificados en un valor.

**Sintaxis:** `num BitAnd(num value1, num value2)`

- value1, value2: valores numéricos que deberían estar unidos por el operador AND

**Comentarios:** esta función convierte ambos parámetros en la representación binaria y establece un bit en:

- 0: si uno o ambos de los bits correspondientes en *máscara* y *marca* son 0
- 1: si ambos de los bits correspondientes son 1.

En otras palabras, devuelve 0 en todos los casos excepto cuando los bits correspondientes de los dos parámetros son 1.

**Ejemplo:**`BitAnd(&HF, &HF7)` devuelve 7 porque los valores hexadecimales "F" y "F7" se evalúan en este valor.

----------
### BitOr

**Descripción:** la función BitOr establece los bits especificados en un valor.

**Sintaxis:** `num BitOr(num value1, num value2)`

- value1, value2: valores numéricos que deberían estar unidos por el operador OR

**Comentarios:** esta función convierte ambos parámetros en la representación binaria y establece un bit en 1 si uno o ambos de los bits correspondientes en la máscara y marca son 1 y en 0 si los bits correspondientes son 0. En otras palabras, devuelve 1 en todos los casos excepto cuando los bits correspondientes de ambos parámetros son 0.

----------
### CBool

**Descripción:** la función CBool devuelve un valor booleano basado en la expresión evaluada.

**Sintaxis:** `bool CBool(exp Expression)`

**Comentarios:** si la expresión se evalúa en un valor distinto de cero, CBool devuelve True. En caso contrario, devuelve False.

**Ejemplo:** `CBool([attrib1] = [attrib2])`

Devuelve True si ambos atributos tienen el mismo valor.

----------
### CDate

**Descripción:** la función CDate devuelve un valor DateTime UTC a partir de una cadena. DateTime no es un tipo de atributo nativo en Sincronización pero lo usan algunas funciones.

**Sintaxis:** `dt CDate(str value)`

- Valor: una cadena con una fecha, hora y opcionalmente, una zona horaria

**Comentarios:** la cadena devuelta siempre es UTC.

**Ejemplo:**`CDate([employeeStartTime])` devuelve un valor DateTime basado en la hora de inicio del empleado.

`CDate("2013-01-10 4:00 PM -8")` Devuelve un valor DateTime que representa "2013-01-11 12:00 AM".

----------
### CGuid

**Descripción:** la función CGuid convierte la representación de cadena de un GUID en su representación binaria.

**Sintaxis:** `bin CGuid(str GUID)`

- Una cadena con un formato que sigue este patrón: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx o {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

----------
### Contains

**Descripción:** la función Contains busca una cadena dentro de un atributo de varios valores

**Sintaxis:** `num Contains (mvstring attribute, str search)` `num Contains (mvstring attribute, str search, enum Casetype)` - distingue mayúsculas y minúsculas`num Contains (mvref attribute, str search)` - distingue mayúsculas y minúsculas

- atributo: el atributo con varios valores de búsqueda.<br>
- búsqueda: cadena para buscar en el atributo.<br>
- Casetype: CaseInsensitive o CaseSensitive.<br>

Devuelve el índice en el atributo de varios valores donde se ha encontrado la cadena. Si no se encuentra la cadena, se devuelve 0.

**Comentarios:** para los atributos de cadena de varios valores, la búsqueda encontrará subcadenas en los valores. Para los atributos de referencia, la cadena de búsqueda debe coincidir exactamente con el valor para que se considere una coincidencia.

**Ejemplo:** `IIF(Contains([proxyAddresses],"SMTP:")>0,[proxyAddresses],Error("No primary SMTP address found."))` si el atributo proxyAddresses tiene una dirección de correo electrónico principal (indicada por mayúsculas "SMTP:"), devuelve el atributo proxyAddress. De lo contrario, devuelve un error.

----------
### ConvertFromBase64

**Descripción:** la función ConvertFromBase64 convierte el valor codificado en base64 especificado en una cadena normal.

**Sintaxis:** `str ConvertFromBase64(str source)` da por hecho que se usará Unicode para la codificación de <br>`str ConvertFromBase64(str source, enum Encoding)`

- source: cadena codificada en Base64  
- Codificación: Unicode, ASCII, UTF8

**Ejemplo** `ConvertFromBase64("SABlAGwAbABvACAAdwBvAHIAbABkACEA")` `ConvertFromBase64("SGVsbG8gd29ybGQh", UTF8)`

Ambos ejemplos devuelven "*Hola a todos*"

----------
### ConvertFromUTF8Hex

**Descripción:** la función ConvertFromUTF8Hex convierte el valor codificado hexadecimal UTF8 especificado en una cadena.

**Sintaxis:** `str ConvertFromUTF8Hex(str source)`

- origen: cadena codificada de 2 bytes UTF8

**Comentarios:** la diferencia entre esta función y ConvertFromBase64(,UTF8) es que el resultado es descriptivo para el atributo DN. Azure Active Directory utiliza este formato como DN.

**Ejemplo:** `ConvertFromUTF8Hex("48656C6C6F20776F726C6421")` devuelve "*Hola a todos*"

----------
### ConvertToBase64

**Descripción:** la función ConvertToBase64 convierte una cadena en una cadena en base64 de Unicode. Convierte el valor de una matriz de enteros en su representación de cadena equivalente que se codifica con dígitos de base 64.

**Sintaxis:** `str ConvertToBase64(str source)`

**Ejemplo:** `ConvertToBase64("Hello world!")` devuelve "SABlAGwAbABvACAAdwBvAHIAbABkACEA"

----------
### ConvertToUTF8Hex

**Descripción:** la función ConvertToUTF8Hex convierte una cadena en un valor codificado hexadecimal UTF8.

**Sintaxis:** `str ConvertToUTF8Hex(str source)`

**Comentarios:** Azure Active Directory usa el formato de salida de esta función como formato de atributo DN.

**Ejemplo:** `ConvertToUTF8Hex("Hello world!")` devuelve 48656C6C6F20776F726C6421

----------
### Recuento

**Descripción:** la función Count devuelve el número de elementos en un atributo de varios valores

**Sintaxis:** `num Count(mvstr attribute)`

----------
### CNum

**Descripción:** la función CNum toma una cadena y devuelve un tipo de datos numérico.

**Sintaxis:** `num CNum(str value)`

----------
### CRef

**Descripción:** convierte una cadena en un atributo de referencia

**Sintaxis:** `ref CRef(str value)`

**Ejemplo:** `CRef("CN=LC Services,CN=Microsoft,CN=lcspool01,CN=Pools,CN=RTC Service," & %Forest.LDAP%)`

----------
### CStr

**Descripción:** la función CStr se convierte en un tipo de datos de cadena.

**Sintaxis:** `str CStr(num value)` `str CStr(ref value)` `str CStr(bool value)`

- value: puede ser un valor numérico, un atributo de referencia o un valor booleano.

**Ejemplo:** `CStr([dn])` podría devolver “cn=Joe,dc=contoso,dc=com”

----------
### DateAdd

**Descripción:** devuelve un valor Date que contiene una fecha a la que se ha agregado un intervalo de tiempo especificado.

**Sintaxis:** `dt DateAdd(str interval, num value, dt date)`

- interval: expresión de cadena que es el intervalo de tiempo que desea agregar. La cadena debe tener uno de los siguientes valores:
 - yyyy Año
 - q Trimestre
 - m Mes
 - y Día del año
 - d Día
 - w Día de la semana
 - ww Semana
 - h Hora
 - n Minuto
 - s Segundo
- value: el número de unidades que desea agregar. Puede ser positivo (para obtener fechas futuras) o negativo (para obtener fechas del pasado).
- date: DateTime que representa la fecha a la que se agrega el intervalo.

**Ejemplo:** `DateAdd("m", 3, CDate("2001-01-01"))` agrega 3 meses y devuelve un valor DateTime que representa "2001-04-01".

----------
### DateFromNum

**Descripción:** la función DateFromNum convierte un valor con formato de fecha de AD en un tipo DateTime.

**Sintaxis:** `dt DateFromNum(num value)`

**Ejemplo:** `DateFromNum([lastLogonTimestamp])` `DateFromNum(129699324000000000)` devuelve un valor DateTime que representa 2012-01-01 23:00:00

----------
### DNComponent

**Descripción:** la función DNComponent devuelve el valor de un componente DN especificado empezando por la izquierda.

**Sintaxis:** `str DNComponent(ref dn, num ComponentNumber)`

- dn: el atributo de referencia que interpretar
- ComponentNumber: el componente en DN que devolver

**Ejemplo:** `DNComponent([dn],1)` si dn es “cn=Joe,ou=…”, devuelve Joe

----------
### DNComponentRev

**Descripción:** la función DNComponentRev devuelve el valor de un componente DN especificado empezando por la derecha (final).

**Sintaxis:** `str DNComponentRev(ref dn, num ComponentNumber)` `str DNComponentRev(ref dn, num ComponentNumber, enum Options)`

- dn: el atributo de referencia que interpretar
- ComponentNumber: el componente en DN que devolver
- Opciones: DC, ignora todos los componentes con “dc=”

**Ejemplo:** si dn es "cn=Joe,ou=Atlanta,ou=GA,ou=US, dc=contoso,dc=com", entonces `DNComponentRev([dn],3)` `DNComponentRev([dn],1,"DC")` Ambos devuelven US.

----------
### Error

**Descripción:** la función Error se usa para devolver un error personalizado.

**Sintaxis:** `void Error(str ErrorMessage)`

**Ejemplo:**`IIF(IsPresent([accountName]),[accountName],Error("AccountName is required"))` si el atributo accountName no está presente, se produce un error en el objeto.

----------
### EscapeDNComponent

**Descripción:** la función EscapeDNComponent toma un componente de DN y genera secuencias de escape para poder representarlo en LDAP.

**Sintaxis:** `str EscapeDNComponent(str value)`

**Ejemplo:** `EscapeDNComponent("cn=" & [displayName]) & "," & %ForestLDAP%)` se asegura de que el objeto se puede crear en un directorio LDAP incluso si el atributo displayName tiene caracteres para los que deben generarse secuencias de escape en LDAP.

----------
### FormatDateTime

**Descripción:** la función FormatDateTime se usa para dar formato a un valor DateTime en una cadena con un formato especificado

**Sintaxis:** `str FormatDateTime(dt value, str format)`

- value: un valor con el formato DateTime
- format: una cadena que representa el formato de conversión.

**Comentarios:** puede encontrar aquí los valores posibles para el formato: [Formatos de fecha y hora definidos por el usuario (función Format)](http://msdn2.microsoft.com/library/73ctwf33(VS.90).aspx)

**Ejemplo:**

`FormatDateTime(CDate("12/25/2007"),"yyyy-mm-dd")` Genera “2007-12-25”.

`FormatDateTime(DateFromNum([pwdLastSet]),"yyyyMMddHHmmss.0Z")` Puede generar “20140905081453.0Z”

----------
### GUID

**Descripción:** la función GUID genera un nuevo GUID aleatorio

**Sintaxis:** `str GUID()`

----------
### IIF

**Descripción:** la función IIF devuelve uno de un conjunto de valores posibles según una condición especificada.

**Sintaxis:** `var IIF(exp condition, var valueIfTrue, var valueIfFalse)`

- condition: cualquier valor o expresión que pueda evaluarse en True o False.
- valueIfTrue: un valor que se devolverá si se evalúa una condición en True.
- valueIfFalse: un valor que se devolverá si se evalúa una condición en False.

**Ejemplo:** `IIF([employeeType]="Intern","t-" & [alias],[alias])` devuelve el alias de un usuario al que agrega "t-" al principio del mismo si el usuario está en prácticas. De lo contrario, el alias del usuario se queda como está.

----------
### InStr

**Descripción:** la función InStr busca la primera ocurrencia de una subcadena en una cadena

**Sintaxis:**

`num InStr(str stringcheck, str stringmatch)` `num InStr(str stringcheck, str stringmatch, num start)` `num InStr(str stringcheck, str stringmatch, num start , enum compare)`

- stringcheck: cadena que se va a buscar
- stringmatch: cadena que se tiene que encontrar
- start: posición de inicio para encontrar la subcadena
- compare: vbTextCompare o vbBinaryCompare

**Comentarios:** devuelve la posición en la que se encontró la subcadena o 0 si no se encuentra.

**Ejemplo:** `InStr("The quick brown fox","quick")` se evalúa en 5

`InStr("repEated","e",3,vbBinaryCompare)` se evalúa en 7

----------
### InStrRev

**Descripción:** la función InStrRev busca la última ocurrencia de una subcadena en una cadena

**Sintaxis:** `num InstrRev(str stringcheck, str stringmatch)` `num InstrRev(str stringcheck, str stringmatch, num start)` `num InstrRev(str stringcheck, str stringmatch, num start, enum compare)`

- stringcheck: cadena que se va a buscar
- stringmatch: cadena que se tiene que encontrar
- start: posición de inicio para encontrar la subcadena
- compare: vbTextCompare o vbBinaryCompare

**Comentarios:** devuelve la posición en la que se encontró la subcadena o 0 si no se encuentra.

**Ejemplo:** `InStrRev("abbcdbbbef","bb")` devuelve 7.

----------
### IsBitSet

**Descripción:** la función IsBitSet prueba si un bit se establece o no

**Sintaxis:** `bool IsBitSet(num value, num flag)`

- value: un valor numérico que es evaluated.flag: un valor numérico que tiene el bit que se va a evaluar

**Ejemplo:** `IsBitSet(&HF,4)` devuelve True porque bit “4” se establece en el valor hexadecimal “F”

----------
### IsDate

**Descripción:** la función IsDate se evalúa en True si la expresión se puede evaluar como un tipo DateTime.

**Sintaxis:** `bool IsDate(var Expression)`

**Comentarios:** se usa para determinar si CDate() será correcto.

----------
### IsEmpty

**Descripción:** la función IsEmpty se evalúa en True si el atributo está presente en CS o MV pero se evalúa en una cadena vacía.

**Sintaxis:** `bool IsEmpty(var Expression)`

----------
### IsGuid

**Descripción:** la función IsGuid evaluada en True si la cadena se pudo convertir en un GUID.

**Sintaxis:** `bool IsGuid(str GUID)`

**Comentarios:** un GUID se define como una cadena que sigue uno de estos patrones: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx or {xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}

Se usa para determinar si CGuid() es correcto.

**Ejemplo:** `IIF(IsGuid([strAttribute]),CGuid([strAttribute]),NULL)` si StrAttribute tiene un formato GUID, devuelve una representación binaria. De lo contrario, devuelve un valor Null.

----------
### IsNull

**Descripción:** la función IsNull devuelve True si la expresión se evalúa como Null.

**Sintaxis:** `bool IsNull(var Expression)`

**Comentarios:** para un atributo, un valor Null se expresa mediante la ausencia del atributo.

**Ejemplo:** `IsNull([displayName])` devuelve True si el atributo no está presente en CS o MV.

----------
### IsNullOrEmpty

**Descripción:** la función IsNullOrEmpty devuelve True si la expresión es Null o una cadena vacía.

**Sintaxis:** `bool IsNullOrEmpty(var Expression)`

**Comentarios:** para un atributo, esto se evaluaría como True si el atributo no está presente o está presente pero es una cadena vacía.<br> La función contraria a esta es IsPresent.

**Ejemplo:**`IsNullOrEmpty([displayName])` devuelve True si el atributo no está presente o si es una cadena vacía en CS o MV.

----------
### IsNumeric

**Descripción:** la función IsNumeric devuelve un valor booleano que indica si una expresión se puede evaluar como un tipo de número.

**Sintaxis:** `bool IsNumeric(var Expression)`

**Comentarios:** se usa para determinar si CNum() analizará correctamente la expresión.

----------
### IsString

**Descripción:** la función IsString se evalúa en True si la expresión se puede evaluar en un tipo de cadena.

**Sintaxis:** `bool IsString(var expression)`

**Comentarios:** se usa para determinar si CStr() analizará correctamente la expresión.

----------
### IsPresent

**Descripción:** la función IsPresent devuelve True si la expresión se evalúa como una cadena que no es Null y no está vacía.

**Sintaxis:** `bool IsPresent(var expression)`

**Comentarios:** la función contraria a esta es IsNullOrEmpty.

**Ejemplo:** `Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

----------
### Elemento

**Descripción:** la función Item devuelve un elemento de un atributo o una cadena de varios valores.

**Sintaxis:** `var Item(mvstr attribute, num index)`

- attribute: atributo de varios valores
- index: índice para un elemento en la cadena de varios valores.

**Comentarios:** la función Item es útil con la función Contains puesto que esta última función devolverá el índice a un elemento en el atributo de varios valores.

Se produce un error si el índice está fuera de los límites.

**Ejemplo:** `Mid(Item([proxyAddress],Contains([proxyAddress], "SMTP:")),6)` devuelve la dirección de correo electrónico principal.

----------
### ItemOrNull

**Descripción:** la función ItemOrNull devuelve un elemento de un atributo o una cadena de varios valores.

**Sintaxis:** `var ItemOrNull(mvstr attribute, num index)`

- attribute: atributo de varios valores
- index: índice para un elemento en la cadena de varios valores.

**Comentarios:** la función ItemOrNull es útil con la función Contains puesto que esta última función devolverá el índice a un elemento en el atributo de varios valores.

Devuelve un valor Null si el índice está fuera de los límites.

----------
### Join

**Descripción:** la función Join toma una cadena de varios valores y devuelve una cadena de valor único con un separador especificado insertado entre cada elemento.

**Sintaxis:** `str Join(mvstr attribute)` `str Join(mvstr attribute, str Delimiter)`

- attribute: atributo de varios valores que contiene cadenas para combinar.
- delimiter: cualquier cadena utilizada para separar las subcadenas en la cadena devuelta. Si se omite, se usa el carácter de espacio (""). Si delimitador es una cadena de longitud cero ("") o nada, todos los elementos de la lista se concatenan sin delimitadores.

**Comentarios:** hay paridad entre las funciones Join y Split. La función Join toma una matriz de cadenas y las combina con una cadena de delimitación para devolver una sola cadena. La función Split toma una cadena y la separa en el delimitador para devolver una matriz de cadenas. Sin embargo, una diferencia clave es que Join puede concatenar cadenas con cualquier cadena de delimitación, mientras que attribute puede separar solo cadenas mediante un delimitador de carácter único.

**Ejemplo:** `Join([proxyAddresses],",")` podría devolver: “SMTP:john.doe@contoso.com,smtp:jd@contoso.com”

----------
### LCase

**Descripción:** la función LCase convierte todos los caracteres de una cadena a minúsculas.

**Sintaxis:** `str LCase(str value)`

**Ejemplo:** `LCase("TeSt")` devuelve “test”.

----------
### Left

**Descripción:** la función Left devuelve un número especificado de caracteres desde la izquierda de una cadena.

**Sintaxis:** `str Left(str string, num NumChars)`

- string: la cadena desde la que devolver los caracteres
- NumChars: un número que identifica el número de caracteres que devolver desde el principio (izquierdo) de la cadena

**Comentarios:** una cadena que contiene los primeros caracteres numChars de la cadena:

- Con numChars = 0, se devuelve una cadena vacía.
- Con numChars < 0, se devuelve una cadena de entrada.
- Si la cadena es null, se devuelve una cadena vacía.

Si la cadena contiene menos caracteres que el número especificado en numChars, se devuelve una cadena idéntica a la cadena (es decir, que contiene todos los caracteres en el parámetro 1).

**Ejemplo:** `Left("John Doe", 3)` devuelve “Joh”.

----------
### Len

**Descripción:** la función Len devuelve el número de caracteres en una cadena.

**Sintaxis:** `num Len(str value)`

**Ejemplo:** `Len("John Doe")` devuelve 8.

----------
### LTrim

**Descripción:** la función LTrim quita los espacios en blanco del principio de una cadena.

**Sintaxis:** `str LTrim(str value)`

**Ejemplo:** `LTrim(" Test ")` devuelve “Test”.

----------
### Mid

**Descripción:** la función Mid devuelve un número especificado de caracteres desde una posición especificada en una cadena.

**Sintaxis:** `str Mid(str string, num start, num NumChars)`

- string: la cadena desde la que devolver los caracteres
- start: un número que identifica la posición de inicio en una cadena desde la que devolver los caracteres
- NumChars: un número que identifica el número de caracteres que devolver desde la posición en una cadena

**Comentarios:** devuelve caracteres numChars que comienzan por la posición de inicio en una cadena. Una cadena que contiene caracteres numChars de la posición de inicio en una cadena:

- Con numChars = 0, se devuelve una cadena vacía.
- Con numChars < 0, se devuelve una cadena de entrada.
- Con start >, la longitud de la cadena devuelve la cadena de entrada.
- Con start <= 0, se devuelve la cadena de entrada.
- Si la cadena es null, se devuelve una cadena vacía.

Si no hay caracteres numChar restantes en la cadena de la posición de inicio, se devuelve el máximo de caracteres que puede devolverse.

**Ejemplo:** `Mid("John Doe", 3, 5)` devuelve “hn Do”.

`Mid("John Doe", 6, 999)` devuelve “Doe”

----------
### Now

**Descripción:** la función Now devuelve DateTime que especifica la fecha y hora actuales, según la fecha y hora del sistema del equipo.

**Sintaxis:** `dt Now()`

----------
### NumFromDate

**Descripción:** la función NumFromDate devuelve una fecha en formato de fecha de AD.

**Sintaxis:** `num NumFromDate(dt value)`

**Ejemplo:** `NumFromDate(CDate("2012-01-01 23:00:00"))` devuelve 129699324000000000.

----------
### PadLeft

**Descripción:** la función PadLeft rellena en la parte izquierda una cadena con una longitud especificada mediante un carácter controlador proporcionado.

**Sintaxis:** `str PadLeft(str string, num length, str padCharacter)`

- string: la cadena que rellenar.
- length: un número entero que representa la longitud deseada de la cadena.
- padCharacter: una cadena que consta de un solo carácter que se usará como el carácter controlador

**Comentarios:**

- Si la longitud de cadena es inferior a la longitud, padCharacter se anexa repetidamente al principio (izquierda) de la cadena hasta que tenga una longitud igual a la longitud.
- PadCharacter puede ser un carácter de espacio, pero no puede ser un valor Null.
- Si la longitud de cadena es igual o mayor que la longitud, la cadena se devuelve sin cambios.
- Si la cadena tiene una longitud mayor que o igual que la longitud, se devuelve una cadena idéntica a la cadena.
- Si la longitud de cadena es menor que la longitud, se devuelve una cadena nueva de la longitud deseada que contiene una cadena rellenada con un padCharacter.
- Si la cadena es Null, la función devuelve una cadena vacía.

**Ejemplo:** `PadLeft("User", 10, "0")` devuelve “000000User”.

----------
### PadRight

**Descripción:** la función PadRight rellena en la parte derecha una cadena con una longitud especificada mediante un carácter controlador proporcionado.

**Sintaxis:** `str PadRight(str string, num length, str padCharacter)`

- string: la cadena que rellenar.
- length: un número entero que representa la longitud deseada de la cadena.
- padCharacter: una cadena que consta de un solo carácter que se usará como el carácter controlador

**Comentarios:**

- Si la longitud de cadena es inferior a la longitud, padCharacter se anexa repetidamente al final (derecha) de la cadena hasta que tenga una longitud igual a la longitud.
- padCharacter puede ser un carácter de espacio, pero no puede ser un valor Null.
- Si la longitud de cadena es igual o mayor que la longitud, la cadena se devuelve sin cambios.
- Si la cadena tiene una longitud mayor que o igual que la longitud, se devuelve una cadena idéntica a la cadena.
- Si la longitud de cadena es menor que la longitud, se devuelve una cadena nueva de la longitud deseada que contiene una cadena rellenada con un padCharacter.
- Si la cadena es Null, la función devuelve una cadena vacía.

**Ejemplo:** `PadRight("User", 10, "0")` devuelve “User000000”.

----------
### PCase

**Descripción:** la función PCase convierte el primer carácter de cada palabra delimitada por espacios de una cadena a mayúsculas, y todos los demás caracteres se convierten a minúsculas.

**Sintaxis:** `String PCase(string)`

**Ejemplo:** `PCase("TEsT")` devuelve “Test”.

----------
### RandomNum

**Descripción:** la función RandomNum devuelve un número aleatorio entre un intervalo especificado.

**Sintaxis:** `num RandomNum(num start, num end)`

- start: un número que identifica el límite inferior del valor aleatorio que se va a generar
- end: un número que identifica el límite superior del valor aleatorio que generar

**Ejemplo:** `Random(100,999)` puede devolver 734.

----------
### RemoveDuplicates

**Descripción:** la función RemoveDuplicates toma una cadena de varios valores y garantiza que cada valor es único.

**Sintaxis:** `mvstr RemoveDuplicates(mvstr attribute)`

**Ejemplo:** `RemoveDuplicates([proxyAddresses])` devuelve un atributo proxyAddress saneado donde se han quitado todos los valores duplicados.

----------
### Sustituya

**Descripción:** la función Replace reemplaza todas las apariciones de una cadena en otra cadena.

**Sintaxis:** `str Replace(str string, str OldValue, str NewValue)`

- string: una cadena en la que se reemplazarán los valores.
- OldValue: la cadena que se va a buscar y reemplazar.
- NewValue: la cadena que reemplazar.

**Comentarios:** la función reconoce los siguientes monikers especiales:

- \\n: nueva línea
- \\r: retorno de carro
- \\t: tabulación

**Ejemplo:** `Replace([address],"\r\n",", ")`reemplaza CRLF por una coma y un espacio y podría originar “One Microsoft Way, Redmond, WA, USA”

----------
### ReplaceChars

**Descripción:** la función ReplaceChars reemplaza todas las apariciones de caracteres encontrados en la cadena ReplacePattern.

**Sintaxis:** `str ReplaceChars(str string, str ReplacePattern)`

- string: una cadena por la que reemplazar los caracteres.
- ReplacePattern: una cadena que contiene un diccionario con caracteres que reemplazar.

El formato es {source1}:{target1},{source2}:{target2},{sourceN},{targetN} donde source es el carácter que buscar y target la cadena por la que reemplazar.

**Comentarios:**

- La función toma cada aparición de los orígenes definidos y los reemplaza por los destinos.
- El origen debe ser exactamente un carácter (unicode).
- El origen no puede estar vacío ni puede tener más de un carácter (error de análisis).
- El destino puede tener varios caracteres, por ejemplo, ö:oe, β:ss.
- El destino puede estar vacío, lo que indica que el carácter debe quitarse.
- El origen distingue mayúsculas y minúsculas, y debe ser una coincidencia exacta.
- La coma (,) y los dos puntos (:) son caracteres reservados y no se puede reemplazar utilizando esta función.
- Se omiten los espacios y otros caracteres en blanco en la cadena ReplacePattern.

**Ejemplo:** `%ReplaceString% = ’:,Å:A,Ä:A,Ö:O,å:a,ä:a,ö,o`

`ReplaceChars("Räksmörgås",%ReplaceString%)` Devuelve Raksmorgas

`ReplaceChars("O’Neil",%ReplaceString%)` Devuelve “ONeil”, se define la marca única para quitarla.

----------
### Right

**Descripción:** la función Right devuelve un número especificado de caracteres desde la derecha (final) de una cadena.

**Sintaxis:** `str Right(str string, num NumChars)`

- string: la cadena desde la que devolver los caracteres
- NumChars: un número que identifica el número de caracteres que devolver desde el final (derecho) de la cadena

**Comentarios:** los caracteres NumChars se devuelven a partir de la última posición de la cadena.

Una cadena que contiene los últimos caracteres numChars de la cadena:

- Con numChars = 0, se devuelve una cadena vacía.
- Con numChars < 0, se devuelve una cadena de entrada.
- Si la cadena es null, se devuelve una cadena vacía.

Si la cadena contiene menos caracteres que el número especificado en NumChars, se devuelve una cadena idéntica a la cadena.

**Ejemplo:** `Right("John Doe", 3)` devuelve “Doe”.

----------
### RTrim

**Descripción:** la función RTrim quita los espacios en blanco del final de una cadena.

**Sintaxis:** `str RTrim(str value)`

**Ejemplo:** `RTrim(" Test ")` devuelve “Test”.

----------
### Dividir

**Descripción:** la función Split toma una cadena separada por un delimitador y la convierte en una cadena de varios valores.

**Sintaxis:** `mvstr Split(str value, str delimiter)` `mvstr Split(str value, str delimiter, num limit)`

- value: la cadena con un carácter delimitador que separar.
- delimiter: carácter único que usar como delimitador.
- limit: número máximo de valores que se devolverá.

**Ejemplo:** `Split("SMTP:john.doe@contoso.com,smtp:jd@contoso.com",",")` devuelve una cadena de varios valores con 2 elementos útiles para el atributo proxyAddress

----------
### StringFromGuid

**Descripción:** la función StringFromGuid toma un GUID binario y lo convierte en una cadena

**Sintaxis:** `str StringFromGuid(bin GUID)`

----------
### StringFromSid

**Descripción:** la función StringFromSid convierte una matriz de bytes o una matriz de bytes de varios valores que contiene un identificador de seguridad en una cadena o en una cadena de varios valores.

**Sintaxis:** `str StringFromSid(bin ObjectSID)` `mvstr StringFromSid(mvbin ObjectSID)`

----------
### Switch

**Descripción:** la función Switch se utiliza para devolver un único valor según las condiciones evaluadas.

**Sintaxis:** `var Switch(exp expr1, var value1[, exp expr2, var value … [, exp expr, var valueN]])`

- expr: expresión variante que desea evaluar.
- value: valor que se va a devolver si la expresión correspondiente es True.

**Comentarios:** la lista de argumentos de la función Switch consta de pares de expresiones y valores. Las expresiones se evalúan de izquierda a derecha y se devuelve el valor asociado a la primera expresión que evaluar en True. Si los elementos no están emparejados correctamente, se produce un error en tiempo de ejecución.

Por ejemplo, si expr1 es True, Switch devuelve value1. Si expr-1 es False, pero expr-2 es True, Switch devuelve value-2, etc.

Switch devuelve Nothing si:

- Ninguna de las expresiones son True.
- La primera expresión True tiene un valor correspondiente que es Null.

Switch evalúa todas las expresiones, aunque devuelva solo una de ellas. Por este motivo, debe vigilar efectos secundarios no deseados. Por ejemplo, si la evaluación de cualquier expresión tiene como resultado una división por cero, se produce un error.

El valor puede ser también la función Error que devolvería una cadena personalizada.

**Ejemplo:** `Switch([city] = "London", "English", [city] = "Rome", "Italian", [city] = "Paris", "French", True, Error("Unknown city"))` devuelve el idioma hablado en las ciudades más importantes. De lo contrario, devuelve un error.

----------
### Trim

**Descripción:** la función Trim quita los espacios en blanco del principio y del final de una cadena.

**Sintaxis:** `str Trim(str value)` `mvstr Trim(mvstr value)`

**Ejemplo:** `Trim(" Test ")` devuelve “Test”.

`Trim([proxyAddresses])` Quita los espacios del principio y del final de cada valor en el atributo proxyAddress.

----------
### UCase

**Descripción:** la función UCase convierte todos los caracteres de una cadena a mayúsculas.

**Sintaxis:** `str UCase(str string)`

**Ejemplo:** `UCase("TeSt")` devuelve “TEST”.

----------
### Word

**Descripción:** la función Word devuelve una palabra incluida en una cadena, según los parámetros que describen los delimitadores que usar y el número de palabras que se devolverá.

**Sintaxis:** `str Word(str string, num WordNumber, str delimiters)`

- string: la cadena desde la que devolver una palabra.
- WordNumber: un número que identifica qué número de palabras debe devolverse.
- delimiters: una cadena que representa los delimitadores que deben usarse para identificar palabras

**Comentarios:** cada cadena de caracteres en la cadena separada por uno de los caracteres en los delimitadores se identifica como palabras:

- Si el número es < 1, se devuelve una cadena vacía.
- Si la cadena es Null, se devuelve una cadena vacía.

Si la cadena contiene menos palabras o si la cadena no contiene palabras identificadas por los delimitadores, se devuelve una cadena vacía.

**Ejemplo:** `Word("The quick brown fox",3," ")` Devuelve “brown”

`Word("This,string!has&many separators",3,",!&#")` devolvería “has”

## Recursos adicionales

* [Explicación de las expresiones declarativas de aprovisionamiento](active-directory-aadconnectsync-understanding-declarative-provisioning-expressions.md)
* [Sincronización de Azure AD Connect: personalización de las opciones de sincronización](active-directory-aadconnectsync-whatis.md)
* [Integración de las identidades locales con Azure Active Directory](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0309_2016-->