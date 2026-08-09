# Paginación Versión 2 - Dos parámetros

:books: Este ejemplo de una variante del proceso de paginación original, pero adaptado para que utilice dos propiedades adicionales a fin de que funcione aún cuando una vista tiene un formulario para realizar filtros de información.  

## Clase `PaginacionV2`

Esta clase hereda las características y comportamientos de la clase `Paginacion` 

```cs
namespace InventaMeCF.Utilidades
{
    public class PaginacionV2:Paginacion
    {
        public int? ProductoMarca { get; set; }
        public string? CadenaBusqueda { get; set; }
        public PaginacionV2() { }
        public PaginacionV2(int totalRegistros, int pagina, int registrosPagina = 10, string controlador = "Home", string accion = "Index"):base(totalRegistros, pagina, registrosPagina, controlador, accion)
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
    [Authorize(Roles = "administrador")]
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

            var paginacion = new PaginacionV2(lista.Count, pg,20, "Producto");
            
            paginacion.ProductoMarca = productoMarca;
            paginacion.CadenaBusqueda = cadenaBusqueda;
            
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


## Plantilla `_PaginacionV2.cshtml`  

```cs
@model InventaMeCF.Utilidades.PaginacionV2;

<div class="container">
    @if (Model.TotalPaginas > 0)
    {
        <ul class="pagination justify-content-end">
            @if (Model.PaginaActual > 1)
            {
                <li class="page-item">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="1" asp-route-productoMarca="@Model.ProductoMarca" asp-route-cadenaBusqueda="@Model.CadenaBusqueda">Primera</a>
                </li>
                <li>
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.PaginaActual - 1)" asp-route-productoMarca="@Model.ProductoMarca" asp-route-cadenaBusqueda="@Model.CadenaBusqueda">Anterior</a>
                </li>
            }
            @for (var pge = Model.PaginaInicio; pge <= Model.UltimaPagina; pge++)
            {   
                <li class="page-item @(pge == Model.PaginaActual ? "active" : "")">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@pge" asp-route-productoMarca="@Model.ProductoMarca" asp-route-cadenaBusqueda="@Model.CadenaBusqueda">@pge</a>
                </li>
            }
            @if (Model.PaginaActual < Model.TotalPaginas)
            {
                <li class="page-item">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.PaginaActual + 1)" asp-route-productoMarca="@Model.ProductoMarca" asp-route-cadenaBusqueda="@Model.CadenaBusqueda">Siguiente</a>
                </li>
                <li>
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.TotalPaginas)" asp-route-productoMarca="@Model.ProductoMarca" asp-route-cadenaBusqueda="@Model.CadenaBusqueda">Última</a>
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
    PaginacionV2 paginacion = new PaginacionV2();
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

<partial name="_paginacionv2" model="@paginacion" />

 // ✂ CÓDIGO OMITIDO

```