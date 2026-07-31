# Inserción de datos fake

## 1. Instalación de Bogus

```bash
Install-Package Bogus
```
o
```bash
dotnet add package Bogus
```

## 2. En la carpeta principal de su proyecto, cree una carpeta llamada `Bogus`

## 3. En la carpeta `Bogus` cree una clase `DbSeeder` 
```cs
using Bogus;
using InventaMeCF.Models;

namespace InventaMeCF.Bogus
{
    public static class DbSeeder
    {
        public static async Task SeedAsync(InventaMeCFContext context)
        {
            // Evita insertar datos si ya existen
            if (context.Productos.Any())
                return;

            var faker = new Faker<Producto>("es")
                .RuleFor(x => x.Nombre, f => f.Commerce.ProductName())
                .RuleFor(x => x.Descripcion, f => f.Commerce.ProductDescription())
                .RuleFor(x => x.PrecioUnitario, f => f.Random.Decimal(10, 500))
                .RuleFor(x => x.MarcaId, f => f.Random.Int(1, 3));

            var productos = faker.Generate(150);

            await context.Productos.AddRangeAsync(productos);
            await context.SaveChangesAsync();
        }
    }
}
```
## 4. Modifique el archivo `Program.cs` para que inserte los datos

:books: Coloque estas línea abajo de `var app = builder.Build();`  

```cs
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<InventaMeCFContext>();

    context.Database.Migrate();

    await DbSeeder.SeedAsync(context);
}
```

## 5. Ejecute la aplicación.


## MAS INFORMACIÓN - PENDIENTE.

```cs
using Bogus;

var usuarioFaker = new Faker<Usuario>()
    .RuleFor(x => x.Nombre, f => f.Name.FullName())
    .RuleFor(x => x.Email, f => f.Internet.Email());

var usuarios = usuarioFaker.Generate(10);
```