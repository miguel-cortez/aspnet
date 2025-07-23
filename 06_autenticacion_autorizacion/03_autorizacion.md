# AUTORIZACIÓN

## Estructura de las tablas

:warning: Estas son las tablas que se usarán para registrar usuarios y roles. La estructura ya se publicó anteriormente, por lo tanto, ya debe tener creadas las tablas necesarias.  


```sql
IF OBJECT_ID(N'Usuarios', N'U') IS NULL
CREATE TABLE Usuarios(
	Id INT IDENTITY(1,1) NOT NULL,
	Login CHAR(20) NOT NULL,
	Password VARCHAR(64) NOT NULL
)
GO
IF OBJECT_ID(N'Roles', N'U') IS NULL
CREATE TABLE Roles(
	Id INT IDENTITY(1,1) NOT NULL,
	Nombre VARCHAR(30) NOT NULL
)
GO

IF OBJECT_ID(N'RolesAsignados', N'U') IS NULL
CREATE TABLE RolesAsignados(
	Id INT IDENTITY(1,1) NOT NULL,
	UsuarioId INT NOT NULL,
	RolId INT NOT NULL,
)
GO

ALTER TABLE Usuarios
	ADD CONSTRAINT PrimaryKeyUsuarios PRIMARY KEY (Id);
GO

ALTER TABLE Roles
	ADD CONSTRAINT PrimaryKeyRoles PRIMARY KEY (Id);
GO

ALTER TABLE RolesAsignados
	ADD CONSTRAINT ForeignKeyRolesAsignadosIdRol
	FOREIGN KEY (RolId) REFERENCES Roles(Id);
GO

ALTER TABLE RolesAsignados
	ADD CONSTRAINT ForeignKeyRolesAsignadosIdUsuario
	FOREIGN KEY (UsuarioId) REFERENCES Usuarios(Id);
GO

```

## Agregue roles

```sql
insert into Roles(Nombre) values('Administrador')
go
insert into Roles(Nombre) values('Estándar')
go
```

## Asigne el Rol Administrador al Usuario con Id 1

```sql
insert into RolesAsignados(UsuarioId,RolId) values(1,1)
go
```

## Descomente el siguiente segmento de código en AccesoController

```csharp
/*
List<Role> lista = (from rls in _context.Roles
    join rlsa in _context.RolesAsignados
    on rls.Id equals rlsa.IdRol
    where rlsa.IdUsuario == usuario.Id
    select rls).ToList();
*/
```
:boom: **Corrija los errores**. Se van a generar unos errores por convenciones en nombres, por ejemplo, tendrá que cambiar `IdRol` por `RolId` 

<details>
<summary>Solución</summary>
<pre>
    List&lt;Role&gt;  lista = (from rls in _context.Roles
    join rlsa in _context.RolesAsignados
    on rls.Id equals rlsa.RolId
    where rlsa.UsuarioId == usuario.Id
    select rls).ToList();
</pre>
</details>

## Realice los cambios indicados en AccesoController.

:books: Inicialmente, se simuló disponer del rol `Administrador` para evitar errores a la hora de ejecutar la aplicación; pero ahora ya tenemos roles en la base de datos y podemos acceder a ellos desde `AccesoController` por lo tanto vamos a cambiar el bloque de código siguiente:  

```csharp
List<string> lista = new List<string>();
foreach (string rol in lista)
{
    claims.Add(new Claim(ClaimTypes.Role, "administrador")); // rol.Nombre
}
```

El segmento de código anterior quedará de la siguiente manera:  

```csharp
foreach (Role rol in lista)
{
    claims.Add(new Claim(ClaimTypes.Role, rol.Nombre));
}
```

:x: Se eliminó la línea `List<string> lista = new List<string>();` porque ahora la variable `lista` tiene los roles asignados al usuario.  

## Proteja la función Index de PruebaController

Con fines demostrativos, vamos restringir la función `ListaPrecioMedio` de `PruebaController` para que solo los usuarios con rol `Administrador` o `Estándar` puedan utilizar dicha función.   

Lo que tiene que hacer es agregar la anotación `[Authorize(Roles = "Administrador, Estandar")]` arriba del nombre de la función.  En la parte superior de la clase debe tener la importación del espacio de nombres `using Microsoft.AspNetCore.Authorization;`  

```csharp
[Authorize(Roles = "Administrador, Estandar")]
public async Task<IActionResult> ListaPrecioMedio(bool mayores)
{
    if (_context.Productos == null)
    {
        return NotFound();
    }
    var precio_medio = await _context.Productos.AverageAsync(a => a.Precio);
    var productos = (mayores) ? await _context.Productos.Where(a => a.Precio > precio_medio).ToListAsync(): await _context.Productos.Where(a => a.Precio < precio_medio).ToListAsync();
    ViewBag.Promedio = precio_medio;
    ViewBag.Titulo = (mayores) ? "mayores" : "menores";
    return View(productos);
}
```

## Visualice rutas del menú de forma condicional en función de los roles de usuario autenticado.  

Agregue la línea `@using Microsoft.AspNetCore.Identity` como primera instrucción de la plantilla `_Layout.cshtml`  

Luego, para ver una opción de menú de forma condicional, puede usar el siguiente ejemplo como referencia:  

```chsarp
@if (User.IsInRole("Estándar") || User.IsInRole("Supervisor"))
{
    <li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Prueba" asp-action="Index">Prueba</a>
    </li>
}
```

## ACTIVIDAD

:one: Agregue un nuevo usuario.  

:two: Asigne el rol `Estándar` al nuevo usuario.  

:three: Se pide que tenga una opción de menú disponible para todos los usuarios, excepto para el usuario que tenga el rol `Estándar` 

:four: Además, de no disponer de la opción en el menú, tampoco debe poder ingresar escribiendo la URL en la barra de direcciones.  

