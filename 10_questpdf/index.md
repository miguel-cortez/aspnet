# QuestPDF

## Paso 1. Instale QuestPdf

`https://www.questpdf.com/quick-start.html`  

## Paso 2. Cree una función en PruebaController.

Esta función permitirá crear un informe en formato PDF.
(Sección ASP.NET)

**Ejecute la aplcicación par ver el informe de muesta**

## Paso 3. Cree una tabla utilizando QuestPdf

```csharp
namespace WebApplication1.Utilidades
{
    public class InvoiceModel
    {
        public string Item { get; set; }
        public int Quantity { get; set; }
        public decimal Price { get; set; }
    }
}
```

```csharp
using QuestPDF.Fluent;
using QuestPDF.Helpers;
using QuestPDF.Infrastructure;
using QuestPDF.Drawing;
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.IO;
namespace WebApplication1.Utilidades
{
    public class TableDocument : IDocument
    {
        private List<InvoiceModel> _items;

        public TableDocument(List<InvoiceModel> items)
        {
            _items = items;
        }

        public DocumentMetadata GetMetadata() => DocumentMetadata.Default;

        public void Compose(IDocumentContainer container)
        {
            container.Page(page =>
            {
                page.Margin(30);
                page.Content()
                    .Table(table =>
                    {
                        table.ColumnsDefinition(columns =>
                        {
                            columns.RelativeColumn(4); // Item
                            columns.RelativeColumn(2); // Quantity
                            columns.RelativeColumn(2); // Price
                        });

                        // Header
                        table.Header(header =>
                        {
                            header.Cell().Element(CellStyle).Text("Item");
                            header.Cell().Element(CellStyle).Text("Qty");
                            header.Cell().Element(CellStyle).Text("Price");

                            static IContainer CellStyle(IContainer container) =>
                                container.DefaultTextStyle(x => x.SemiBold()).PaddingVertical(5).BorderBottom(1).BorderColor(Colors.Grey.Medium);
                        });

                        // Rows
                        foreach (var item in _items)
                        {
                            table.Cell().Element(CellStyle).Text(item.Item);
                            table.Cell().Element(CellStyle).Text(item.Quantity.ToString());
                            table.Cell().Element(CellStyle).Text($"${item.Price:N2}");

                            static IContainer CellStyle(IContainer container) =>
                                container.PaddingVertical(5);
                        }
                    });
            });
        }
    }
}
```

```csharp
        [HttpGet(Name = "GeneratePdf")]
        public IResult GeneratePdf()
        {
            var data = new List<InvoiceModel>
            {
                new InvoiceModel { Item = "Producto A", Quantity = 2, Price = 19.99m },
                new InvoiceModel { Item = "Producto B", Quantity = 1, Price = 9.99m },
                new InvoiceModel { Item = "Producto C", Quantity = 5, Price = 4.99m }
            };

            var document = new TableDocument(data);
            var pdfStream = document.GeneratePdf();
            return Results.File(pdfStream, "application/pdf", "hello-world.pdf");
        }
```

## Paso 5. Ejercicio

Cree una tabla con la información de los usuarios y roles.

