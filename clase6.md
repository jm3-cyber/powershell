# VARIABLES, ENTRADA Y SALIDA

## VARIABLES

Powershell permite almacenar valores en variables. Todas los nombres de variable
comienzan con el signo ``$``, y se componen de letras, números y la raya de
subrayar (``_``).

El operador de asignación es el signo igual (``=``). Algunos ejemplos:

Comando | Descripción
------- | -----------
``$servidor = "localhost"`` | La variable contendrá una cadena de texto
``$numero = 5``             | La variable contendrá un número entero
``$servicios = Get-Service`` | La variable contendrá una lista de servicios

Si se desea consultar el tipo de datos que contiene una variable, se puede
hacer con el cmdlet ``Get-Member``. Por ejemplo:

```console
PS> $servidor | gm

   TypeName: System.String

Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System...
CompareTo        Method                int CompareTo(System.Object v...
Contains         Method                bool Contains(string value)

PS> $numero | gm

   TypeName: System.Int32

Name        MemberType Definition
----        ---------- ----------
CompareTo   Method     int CompareTo(System.Object value), int Compa...
Equals      Method     bool Equals(System.Object obj), bool Equals(i...
GetHashCode Method     int GetHashCode()
```

``Get-Member`` también despliega las propiedades y métodos de la variable. Por
ejemplo, las variables tipo ``System.String`` poseen el método ``ToUpper()``, que
convierte el contenido de la variable a mayúsculas:

```console
PS> $servidor.ToUpper()
LOCALHOST
```

Las variables pueden emplearse para sustituir cualquier parámetro en la llamada
a un cmdlet. Teniendo en cuenta el valor de la variable ``$servidor`` en esta
guía, estas dos órdenes son equivalentes:

```Powershell
Get-WmiObject Win32_ComputerSystem -computername localhost
Get-WmiObject Win32_ComputerSystem -computername $servidor
```

## USO DE LAS COMILLAS EN LA ASIGNACIÓN A VARIABLES
Las comillas sencillas se emplean para asignar a una variable el texto exacto
que se ponga entre ellas, por ejemplo:

```console
PS> $var = 'Hola'
PS> $var = 'El contenido de la variable es $var'
PS> $var
El contenido de la variable es $var
```

Se podría pensar que Powershell desplegaría **El contenido de la variable es
hola**, pero las comillas sencillas hacen que el signo ``$`` no se interprete
como inicio del nombre de una variable.

Para lograr que Powershell interprete el signo ``$`` como inicio del nombre de
una variable, se efectúa la asignación con comillas dobles:

```console
PS> $var = 'Hola'
PS> $var="La variable contiene $var"
PS> $var
La variable contiene Hola
```

La comilla invertida `` ` `` hace que Powershell ignore el significado del
siguiente caracter especial, por ejemplo:

```console
PS> $var = 'Hola'
PS> $var="La variable `$var contiene $var"
PS> $var
La variable $var contiene Hola
```

También puede emplearse para dar significado especial a ciertos caracteres
(equivale a ``\`` en C y Java, por ejemplo ``\n``, ``\t``...). Un ejemplo de
uso es el siguiente:

```Powershell
PS> $nombrecompu = 'localhost'
PS> $frase = "`$nombrecompu`ncontiene`n$nombrecompu"
PS> $frase
$nombrecompu
contiene
localhost
```

Como se puede ver, `` `n`` simboliza el caracter de retorno de carro.

## VARIABLES TIPO LISTA

Es posible crear una variable tipo lista, separando los valores de la lista
con comas:

```console
PS> $computadores = 'servidor', 'localhost', 'servidor_2'
PS> $computadores
servidor
localhost
servidor_2
```

Los elementos de la lista se pueden acceder por número índice. Las listas se
numeran desde el cero:

```console
PS> $computadores[0]
servidor
PS> $computadores.count
3
```

A partir de la versión 3 de Powershell, si se pasa una lista como parámetro de
un cmdlet, Powershell itera sobre los elementos de la lista. Por ejemplo, el
siguiente comando despliega información sobre los tres computadores que figuran
en la lista:

```Powershell
get-wmiobject win32_computersystem -comp $computadores
```

También es posible iterar sobre una lista empleando el cmdlet ``Foreach-Object``.
Estas dos órdenes hacen lo mismo:

```Powershell
$computadores = $computadores.tolower()
$computadores = $computadores | ForEach-Object {$_.tolower()}
```

## DECLARACIÓN DEL TIPO DE UNA VARIABLE
Hay ocasiones en que es necesario asegurarse que el tipo de una variable es el
correcto para la operación que se desea realizar. Analice el siguiente ejemplo:

```console
PS> $numero = read-host "Introduzca un número"
Introduzca un número: 1024
PS> $numero = $numero * 10
PS> $numero
1024102410241024102410241024102410241024
```

El resultado de esta operación no es el esperado, pero esto ocurre porque el
cmdlet ``read-host`` retorna un valor tipo ``System.String``.

Se puede corregir esta situación, forzando a Powershell a hacer la conversión
del valor digitado a tipo entero:

```console
PS> [int]$numero = read-host "Introduzca un número"
Introduzca un número: 1024
PS> $numero = $numero * 10
PS> $numero
10240
```

Los tipos de datos que se pueden emplear son:
- ``[int]`` para enteros,
- ``[single]`` y ``[double]`` para números de punto flotante de precisión
 sencilla y doble,
- ``[string]`` para cadenas,
- ``[char]`` para caracteres,
- ``[xml]`` para un documento XML; el valor es validado como contenido XML.

## COMANDOS PARA MANIPULACIÓN DE VARIABLES

Powershell incluye los siguientes comandos para manipular variables:

- ``New-Variable``
- ``Set-Variable``
- ``Remove-Variable``
- ``Get-Variable``
- ``Clear-Variable``

Con excepción del comando ``Remove-Variable``, todos los demás se ejecutan
de forma implícita empleando la sintaxis que se ha usado en esta guía.

## SOLICITANDO INFORMACIÓN AL USUARIO

El cmdlet ``Read-Host`` despliega un mensaje de solicitud, y luego lee una
respuesta del usuario. Por ejemplo:

```console
PS> Read-Host "Introduzca un nombre de computador"
Introduzca un nombre de computador: SERVIDOR
SERVIDOR
```

El cmdlet agrega dos puntos (:) después del mensaje de solicitud, y coloca
la respuesta del usuario en el pipeline. En la mayoría de los casos, la
respuesta del usuario se captura en una variable, como en el siguiente ejemplo:

```console
PS> $computador = Read-Host "Introduzca un nombre de computador"
Introduzca un nombre de computador: SERVIDOR
```

## ESCRIBIENDO INFORMACIÓN A LA CONSOLA

Para escribir datos a la consola, existen dos métodos:

- ``Write-Host``: Permite escribir datos directamente a la consola, saltándose el
  pipeline. Por esta razón, su salida no puede ser capturada ni redirigida a
  logs. Se recomienda no usarlo.
- ``Write-Output``: Permite escribir datos al pipeline. Si no hay redirección a
  otro comando, los datos saldrán por pantalla. Esta es la manera recomendada
  de imprimir datos por consola.

## OTRAS FORMAS DE ESCRIBIR A LA CONSOLA

Powershell tiene otros cmdlets para producir salida a consola. Ninguna de ellas
emplea el pipeline (funcionan como ``Write-Host``), pero su salida se puede
suprimir a voluntad, mediante el uso de variables de entorno. Los comandos son
los siguientes:

Comando | Descripción
------- | -----------
``Write-Warning``| Imprime el mensaje en color naranja, precedido por la palabra WARNING. Su impresión depende del valor de la variable ``$WarningPerference``, que por omisión tiene el valor ``Continue``.
``Write-Verbose``| Imprime el mensaje en color azul, precedido por la palabra VERBOSE. Su impresión depende del valor de la variable ``$VerbosePreference``, que por omisión se fija en ``SilentlyContinue``.
``Write-Debug``| Imprime el mensaje en color azul, precedido por la palabra DEBUG. Su impresión depende del valor de la variable ``$DebugPreference``, que por omisión tiene el valor ``SilentlyContinue``.
``Write-error``| Genera un mensaje de error. Su impresión depende del valor de la variable ``$ErrorActionPreference``, que por omisión tiene el valor ``Continue``.

Si la variable correspondiente al comando tiene el valor ``Continue``, el mensaje
se imprime. Si el valor es ``SilentlyContinue``, se suprime la impresión del
mensaje.
