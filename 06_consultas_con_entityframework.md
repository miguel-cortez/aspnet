# Uso de EntityFramework

Ejemplos:  

```csharp
var productos = _context.Productos.ToListAsync()
```

```csharp
var productos = _context.Productos.Where(a => a.Existencia < 10).ToListAsync();
```

```csharp
var ord = _context.Productos.Include("Detalles").Where(o => o.Nombre == "Producto A").First();
```

```csharp
from od in _context.DetalleOrdenes.AsEnumerable()
    where od.Orden.CustomerID == "ALFKI"
    select new
    {
        od.OrderID,
        od.ProductID,
        od.Quantity,
        od.Discount,
        DetailTotal = (1 - (decimal)od.Discount) *
            (od.Quantity * od.UnitPrice)
    }).ToList();
```