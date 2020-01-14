# FORMATEO DE LA SALIDA, MÁS ESTRATEGIAS DE FILTRADO

## FORMATEO DE LA SALIDA: TABLAS, LISTAS, PRESENTACIÓN ANCHA

Los comandos de Powershell presentan su salida en tablas o en listas,
dependiendo del comando que se emplee y de los atributos que se le pida
mostrar.

Anteriormente, se vio que se puede emplear el comando ``Select-Object``
(abreviado como ``Select``) para determinar los atributos que se despliegan. Sin
embargo, el despliegue está limitado a los atributos relacionados con el comando
invocado. Por ejemplo:

```Powershell
Get-Process | select name,vm
```

...despliega una lista de procesos, compuesta solamente por el nombre de cada
proceso y la cantidad de memoria virtual que emplea.

Los comandos de formateo permiten, además de imprimir los atributos estándar,
desplegar atributos calculados a partir de los atributos básicos. El despliegue
se puede hacer en modalidad de tabla, lista o formato ancho; a continuación se
explica en detalle cada modalidad.

## FORMATEO DE TABLAS

El cmdlet ``Format-Table`` (abreviado como ``ft``) permite desplegar la
salida de un comando en formato de tabla. Los parámetros que recibe este comando son los siguientes:

 Parámetros | Significado
 --------- | -----------
``-Property`` | Permite especificar los atributos (nativos o calculados) que se desea desplegar. Se puede emplear el comodín ``*`` para especificarlos todos.
``-Autosize`` | Permite que el comando acomode el ancho de las columnas de la mejor manera posible. Normalmente dicho ancho es fijo, pero este parámetro ajusta el ancho de las columnas al valor más largo de cada atributo.
``-Groupby`` | Con este parámetro se especifica uno de los campos. Cada vez que haya un cambio en el valor de este campo, se imprime un nuevo juego de encabezados. Se recomienda emplear ``Sort-Object`` antes de hacer un ``Format-Table`` con ``Groupby``, para evitar repetición innecesaria de los encabezados.
``-Wrap`` | Cuando el valor de un campo es excesivamente largo, Powershell lo recorta, e indica esto con puntos suspensivos (...). El parámetro ``Wrap`` hace que los valores largos se extiendan a una o varias líneas adicionales, según la longitud del valor.

Ejemplos:

```Powershell
get-process | ft -Property *
```
... trata de imprimir todos los atributos de los procesos. Powershell revisa
todas las líneas de la salida, y hace el mejor esfuerzo para acomodar todos
los campos posibles. Debido a esta revisión, el comando demora un tiempo
relativamente largo en ejecutarse.

```Powershell
get-process | ft -Property ID,Name,Responding -AutoSize
```
... obtiene una lista de procesos con los atributos ID, Name y Responding. El
ancho de las columnas se optimiza.

```Powershell
get-process | ft -Property * -autosize
```

... trata de imprimir todos los atributos de los procesos. Sin embargo, este
comando corre más rápido, ya que se optimiza el ancho de las columnas.

```Powershell
Get-Service | sort Status | ft Name,Status,DisplayName -groupby Status
```

... imprime una lista de servicios dividida en dos secciones: Una de servicios
detenidos (Stopped) y otra de servicios en ejecución (Running).

```Powershell
Get-Service | ft Name,Status,DisplayName -autosize -wrap
```

... imprime una lista de servicios con los atributos Name, Status y DisplayName.
El atributo DisplayName (que es el más largo) se extenderá a varias líneas si
es necesario.

## FORMATEO DE LISTAS

El cmdlet ``Format-List`` (abreviado como ``fl``) permite mostrar los atributos
como una serie de duplas nombre-valor, por ejemplo:

```console
Get-Service -name bits | fl -Property *

Name                : bits
RequiredServices    : {RpcSs}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
DisplayName         : Background Intelligent Transfer Service
DependentServices   : {}
MachineName         : .
ServiceName         : bits
ServicesDependedOn  : {RpcSs}
ServiceHandle       :
Status              : Stopped
ServiceType         : Win32ShareProcess
StartType           : Manual
Site                :
Container           :
```

Nótese que aquí se filtró la salida del comando ``Get-Service``, para obtener
únicamente la información del servicio BITS (de lo contrario, ``Format-List``
hubiera arrojado un grupo de líneas similar por cada servicio). Aquí se le
pidió a ``Format-List`` que mostrara todos los atributos del servicio (el
parámetro ``-Property`` con comodín).

## FORMATO ANCHO

El formato ancho permite mostrar dos o más columnas de una propiedad del objeto
en particular, al estilo del comando ``ls`` de Linux. Se emplea el cmdlet
``Format-Wide`` (abreviado ``fw``) para este propósito.

Ejemplos:

```Powershell
get-process | format-wide
```

... muestra dos columnas de nombres de procesos (``Format-wide`` escoge por
omisión la propiedad Name).

```Powershell
get-process | format-wide name -col 4
```
... muestra 4 columnas de nombres de procesos.

## CAMBIO DE NOMBRES DE ATRIBUTOS, Y CÁLCULO DE NUEVOS ATRIBUTOS

Un ejemplo de cambio de nombre de un atributo es el siguiente:

```Powershell
get-service | ft @{name='Servicio';expression={$_.Name}},Status,DisplayName
```

En este caso, se define una nueva columna, empleando la expresión que va entre
los símbolos ``@{ }`` .
* ``Name`` indica el nombre que tendrá la columna. Se puede abreviar como ``n``.
* ``Expression`` define el contenido de la columna. Se puede abreviar como ``e``.
* ``$_`` es una variable especial que contiene el objeto que se está procesando
  en ese momento. Para este ejemplo, ``$_.Name`` quiere decir: "Del objeto
  actual, tome la propiedad ``Name``".

El comando anterior se podría abreviar así:

```Powershell
get-service | ft @{n='Servicio';e={$_.Name}},Status,DisplayName
```

El siguiente comando:

```Powershell
get-process | ft Name,@{n='VM (MB)';e={$_.VM / 1MB}} -
```

...despliega una tabla de procesos con dos columnas: **Name** (propiedad nativa)
y **VM (MB)**, que muestra la cantidad de memoria virtual usada por el
proceso, en megabytes.

El comando:

```Powershell
get-process | ft Name,@{n='VM (MB)';e={$_.VM / 1MB -as [int]}} -AutoSize
```

...despliega una tabla similar a la del ejemplo anterior, pero redondea la
memoria virtual a un valor entero.

## USO DE LOS COMANDOS DE FORMATEO

El comando de formateo que se vaya a emplear (``ft``, ``fl`` ó ``fw``) debe ser
el **último** en el pipeline antes de imprimir en pantalla. La salida de un
comando de formateo solamente puede ser redirigida a un archivo de **TEXTO**.
Si se intenta convertir la salida a otro formato (CSV, HTML, XML) los
resultados serán inconsistentes.

## MÁS ESTRATEGIAS DE FILTRADO
Los resultados de los comandos se pueden filtrar de varios modos:

* Empleando comodines en los parámetros que así lo soporten. Por ejemplo:

```Powershell
get-service -name s*
```

...muestra una lista de servicios cuyo nombre empieza por S.

* Si el parámetro no soporta comodines, se puede emplear el cmdlet
``Where-Object`` (abreviado ``Where``) en el pipeline. Por ejemplo:

```Powershell
Get-Service | where -filter { $_.Status -like "Run*" }
```

...muestra una lista de servicios cuyo estado (Status) comienza con "Run"

Algunos de los operadores de comparación que se pueden usar son:

Operador | Significado
-------- | -----------
``-eq``      | Igual
``-ne``      | No igual
``-gt``      | Mayor que
``-ge``      | Mayor que o igual
``-lt``      | Menor que
``-le``      | Menor que o igual
``-like``    | Coincide con la expresión con comodines
``-notlike`` | No coincide con la expresión con comodines

Por ejemplo:

```Powershell
Get-Service | where -filter { $_.Status -eq "Running" }
```

Las comparaciones de cadenas no hacen normalmente distinción entre mayúsculas
y minúsculas. Si se requiere hacerla, se antepone una ``c`` a los operadores
(``-ceq``, ``-cne``, ``-cgt``, ``-cge``...).

Se pueden emplear también los conectivos ``-and`` y ``-or``.

Para mayor información, puede consultarse el tópico de ayuda
``about_comparison_operators``.

## UN EJEMPLO COMPLETO DE FILTRADO COMPLEJO
Se requiere sacar un listado que incluya los 10 procesos que más memoria
virtual están consumiendo, sin incluir a Powershell. Al final, debe presentarse
el total de memoria virtual que están consumiendo estos 10 procesos. El listado
solamente debe incluir las columnas Name y VM.

- [x] El primer paso (filtrar los procesos Powershell) se puede hacer así:

```Powershell
Get-Process | where -filter { $_.Name -notlike "Powershell*" }
```

- [x] Luego se organiza por la columna VM, en orden descendente, y se
especifican las columnas que se desea mostrar:

```Powershell
Get-Process | where -filter { $_.Name -notlike "Powershell*" } | sort VM -desc | select name,vm
```

- [x] Por último, se emplea el cmdlet ``Measure-Object`` para hallar el total
de uso de memoria virtual:

```Powershell
Get-Process | where -filter { $_.Name -notlike "Powershell*" } | sort VM | select name,vm -first 10 | Measure-Object -Property vm -sum
```

## EJERCICIOS
1. Mostrar una tabla de procesos que incluya únicamente los nombres de los
   procesos, sus IDs, y si están respondiendo a Windows (la propiedad
   ``Responding`` muestra eso). Haga que la tabla tome el mínimo de espacio
   horizontal, pero no permita que la información se trunque.

2. Muestre una tabla de procesos que incluya los nombres de los procesos y sus
   IDs. También incluya columnas para uso de memoria virtual y física;
   exprese dichos valores en megabytes (MB).

3. Emplee ``Get-EventLog`` para mostrar una lista de los logs de eventos
   disponibles (revise la ayuda para encontrar el parámetro que le permitirá
   obtener dicha información). Formatee la salida como una tabla que incluya
   el nombre de despliegue del log y el período de retención. Los encabezados
   de columna deben ser NombreLog y Per-Retencion.

4. Muestre una lista de servicios, de tal manera que aparezcan agrupados los
   servicios que están iniciados y los que están detenidos. Los que están
   iniciados deben aparecer primero.

5. Mostrar una lista a cuatro columnas de todos los directorios que están en
   el raíz de la unidad ``C:``

6. Cree una lista formateada de todos los archivos ``.exe`` del directorio
   ``C:\Windows``. Debe mostrarse el nombre, la información de versión, y el
   tamaño del archivo. La propiedad de tamaño se llama ``length`` en Powershell,
   pero para mayor claridad, la columna se debe llamar **Tamaño** en su listado.

7. Importe el módulo ``NetAdapter`` (empleando el comando ``Import-Module
   NetAdapter``).
   Empleando el cmdlet ``Get-NetAdapter``, muestre una lista de adaptadores no
   virtuales (adaptadores cuya propiedad Virtual sea falsa. El valor lógico
   falso es representado por Powershell como ``$False``).

8. Importe el módulo ``DnsClient``. Empleando el cmdlet ``Get-DnsClientCache``,
   muestre una lista de los registros ``A`` y ``AAAA`` que estén en el caché.
   Sugerencia: Si el caché está vacío, visite algunos sitios web para poblarlo.

9. Genere una lista de todos los archivos ``.exe`` del directorio
   ``C:\Windows\System32`` que tengan más de 5 MB.

10. Muestre una lista de parches que sean actualizaciones de seguridad.

11. Muestre una lista de parches que hayan sido instalados por el
    usuario ``Administrador``, que sean actualizaciones. Si no tiene ninguno,
    busque parches instalados por el usuario ``System``. Note que algunos parches
    no tienen valor en el campo ``Installed By``.

12. Genere una lista de todos los procesos que estén corriendo con el nombre
    **Conhost** o **Svchost**.
