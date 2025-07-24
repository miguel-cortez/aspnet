# Paginación

```csharp
namespace WebApplication1.Utilidades
{
    public class Paginacion
    {
        public int TotalRegistros { get; private set; }
        public int PaginaActual { get; private set; }
        public int RegistrosPagina { get; private set; }
        public int TotalPaginas { get; private set; }
        public int PaginaInicio { get; private set; }
        public int UltimaPagina { get; private set; }
        public string Controlador { get; private set; }
        public string Accion { get; private set; }
        public int Salto { get; private set; }
        public Paginacion(int totalRegistros, int pagina, int registrosPagina = 10, string controlador = "Home", string accion = "Index")
        {
            int totalPaginas = (int)Math.Ceiling((decimal)totalRegistros / (decimal)registrosPagina);
            int paginaActual = (pagina < 1) ? 1 : pagina;
            int paginaInicio = paginaActual - 5;
            int ultimaPagina = paginaActual + 4;
            int salto = (paginaActual - 1) * registrosPagina;
            if (paginaInicio <= 0)
            {
                ultimaPagina = ultimaPagina - (paginaInicio - 1);
                paginaInicio = 1;
            }
            if (ultimaPagina > totalPaginas)
            {
                ultimaPagina = totalPaginas;
                if (ultimaPagina > 10)
                {
                    paginaInicio = ultimaPagina - 9;
                }
            }
            TotalRegistros = totalRegistros;
            PaginaActual = paginaActual;
            RegistrosPagina = registrosPagina;
            TotalPaginas = totalPaginas;
            PaginaInicio = paginaInicio;
            UltimaPagina = ultimaPagina;
            Controlador = controlador;
            Accion = accion;
            Salto = salto;
        }
    }
}
```

## Cree una plantilla de Razor llamada _Paginacion.cshtml en la carpeta Shared

```html
@model WebApplication1.Utilidades.Paginacion;

<div class="container">
    @if (Model.TotalPaginas > 0)
    {
        <ul class="pagination justify-content-end">
            @if (Model.PaginaActual > 1)
            {
                <li class="page-item">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="1">Primera</a>
                </li>
                <li>
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.PaginaActual - 1)">Anterior</a>
                </li>
            }
            @for (var pge = Model.PaginaInicio; pge <= Model.UltimaPagina; pge++)
            {
                <li class="page-item @(pge == Model.PaginaActual ? "active" : "")">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@pge">@pge</a>
                </li>
            }
            @if (Model.PaginaActual < Model.TotalPaginas)
            {
                <li class="page-item">
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.PaginaActual + 1)">Siguiente</a>
                </li>
                <li>
                    <a class="page-link" asp-controller="@(Model.Controlador)" asp-action="@(Model.Accion)" asp-route-pg="@(Model.TotalPaginas)">Última</a>
                </li>
            }
        </ul>
    }
</div>
```