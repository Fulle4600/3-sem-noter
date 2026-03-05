# Uge 11 — Softwarearkitektur & Design Patterns

Denne uge handler om at designe software der er holdbart, fleksibelt og let at vedligeholde. Vi kigger på SOLID-principper, Repository Pattern, MVC og Dependency Injection — det fundamentale fundament for professionel softwarearkitektur.

---

## SOLID Principper

SOLID er et akronym for 5 designprincipper der gør objektorienteret kode nemmere at forstå, teste og ændre over tid.

### S — Single Responsibility Principle (SRP)

**En klasse skal kun have ét ansvar — én grund til at ændre den.**

Hvis en klasse håndterer både forretningslogik, databaseadgang OG e-mail udsendelse, er den svær at teste og ændre. Del ansvaret op.

```csharp
// FORKERT — UserService gør for meget
class UserService {
    public void CreateUser(User u) { /* gem i DB */ }
    public void SendWelcomeEmail(User u) { /* send mail */ }
    public string FormatUserReport(User u) { /* lav rapport */ }
}

// RIGTIGT — ét ansvar per klasse
class UserRepository  { public void Save(User u) { ... } }
class EmailService    { public void SendWelcome(User u) { ... } }
class UserReportService { public string Format(User u) { ... } }
```

**Hvorfor?** Hvis database-logikken skal ændres, påvirker det ikke e-mail-koden. Nemmere at teste isoleret.

---

### O — Open/Closed Principle (OCP)

**Klasser skal være åbne for udvidelse men lukkede for ændring.**

Du skal kunne tilføje ny funktionalitet *uden* at ændre eksisterende, testede kode. Brug abstraktion (interfaces/inheritance).

```csharp
// FORKERT — skal ændres hver gang ny rabattype tilføjes
class DiscountCalculator {
    public decimal Calculate(string type, decimal price) {
        if (type == "student") return price * 0.9m;
        if (type == "senior")  return price * 0.8m;
        // Ny type = skal ændre denne klasse
    }
}

// RIGTIGT — udvid med ny klasse, rør ikke eksisterende kode
interface IDiscount { decimal Apply(decimal price); }
class StudentDiscount : IDiscount { public decimal Apply(decimal p) => p * 0.9m; }
class SeniorDiscount  : IDiscount { public decimal Apply(decimal p) => p * 0.8m; }
class BlackFridayDiscount : IDiscount { public decimal Apply(decimal p) => p * 0.5m; } // Ny!
```

---

### L — Liskov Substitution Principle (LSP)

**En subklasse skal altid kunne erstatte sin superklasse uden at programmet går i stykker.**

Hvis du har kode der virker med `Bird`, skal den også virke hvis du bytter til en `Duck` (der arver fra `Bird`).

```csharp
// PROBLEM — Penguin arver fra Bird men kan ikke flyve!
class Bird    { public virtual void Fly() { ... } }
class Penguin : Bird {
    public override void Fly() => throw new Exception("Kan ikke flyve!"); // Bryder LSP!
}

// LØSNING — adskil flyve-adfærd
interface IFlyable { void Fly(); }
class Sparrow : Bird, IFlyable { public void Fly() { ... } }
class Penguin : Bird { /* ingen Fly */ }
```

**Tommelfingerregel:** Hvis en subklasse *undtagelsesvist* kaster fejl for metoder den arver, er det et LSP-brud.

---

### I — Interface Segregation Principle (ISP)

**Mange specifikke interfaces er bedre end ét stort generelt interface.**

Klasser bør ikke tvinges til at implementere metoder de ikke bruger.

```csharp
// FORKERT — IWorker tvinger Robot til at implementere Eat()
interface IWorker {
    void Work();
    void Eat();   // Robotter spiser ikke!
    void Sleep(); // Robotter sover ikke!
}

// RIGTIGT — opdel i specifikke interfaces
interface IWorkable { void Work(); }
interface IEatable  { void Eat(); }
class Human : IWorkable, IEatable { ... }
class Robot : IWorkable { ... } // Kun det den faktisk kan
```

---

### D — Dependency Inversion Principle (DIP)

**Afhæng af abstraktioner (interfaces), ikke konkrete implementationer.**

High-level moduler (forretningslogik) skal ikke afhænge direkte af low-level moduler (database). Begge skal afhænge af et interface.

```csharp
// FORKERT — UserService er hårdt koblet til SQL
class UserService {
    private SqlUserRepository _repo = new SqlUserRepository(); // Hårdt koblet!
}

// RIGTIGT — afhænger af interface
class UserService {
    private IUserRepository _repo;
    public UserService(IUserRepository repo) { _repo = repo; } // Injiceret!
}

// Nu kan man bruge SQL, MongoDB eller et fake repository til tests
```

---

## Repository Pattern

Abstraktionslag mellem forretningslogik og dataadgang. Forretningslogik kalder altid et interface — ved ikke hvilken database der er bagved.

```csharp
// Interface — kontrakten
interface IProductRepository {
    Product GetById(int id);
    IEnumerable<Product> GetAll();
    void Add(Product p);
    void Update(Product p);
    void Delete(int id);
}

// Konkret implementering
class SqlProductRepository : IProductRepository {
    private readonly AppDbContext _db;
    public SqlProductRepository(AppDbContext db) { _db = db; }

    public Product GetById(int id) => _db.Products.Find(id);
    public IEnumerable<Product> GetAll() => _db.Products.ToList();
    public void Add(Product p) { _db.Products.Add(p); _db.SaveChanges(); }
    public void Update(Product p) { _db.Products.Update(p); _db.SaveChanges(); }
    public void Delete(int id) { var p = _db.Products.Find(id); _db.Products.Remove(p); _db.SaveChanges(); }
}

// Test-implementering (InMemory)
class InMemoryProductRepository : IProductRepository {
    private List<Product> _data = new();
    public Product GetById(int id) => _data.FirstOrDefault(p => p.Id == id);
    public IEnumerable<Product> GetAll() => _data;
    public void Add(Product p) { p.Id = _data.Count + 1; _data.Add(p); }
    public void Update(Product p) { /* find og erstat */ }
    public void Delete(int id) { _data.RemoveAll(p => p.Id == id); }
}
```

**Fordele:**
- Skift database (SQL → MongoDB) = kun ny implementering af interfacet
- Unit tests bruger InMemory-versionen — ingen rigtig database nødvendig
- Forretningslogik er isoleret fra datadetaljer

---

## MVC — Model View Controller

Arkitekturmønster der opdeler applikationen i 3 klare ansvarsområder.

```
Browser → Controller → Model (henter data) → Controller → View (renderer HTML) → Browser
```

| Del | Ansvar | Kender til |
|---|---|---|
| **Model** | Data og forretningslogik | Database, regler |
| **View** | Præsentation (HTML/CSS) | Kun data den modtager |
| **Controller** | Koordinator | Modtager requests, kalder Model, returnerer View |

```csharp
// ASP.NET Core MVC eksempel
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase {
    private readonly IProductRepository _repo;

    public ProductsController(IProductRepository repo) {
        _repo = repo; // Dependency Injection
    }

    [HttpGet]
    public IActionResult GetAll() {
        var products = _repo.GetAll();
        return Ok(products);
    }

    [HttpPost]
    public IActionResult Create(Product product) {
        _repo.Add(product);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }
}
```

**Separation of concerns:** Skift UI (View) uden at røre forretningslogik (Model). Skift database uden at røre UI.

---

## Dependency Injection (DI)

En klasse *modtager* sine afhængigheder udefra fremfor at oprette dem selv.

**Uden DI (tæt koblet):**
```csharp
class OrderService {
    private SqlProductRepository _repo = new SqlProductRepository(); // Opretter selv!
    // Umuligt at teste uden rigtig database
    // Umuligt at skifte til MongoDB
}
```

**Med DI (løs kobling):**
```csharp
class OrderService {
    private IProductRepository _repo;
    public OrderService(IProductRepository repo) { _repo = repo; } // Modtager!
}
```

**Registrering i ASP.NET Core (Program.cs):**
```csharp
// Fortæl DI-container: "Når nogen beder om IProductRepository, giv dem SqlProductRepository"
builder.Services.AddScoped<IProductRepository, SqlProductRepository>();
builder.Services.AddScoped<IEmailService, SmtpEmailService>();

// Til tests kan man registrere:
builder.Services.AddScoped<IProductRepository, InMemoryProductRepository>();
```

**Livstider:**
| Livstid | Beskrivelse |
|---|---|
| `AddSingleton` | Én instans for hele applikationens levetid |
| `AddScoped` | Én instans per HTTP request (mest brugt til repositories) |
| `AddTransient` | Ny instans hver gang nogen beder om den |

---

## Design Patterns

Genbrugelige løsninger på kendte softwareproblemer. Tre kategorier:

### Creational Patterns (objekt-oprettelse)

**Singleton** — Garanterer at der kun er én instans af en klasse.
```csharp
class DatabaseConnection {
    private static DatabaseConnection _instance;
    private DatabaseConnection() { } // Privat constructor
    public static DatabaseConnection Instance =>
        _instance ??= new DatabaseConnection();
}
```

**Factory** — Delegerer objekt-oprettelse til en separat klasse.
```csharp
interface IAnimal { void Speak(); }
class Dog : IAnimal { public void Speak() => Console.WriteLine("Vov!"); }
class Cat : IAnimal { public void Speak() => Console.WriteLine("Miau!"); }

class AnimalFactory {
    public static IAnimal Create(string type) => type switch {
        "dog" => new Dog(),
        "cat" => new Cat(),
        _ => throw new ArgumentException("Ukendt dyr")
    };
}
```

### Structural Patterns (sammensætning)

**Adapter** — Gør to inkompatible interfaces kompatible.
```csharp
// Tredjeparts bibliotek har et interface du ikke kan ændre
class LegacyPrinter { public void PrintOldFormat(string s) { ... } }

// Din kode forventer IModernPrinter
interface IModernPrinter { void Print(Document doc); }

class PrinterAdapter : IModernPrinter {
    private LegacyPrinter _legacy = new();
    public void Print(Document doc) => _legacy.PrintOldFormat(doc.ToString());
}
```

### Behavioral Patterns (kommunikation)

**Observer** — Objekter "abonnerer" på ændringer hos et andet objekt.
```csharp
// Eksempel: Event-systemet i C# er Observer pattern
class StockPrice {
    public event Action<decimal> PriceChanged;
    private decimal _price;
    public decimal Price {
        set { _price = value; PriceChanged?.Invoke(value); }
    }
}
// Abonnenter notificeres automatisk når prisen ændrer sig
```

---

## Incremental Design

I XP starter du med det *simpleste* design der løser problemet nu — og forbedrer det løbende efterhånden som du lærer mere.

**Modsætning: Big Design Up Front (BDUF)**
- BDUF: Design hele systemet på forhånd → mister tid, laver forkerte antagelser
- Incremental Design: Design til dagens behov → refaktorer når du forstår mere

**Princippet i praksis:**
1. Lav det simpleste der virker (YAGNI — You Ain't Gonna Need It)
2. Skriv tests der beskriver adfærden
3. Refaktorer modig når du ser et bedre mønster
4. Design og kode vokser synkront med forståelsen

> "Gør det simpelt i dag, refaktorer i morgen når du ved mere."

---

## Opsummering

| Begreb / Princip          | Beskrivelse                                                                          |
|---------------------------|--------------------------------------------------------------------------------------|
| SRP (Single Responsibility)| En klasse har kun ét ansvar — én grund til at ændre den                             |
| OCP (Open/Closed)         | Åben for udvidelse, lukket for ændring — brug interfaces/abstraktion                |
| LSP (Liskov Substitution) | En subklasse skal altid kunne erstatte sin superklasse uden at bryde opførslen      |
| ISP (Interface Segregation)| Mange specifikke interfaces er bedre end ét stort generelt interface               |
| DIP (Dependency Inversion)| Afhæng af interfaces (abstraktioner), ikke konkrete implementationer               |
| Repository Pattern        | Abstraktionslag der isolerer forretningslogik fra datahåndtering                    |
| MVC                       | Model-View-Controller: adskiller data, præsentation og koordination                 |
| Dependency Injection (DI) | Klassen modtager sine afhængigheder udefra fremfor at oprette dem selv              |
| Singleton Pattern         | Sikrer at kun én instans af en klasse eksisterer                                    |
| Factory Pattern           | Delegerer objekt-oprettelse til en separat fabriksklasse                            |
| Adapter Pattern           | Gør to inkompatible interfaces kompatible                                            |
| Observer Pattern          | Objekter abonnerer på ændringer hos et andet objekt — C# events er Observer Pattern |
| Incremental Design        | Design til nutidens behov — refaktorer løbende (modsat Big Design Up Front)         |
| YAGNI                     | "You Ain't Gonna Need It" — byg ikke funktioner du ikke har brug for nu             |

---

## Opgaver til selvstudium

1. Tag et stykke kode du har skrevet tidligere — identificer et SRP-brud og omskriv det så det følger SRP
2. Implementer OCP-eksemplet med rabatter: tilføj en ny rabattype (fx `VIPDiscount`) uden at ændre eksisterende kode
3. Forklar LSP med egne ord: Hvad er problemet med at `Penguin` arver `Fly()` fra `Bird`? Hvad er løsningen?
4. Beskriv hvad DI-containerens tre livstider (`Singleton`, `Scoped`, `Transient`) bruges til — og hvornår ville du vælge den ene over den anden?
5. Tegn et diagram over MVC-arkitekturen for en webshop-applikation med produkter og ordrer
6. Implementer Observer Pattern uden at bruge C# events — brug et interface `IObserver` og en `Subject`-klasse der holder en liste af observers
