# SCRIPTING BÁSICO EN POWERSHELL

Los scripts de Powershell permiten almacenar un comando o serie de comandos,
para ejcutarlos posteriormente con más facilidad. Para producir un script,
basta con grabar los comandos requeridos en un archivo de texto con
extensión ``.ps1`` .

Por ejemplo, considérese el siguiente comando, que imprime una lista de los
discos fijos conectados al sistema, junto con su espacio libre, tamaño total
y porcentaje de espacio libre:

```POWERSHELL
Get-WmiObject -Class Win32_logicaldisk `
 -ComputerName localhost `
 -filter "drivetype=3" |
 Sort-Object -Property DeviceID |
 format-table -Property DeviceID,
 @{n='EspacioLibre(MB)';e={$_.FreeSpace / 1MB -as [int]}},
 @{n='Tamaño(GB)';e={$_.Size / 1GB -as [int]}},
 @{n='%Libre';e={$_.FreeSpace / $_.Size * 100 -as [int]}}
```

Al final de las líneas 1 y 2 se emplea la comilla invertida `` ` `` antes del final
de cada línea, para indicar que el comando aún no termina. En el resto de las
líneas, este oficio lo hacen las barras verticales y las comas. Se presenta
así para que sea más fácil de leer.

Copiando estas líneas de código y grabándolas en un archivo de texto
llamado ``Get-DiskInventory.ps1``, quedará creado un script. Se puede ejecutar,
digitando el nombre del mismo en consola. Si el script está grabado en un
directorio diferente al actual, debe escribirse la ruta completa al archivo
para ejectuarlo.

En caso de que el script no ejecute, es necesario cambiar la política de
ejecución de scripts a un valor más permisivo, empleando el cmdlet
``Set-ExecutionPolicy`` como **administrador**. Ver la ayuda del cmdlet para más
detalles.

## PARAMETRIZACIÓN DE SCRIPTS

Los scripts se pueden parametrizar, es decir, se pueden crear parámetros que
se pueden especificar en el momento de ejecutar el script. Por ejemplo, en el
script del ejemplo anterior es posible volver parámetros el nombre del computador y el tipo de unidad.

El script parametrizado se vería así:

```Powershell
param (
  $ComputerName = 'localhost',
  $DriveType = 3
)

Get-WmiObject -Class Win32_logicaldisk `
  -ComputerName $ComputerName `
  -filter "drivetype=$DriveType" |
  Sort-Object -Property DeviceID |
  format-table -Property DeviceID,
  @{n='EspacioLibre(MB)';e={$_.FreeSpace / 1MB -as [int]}},
  @{n='Tamaño(GB)';e={$_.Size / 1GB -as [int]}},
  @{n='%Libre';e={$_.FreeSpace / $_.Size * 100 -as [int]}}
```

Opcionalmente, a cada parámetro se le puede especificar un valor por omisión,
como se ve en el ejemplo anterior.

## DOCUMENTACIÓN DE SCRIPTS

Los scripts se pueden documentar, de tal manera que proporcionen ayuda similar
a la que brinda Powershell. El script documentado se vería de la siguiente
manera:

```Powershell
<#
.SYNOPSIS
Get-DiskInventory obtiene información de discos lógicos para uno
o más computadores.
.DESCRIPTION
Get-DiskInventory emplea WMI para consultar las instancias
Win32_LogicalDisk de uno o más computadores. Despliega para cada disco
la letra de unidad, el espacio libre, el espacio total, y el porcentaje
de espacio libre.
.PARAMETER ComputerName
El nombre, o nombres, de los computadores a consultar. El valor por omisión
es localhost.
.PARAMETER DriveType
El tipo de unidad a consultar. Ver la documentación de Win32_LogicalDisk
para más información. 3 indica un disco fijo, y es el valor por omisión.
.EXAMPLE
Get-DiskInventory -ComputerName SERVIDOR -DriveType 3
#>

param (
  $ComputerName = 'localhost',
  $DriveType = 3
)

Get-WmiObject -Class Win32_logicaldisk `
 -ComputerName $ComputerName `
 -filter "drivetype=$DriveType" |
 Sort-Object -Property DeviceID |
 Select-Object -Property DeviceID,
 @{n='EspacioLibre(MB)';e={$_.FreeSpace / 1MB -as [int]}},
 @{n='Tamaño(GB)';e={$_.Size / 1GB -as [int]}},
 @{n='%Libre';e={$_.FreeSpace / $_.Size * 100 -as [int]}}
```

Los comentarios se pueden especificar con un símbolo ``#`` antes de cada línea de
comentario, o con la notación ``<# #>`` si se trata de varias líneas seguidas de
comentarios.

Se hizo también un ligero cambio al script: en lugar de emplear el cmdlet
``Format-Table`` se emplea ``Select-Object``, para no producir como salida una
tabla formateada. Empleando ``Select-Object`` la salida final del script es un
objeto, con lo cual el usuario final puede encadenar el script con otros
comandos empleando el pipeline.

## SCRIPTS AVANZADOS

Agregando la línea ``[CmdletBinding()]`` después de los comentarios de la ayuda
el script se convierte en avanzado, lo cual permite volver los parámetros
obligatorios, fijarles tipo de datos, agregar alias para los parámetros, etc.

La versión final del script del ejemplo es la siguiente:

```Powershell
<#
.SYNOPSIS
Get-DiskInventory obtiene información de discos lógicos para uno
o más computadores.
.DESCRIPTION
Get-DiskInventory emplea WMI para consultar las instancias
Win32_LogicalDisk de uno o más computadores. Despliega para cada disco
la letra de unidad, el espacio libre, el espacio total, y el porcentaje
de espacio libre.
.PARAMETER ComputerName
El nombre, o nombres, de los computadores a consultar.
.PARAMETER DriveType
El tipo de unidad a consultar. Ver la documentación de Win32_LogicalDisk
para más información. 3 indica un disco fijo, y es el valor por omisión.
.EXAMPLE
Get-DiskInventory -ComputerName SERVIDOR -DriveType 3
#>
[CmdletBinding()]
param (
  [Parameter(Mandatory=$True)]
  [Alias('HostName')]
  [string]$ComputerName,
  [ValidateSet(2,3)]
  [int]$DriveType = 3
)

Get-WmiObject -Class Win32_logicaldisk `
 -ComputerName $ComputerName `
 -filter "drivetype=$DriveType" |
 Sort-Object -Property DeviceID |
 Select-Object -Property DeviceID,
 @{n='EspacioLibre(MB)';e={$_.FreeSpace / 1MB -as [int]}},
 @{n='Tamaño(GB)';e={$_.Size / 1GB -as [int]}},
 @{n='%Libre';e={$_.FreeSpace / $_.Size * 100 -as [int]}}
```

Los cambios con respecto al ejemplo anterior son los siguientes:
- Se incluye la directiva para volver avanzado el script
- El parámetro ``ComputerName`` ahora es obligatorio, y se le crea el alias
  ``Hostname``.
- El parámetro ``DriveType`` se declara como entero, y los únicos valores válidos
  que recibe son ``2`` y ``3``.
