# Uge 8 — Database & Thread Safety

Denne uge løfter vi vores applikation fra en in-memory løsning til rigtig persistering med Entity Framework Core og SQL Server. Vi introducerer Interface-mønsteret for at gøre koden fleksibel og testbar, og vi undersøger Thread Safety — hvad sker der, når flere tråde tilgår de samme data samtidig, og hvordan undgår vi fejl.

---

## Entity Framework Core (EF Core)

**Entity Framework Core** er Microsofts ORM (Object-Relational Mapper) til .NET. Det lader os arbejde med en SQL-database ved hjælp af C#-objekter — vi behøver ikke skrive SQL manuelt til standard CRUD-operationer.

### Installer NuGet-pakker

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

Eller via NuGet Package Manager i Visual Studio.

### DbContext — databaseforbindelsen

`DbContext` er den centrale klasse i EF Core. Den repræsenterer forbindelsen til databasen og indeholder `DbSet<T>` for hver tabel.

```csharp
using Microsoft.EntityFrameworkCore;

namespace WebshopApp.Data
{
    public class WebshopDbContext : DbContext
    {
        // Konstruktor — modtager konfiguration via Dependency Injection
        public WebshopDbContext(DbContextOptions<WebshopDbContext> options)
            : base(options)
        {
        }

        // Tabeller som DbSet-properties
        public DbSet<Product> Products { get; set; }
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Order> Orders { get; set; }

        // Valgfrit: konfigurer modellen
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Product>(entity =>
            {
                entity.HasKey(p => p.Id);
                entity.Property(p => p.Name).IsRequired().HasMaxLength(200);
                entity.Property(p => p.Price).HasPrecision(18, 2);
            });
        }
    }
}
```

### Konfiguration i Program.cs (ASP.NET Core)

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Tilføj DbContext med SQL Server
builder.Services.AddDbContext<WebshopDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Tilføj controllers
builder.Services.AddControllers();
```

### Connection string i appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=WebshopDb;Trusted_Connection=True;"
  }
}
```

---

## Migrationer

Migrationer er EF Cores måde at holde styr på ændringer i databaseskemaet over tid. Hver migration er en C#-klasse, der beskriver hvad der er ændret.

### Opret første migration

```bash
# I terminalen (fra projektmappen):
dotnet ef migrations add InitialCreate

# Anvend migrationen på databasen:
dotnet ef database update
```

### Hvad genereres?

```
Migrations/
    20240215093000_InitialCreate.cs
    20240215093000_InitialCreate.Designer.cs
    WebshopDbContextModelSnapshot.cs
```

### Eksempel på migrationsklasse (automatisk genereret)

```csharp
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Products",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 200, nullable: false),
                Price = table.Column<decimal>(precision: 18, scale: 2, nullable: false),
                StockQuantity = table.Column<int>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Products", x => x.Id);
            });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Products");
    }
}
```

---

## Database Repository med EF Core

```csharp
using Microsoft.EntityFrameworkCore;

namespace WebshopApp.Repositories
{
    public class ProductDbRepository
    {
        private readonly WebshopDbContext _context;

        public ProductDbRepository(WebshopDbContext context)
        {
            _context = context;
        }

        // CREATE
        public async Task<Product> AddAsync(Product product)
        {
            _context.Products.Add(product);
            await _context.SaveChangesAsync();
            return product;
        }

        // READ — alle
        public async Task<List<Product>> GetAllAsync()
        {
            return await _context.Products.ToListAsync();
        }

        // READ — én
        public async Task<Product?> GetByIdAsync(int id)
        {
            return await _context.Products.FindAsync(id);
        }

        // UPDATE
        public async Task<Product?> UpdateAsync(int id, Product updatedProduct)
        {
            var existing = await _context.Products.FindAsync(id);
            if (existing == null) return null;

            existing.Name = updatedProduct.Name;
            existing.Price = updatedProduct.Price;
            existing.StockQuantity = updatedProduct.StockQuantity;

            await _context.SaveChangesAsync();
            return existing;
        }

        // DELETE
        public async Task<bool> DeleteAsync(int id)
        {
            var product = await _context.Products.FindAsync(id);
            if (product == null) return false;

            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
            return true;
        }
    }
}
```

---

## Interface Pattern (IRepository)

Et **interface** i C# er en kontrakt: det definerer hvilke metoder en klasse SKAL implementere, men ikke HVORDAN de implementeres.

### Hvorfor bruge interfaces?

1. **Udskiftelighed** — skift fra InMemory til Database uden at ændre controllers
2. **Testbarhed** — brug en mock/fake repository i tests
3. **Dependency Injection** — registrer konkret implementering bag interface
4. **SOLID** — understøtter Dependency Inversion Principle

### Definér interfacet

```csharp
namespace WebshopApp.Repositories
{
    public interface IProductRepository
    {
        Task<Product> AddAsync(Product product);
        Task<List<Product>> GetAllAsync();
        Task<Product?> GetByIdAsync(int id);
        Task<Product?> UpdateAsync(int id, Product updatedProduct);
        Task<bool> DeleteAsync(int id);
        Task<List<Product>> SearchByNameAsync(string term);
    }
}
```

### In-Memory implementering (til tests)

```csharp
public class InMemoryProductRepository : IProductRepository
{
    private readonly List<Product> _products = new();
    private int _nextId = 1;

    public Task<Product> AddAsync(Product product)
    {
        product.Id = _nextId++;
        _products.Add(product);
        return Task.FromResult(product);
    }

    public Task<List<Product>> GetAllAsync()
        => Task.FromResult(new List<Product>(_products));

    public Task<Product?> GetByIdAsync(int id)
        => Task.FromResult(_products.FirstOrDefault(p => p.Id == id));

    public Task<Product?> UpdateAsync(int id, Product updated)
    {
        var existing = _products.FirstOrDefault(p => p.Id == id);
        if (existing == null) return Task.FromResult<Product?>(null);
        existing.Name = updated.Name;
        existing.Price = updated.Price;
        return Task.FromResult<Product?>(existing);
    }

    public Task<bool> DeleteAsync(int id)
    {
        var p = _products.FirstOrDefault(x => x.Id == id);
        if (p == null) return Task.FromResult(false);
        _products.Remove(p);
        return Task.FromResult(true);
    }

    public Task<List<Product>> SearchByNameAsync(string term)
    {
        var result = _products
            .Where(p => p.Name.Contains(term, StringComparison.OrdinalIgnoreCase))
            .ToList();
        return Task.FromResult(result);
    }
}
```

### Database implementering

```csharp
public class ProductDbRepository : IProductRepository
{
    private readonly WebshopDbContext _context;

    public ProductDbRepository(WebshopDbContext context)
    {
        _context = context;
    }

    public async Task<Product> AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
        return product;
    }

    // ... resten af metoderne som vist ovenfor ...
}
```

### Registrer i DI-containeren

```csharp
// Program.cs — skift nemt mellem implementeringer
builder.Services.AddScoped<IProductRepository, ProductDbRepository>();
// ELLER til tests / uden database:
// builder.Services.AddScoped<IProductRepository, InMemoryProductRepository>();
```

### Brug i controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repo;

    // Dependency Injection — DI-containeren leverer den konkrete type
    public ProductsController(IProductRepository repo)
    {
        _repo = repo;
    }

    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetAll()
    {
        return await _repo.GetAllAsync();
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _repo.GetByIdAsync(id);
        if (product == null) return NotFound();
        return product;
    }
}
```

---

## Thread Safety

**Thread Safety** handler om hvad der sker, når flere tråde tilgår og ændrer de samme data på samme tid. Dette er særligt relevant i webapplikationer, hvor mange HTTP-requests behandles parallelt.

### Race Condition — et klassisk problem

```csharp
// IKKE thread-safe — race condition!
public class UnsafeCounter
{
    private int _count = 0;

    public void Increment()
    {
        _count++;  // Dette er faktisk 3 operationer:
                   // 1. Læs _count
                   // 2. Tilføj 1
                   // 3. Skriv tilbage
                   // Tråd A og B kan begge læse 5, begge lægge 1 til, og begge skrive 6
                   // Resultatet burde være 7, men bliver 6!
    }

    public int GetCount() => _count;
}
```

### Problem visualiseret

```
Tråd A:  Læs _count = 5
Tråd B:  Læs _count = 5    <- B læser INDEN A skriver
Tråd A:  _count = 5 + 1 = 6
Tråd B:  _count = 5 + 1 = 6  <- B overskriver A's resultat!
Resultat: 6 (forventet: 7)
```

---

## Interlocked.Increment

`Interlocked`-klassen i .NET giver atomare operationer — operationer der ikke kan afbrydes af andre tråde.

```csharp
using System.Threading;

public class SafeCounter
{
    private int _count = 0;

    // Atomær increment — thread-safe!
    public void Increment()
    {
        Interlocked.Increment(ref _count);
    }

    // Atomær decrement
    public void Decrement()
    {
        Interlocked.Decrement(ref _count);
    }

    // Atomær add
    public void Add(int value)
    {
        Interlocked.Add(ref _count, value);
    }

    public int GetCount() => _count;
}
```

### Interlocked vs lock

```csharp
// Interlocked — kun til simple numeriske operationer
Interlocked.Increment(ref _count);    // Hurtigst
Interlocked.Decrement(ref _count);
Interlocked.Exchange(ref _count, 0);  // Sæt til 0 atomært
Interlocked.CompareExchange(ref _count, newVal, expected); // CAS

// lock — til mere komplekse operationer
private readonly object _lockObject = new object();

public void ComplexOperation()
{
    lock (_lockObject)
    {
        // Kun én tråd ad gangen her
        _list.Add(newItem);
        _count++;
        _lastModified = DateTime.Now;
    }
}
```

### Thread-safe In-Memory Repository

```csharp
public class ThreadSafeProductRepository : IProductRepository
{
    private readonly List<Product> _products = new();
    private int _nextId = 0;
    private readonly object _lock = new object();

    public Task<Product> AddAsync(Product product)
    {
        lock (_lock)
        {
            product.Id = Interlocked.Increment(ref _nextId);
            _products.Add(product);
            return Task.FromResult(product);
        }
    }

    public Task<List<Product>> GetAllAsync()
    {
        lock (_lock)
        {
            return Task.FromResult(new List<Product>(_products));
        }
    }

    public Task<bool> DeleteAsync(int id)
    {
        lock (_lock)
        {
            var product = _products.FirstOrDefault(p => p.Id == id);
            if (product == null) return Task.FromResult(false);
            _products.Remove(product);
            return Task.FromResult(true);
        }
    }

    // ... resten af metoderne ...
}
```

### ConcurrentDictionary — et alternativ

For simple key-value stores er `ConcurrentDictionary<TKey, TValue>` et thread-safe alternativ til `Dictionary`:

```csharp
using System.Collections.Concurrent;

public class ConcurrentProductStore
{
    private readonly ConcurrentDictionary<int, Product> _products = new();
    private int _nextId = 0;

    public Product Add(Product product)
    {
        product.Id = Interlocked.Increment(ref _nextId);
        _products[product.Id] = product;
        return product;
    }

    public Product? GetById(int id)
    {
        _products.TryGetValue(id, out var product);
        return product;
    }

    public bool Delete(int id)
    {
        return _products.TryRemove(id, out _);
    }
}
```

---

## Async/Await i C#

EF Core bruger asynkrone metoder. Forstå det grundlæggende:

```csharp
// Synkron — blokerer tråden mens den venter
public Product GetById(int id)
{
    return _context.Products.Find(id); // Blokerer!
}

// Asynkron — frigiver tråden mens den venter
public async Task<Product?> GetByIdAsync(int id)
{
    return await _context.Products.FindAsync(id); // Frigiver tråden
}

// Controller-metode SKAL også være async
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetById(int id)
{
    var product = await _repo.GetByIdAsync(id);
    if (product == null) return NotFound();
    return product;
}
```

---

## Opsummering

| Begreb                  | Beskrivelse                                                          |
|-------------------------|----------------------------------------------------------------------|
| EF Core                 | ORM til .NET — kortlægger C#-klasser til databasetabeller            |
| DbContext               | Repræsenterer databaseforbindelsen og DbSet for hver tabel           |
| Migrationer             | Versionsstyring af databaseskemaet                                   |
| IRepository             | Interface der definerer kontrakten for datahåndtering                |
| Dependency Injection    | DI-containeren leverer konkrete implementeringer                     |
| Thread Safety           | Sikring af at data ikke korrupteres ved samtidige tråde              |
| Race Condition          | Fejl der opstår når to tråde ændrer data på samme tid                |
| Interlocked             | Atomare operationer — thread-safe uden lock                          |
| lock                    | Blokerer alle andre tråde fra en kodeblok                            |
| ConcurrentDictionary    | Thread-safe dictionary fra .NET BCL                                  |

---

## Opgaver til selvstudium

1. Opret et nyt ASP.NET Web API projekt med EF Core og SQL Server
2. Definer et `IProductRepository` interface og to implementeringer (InMemory og DB)
3. Registrer i DI-containeren og test skift mellem implementeringer
4. Skriv en simpel test der demonstrerer race condition med en usikker counter
5. Ret testen ved at bruge `Interlocked.Increment` og verificer at den nu består
