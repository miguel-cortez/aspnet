# CODE FIRST

## Crear la migración inicial

Para realizar este proceso se tinen dos opciones:
a. Consola del Administrador de paquetes (NuGet)
b. Terminal del sistema operativo (CMD)

### NuGet

Este método será estudiado posteriormente, por el momento prefiero utilizar la terminal con `dotnet` 

```
Add-Migration MigracionInicial
```

### Terminal

Nota: Después de --project se debe apuntar al directorio que tiene el archivo con extensión `.csproj` que es un archivo XML con la información necesaria para compilar el proyecto.  

```
dotnet ef migrations add MigracionInicial --project .\MacvDatabase
```

Donde MacvDatabase es el proyecto de destino, no es el proyecto donde está instalado dotnet-ef (Digo porque en mi caso tengo un proyecto para API y otro para DB donde están las clases y los archivos de migraciones). No especifico el proyecto API que es donde tengo el archivo appsettings.json de la cadena de conexión sino el proyecto donde están las clases.

El paquete dotnet-ef localmente puede ser instalado en cualquier directorio de la aplicación. Lo que se instala en directorio he visto que no afecta a otros directorios, es decir que no hay problema que se instale en diferentes directorios pero tampoco tiene sentido hacerlo.

![imagage](./img/Comandos/migrations_initdb.png)  

Si intento agregar una nueva migración el mismo nombre:  

![image](./img/Comandos/migrations_initdb_exists.png)  

Es claro que no se puede agregar una nueva migración con un identificador existente.

#### Listar las migraciones
```
dotnet ef migrations list
```

#### Ejecutar las migraciones

 ```
 dotnet ef database update
 ```

## Ejecutar nuevas migraciones

:warning: Nunca cambiar una migración desplegada (ejecutada) porque Entity Framework no lo permite (no del todo comprobado por mi persona). LO QUE SE TIENE que hacer es una nueva migración aún cuando solo se haya cambiado el tipo de dato de un campo.  

Nota. Si hace modificaciones en las entidades será necesario crear una nueva migración para actualizar la base de datos.  
El comando será el mismo; pero se debe cambiar la identificación de **MigracionInicial** a cualquier otro identificador que describa las modificaicones realizadas.  

Las migraciones posteriores a la inicial son para funcionalidades concretas, por lo que será necesario agregar un identificador específico. En cambio la primera migración tienen muchas funcionalidades en una misma migración y por ello la identificamos como MigracionInicial por ejemplo.

### Terminal

```
dotnet ef migrations add AddEmailToUser --project .\MacvCodeFirst\
```

Cuando se crean migraciones, revertir una migración no se hace borrando los archivos de migraciones, sino por línea de comandos porque las migraciones llevan un historial que ser rompería si solo borramos archivos.  

## Revertir migracion

El siguiente comando solo elimina la última migración (del proyecto)

```
dotnet ef migrations remove --project ..\MacvDatabase
```

:warning: El comando anterior NO FUNCIONA si la migración (en este caso la última migración) ya fue aplicada en la base de datos, en caso contrario, se presentará un error como el siguiente:  

```
The migration '20250830163846_AddClienteTable' has already been applied to the database. Revert it and try again. If the migration has been applied to other databases, consider reverting its changes using a new migration instead.
```

Para eliminar de la base de datos las últimas migraciones y volver a una versión específica se puede utilizar `dotnet ef database update` de la siguiente manera 

```
dotnet ef database update InitDB --project ..\MacvDatabase
```

Después de revertir de la base de dato las migraciones ya se pueden eliminar los archivos de las migraciones correspondientes.

```
dotnet ef migrations remove --project ..\MacvDatabase
```

así se mueve la última migración definida en el proyecto (archivo de la última migración).

Si a la hora de agregar una migración se presenta algún error, posiblemente el comando `dotnet build` le ayude porque ejecuta el comando de forma tal que se muestran las operaciones que va realizando e información de los errores. A continuación presento un caso.

![image](./img/Comandos/incoherencia_accesibilidad.png)  


### Por Package Manager


```
Update-Database InitDB
```

Donde `InitDB` sería el nombre de la migración a la cual se quiere actualizar.

### Terminal

```
dotnet ef database update MigracionInicial --project .\MacvCodeFirst\
```

donde MigracionInicial es la migración a la cual queremos saltar. Esto va eliminando todas las migraciones desde la última hasta llegar a la migración que queremos saltar.

NOTAS:
1. En relaciones entre entidades podemos utilizar IEnumerable o ICollection para representar colecciones de datos
2. ICollection permite agregar, eliminar o actualizar elementos mientras que IEnumerable NO PERMITE. por eso se recomienta usar ICollection.

NOTA:
Cuando ejecutas dotnet --version y ves una versión válida, significa que el SDK de .NET está correctamente instalado. Sin embargo, si dotnet ef no funciona, es probable que falte la herramienta de Entity Framework Core CLI (Command Line Interface).

OPCION 1. Instalar dotnet-ef globalmente.
Esto instalará el comando dotnet ef de forma global en tu sistema.

```
dotnet tool install --global dotnet-ef
```

OPCION 2, Agregar dotnet-ef como herramienta local en el proyecto.
Dentro de la carpeta de tu proyecto, puedes ejecutar:  

Antes de instalar dotnet-ef debes tener un archivo de manimiesto en la carpeta .config. Si aún no existe, debes ejecutar el siguiente comando:  

```
dotnet new tool-manifest
```

y luego instalas dotnet-ef

```
dotnet tool install dotnet-ef
```

### Ver la versión de dotnet-ef instalado

```
dotnet ef --version
```

El comando `dotnet new tool-manifest` creo dentro del proyecto (en la ubicación del archivo .csproj) una nueva carpeta llamada .config y dentro creó un archivo llamado `dotnet-tools.json` con el siguiente contenido:  
```json
{
  "version": 1,
  "isRoot": true,
  "tools": {}
}
```

## Desinstalar dotnet-ef
### Si dotnet-ef se instaló de forma global en el equipo

```
dotnet tool uninstall --global dotnet-ef
```

### Si dotnet-ef se instaló localmente en un proyecto.

```
dotnet tool uninstall dotnet-ef
```

Si quieres borrar completamente cualquier configuración local relacionada:

Elimina el archivo dotnet-tools.json si ya no necesitas ninguna otra herramienta local.

Borra la carpeta .store o .dotnet/tools (en el home) si necesitas limpiar caché global/local manualmente.

### Listar herramientas instaladas a nivel global o a nivel de proyecto

```
dotnet tool list --global
dotnet tool list
```