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