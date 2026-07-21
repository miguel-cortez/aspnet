# CÓDIGO AUXILIAR

## Contenido de la clase Producto

```cs
using Microsoft.EntityFrameworkCore;
using System.ComponentModel;
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;
namespace InventaMeCF.Models
{
    public class Producto
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }
        [Column("Nombre", TypeName = "varchar(80)")]
        [DisplayName("Nombre del producto")]
        [Required(ErrorMessage = "El nombre del producto es requerido.")]
        [StringLength(80, ErrorMessage = "El nombre del producto debe tener una longitud mínima de 3 caracteres y como máximo 80", MinimumLength = 3)]
        public string? Nombre { get; set; }
        [Precision(10, 4)]
        [DisplayName("Precio unitario")]
        public decimal PrecioUnitario { get; set; }

        [Required(ErrorMessage = "La descripción es obligatoria")]
        [Column(TypeName = "varchar(200)")]
        [DisplayName("Descripción")]
        [StringLength(200, MinimumLength = 3)]
        public string? Descripcion { get; set; }
        public int MarcaId { get; set; }

        [ForeignKey("MarcaId")]
        public virtual Marca Marca { get; set; }
    }
}
```

## Obtener marcas para llenar un Select

```cs
            ViewBag.Marcas = await _context.Marcas
                .OrderBy(m => m.Nombre)
                .Select(m => new SelectListItem
                {
                    Value = m.Id.ToString(),
                    Text = m.Nombre
                })
                .ToListAsync();
```

## Vista para crear un producto

```cs
@model InventaMeCF.Models.Producto
<h1>Catálogo de productos</h1>
<form asp-action="Guardar">
    <div>
        <label asp-for="Nombre" class="control-label"></label>
        <input asp-for="Nombre" class="form-control" />
    </div>

    <div>
        <label asp-for="PrecioUnitario" class="control-label"></label>
        <input asp-for="PrecioUnitario" class="form-control" />
    </div>
    <div>
        <label asp-for="Descripcion" class="control-label"></label>
        <input asp-for="Descripcion" class="form-control" />
    </div>
    <div>
    <select asp-for="MarcaId"
        asp-items="ViewBag.Marcas" class="form-select">
            <option value="">Seleccione una marca</option>
    </select>
    </div>

    <input type="submit" value="Guardar" class="btn btn-primary" />
</form>
<a asp-action="Tarjeta" asp-route-id="1">Ver el producto con ID 1</a>
```