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