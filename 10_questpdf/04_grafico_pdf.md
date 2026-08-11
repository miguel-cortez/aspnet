# Creación de gráfico en PDF (con migraciones)


## 1. Agregue tres clases: Cliente, Venta, DetalleVenta

### Clase `Cliente` 
```cs
    public class Cliente
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Column("Nombre", TypeName = "varchar(80)")]
        public string? Nombre { get; set; }

        public virtual ICollection<Venta> Ventas { get; set; }
    }
```

### Clase `Venta` 
```cs
    public class Venta
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Column("NumeroComprobante", TypeName = "varchar(25)")]
        public string? NumeroComprobante { get; set; }
        
        public DateTime ? Fecha { get; set; }

        [Precision(10, 4)]
        public decimal SubTotal { get; set; }

        [Precision(10, 4)]
        public decimal Iva { get; set; }

        [Precision(10, 4)]
        public decimal Total { get; set; }

        public int ClienteId { get; set; }

        [ForeignKey("ClienteId")]
        public virtual Cliente? Cliente { get; set; }

        public int UsuarioId { get; set; }

        [ForeignKey("UsuarioId")]
        public virtual Usuario? Usuario { get; set; }
    }
```

### Clase `DetalleVenta` 
```cs
    public class DetalleVenta
    {
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }


        public int Cantidad { get; set; }

        [Precision(10, 4)]
        public decimal PrecioUnitario { get; set; }

        [Precision(10, 4)]
        public decimal Monto { get; set; }

        public int VentaId { get; set; }

        [ForeignKey("VentaId")]
        public virtual Venta? Venta { get; set; }

        public int ProductoId { get; set; }

        [ForeignKey("ProductoId")]
        public virtual Producto? Producto { get; set; }
    }
```

## 2. Modifique la clase `Usuario` y la clase `Producto` 

:books: La clase usuario va a permitir navegar en la colección de ventas registradas.

```cs
    public class Usuario
    {
        // ✂️ código omitido
        public virtual ICollection<Venta> Ventas { get; set; } // Línea agregada.
    }
```

```cs
    public class Producto
    {
        // ✂️ código omitido
        public virtual ICollection<DetalleVenta> DetalleVentas { get; set; } // Línea agregada
    }
```

## 3. Modifique la clase de contexto

```cs
    public class InventaMeCFContext:DbContext
    {
        public InventaMeCFContext(DbContextOptions<InventaMeCFContext> options) : base(options)
        {

        }
        // ✂️ código omitido

        public DbSet<Cliente> Clientes { get; set; } // Línea agregada
        public DbSet<Venta> Ventas { get; set; } // Línea agregada
        public DbSet<DetalleVenta> DetalleVentas { get; set; } // Línea agregada

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // ✂️ código omitido
        }
    }
```

## 4. Cree una nueva migración

```bash
Add-Migration AddTablesVentas
```

## 5. Actualice la base de datos

```bash
Update-Database
```

## 6. Agregue datos de 5 ventas

:books: Para ello será necesario que registre Clientes (por lo menos 3), registre detalles de ventas y ventas.

### Agregue clientes
![image](./img2/lista_clientes.png)  

### Consulte la lista de productos para ver `ID` y `PrecioUnitario`

![image](./img2/lista_productos.png)  

### Registre Ventas
:books: Los campos SubTotal, Iva y Total pueden ser modificados posteriormente, cuando ya haya registrados los detalles de ventas porque sus valores dependen de los detalles.  
![image](./img2/ventas.png)  

### Registre detalle de ventas
![image](./img2/detalle_ventas.png)  

<details>
<summary>Ejemplos de inserciones</summary>
<pre>
USE InventaMeCF
GO
SET IDENTITY_INSERT Clientes ON 
GO
INSERT Clientes (Id, Nombre) VALUES (1, N'Miguel')
GO
INSERT Clientes (Id, Nombre) VALUES (2, N'Juan')
GO
INSERT Clientes (Id, Nombre) VALUES (3, N'Andrés')
GO
SET IDENTITY_INSERT Clientes OFF
GO
SET IDENTITY_INSERT Ventas ON 
GO
INSERT Ventas ( Id,  NumeroComprobante,  Fecha,  SubTotal,  Iva,  Total,  ClienteId,  UsuarioId) VALUES (1, N'1415', CAST(N'2026-08-10T00:00:00.0000000' AS DateTime2), CAST(525.2814 AS Decimal(10, 4)), CAST(68.2866 AS Decimal(10, 4)), CAST(593.5680 AS Decimal(10, 4)), 1, 1)
GO
INSERT Ventas ( Id,  NumeroComprobante,  Fecha,  SubTotal,  Iva,  Total,  ClienteId,  UsuarioId) VALUES (2, N'1617', CAST(N'2026-08-10T00:00:00.0000000' AS DateTime2), CAST(116.5410 AS Decimal(10, 4)), CAST(15.1503 AS Decimal(10, 4)), CAST(131.6913 AS Decimal(10, 4)), 1, 1)
GO
INSERT Ventas ( Id,  NumeroComprobante,  Fecha,  SubTotal,  Iva,  Total,  ClienteId,  UsuarioId) VALUES (3, N'1819', CAST(N'2026-08-10T00:00:00.0000000' AS DateTime2), CAST(19.7433 AS Decimal(10, 4)), CAST(2.5667 AS Decimal(10, 4)), CAST(22.3100 AS Decimal(10, 4)), 3, 1)
GO
INSERT Ventas ( Id,  NumeroComprobante,  Fecha,  SubTotal,  Iva,  Total,  ClienteId,  UsuarioId) VALUES (4, N'2021', CAST(N'2026-08-10T00:00:00.0000000' AS DateTime2), CAST(156.9582 AS Decimal(10, 4)), CAST(20.4046 AS Decimal(10, 4)), CAST(177.3628 AS Decimal(10, 4)), 1, 1)
GO
INSERT Ventas ( Id,  NumeroComprobante,  Fecha,  SubTotal,  Iva,  Total,  ClienteId,  UsuarioId) VALUES (5, N'2223', CAST(N'2026-08-10T00:00:00.0000000' AS DateTime2), CAST(44.8305 AS Decimal(10, 4)), CAST(5.8280 AS Decimal(10, 4)), CAST(50.6585 AS Decimal(10, 4)), 2, 1)
GO
SET IDENTITY_INSERT Ventas OFF
GO
SET IDENTITY_INSERT DetalleVentas ON 
GO
INSERT DetalleVentas ( Id,  Cantidad,  PrecioUnitario,  Monto,  VentaId,  ProductoId) VALUES (1, 2, CAST(184.1616 AS Decimal(10, 4)), CAST(368.3232 AS Decimal(10, 4)), 1, 304)
GO
INSERT DetalleVentas ( Id,  Cantidad,  PrecioUnitario,  Monto,  VentaId,  ProductoId) VALUES (2, 5, CAST(23.3082 AS Decimal(10, 4)), CAST(116.5410 AS Decimal(10, 4)), 2, 327)
GO
INSERT DetalleVentas ( Id,  Cantidad,  PrecioUnitario,  Monto,  VentaId,  ProductoId) VALUES (3, 1, CAST(19.7433 AS Decimal(10, 4)), CAST(19.7433 AS Decimal(10, 4)), 3, 353)
GO
INSERT DetalleVentas ( Id,  Cantidad,  PrecioUnitario,  Monto,  VentaId,  ProductoId) VALUES (4, 6, CAST(26.1597 AS Decimal(10, 4)), CAST(156.9582 AS Decimal(10, 4)), 1, 427)
GO
INSERT DetalleVentas ( Id,  Cantidad,  PrecioUnitario,  Monto,  VentaId,  ProductoId) VALUES (5, 1, CAST(44.8305 AS Decimal(10, 4)), CAST(44.8305 AS Decimal(10, 4)), 4, 450)
GO
SET IDENTITY_INSERT DetalleVentas OFF
GO
</pre>
</details>

## 7. En la carpeta `Pdf` agregue dos clases: `VolumenVentasModel` y `VolumenVentasDocument` 

### Clase `VolumenVentasModel`  
```cs
    public class VolumenVentasModel
    {
        public int Id { get; set; }
        public string? Nombre { get; set; }
        public int? Volumen { get; set; }
    }
```

### Clase `VolumenVentasDocument`  
```cs
using QuestPDF.Fluent;
using QuestPDF.Helpers;
using QuestPDF.Infrastructure;
using ScottPlot;
using ScottPlot.Plottables;
using Colors = QuestPDF.Helpers.Colors;
namespace InventaMeCF.Pdf
{
    public class VolumenVentasDocument : IDocument
    {
        private List<VolumenVentasModel> Model { get; }
        public VolumenVentasDocument(List<VolumenVentasModel> model)
        {
            Model = model;
        }
        public void Compose(IDocumentContainer container)
        {
            container.Page(page =>
            {
                page.Size(PageSizes.A4);
                page.Margin(2, Unit.Centimetre);
                page.PageColor(Colors.White);
                page.DefaultTextStyle(x => x.FontSize(14));

                page.Header()
                    .AlignCenter()
                    .Column(column =>
                    {
                        column.Item()
                            .Text("VOLUMEN DE VENTAS")
                            .SemiBold()
                            .FontSize(24)
                            .FontColor(Colors.Blue.Medium);

                        column.Item()
                            .Text("Top 3 productos con mayor volumen de ventas")
                            .FontSize(12)
                            .FontColor(Colors.Grey.Medium);
                    });

                page.Content()
                    .Column(column =>
                    {
                        column.Spacing(20);

                        column.Item().Column(column =>
                        {
                            column.Spacing(10);

                            column.Item()
                                .AspectRatio(1)
                                .Svg(size =>
                                {
                                    ScottPlot.Plot plot = new();
                                    PieSlice[] slices = new PieSlice[3];
                                    int i = 0;
                                    ScottPlot.Color[] cl = new ScottPlot.Color[] { new ScottPlot.Color(Colors.Yellow.Medium.Hex), new ScottPlot.Color(Colors.Green.Medium.Hex), new ScottPlot.Color(Colors.Blue.Medium.Hex) };
                                    foreach (var item in Model)
                                    {
                                        slices[i] = new() { Value = (double)item.Volumen, FillColor = cl[i], Label = item.Nombre };
                                        i++;
                                    }

                                    var pie = plot.Add.Pie(slices);
                                    pie.DonutFraction = 0.5;
                                    pie.SliceLabelDistance = 1.5;
                                    pie.LineColor = ScottPlot.Colors.White;
                                    pie.LineWidth = 3;

                                    foreach (var pieSlice in pie.Slices)
                                    {
                                        pieSlice.LabelStyle.FontName = "Lato";
                                        pieSlice.LabelStyle.FontSize = 16;
                                    }

                                    plot.Axes.Frameless();
                                    plot.HideGrid();

                                    return plot.GetSvgXml((int)size.Width, (int)size.Height);
                                });
                        });
                        column.Item().Table(table =>
                        {
                            table.ColumnsDefinition(columns =>
                            {
                                columns.RelativeColumn(2); // Id
                                columns.RelativeColumn(2); // Nombre
                                columns.RelativeColumn(2); // Volumen
                            });
                            // Header
                            table.Header(header =>
                            {
                                header.Cell().Element(CellStyle).Text("ID");
                                header.Cell().Element(CellStyle).Text("NOMBRE");
                                header.Cell().Element(CellStyle).Text("VOLUMEN").AlignCenter();
                                static IContainer CellStyle(IContainer container) =>
                                    container
                                    .Background(Colors.Blue.Medium.Hex)
                                    .DefaultTextStyle(x => x
                                        .SemiBold()
                                        .FontColor(Colors.White)
                                        .FontFamily("Arial")
                                        .FontSize(12)
                                     )
                                    .PaddingVertical(5)
                                    .BorderBottom(1)
                                    .BorderColor(Colors.White);
                            });

                            // Rows
                            foreach (var item in Model)
                            {
                                table.Cell().Element(CellStyle).Text(item.Id.ToString());
                                table.Cell().Element(CellStyle).Text(item.Nombre);
                                table.Cell().Element(CellStyle).Text(item.Volumen.ToString());
                                static IContainer CellStyle(IContainer container) =>
                                        container.PaddingVertical(5);
                            }
                        });
                    });

                page.Footer()
                    .AlignCenter()
                    .Text(x =>
                    {
                        x.Span("Página ");
                        x.CurrentPageNumber();
                    });

            });
        }
    }
}
```

## 8. Agregue una nueva función a `ProductoController` 

```cs
        [HttpGet(Name = "GraficoVolumenVentasPdf")]
        public async Task<IResult> GraficoVolumenVentasPdf(int n)
        {
            List <VolumenVentasModel> data = await _context.Productos
                .Select(p => new VolumenVentasModel
                {
                    Id = p.Id,
                    Nombre = p.Nombre,
                    Volumen = p.DetalleVentas.Sum(d => d.Cantidad)
                })
                .OrderByDescending(x => x.Volumen)
                .Take(n)
                .ToListAsync();
            var document = new VolumenVentasDocument(data);
            var pdfStream = document.GeneratePdf();
            return Results.File(pdfStream, "application/pdf", "volumen_ventas.pdf");
        }
```

## 9. Agregue un link para generar el archivo PDF con el gráfico.

:books: El link ha sido agregado en la vista `Index` de `ProductoController` 

```bash
<a asp-controller="Producto" asp-action="GraficoVolumenVentasPdf" asp-route-n="3">Volumen de ventas</a>
```