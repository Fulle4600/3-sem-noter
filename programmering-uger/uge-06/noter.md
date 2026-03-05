# Uge 6 — Repository Pattern & Unit Tests

Denne uge introducerer to centrale begreber i professionel softwareudvikling: Repository Pattern og Unit Testing med xUnit. Vi lærer at strukturere vores kode med klart adskilte lag (model, repository, test) og at skrive automatiserede tests, der verificerer vores kodes korrekthed. Disse teknikker er grundlaget for vedligeholdbar og testbar kode i erhvervslivet.

---

## Model-klassen

En **model-klasse** (også kaldet en domæneklasse eller entity) repræsenterer et objekt fra den virkelige verden i vores program. Den indeholder data (properties) og eventuelt simpel logik.

### Principper for en god model-klasse

- Kun data og enkle beregninger — ingen databaselogik, ingen UI-logik
- Properties med `get` og `set`
- Konstruktorer til initialisering
- `override ToString()` til debugging

### Eksempel: `Product`-klassen

```csharp
namespace WebshopApp.Models
{
    public class Product
    {
        // Auto-property med Nullable Reference Type
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public string? Description { get; set; }  // Nullable — kan være null
        public decimal Price { get; set; }
        public int StockQuantity { get; set; }
        public DateTime CreatedAt { get; set; }

        // Standard konstruktor (krævet af f.eks. EF Core)
        public Product() { }

        // Convenience-konstruktor
        public Product(string name, decimal price, int stock)
        {
            Name = name;
            Price = price;
            StockQuantity = stock;
            CreatedAt = DateTime.Now;
        }

        // Nyttigt til debugging
        public override string ToString()
        {
            return $"Product [Id={Id}, Name={Name}, Price={Price:C}, Stock={StockQuantity}]";
        }
    }
}
```

---

## Nullable Reference Types (NRT)

Fra C# 8.0 kan man aktivere **Nullable Reference Types** i sit projekt. Det betyder at kompilatoren advarer dig, hvis du forsøger at bruge en reference-type variabel, der potentielt er `null`, uden at tjekke det først.

### Aktivering i `.csproj`

```xml
<PropertyGroup>
  <Nullable>enable</Nullable>
</PropertyGroup>
```

### Syntaks

| Syntaks       | Betydning                                     |
|---------------|-----------------------------------------------|
| `string name` | Kan IKKE være null — kompilatoren advarer     |
| `string? name`| KAN være null — du skal tjekke før brug       |
| `name!`       | Null-forgiving operator — du siger "jeg ved den ikke er null" |

### Eksempel med null-tjek

```csharp
public void PrintDescription(Product product)
{
    // product.Description er string? — vi skal tjekke
    if (product.Description != null)
    {
        Console.WriteLine(product.Description);
    }

    // Eller med null-conditional operator:
    Console.WriteLine(product.Description?.ToUpper() ?? "Ingen beskrivelse");
}
```

---

## Repository Pattern

Et **Repository** er en klasse, der håndterer al datahåndtering for en bestemt model. I stedet for at sprede databasekald rundt i hele applikationen, samler vi det ét sted.

### Fordele ved Repository Pattern

1. **Separation of Concerns** — model ved ikke noget om lagring
2. **Testbarhed** — vi kan nemt udskifte en in-memory repository med en database-repository
3. **Vedligeholdbarhed** — ændringer i datahåndtering sker kun ét sted
4. **DRY (Don't Repeat Yourself)** — ingen gentaget kode til CRUD

### In-Memory Repository eksempel

```csharp
using System.Collections.Generic;
using System.Linq;

namespace WebshopApp.Repositories
{
    public class ProductRepository
    {
        // Intern liste — simulerer en database i hukommelsen
        private List<Product> _products = new List<Product>();
        private int _nextId = 1;

        // CREATE
        public Product Add(Product product)
        {
            product.Id = _nextId++;
            _products.Add(product);
            return product;
        }

        // READ — hent alle
        public List<Product> GetAll()
        {
            return new List<Product>(_products); // Returnér en kopi
        }

        // READ — hent én
        public Product? GetById(int id)
        {
            return _products.FirstOrDefault(p => p.Id == id);
        }

        // UPDATE
        public Product? Update(int id, Product updatedProduct)
        {
            Product? existing = _products.FirstOrDefault(p => p.Id == id);
            if (existing == null) return null;

            existing.Name = updatedProduct.Name;
            existing.Price = updatedProduct.Price;
            existing.StockQuantity = updatedProduct.StockQuantity;
            existing.Description = updatedProduct.Description;

            return existing;
        }

        // DELETE
        public bool Delete(int id)
        {
            Product? product = _products.FirstOrDefault(p => p.Id == id);
            if (product == null) return false;

            _products.Remove(product);
            return true;
        }

        // Hjælpemetode
        public int Count()
        {
            return _products.Count;
        }
    }
}
```

---

## Unit Testing med xUnit

**Unit tests** er automatiserede tests, der tester én enkelt "enhed" (typisk en metode eller klasse) isoleret fra resten af systemet.

### Hvorfor skrive tests?

- Opdager fejl tidligt i udviklingsprocessen
- Giver tillid til at ændringer ikke bryder eksisterende funktionalitet (regression testing)
- Fungerer som levende dokumentation
- Tvinger til at skrive testbar (og dermed bedre struktureret) kode

### Opret xUnit testprojekt

```bash
# I terminalen / Package Manager Console:
dotnet new xunit -n WebshopApp.Tests
dotnet add WebshopApp.Tests/WebshopApp.Tests.csproj reference WebshopApp/WebshopApp.csproj
```

Eller i Visual Studio: Højreklik på solution → Add → New Project → xUnit Test Project.

### AAA-mønsteret (Arrange, Act, Assert)

Alle unit tests bør følge dette mønster:

1. **Arrange** — Forbered data og objekter
2. **Act** — Kald den metode du tester
3. **Assert** — Verificer at resultatet er korrekt

### Eksempel: Test af `ProductRepository`

```csharp
using Xunit;
using WebshopApp.Models;
using WebshopApp.Repositories;

namespace WebshopApp.Tests
{
    public class ProductRepositoryTests
    {
        // Felter der bruges i alle tests
        private readonly ProductRepository _repo;

        // Konstruktoren kører FØR hver enkelt test
        public ProductRepositoryTests()
        {
            _repo = new ProductRepository();
        }

        // --- ADD tests ---

        [Fact]
        public void Add_ValidProduct_ReturnsProductWithId()
        {
            // Arrange
            var product = new Product("Laptop", 8999.99m, 10);

            // Act
            var result = _repo.Add(product);

            // Assert
            Assert.NotNull(result);
            Assert.Equal(1, result.Id);  // Første produkt får Id = 1
            Assert.Equal("Laptop", result.Name);
        }

        [Fact]
        public void Add_MultipleProducts_IdsAreIncremented()
        {
            // Arrange & Act
            var p1 = _repo.Add(new Product("Laptop", 8999m, 5));
            var p2 = _repo.Add(new Product("Mus", 299m, 20));
            var p3 = _repo.Add(new Product("Tastatur", 499m, 15));

            // Assert
            Assert.Equal(1, p1.Id);
            Assert.Equal(2, p2.Id);
            Assert.Equal(3, p3.Id);
        }

        // --- GET ALL tests ---

        [Fact]
        public void GetAll_EmptyRepository_ReturnsEmptyList()
        {
            // Act
            var result = _repo.GetAll();

            // Assert
            Assert.NotNull(result);
            Assert.Empty(result);
        }

        [Fact]
        public void GetAll_AfterAddingProducts_ReturnsCorrectCount()
        {
            // Arrange
            _repo.Add(new Product("Laptop", 8999m, 5));
            _repo.Add(new Product("Mus", 299m, 20));

            // Act
            var result = _repo.GetAll();

            // Assert
            Assert.Equal(2, result.Count);
        }

        // --- GET BY ID tests ---

        [Fact]
        public void GetById_ExistingId_ReturnsCorrectProduct()
        {
            // Arrange
            _repo.Add(new Product("Laptop", 8999m, 5));

            // Act
            var result = _repo.GetById(1);

            // Assert
            Assert.NotNull(result);
            Assert.Equal("Laptop", result.Name);
        }

        [Fact]
        public void GetById_NonExistingId_ReturnsNull()
        {
            // Act
            var result = _repo.GetById(999);

            // Assert
            Assert.Null(result);
        }

        // --- UPDATE tests ---

        [Fact]
        public void Update_ExistingProduct_ReturnsUpdatedProduct()
        {
            // Arrange
            _repo.Add(new Product("Gammelt navn", 100m, 5));
            var updatedData = new Product("Nyt navn", 200m, 10);

            // Act
            var result = _repo.Update(1, updatedData);

            // Assert
            Assert.NotNull(result);
            Assert.Equal("Nyt navn", result.Name);
            Assert.Equal(200m, result.Price);
        }

        [Fact]
        public void Update_NonExistingId_ReturnsNull()
        {
            // Act
            var result = _repo.Update(999, new Product("Test", 100m, 1));

            // Assert
            Assert.Null(result);
        }

        // --- DELETE tests ---

        [Fact]
        public void Delete_ExistingProduct_ReturnsTrueAndRemovesProduct()
        {
            // Arrange
            _repo.Add(new Product("Laptop", 8999m, 5));

            // Act
            bool deleted = _repo.Delete(1);

            // Assert
            Assert.True(deleted);
            Assert.Equal(0, _repo.Count());
        }

        [Fact]
        public void Delete_NonExistingId_ReturnsFalse()
        {
            // Act
            bool deleted = _repo.Delete(999);

            // Assert
            Assert.False(deleted);
        }
    }
}
```

---

## Kørsel af tests

### Via terminal

```bash
cd WebshopApp.Tests
dotnet test
```

### Output eksempel

```
Test run for WebshopApp.Tests.dll
  Passed! - Failed: 0, Passed: 9, Skipped: 0, Total: 9
```

### Via Visual Studio

- Åbn **Test Explorer** (Test → Test Explorer)
- Klik **Run All Tests**
- Grønne flueben = bestået, røde kryds = fejlet

---

## Opsummering

| Begreb                  | Beskrivelse                                               |
|-------------------------|-----------------------------------------------------------|
| Model-klasse            | Repræsenterer et domæneobjekt med data                    |
| Nullable Reference Types| Kompilatoren hjælper med at undgå NullReferenceException  |
| Repository Pattern      | Samler al datahåndtering for en model ét sted             |
| Unit Test               | Automatiseret test af én isoleret enhed                   |
| xUnit                   | Populært testframework til .NET                           |
| AAA-mønsteret           | Arrange, Act, Assert — standardstruktur for en test       |
| `[Fact]`                | Attribut der markerer en testmetode i xUnit               |

---

## Opgaver til selvstudium

1. Opret en `Customer`-model med relevante properties (Id, Name, Email, PhoneNumber?)
2. Opret en `CustomerRepository` med CRUD-metoder
3. Skriv mindst 8 unit tests for `CustomerRepository`
4. Sørg for at alle dine properties bruger Nullable Reference Types korrekt
