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