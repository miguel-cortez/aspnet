# Autenticación y autorización

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

***Arriba de var app = builder.Build(); agregue el siguiente bloque:***


```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(option =>
    {
        option.LoginPath = "/Acceso/Index";
        option.ExpireTimeSpan = TimeSpan.FromMinutes(20);
        option.AccessDeniedPath = "/Home/Privacy";
    });
```

***En el siguiente bloque de código, cambie el controlador Home por Acceso***

```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

***El código quedará de la siguiente manera***
```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Acceso}/{action=Index}/{id?}");
```
***Abajo de app.UseRouting(); agregue la siguiente línea***

```csharp
app.UseAuthentication();
```

## Diseñe un formulario de inicio de sesión

```html
@model WebApplication1.Models.InfoLogin
@{
    Layout = null;
}

<!DOCTYPE html>

<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Index</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.2/font/bootstrap-icons.css">
</head>
<body>
    <div class="row mt-5">
        <div class="col-sm-4 offset-sm-4">
            <form asp-controller="Acceso" asp-action="Index" method="post">
                <div class="mb-3">
                    <label class="form-label"><i class="bi bi-person-circle" style="font-size: 2rem; color: cornflowerblue;">&nbsp;Usuario:</i></label>
                    <input type="text" class="form-control" asp-for="Login">
                </div>
                <div class="mb-3">
                    <label class="form-label"><i class="bi bi-key" style="font-size: 2rem; color: cornflowerblue;">&nbsp;Clave</i></label>
                    <input type="password" class="form-control" asp-for="Password">
                </div>
                <button type="submit" class="btn btn-primary">Ingresar</button>
            </form>
        </div>
    </div>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```