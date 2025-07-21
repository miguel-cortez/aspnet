# Autenticación y autorización

## Estructura de tablas


```sql
IF OBJECT_ID(N'Usuarios', N'U') IS NULL
CREATE TABLE Usuarios(
	Id BIGINT IDENTITY(1,1) NOT NULL,
	Login CHAR(20) NOT NULL,
	Password VARBINARY(256) NOT NULL
)
GO
IF OBJECT_ID(N'Roles', N'U') IS NULL
CREATE TABLE Roles(
	Id BIGINT IDENTITY(1,1) NOT NULL,
	Nombre VARCHAR(30) NOT NULL
)
GO

IF OBJECT_ID(N'RolesAsignados', N'U') IS NULL
CREATE TABLE RolesAsignados(
	Id BIGINT IDENTITY(1,1) NOT NULL,
	UsuarioId BIGINT NOT NULL,
	RolId BIGINT NOT NULL,
)
GO
```

## Cree un controlador llamado AccesoController

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Mvc;
using System.Data;
using System.Security.Claims;
using WebApplication1.Models;

namespace WebApplication1.Controllers
{
    public class AccesoController : Controller
    {
        private readonly Bd1Context _context;

        public AccesoController(Bd1Context context)
        {
            _context = context;
        }
        public IActionResult Index()
        {
            return View();
        }
        [HttpPost]
        public async Task<IActionResult> Index(InfoLogin infoLogin)
        {
            if (infoLogin != null)
            {
                //string sql = String.Format("select Id, Login, Password from Usuarios a where Login='{0}' and convert(varchar(256), DecryptByPassPhrase('osni',a.Password)) = '{1}'", infoLogin.Login, infoLogin.Password);
                //Usuario? usuario = _context.Usuarios.FromSqlRaw(sql).FirstOrDefault<Usuario>();
                string? usuario = null;
                if (usuario != null)
                {
                    var claims = new List<Claim> {
                        new Claim(ClaimTypes.Name,"miguel"), // usuario.Login
                        new Claim("Otro","otro dato")
                    };
                    /*
                    List<Role> lista = (from rls in _context.Roles
                                        join rlsa in _context.RolesAsignados
                                        on rls.Id equals rlsa.IdRol
                                        where rlsa.IdUsuario == usuario.Id
                                        select rls).ToList();
                    */
                    List<string> lista = new List<string>();
                    foreach (string rol in lista)
                    {
                        claims.Add(new Claim(ClaimTypes.Role, "administrador")); // rol.Nombre
                    }
                    var claimsIdentity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);

                    await HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme, new ClaimsPrincipal(claimsIdentity));

                    return RedirectToAction("Index", "Home");
                }
                else
                {
                    return View();
                }
            }
            else
            {
                return View();
            }

        }
        public async Task<IActionResult> Salir()
        {
            await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            return RedirectToAction("Index", "Acceso");
        }
    }
}
```

## Vamos a modificar el archivo Program.cs

## Arriba de var app = builder.Build(); agregue el siguiente bloque

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(option =>
    {
        option.LoginPath = "/Acceso/Index";
        option.ExpireTimeSpan = TimeSpan.FromMinutes(20);
        option.AccessDeniedPath = "/Home/Privacy";
    });
```

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(option =>
    {
        option.LoginPath = "/Acceso/Index";
        option.ExpireTimeSpan = TimeSpan.FromMinutes(20);
        option.AccessDeniedPath = "/Home/Privacy";
    });
```