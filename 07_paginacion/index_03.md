# Paginación Versión 3 - Múltiples parámetros

:books: Este ejemplo está desarrollado para paginar los resultados de productos que han sido filtrados mediante un combo de selección de marca y una cadena de búsqueda. Algo interesante de este ejemplo es que ofrece la posibilidad de definir múltiples parámetros en la función del controlador y se envían en el paginador. Internamente en la plantilla de Razor (para paginación) se utiliza `Url.Action(Model.Accion, Model.Controlador, ruta);` para generar cada link.  

## Clase `PaginacionV3`

Esta clase hereda las características y comportamientos de la clase `Paginacion` 

```cs
namespace InventaMeCF.Utilidades
{
    public class PaginacionV3:Paginacion
    {
        public Dictionary<string, string> Parametros { get; set; } = new Dictionary<string, string>();
        public PaginacionV3() { }
        public PaginacionV3(int totalRegistros, int pagina, int registrosPagina = 10, string controlador = "Home", string accion = "Index"):base(totalRegistros, pagina, registrosPagina, controlador, accion)
        {

        }
    }
}
```

## Función `Index` de `ProductoController`  

```cs
using InventaMeCF.Models;
using InventaMeCF.Utilidades;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.EntityFrameworkCore;
namespace InventaMeCF.Controllers
{
    public class ProductoController : Controller
    {
        private readonly InventaMeCFContext _context;
        public ProductoController(InventaMeCFContext context)
        {
            _context = context;
        }
        public async Task<IActionResult> Index(int pg = 1,int? productoMarca = null, string? cadenaBusqueda= null)
        {
            if (_context.Productos == null)
            {
                return Problem("El conjunto 'InventaMeCFContext.Productos' está vacío.");
            }


            IQueryable<Marca> marcaQuery = from m in _context.Marcas select m;

            var productos = from m in _context.Productos
                            select m;
            if (!string.IsNullOrEmpty(cadenaBusqueda))
            {
                productos = productos.Where(s => s.Nombre!.ToUpper().Contains(cadenaBusqueda.ToUpper()));
            }

            if (productoMarca != 0)
            {
                productos = productos.Where(x => x.Marca!.Id == productoMarca);
            }

            var lista = await productos.ToListAsync();
            // Inicio paginación.

            var paginacion = new PaginacionV3(lista.Count, pg,20, "Producto");
            paginacion.Parametros.Add("pg", "1"); // El valor 1 no tiene relevancia porque se modifica en la vista (en la plantilla para paginación) de Razor.
            paginacion.Parametros.Add("productoMarca", productoMarca?.ToString() ?? "10");
            paginacion.Parametros.Add("cadenaBusqueda", cadenaBusqueda ?? "");

            var data = lista.Skip(paginacion.Salto).Take(paginacion.RegistrosPagina).ToList();
            this.ViewBag.Paginacion = paginacion;
            // fin paginación.

            var productoMarcaVM = new ProductoMarcaViewModel
            {
                Marcas = new SelectList(marcaQuery, "Id", "Nombre"),
                Productos = data
            };
            return View(productoMarcaVM);
        }
    }
}
```

:high_brightness: Una ventaja a destacar es que se pueden crear los parámetros necesarios sin crear una propiedad adicional en la clase de `PaginacionV3`, solo basta con el diccionario que se agregó. En cada controlador se pueden agregar parámetros diferentes y se modifica la firma del método para que se adapte a las necesidades.   


## Plantilla `_PaginacionV3.cshtml`  

```cs
@model InventaMeCF.Utilidades.PaginacionV3;

<div class="container">
    @{
        var ruta = new Dictionary<string, string>();

        foreach (var item in Model.Parametros)
        {
            ruta[item.Key] = item.Value;
        }
        string? GenerarUrl(int pagina)
        {
            ruta["pg"] = pagina.ToString();

            return Url.Action(
                Model.Accion,
                Model.Controlador,
                ruta
            );
        }
    }
    @if (Model.TotalPaginas > 0)
    {
        <ul class="pagination justify-content-end">
            @if (Model.PaginaActual > 1)
            {
                <li class="page-item">
                    <a class="page-link" href="@GenerarUrl(1)">Primera</a>
                </li>
                <li>
                    <a class="page-link" href="@GenerarUrl(Model.PaginaActual - 1)">Anterior</a>
                </li>
            }
            @for (var pge = Model.PaginaInicio; pge <= Model.UltimaPagina; pge++)
            {   
                  <li class="page-item @(pge == Model.PaginaActual ? "active" : "")">
                    <a class="page-link" href="@GenerarUrl(pge)">@pge</a>
                </li>
            }
            @if (Model.PaginaActual < Model.TotalPaginas)
            {
                <li class="page-item">
                    <a class="page-link" href="@GenerarUrl(Model.PaginaActual + 1)">Siguiente</a>
                </li>
                <li>
                    <a class="page-link" href="@GenerarUrl(Model.TotalPaginas)">Última</a>
                </li>
            }
        </ul>
    }
</div>
```

## Vista `Index` de `ProductoController`  

```cs
@model InventaMeCF.Models.ProductoMarcaViewModel
@using InventaMeCF.Utilidades;
@{
    ViewData["Title"] = "Index";
    // A PARTIR DE AQUÍ SE AGREGARON ESTAS LÍNEAS
    PaginacionV3 paginacion = new PaginacionV3();
    int paginaActual = 0;
    if (ViewBag.Paginacion != null)
    {
        paginacion = ViewBag.Paginacion;
        paginaActual = paginacion.PaginaActual;
    }
    // HASTA AQUÍ
}

<h1>Index</h1>

<p>
    <a asp-action="Crear">Crear Nuevo</a>
</p>
<form asp-controller="Producto" asp-action="Index" method="get">
    <p>
        <select asp-for="ProductoMarca" asp-items="Model.Marcas">
            <option value="0">Todas</option>
        </select>

        <label>Nombre: <input type="text" asp-for="CadenaBusqueda" /></label>
        <input type="submit" value="Filtro" />
    </p>
</form>

<table class="table table-hover">
    <thead>
        <tr>
            <th>
                @Html.DisplayNameFor(model => model.Productos![0].Nombre)
            </th>
            <th>
                @Html.DisplayNameFor(model => model.Productos![0].Descripcion)
            </th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model.Productos!)
        {
            <tr>
                <td>
                    @Html.DisplayFor(modelItem => item.Nombre)
                </td>
                <td>
                    @Html.DisplayFor(modelItem => item.Descripcion)
                </td>
                <td>
                    <a asp-action="Editar" asp-route-id="@item.Id" class="btn btn-success btn-sm">Editar</a> |
                    <a asp-action="Eliminar" asp-route-id="@item.Id" class="btn btn-danger btn-sm">Eliminar</a>
                </td>
            </tr>
        }
    </tbody>
</table>

<partial name="_paginacionv3" model="@paginacion" />

 // ✂ CÓDIGO OMITIDO

```