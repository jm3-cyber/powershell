# EXPORTACIÓN DE DATOS, PROFUNDIZACIÓN EN EL PIPELINE

## EXPORTAR DATOS EN VARIOS FORMATOS

El cmdlet ``export-CSV`` permite exportar la salida de un comando hacia un
archivo de texto, en formato separado por comas. Un ejemplo:

```powershell
get-process | export-csv procesos.csv
```

Abriendo el archivo resultante con un editor de texto, se puede comprobar que
cada línea contiene **todos** los atributos de los procesos, a diferencia del
listado por pantalla, que solamente muestra un subconjunto predeterminado de
campos.

El archivo resultante se puede cargar en una herramienta como Excel, en una
base de datos, o se puede volver a cargar en Powershell para analizarlo como
una instantánea (snapshot) del sistema, con el siguiente comando:

```powershell
import-csv procesos.csv
```

El cmdlet ``export-clixml`` permite exportar la salida de un comando en formato
XML, que es más completo que el formato CSV:

```powershell
get-process | export-clixml procesos.xml
```

El comando para importar un archivo grabado por Powershell en formato XML
es ``import-clixml``. Un ejemplo:

```powershell
import-clixml procesos.xml
```

## COMPARACIÓN DE OBJETOS

El cmdlet ``Compare-Object`` (que tiene el alias ``diff``) permite comparar
dos objetos y mostrar las diferencias entre ambos. En este sentido, permite
comparar un snapshot de los procesos (o servicios, o cualquier otra cosa)
de un sistema con un snapshot más actual.

Por ejemplo, se puede generar un snapshot de los procesos del sistema:

```powershell
get-process | export-clixml procesos.xml
```

...y más tarde, comparar con los procesos que haya en ese momento:

```powershell
diff -Ref (Import-Clixml .\procesos.xml) -Diff (get-process) -Property name
```

El comando anterior se interpreta de la siguiente manera:

- [x] Los paréntesis se interpretan igual que en álgebra: Los comandos que están
  entre paréntesis se ejecutan primero, y sus resultados se pasan al comando
  de más afuera.
- [x] El parámetro ``-Ref`` (``-ReferenceObject`` por extenso) indica el objeto
  que se va a emplear como base para la comparación (en este caso,
  el snapshot que se había grabado previamente).
- [x] El parámetro ``-Diff`` (``-DifferenceObject`` por extenso) indica
  el objeto que se va a comparar contra la base (en este caso, la lista
  actual de procesos).
- [x] El parámetro ``-Property`` indica la propiedad que se compara,
  en este caso, los nombres de los procesos

La salida del comando es similar a ésta:

```console
name                                                           SideIndicator
----                                                           -------------
WindowsInternal.ComposableShell.Experiences.TextInput.InputApp =>           
YourPhone                                                      =>           
LockApp                                                        <=           
Microsoft.Photos                                               <=           
notepad                                                        <=           
```

Aparecen los nombres de los procesos, con una flecha que apunta a izquierda
o a derecha:
- [x] Si la flecha apunta a la derecha, indica un nombre que está en el snapshot
  nuevo, pero no en el original.
- [x] Si la flecha apunta a la izquierda, indica un nombre que está en el
  snapshot original, pero no en el nuevo.

## SALIDA A ARCHIVO O A IMPRESORA

Si se desea grabar la salida de un comando a un archivo de texto plano sin
formato, se pueden emplear cualquiera de estas maneras (se toma como
ejemplo el comando ``get-process``):

- [x] ``get-process > procesos.txt`` (estilo redirección)
- [x] ``get-process | out-file procesos.txt`` (usando el pipeline)

Las dos versiones son funcionalmente equivalentes, pero el comando ``out-file``
recibe parámetros para cambiar el ancho de línea, y para evitar la
sobreescritura de un archivo existente.

Para leer un archivo de texto plano como cadenas de texto, se emplea el cmdlet
``Get-Content`` (con alias ``cat`` ó ``type``):

```powershell
Get-Content procesos.txt
```

Si se desea enviar la salida de un comando a la impresora, se emplea el cmdlet
``out-printer``:

```powershell
get-process | out-printer
```

## CONVERSIÓN DE LA SALIDA A HTML

La salida de un cmdlet cualquiera puede convertirse a HTML, de la siguiente
manera (empleando como ejemplo ``get-process``):

```powershell
get-process | ConvertTo-HTML
```

Este comando manda sus resultados a la pantalla. Para crear un archivo HTML,
se requiere combinarlo con ``Out-File``:

```powershell
get-process | ConvertTo-HTML | Out-file procesos.html
```

## EMPLEO DEL PIPELINE PARA ENLAZAR COMANDOS QUE GESTIONAN RECURSOS

El pipeline puede emplearse en forma muy poderosa para gestionar recursos
de la máquina. Por ejemplo, el comando:

```powershell
get-process | stop-process
```

... pararía todos los procesos de la máquina si Powershell está corriendo con
privilegios de administrador (por favor NO ejecute este comando, ya que podría
bloquear la máquina!).

Para simular el efecto de este comando, lo puede correr de la siguiente manera:

```powershell
get-process | stop-process -whatif
```

Un mejor ejemplo para un administrador sería buscar un proceso específico y
detenerlo. El siguiente comando busca todos los procesos llamados ``notepad``
y trata de detenerlos:

```powershell
get-process -name notepad | stop-process
```

## ORDENANDO Y SELECCIONANDO OBJETOS EMPLEANDO EL PIPELINE

El pipeline se puede emplear para seleccionar propiedades de los objetos que
no aparecen por omisión cuando se emplea un cmdlet. También se puede emplear
para filtrar los resultados (empleando criterios de búsqueda).

Para ordenar, se emplea el cmdlet ``Sort-Object`` (abreviado ``sort``), y para
seleccionar propiedades, ``Select-Object`` (abreviado ``select``).

Por ejemplo, la lista de procesos aparece normalmente en orden alfabético
por nombre de proceso. Para ordenarla por Process ID, se emplea el comando:

```powershell
get-process | sort id
```

Para ordenarla por uso de memoria virtual, en orden decreciente:

```powershell
get-process | sort vm -desc
```

Debe notarse que en este último ejemplo se está ordenando por un campo que
no se muestra normalmente en pantalla. Para cambiar los campos que se
despliegan, se emplea ``select``:

```powershell
get-process | select -property id,name,vm | sort vm -desc
```

Este ejemplo muestra una tabla con el identificador, nombre y uso de memoria
virtual de los procesos de la máquina, ordenada en forma descendente por uso
de memoria virtual.

Recordar que para ver las propiedades de un objeto (los campos que es
posible mostrar) se emplea el cmdlet ``Get-Member`` (abreviado ``gm``):

```powershell
Get-Process | gm
```

## PASO DE PARÁMETROS EN EL PIPELINE

En un pipeline del tipo:

``Comando_A | Comando_B``

...los parámetros se pueden pasar de dos maneras:

- [x] Por valor (ByValue): En este caso, Powershell analiza el tipo de salida
  que da el Comando_A, y determina qué parámetro del Comando_B puede
  recibir esta salida.

Ejemplo: Analizar el comando

```powershell
Get-Process | Stop-Process
```

En este caso, ``Get-Process`` produce objetos tipo **Process**. Examinando la
ayuda de ``Stop-Process``, se encuentra el siguiente parámetro:

```console
-InputObject <Process[]>
     Specifies the process objects to stop. Enter a variable that contains the
     objects, or type a command or expression that gets the objects.

     Required?                    true
     Position?                    0
     Default value                None
     Accept pipeline input?       True (ByValue)
     Accept wildcard characters?  false
```

Como se ve, este parámetro puede recibir valores del pipeline, empleando el
método ByValue. Por esta razón, el comando funciona.

Si se intenta conectar con el pipeline dos comandos que no tienen salida y
parámetros compatibles, se produce un error. También puede darse el caso de que
el paso de datos se dé por el parámetro incorrecto. Por ejemplo, suponga que
el archivo ``computadores.txt`` contiene los nombres de varios computadores,
y se desea desplegar la lista de servicios de cada uno de estos computadores. Se
podría usar el comando:

```powershell
Get-Content computadores.txt | get-service
```

Este comando produce un error, porque el parámetro requerido (``ComputerName``)
no acepta entrada por el pipeline por el método ByValue.

El comando se puede ejecutar entonces empleando paréntesis:

```powershell
get-service -computername (get-content computadores.txt)
```

- [x] Por nombre de parámetro (ByPropertyName): En este método se deben
  especificar los nombres de los parámetros. Por ejemplo, considere un archivo
  llamado ``alias.txt`` con el siguiente contenido:

```console
Name,Value
np,notepad
sel,Select-Object
go,Invoke-Command
```

La idea es emplear el contenido de este archivo para introducirlo al comando
``new-alias``, y crear los alias listados en el archivo.

Nótese que la primera línea del archivo equivale a los encabezados de columna.
Si se importa este archivo con ``import-csv``, se obtiene lo siguiente:

```console
PS C:\Users\Usuario\powershell> Import-Csv .\alias.txt

Name Value         
---- -----         
np   notepad       
sel  Select-Object
go   Invoke-Command
```

Si se emplea ``Get-Member`` para analizar esta salida, se obtiene:

```console
   TypeName: System.Management.Automation.PSCustomObject

Name        MemberType   Definition                    
----        ----------   ----------                    
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()             
GetType     Method       type GetType()                
ToString    Method       string ToString()             
Name        NoteProperty string Name=np                
Value       NoteProperty string Value=notepad
```

Se puede ver que las dos últimas propiedades son tipo **String**. Analicemos
ahora los parámetros del comando ``New-Alias``:

```console
New-Alias [-Name] <String> [-Value] <String> [-Confirm] [-Description
<String>] [-Force] [-Option {None | ReadOnly | Constant | Private | AllScope |
Unspecified}] [-PassThru] [-Scope <String>] [-WhatIf] [<CommonParameters>]
```

Los parámetros ``Name`` y ``Value`` reciben entradas tipo **String**. Y si se
revisa la ayuda completa, se puede comprobar que ambos parámetros reciben
valores por el pipeline empleando la modalidad ByPropertyName:

```console
-Name <String>
        Specifies the new alias. You can use any alphanumeric characters in an
        alias, but the first character cannot be a number.

        Required?                    true
        Position?                    0
        Default value                None
        Accept pipeline input?       True (ByPropertyName)
        Accept wildcard characters?  false

-Value <String>
        Specifies the name of the cmdlet or command element that is being aliased.

        Required?                    true
        Position?                    1
        Default value                None
        Accept pipeline input?       True (ByPropertyName)
        Accept wildcard characters?  false
```

Se puede entonces pasar la información de un comando a otro, de la siguiente
manera:

```powershell
import-csv alias.txt | new-alias
```

## EJERCICIOS

1. Cree dos archivos de texto similares (con una o dos líneas diferentes).
   Compárelos empleando ``diff``.
2. Qué ocurre si se ejecuta:
   ```powershell
   get-service | export-csv servicios.csv | out-file
   ```
   Por qué?
3. Cómo haría para crear un archivo delimitado por puntos y comas (;)?
   PISTA: Se emplea ``export-csv``, pero con un parámetro adicional.
4. ``Export-cliXML`` y ``Export-CSV`` modifican el sistema, porque pueden crear
   y sobreescribir archivos. Existe algún parámetro que evite la
   sobreescritura de un archivo existente? Existe algún parámetro que
   permita que el comando pregunte antes de sobresscribir un archivo?
5. Windows emplea configuraciones regionales, lo que incluye el separador de
   listas. En Windows en inglés, el separador de listas es la coma (,).
   Cómo se le dice a ``Export-CSV`` que emplee el separador del sistema en lugar
   de la coma?
6. Identifique un cmdlet que permita generar un número aleatorio.
7. Identifique un cmdlet que despliegue la fecha y hora actuales.
8. Qué tipo de objeto produce el cmdlet de la pregunta 7?
9. Usando el cmdlet de la pregunta 7 y ``select-object``, despliegue solamente
   el día de la semana, así:

```console
   DayOfWeek
   ---------
    Thursday
```

10. Identifique un cmdlet que muestre información acerca de parches (hotfixes)
    instalados en el sistema.
11. Empleando el cmdlet de la pregunta 10, muestre una lista de parches
    instalados. Luego extienda la expresión para ordenar la lista por fecha
    de instalación, y muestre en pantalla únicamente la fecha de instalación,
    el usuario que instaló el parche, y el ID del parche. Recuerde examinar
    los nombres de las propiedades.
12. Complemente la solución a la pregunta 11, para que el sistema ordene los
    resultados por la descripción del parche, e incluya en el listado la
    descripción, el ID del parche, y la fecha de instalación.
    Escriba los resultados a un archivo HTML.
13. Muestre una lista de las 50 entradas más nuevas del log de eventos System.
    Ordene la lista de modo que las entradas más antiguas aparezcan primero;
    las entradas producidas al mismo tiempo deben ordenarse por número índice.
    Muestre el número índice, la hora y la fuente para cada entrada. Escriba
    esta información en un archivo de texto plano.
