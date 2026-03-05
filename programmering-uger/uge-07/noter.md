# Uge 7 — Mere Unit Test & Git

Denne uge bygger videre på unit testing ved at teste søgning og sortering i vores repository. Vi introducerer også to nyttige C#-funktioner: named arguments og optional arguments, som gør vores metoder mere fleksible og letlæselige. Derudover dykker vi ned i Git branches og Pull Requests — det workflow som professionelle teams bruger til at samarbejde om kode.

---

## Søgning og sortering i Repository

### Søgemetoder

Vi udvider vores `ProductRepository` med søgefunktionalitet. LINQ (Language Integrated Query) er det primære redskab i C# til at filtrere og transformere samlinger.

```csharp
public class ProductRepository
{
    private List<Product> _products = new List<Product>();
    private int _nextId = 1;

    // Eksisterende metoder (Add, GetAll, GetById, Update, Delete)...

    // Søg på navn (case-insensitiv)
    public List<Product> SearchByName(string searchTerm)
    {
        return _products
            .Where(p => p.Name.Contains(searchTerm, StringComparison.OrdinalIgnoreCase))
            .ToList();
    }

    // Filtrer på prisinterval
    public List<Product> GetByPriceRange(decimal minPrice, decimal maxPrice)
    {
        return _products
            .Where(p => p.Price >= minPrice && p.Price <= maxPrice)
            .ToList();
    }

    // Hent produkter med lav lagerbeholdning
    public List<Product> GetLowStock(int threshold = 5)
    {
        return _products
            .Where(p => p.StockQuantity <= threshold)
            .ToList();
    }
}
```

### Sorteringsmetoder

```csharp
public class ProductRepository
{
    // ... eksisterende kode ...

    // Sorter på pris (stigende som standard)
    public List<Product> GetAllSortedByPrice(bool ascending = true)
    {
        if (ascending)
            return _products.OrderBy(p => p.Price).ToList();
        else
            return _products.OrderByDescending(p => p.Price).ToList();
    }

    // Sorter på navn
    public List<Product> GetAllSortedByName()
    {
        return _products.OrderBy(p => p.Name).ToList();
    }

    // Generisk sortering med lambda
    public List<Product> GetAllSorted(Func<Product, object> sortSelector, bool ascending = true)
    {
        if (ascending)
            return _products.OrderBy(sortSelector).ToList();
        else
            return _products.OrderByDescending(sortSelector).ToList();
    }
}
```

---

## Unit tests for søgning og sortering

```csharp
public class ProductRepositorySearchTests
{
    private readonly ProductRepository _repo;

    public ProductRepositorySearchTests()
    {
        _repo = new ProductRepository();

        // Seed testdata
        _repo.Add(new Product("Apple MacBook", 14999m, 3));
        _repo.Add(new Product("Dell Laptop", 8999m, 8));
        _repo.Add(new Product("Apple Magic Mouse", 799m, 2));
        _repo.Add(new Product("Logitech Tastatur", 499m, 15));
        _repo.Add(new Product("USB-C Hub", 299m, 1));
    }

    // --- Søgetests ---

    [Fact]
    public void SearchByName_ExistingTerm_ReturnsMatchingProducts()
    {
        // Act
        var result = _repo.SearchByName("Apple");

        // Assert
        Assert.Equal(2, result.Count);
        Assert.All(result, p => Assert.Contains("Apple", p.Name));
    }

    [Fact]
    public void SearchByName_CaseInsensitive_ReturnsResults()
    {
        // Act
        var result = _repo.SearchByName("apple");

        // Assert
        Assert.Equal(2, result.Count);
    }

    [Fact]
    public void SearchByName_NonExistingTerm_ReturnsEmptyList()
    {
        // Act
        var result = _repo.SearchByName("Samsung");

        // Assert
        Assert.Empty(result);
    }

    [Fact]
    public void GetByPriceRange_ValidRange_ReturnsCorrectProducts()
    {
        // Act
        var result = _repo.GetByPriceRange(300m, 900m);

        // Assert — forventer: Magic Mouse (799), Logitech (499), USB-C Hub (299)
        Assert.Equal(3, result.Count);
    }

    // --- Sorteringstests ---

    [Fact]
    public void GetAllSortedByPrice_Ascending_FirstIsCheapest()
    {
        // Act
        var result = _repo.GetAllSortedByPrice(ascending: true);

        // Assert — USB-C Hub til 299 skal være først
        Assert.Equal("USB-C Hub", result.First().Name);
    }

    [Fact]
    public void GetAllSortedByPrice_Descending_FirstIsMostExpensive()
    {
        // Act
        var result = _repo.GetAllSortedByPrice(ascending: false);

        // Assert — MacBook til 14999 skal være først
        Assert.Equal("Apple MacBook", result.First().Name);
    }

    [Fact]
    public void GetAllSortedByName_ReturnsAlphabetically()
    {
        // Act
        var result = _repo.GetAllSortedByName();

        // Assert — "Apple MacBook" < "Apple Magic Mouse" < "Dell..." osv.
        Assert.Equal("Apple MacBook", result.First().Name);
    }

    // --- Lav lagerbeholdning ---

    [Fact]
    public void GetLowStock_DefaultThreshold_ReturnsBelowFive()
    {
        // Act
        var result = _repo.GetLowStock(); // threshold = 5

        // Assert — MacBook (3), Magic Mouse (2), USB-C Hub (1)
        Assert.Equal(3, result.Count);
    }

    [Fact]
    public void GetLowStock_CustomThreshold_ReturnsCorrectProducts()
    {
        // Act
        var result = _repo.GetLowStock(threshold: 2);

        // Assert — Magic Mouse (2), USB-C Hub (1)
        Assert.Equal(2, result.Count);
    }
}
```

---

## Named Arguments i C#

**Named arguments** lader dig angive argument-navne eksplicit, når du kalder en metode. Det forbedrer læsbarheden, specielt når metoder har mange parametre.

### Uden named arguments (svært at læse)

```csharp
// Hvad betyder true og false her?
CreateOrder(1001, true, false, 3);
```

### Med named arguments (klart og tydeligt)

```csharp
CreateOrder(
    customerId: 1001,
    isPriority: true,
    requiresSignature: false,
    quantity: 3
);
```

### Regler for named arguments

- Du kan blande positional og named arguments
- Named arguments behøver ikke stå i rækkefølge
- Nyttigt ved metoder med mange bool-parametre

```csharp
public void SendEmail(
    string recipient,
    string subject,
    string body,
    bool isHtml = false,
    bool sendCopy = false)
{
    // implementation...
}

// Kald med named arguments — meget læseligt
SendEmail(
    recipient: "bruger@example.com",
    subject: "Bekræftelse",
    body: "<h1>Tak</h1>",
    isHtml: true
    // sendCopy bruger default-værdien false
);
```

---

## Optional Arguments i C#

**Optional arguments** (valgfrie parametre) har en standardværdi. Hvis du ikke angiver argumentet, bruges standardværdien automatisk.

### Syntaks

```csharp
public List<Product> GetProducts(
    int page = 1,
    int pageSize = 20,
    bool includeInactive = false)
{
    // ...
}

// Alle disse kald er gyldige:
GetProducts();                    // page=1, pageSize=20, includeInactive=false
GetProducts(2);                   // page=2, pageSize=20, includeInactive=false
GetProducts(1, 10);               // page=1, pageSize=10, includeInactive=false
GetProducts(pageSize: 5);         // Named + optional kombineret
GetProducts(1, 20, true);         // Alle eksplicitte
```

### Regler for optional arguments

- Optional parametre skal stå EFTER obligatoriske parametre
- Standardværdien skal være en compile-time konstant
- Brug dem til at gøre API'er mere fleksible uden at overloade

```csharp
// FORKERT — optional parameter før obligatorisk
public void Method(int optional = 5, string required)  // Kompilerer ikke!

// KORREKT
public void Method(string required, int optional = 5)
```

### Praktisk eksempel: Søgemetode med optional parametre

```csharp
public List<Product> Search(
    string? nameTerm = null,
    decimal? minPrice = null,
    decimal? maxPrice = null,
    string sortBy = "name",
    bool ascending = true)
{
    var query = _products.AsEnumerable();

    if (nameTerm != null)
        query = query.Where(p => p.Name.Contains(nameTerm, StringComparison.OrdinalIgnoreCase));

    if (minPrice.HasValue)
        query = query.Where(p => p.Price >= minPrice.Value);

    if (maxPrice.HasValue)
        query = query.Where(p => p.Price <= maxPrice.Value);

    query = sortBy.ToLower() switch
    {
        "price" => ascending ? query.OrderBy(p => p.Price) : query.OrderByDescending(p => p.Price),
        "name"  => ascending ? query.OrderBy(p => p.Name)  : query.OrderByDescending(p => p.Name),
        _       => query
    };

    return query.ToList();
}

// Fleksible kald:
var alleAppleProdukter = Search(nameTerm: "Apple");
var billigeProdukter   = Search(maxPrice: 500m, sortBy: "price");
var dyresteFørst       = Search(sortBy: "price", ascending: false);
```

---

## Git Branches

En **branch** (gren) i Git er en uafhængig udviklingslinje. Den primære branch hedder typisk `main` (tidligere `master`).

### Hvorfor bruge branches?

- Isoler ny funktionalitet fra stabil kode
- Flere udviklere kan arbejde parallelt
- Nemt at opgive en feature uden at ødelægge main
- Giver mulighed for code review inden merge

### Grundlæggende branch-kommandoer

```bash
# Se alle branches (lokal og remote)
git branch -a

# Opret ny branch
git branch feature/product-search

# Skift til en branch
git checkout feature/product-search

# Opret OG skift i én kommando (moderne syntax)
git switch -c feature/product-search

# Se hvilken branch du er på
git status

# Slet en branch (kun hvis merged)
git branch -d feature/product-search

# Tving sletning (uanset om merged)
git branch -D feature/product-search
```

### Typisk workflow

```bash
# 1. Start fra main og hent seneste ændringer
git checkout main
git pull origin main

# 2. Opret feature branch
git switch -c feature/add-product-search

# 3. Lav dine ændringer og commit
git add ProductRepository.cs
git commit -m "Add SearchByName method to ProductRepository"

git add ProductRepositoryTests.cs
git commit -m "Add unit tests for SearchByName"

# 4. Push branch til GitHub
git push origin feature/add-product-search

# 5. Opret Pull Request på GitHub
# (se næste sektion)
```

---

## Pull Requests (PR)

En **Pull Request** er en anmodning om at merge dine ændringer fra en feature branch ind i main. Det er standarden for code review i professionelle teams.

### Processen

1. **Push** din branch til GitHub
2. Gå til repository på github.com
3. Klik **"Compare & pull request"**
4. Udfyld titel og beskrivelse
5. Vælg reviewers
6. Klik **"Create pull request"**
7. Reviewer gennemgår koden og skriver kommentarer
8. Du retter eventuelle problemer med nye commits
9. Reviewer godkender
10. Merge ind i main

### God PR-beskrivelse indeholder

```markdown
## Hvad gør denne PR?
Tilføjer søge- og sorteringsfunktionalitet til ProductRepository.

## Ændringer
- Ny `SearchByName(string term)` metode
- Ny `GetAllSortedByPrice(bool ascending)` metode
- Unit tests for begge metoder

## Test
- Alle 9 nye unit tests er grønne
- Ingen eksisterende tests er brudt

## Screenshots
[Evt. screenshot af test output]
```

### Håndtering af merge conflicts

Konflikter opstår, når to branches ændrer de samme linjer i en fil.

```bash
# Opdater din branch med seneste main inden merge
git checkout feature/add-product-search
git merge main  # eller: git rebase main

# Hvis conflict — åbn filen og løs det manuelt:
# <<<<<<< HEAD (din kode)
# public List<Product> Search(string term) { ... }
# =======
# public IEnumerable<Product> Search(string term) { ... }
# >>>>>>> main (main's kode)

# Efter løsning:
git add ProductRepository.cs
git commit -m "Resolve merge conflict in ProductRepository"
```

---

## Branch-navngivningskonventioner

| Præfiks         | Brug                                    | Eksempel                          |
|-----------------|-----------------------------------------|-----------------------------------|
| `feature/`      | Ny funktionalitet                       | `feature/user-authentication`     |
| `bugfix/`       | Rettelse af fejl                        | `bugfix/fix-null-reference`       |
| `hotfix/`       | Kritisk rettelse til produktion         | `hotfix/security-patch`           |
| `refactor/`     | Omskrivning uden ny funktionalitet      | `refactor/clean-repository-code`  |
| `test/`         | Tilføjelse af tests                     | `test/add-product-search-tests`   |
| `docs/`         | Dokumentation                           | `docs/update-readme`              |

---

## Commit-beskeder — best practices

```bash
# Dårlig commit-besked:
git commit -m "fix stuff"
git commit -m "changes"
git commit -m "done"

# God commit-besked — imperativ, kort, beskrivende:
git commit -m "Add SearchByName method to ProductRepository"
git commit -m "Fix null reference in GetById when list is empty"
git commit -m "Refactor price sorting to use LINQ OrderBy"

# Commit med længere beskrivelse:
git commit -m "Add pagination to GetAll endpoint

Returns products in pages instead of all at once.
Default page size is 20, configurable via query parameter.

Closes #42"
```

---

## Opsummering

| Begreb              | Beskrivelse                                                         |
|---------------------|---------------------------------------------------------------------|
| LINQ Where          | Filtrerer en samling baseret på en betingelse                       |
| LINQ OrderBy        | Sorterer en samling stigende                                        |
| Named arguments     | Angiver parameter-navne eksplicit ved metodekald                    |
| Optional arguments  | Parametre med standardværdier — kan udelades ved kald               |
| Git branch          | Uafhængig udviklingslinje i et Git repository                       |
| Pull Request        | Anmodning om at merge en branch — muliggør code review              |
| Merge conflict      | To branches har ændret de samme linjer — skal løses manuelt         |

---

## Opgaver til selvstudium

1. Tilføj en `SearchByEmail(string email)` metode til din `CustomerRepository`
2. Tilføj sortering på `CustomerRepository` (sorter på navn og på oprettelsesdato)
3. Skriv unit tests for alle nye metoder
4. Opret en ny branch i dit Git-repository, tilføj en ny feature, og lav en Pull Request
5. Brug named og optional arguments i mindst to af dine metoder
