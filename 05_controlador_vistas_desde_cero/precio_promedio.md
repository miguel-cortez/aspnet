# Lista de productos en función del precio promedio

## Agregue el siguiente código al final de la vista Index
La vista `Index` que corresponde a `PruebaController`  

```csharp
<hr />

<div class="card" style="width: 18rem;">
    <div class="card-header">
        LISTA DE PRODUCTOS
    </div>
  <div class="card-body">
    <h5 class="card-title">Según promedio de precios</h5>
    <p class="card-text">Utilice esta opción para ver la lista de productos cuyo precio sea menor o mayor que el promedio de precio</p>
    <a asp-action="ListaPrecioMedio" class="btn btn-danger" asp-route-mayores="false">Menores</a>&nbsp;<a asp-action="ListaPrecioMedio" class="btn btn-success" asp-route-mayores="true">Mayores</a>
  </div>
</div>
```

## Agregue la siguiente función a PruebaController

```csharp
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

## Agregue la vista ListaPrecioMedio

```csharp
@model IEnumerable<WebApplication1.Models.Producto>

<h2 class="text-success">Lista de productos @ViewBag.Titulo que el promedio</h2>
<h4 class="text-info">Precio promedio: $ @ViewBag.Promedio</h4>

<table class="table">
    <thead>
        <tr>
            <th scope="col">Cantidad</th>
            <th scope="col">Nombre</th>
            <th scope="col">Precio</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
        <tr>
            <td>@item.Existencia</td>
            <td>@item.Nombre</td>
            <td>@item.Precio</td>
        </tr>
        }
    </tbody>
</table>
```