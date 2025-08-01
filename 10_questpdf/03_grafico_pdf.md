# Creación de gráfico en PDF

```csharp
namespace WebApplication1.Pdf
{
    public class VolumenVentasModel
    {
        public int Id { get; set; }
        public string? Nombre { get; set; }
        public int? Volumen { get; set; }
    }
}
```

```csharp
using QuestPDF.Fluent;
using QuestPDF.Helpers;
using QuestPDF.Infrastructure;

namespace WebApplication1.Pdf
{
    public class VolumenVentasDocument : IDocument
    {
        private VolumenVentasModel Model { get; }
        public VolumenVentasDocument(VolumenVentasModel model)
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
                page.DefaultTextStyle(x => x.FontSize(20));

                page.Header()
                    .Text("Roles asignados")
                    .SemiBold().FontSize(36).FontColor(Colors.Blue.Medium);

                page.Content()
                    .Column(column =>
                    {
                        column.Spacing(20);

                        column.Item().Text("Volumen de vantas");

                        column.Item().Table(table =>
                        {
                            table.ColumnsDefinition(columns =>
                            {
                                columns.RelativeColumn(2); // Id
                                columns.RelativeColumn(2); // Nombre
                            });

                            // Header
                            table.Header(header =>
                            {
                                header.Cell().Element(CellStyle).Text("ID");
                                header.Cell().Element(CellStyle).Text("NOMBRE");

                                static IContainer CellStyle(IContainer container) =>
                                    container.DefaultTextStyle(x => x.SemiBold()).PaddingVertical(5).BorderBottom(1).BorderColor(Colors.Grey.Medium);
                            });

                            // Rows
                            //foreach (var item in Model.Roles)
                            //{
                                table.Cell().Element(CellStyle).Text("dato 1");
                                table.Cell().Element(CellStyle).Text("dato 2");

                                static IContainer CellStyle(IContainer container) =>
                                    container.PaddingVertical(5);
                            //}
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


```chsarp
IF OBJECT_ID(N'Ventas',N'U') IS NULL
CREATE TABLE Ventas(
	Id INT NOT NULL IDENTITY(1,1),
	NumeroComprobante VARCHAR(25) NULL,
	Fecha DATE NOT NULL,
	SubTotal MONEY NOT NULL,
	Iva MONEY NOT NULL,
	Total MONEY NOT NULL,
	ClienteId INT NOT NULL,
	UsuarioId INT NOT NULL,
	CONSTRAINT VentasPk PRIMARY KEY(Id),
	CONSTRAINT VentasFk1 FOREIGN KEY (ClienteId) REFERENCES Clientes(Id),
	CONSTRAINT VentasFk2 FOREIGN KEY (UsuarioId) REFERENCES Usuarios(Id)
)
GO

IF OBJECT_ID(N'DetalleVentas',N'U') IS NULL
CREATE TABLE DetalleVentas(
	Id INT NOT NULL IDENTITY(1,1),
	Cantidad SMALLINT NOT NULL,
	PrecioUnitario MONEY NOT NULL,
	Monto MONEY NOT NULL,
	VentaId INT NOT NULL,
	ProductoId INT NOT NULL,
	CONSTRAINT DetalleVentasPk PRIMARY KEY(Id),
	CONSTRAINT DetalleVentasFk1 FOREIGN KEY (VentaId) REFERENCES Ventas(Id),
	CONSTRAINT DetalleVentasFk2 FOREIGN KEY (ProductoId) REFERENCES Productos(Id)
)
GO
```