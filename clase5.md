# ACCESO A CARACTERÍSTICAS AVANZADAS DE ADMINISTRACIÓN

## ASPECTOS BÁSICOS DE WMI Y CIM

Un computador Windows contiene miles de elementos de información de gestión.
Los frameworks **WMI** (Windows Management Instrumentation) y **CIM** (Common
Information Model) buscan facilitar y organizar dichos elementos.

La información en WMI está organizada por namespaces (espacios con nombre).
- El namespace ``root\CIMv2`` contiene toda la información acerca del sistema
  operativo Windows y el hardware subyacente.
- En computadores cliente, el namespace ``root\SecurityCenter2``
  (``root\SecurityCenter`` en versiones anteriores del sistema operativo)
  contiene información acerca de software firewall, antivirus y antispyware
  instalados en el computador.

Dentro de cada namespace, WMI incluye una serie de **clases**. Cada clase
representa un tipo de componente de gestión que WMI sabe cómo consultar. Por
ejemplo, la clase ``Win32_LogicalDisk`` del namespace ``root\CIMv2`` contiene
información acerca de discos lógicos, mientras que la clase ``AntiSpywareProduct``
del namespace ``root\SecurityCenterv2`` contiene información acerca de productos
antispyware.

Si existen componentes gestionables para una cierta clase, aparecerán como
**instancias** de dicha clase. Los nombres de las clases en el namespace
``root\CIMv2`` suelen iniciar con ``Win32_`` (inclusive en máquinas de 64 bits)
o con ``CIM_`` . Por lo general, las propiedades de dichas instancias son de sólo
lectura (es decir, no se pueden efectuar cambios a los valores de los
parámetros).

Existen diferentes herramientas diseñadas para explorar las distintas clases
de WMI. Una de ellas es **WMI Explorer**, que se puede descargar desde:
https://powershell.org/2013/03/wmi-explorer/

Esta herramienta es un script de Powershell. Para correrla, basta con cambiarse
al directorio donde se la descomprimió, y ejecutarla como ``.\wmiexplorer.ps1``.
Si sale algún error indicando que no se tiene permiso para correr el script, se
puede cambiar la política de ejecución corriendo el siguiente comando como
administrador:

```powershell
Set-ExecutionPolicy unrestricted
```

## EXPLORACIÓN DE WMI EMPLEANDO GET-WMIOBJECT

Se puede emplear el cmdlet ``Get-WmiObject`` para explorar los namespaces de WMI.
Por ejemplo, si se quisiera consultar las clases relacionadas con discos,
se puede emplear la siguiente orden:

```Powershell
Get-WmiObject -Namespace root\CIMv2 -list | where name -like '*disk*'
```

La siguiente orden muestra una lista completa de las clases dentro del
namespace ``root\CIMv2``:

```Powershell
Get-WmiObject -Namespace root\CIMv2 -list
```

Para interrogar una clase específica, se emplea el parámetro ``-class``:

```Powershell
Get-WmiObject -Namespace root\CIMv2 -class win32_desktop
```

El comando anterior se puede abreviar, debido a que el namespace por omisión
es ``root\CIMv2``, y el parámetro ``-class`` es posicional:

```Powershell
Get-WmiObject win32_desktop
```

## EXPLORACIÓN DE WMI EMPLEANDO GET-CIMINSTANCE

El comando ``Get-CimInstance`` se introdujo en Powershell versión 3, y funciona
de manera muy parecida a ``Get-WmiObject``, excepto por lo siguiente:

- Se emplea el parámetro ``-ClassName`` en lugar de ``-Class``.
- No existe el parámetro ``-list`` para listar todas las clases de un namespace.
  Debe emplearse el cmdlet ``Get-CimClass`` con el parámetro ``-Namespace``.

Ejemplos:

Listar las instancias de la clase ``Win32_LogicalDisk``:

```powershell
Get-CimInstance -ClassName win32_logicaldisk
```

Obtener la lista de clases de ``root\CIMv2``, filtrando las que tienen que ver
con discos:

```Powershell
Get-CimClass -Namespace root\CIMv2 | where cimclassname -Like '*disk*'
```

## EJERCICIOS
1. ¿Cuál clase puede emplearse para consultar la dirección IP de un adaptador
   de red? ¿Posee dicha clase algún método para liberar un préstamo de
   dirección (lease) DHCP?
2. Despliegue una lista de parches empleando WMI (Microsoft se refiere a los
   parches con el nombre **quick-fix engineering**). Es diferente el listado al
   que produce el cmdlet ``Get-Hotfix``?
3. Empleando WMI, muestre una lista de servicios, que incluya su status actual,
   su modalidad de inicio, y las cuentas que emplean para hacer login.
4. Empleando cmdlets de CIM, liste todas las clases del namespace
   ``SecurityCenter2``, que tengan **product** como parte del nombre.
5. Empleando cmdlets de CIM, y los resultados del ejercicio anterior, muestre
   los nombres de las aplicaciones antispyware instaladas en el sistema.
   También puede consultar si hay productos antivirus instalados en el sistema.
