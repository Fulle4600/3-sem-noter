# Uge 15 — Sortering: Frontend vs Backend

Denne uge undersøger vi et centralt arkitekturspørgsmål: Skal sortering og filtrering ske i frontend (JavaScript/Vue.js) eller backend (REST API)? Begge tilgange har fordele og ulemper, og valget afhænger af datamængde, performance-krav og kompleksitet. Vi implementerer begge løsninger og diskuterer hvornår man bør vælge hvad.

---

## Query Parameters i REST API

Inden vi sammenligner de to tilgange, skal vi forstå **query parameters** — den mekanisme der bruges til at sende filtrerings- og sorteringsinstruktioner til backend.

### Hvad er query parameters?

Query parameters tilføjes til en URL efter `?` og adskilles med `&`:

```
https://api.example.com/api/products?sortBy=price&ascending=false&search=laptop&maxPrice=5000
```

| Parameter  | Værdi   |
|------------|---------|
| sortBy     | price   |
| ascending  | false   |
| search     | laptop  |
| maxPrice   | 5000    |

### ASP.NET Core — modtag query parameters

```csharp
[HttpGet]
public async Task<ActionResult<List<Product>>> GetAll(
    [FromQuery] string? search = null,
    [FromQuery] string sortBy = "name",
    [FromQuery] bool ascending = true,
    [FromQuery] decimal? minPrice = null,
    [FromQuery] decimal? maxPrice = null,
    [FromQuery] int page = 1,
    [FromQuery] int pageSize = 20)
{
    var produkter = await _repo.GetAllAsync();
    IEnumerable<Product> query = produkter;

    // Filtrering
    if (!string.IsNullOrWhiteSpace(search))
        query = query.Where(p => p.Name.Contains(search, StringComparison.OrdinalIgnoreCase));

    if (minPrice.HasValue)
        query = query.Where(p => p.Price >= minPrice.Value);

    if (maxPrice.HasValue)
        query = query.Where(p => p.Price <= maxPrice.Value);

    // Sortering
    query = sortBy.ToLower() switch
    {
        "price"   => ascending ? query.OrderBy(p => p.Price) : query.OrderByDescending(p => p.Price),
        "name"    => ascending ? query.OrderBy(p => p.Name)  : query.OrderByDescending(p => p.Name),
        "stock"   => ascending ? query.OrderBy(p => p.StockQuantity) : query.OrderByDescending(p => p.StockQuantity),
        _         => query.OrderBy(p => p.Id)
    };

    // Paginering
    var total = query.Count();
    var result = query.Skip((page - 1) * pageSize).Take(pageSize).ToList();

    // Tilføj metadata i response headers
    Response.Headers.Add("X-Total-Count", total.ToString());
    Response.Headers.Add("X-Page", page.ToString());
    Response.Headers.Add("X-Page-Size", pageSize.ToString());

    return Ok(result);
}
```

### Axios — send query parameters

```javascript
// Eksplicitte params
const res = await axios.get(`${API_BASE}/products`, {
    params: {
        search: "laptop",
        sortBy: "price",
        ascending: false,
        minPrice: 1000,
        maxPrice: 10000
    }
});
// Resulterer i: /api/products?search=laptop&sortBy=price&ascending=false&minPrice=1000&maxPrice=10000
```

---

## Tilgang 1: Frontend Sortering/Filtrering

Her hentes ALT data fra API'et, og sortering/filtrering sker i JavaScript/Vue.js.

### Vue.js Implementation

```html
<div id="app" class="container py-4">

    <h1>Produkter (Frontend Sortering)</h1>

    <!-- Søg og sortér -->
    <div class="row mb-4 g-3">
        <div class="col-md-4">
            <input v-model="søgeTerm" class="form-control" placeholder="Søg...">
        </div>
        <div class="col-md-3">
            <select v-model="sorterEfter" class="form-select">
                <option value="name">Navn</option>
                <option value="price">Pris</option>
                <option value="stockQuantity">Lager</option>
            </select>
        </div>
        <div class="col-md-2">
            <select v-model="stigende" class="form-select">
                <option :value="true">Stigende</option>
                <option :value="false">Faldende</option>
            </select>
        </div>
        <div class="col-md-3">
            <input v-model.number="maxPris" type="number" class="form-control" placeholder="Max pris">
        </div>
    </div>

    <p class="text-muted">
        Viser {{ filtreredeSorteredeProdukter.length }} af {{ alleProdukter.length }} produkter
    </p>

    <!-- Produktliste -->
    <div class="row row-cols-1 row-cols-md-3 g-4">
        <div class="col" v-for="p in filtreredeSorteredeProdukter" :key="p.id">
            <div class="card h-100 shadow-sm">
                <div class="card-body">
                    <h5 class="card-title">{{ p.name }}</h5>
                    <p class="text-success fw-bold">{{ formatPris(p.price) }}</p>
                </div>
            </div>
        </div>
    </div>

</div>

<script>
createApp({
    data() {
        return {
            alleProdukter: [],    // Alle produkter hentet fra API
            søgeTerm: "",
            sorterEfter: "name",
            stigende: true,
            maxPris: null
        };
    },

    async mounted() {
        // Hent ALT data én gang
        const res = await axios.get(`${API_BASE}/products`);
        this.alleProdukter = res.data;
    },

    computed: {
        // Filtrering og sortering sker udelukkende i JavaScript
        filtreredeSorteredeProdukter() {
            let result = [...this.alleProdukter];  // Kopier array

            // Filtrér på søgeTerm
            if (this.søgeTerm.trim()) {
                const term = this.søgeTerm.toLowerCase();
                result = result.filter(p => p.name.toLowerCase().includes(term));
            }

            // Filtrér på maxPris
            if (this.maxPris != null && this.maxPris > 0) {
                result = result.filter(p => p.price <= this.maxPris);
            }

            // Sortér
            result.sort((a, b) => {
                let valA = a[this.sorterEfter];
                let valB = b[this.sorterEfter];

                if (typeof valA === "string") valA = valA.toLowerCase();
                if (typeof valB === "string") valB = valB.toLowerCase();

                if (valA < valB) return this.stigende ? -1 : 1;
                if (valA > valB) return this.stigende ? 1 : -1;
                return 0;
            });

            return result;
        }
    }
}).mount("#app");
</script>
```

---

## Tilgang 2: Backend Sortering/Filtrering

Her sendes sortering/filtrering som query parameters til API'et, og backend returnerer allerede sorterede/filtrerede data.

### Vue.js Implementation

```javascript
createApp({
    data() {
        return {
            produkter: [],         // Kun det der vises nu
            søgeTerm: "",
            sorterEfter: "name",
            stigende: true,
            maxPris: null,
            søgeTimeout: null,
            indlæser: false
        };
    },

    async mounted() {
        await this.hentProdukterFraBackend();
    },

    methods: {
        async hentProdukterFraBackend() {
            this.indlæser = true;
            try {
                const params = {
                    sortBy: this.sorterEfter,
                    ascending: this.stigende
                };

                if (this.søgeTerm.trim()) params.search = this.søgeTerm;
                if (this.maxPris > 0) params.maxPrice = this.maxPris;

                const res = await axios.get(`${API_BASE}/products`, { params });
                this.produkter = res.data;
            } catch (e) {
                console.error("Fejl ved hentning:", e);
            } finally {
                this.indlæser = false;
            }
        },

        // Kald API igen når søgeparametre ændres (med debounce)
        søgMedDebounce() {
            clearTimeout(this.søgeTimeout);
            this.søgeTimeout = setTimeout(() => {
                this.hentProdukterFraBackend();
            }, 400);
        }
    },

    watch: {
        // Lyt på ændringer og hent nyt data fra backend
        sorterEfter() { this.hentProdukterFraBackend(); },
        stigende()    { this.hentProdukterFraBackend(); },
        søgeTerm()    { this.søgMedDebounce(); },
        maxPris()     { this.søgMedDebounce(); }
    }
}).mount("#app");
```

---

## Sammenligning: Frontend vs Backend

### Frontend Sortering/Filtrering

**Fordele:**
- Ingen ekstra API-kald ved sortering/filtrering
- Øjeblikkelig respons — ingen netværkslatens
- Simpel at implementere
- Offline capable — data er allerede hentet

**Ulemper:**
- Skalerer dårligt — hvad hvis der er 50.000 produkter?
- Unødvendigt meget data overføres
- Høj hukommelsesbrug i browseren
- Søgemaskiner (SEO) ser ikke filtrerede resultater

**Hvornår:**
- Få data (< 500-1000 elementer)
- Data ændres sjældent
- Hurtig, responsiv filtrering er vigtig
- Interne admin-tools

---

### Backend Sortering/Filtrering

**Fordele:**
- Skalerer til millioner af rækker (database-indeks!)
- Kun relevant data overføres
- Kan kombineres med paginering
- Serverside caching mulig
- SEO-venlig (søgemaskiner kan indeksere filtrerede sider)

**Ulemper:**
- API-kald ved hver ændring — netværkslatens
- Mere kompleks implementation
- Kræver debounce for søgefelter
- Loading-spinner nødvendig

**Hvornår:**
- Mange data (> 1000 elementer)
- Data ændres ofte
- Paginering ønskes
- Søgeoptimering er vigtig

---

## Paginering i Vue.js

En vigtig tilføjelse til backend-sortering er paginering:

```html
<!-- Pagineringskomponent -->
<nav v-if="totalAntal > sidestørrelse" class="mt-4">
    <ul class="pagination justify-content-center">

        <li class="page-item" :class="{ disabled: nuværendeSide === 1 }">
            <button class="page-link" @click="skiftSide(nuværendeSide - 1)">
                Forrige
            </button>
        </li>

        <li
            v-for="side in antalSider"
            :key="side"
            class="page-item"
            :class="{ active: side === nuværendeSide }"
        >
            <button class="page-link" @click="skiftSide(side)">{{ side }}</button>
        </li>

        <li class="page-item" :class="{ disabled: nuværendeSide === antalSider }">
            <button class="page-link" @click="skiftSide(nuværendeSide + 1)">
                Næste
            </button>
        </li>

    </ul>
    <p class="text-center text-muted">
        Side {{ nuværendeSide }} af {{ antalSider }} ({{ totalAntal }} produkter i alt)
    </p>
</nav>
```

```javascript
data() {
    return {
        produkter: [],
        nuværendeSide: 1,
        sidestørrelse: 12,
        totalAntal: 0
    };
},

computed: {
    antalSider() {
        return Math.ceil(this.totalAntal / this.sidestørrelse);
    }
},

methods: {
    async hentProdukter() {
        const res = await axios.get(`${API_BASE}/products`, {
            params: {
                page: this.nuværendeSide,
                pageSize: this.sidestørrelse,
                sortBy: this.sorterEfter
            }
        });
        this.produkter = res.data;
        // Hent total fra response header
        this.totalAntal = parseInt(res.headers["x-total-count"] || "0");
    },

    async skiftSide(side) {
        if (side < 1 || side > this.antalSider) return;
        this.nuværendeSide = side;
        await this.hentProdukter();
        window.scrollTo({ top: 0, behavior: "smooth" });
    }
}
```

---

## EF Core — Sortering og filtrering direkte i databaseforespørgsel

Når vi bruger EF Core, kan vi lade databasen klare sortering og filtrering — meget mere effektivt end at hente alt og sortere i C#.

```csharp
public async Task<(List<Product> Items, int Total)> GetPagedAsync(
    string? search,
    string sortBy,
    bool ascending,
    decimal? maxPrice,
    int page,
    int pageSize)
{
    // Start med IQueryable — ingen databasekald endnu
    IQueryable<Product> query = _context.Products;

    // Tilføj WHERE-klausuler
    if (!string.IsNullOrEmpty(search))
        query = query.Where(p => p.Name.Contains(search));

    if (maxPrice.HasValue)
        query = query.Where(p => p.Price <= maxPrice.Value);

    // Tæl total FØR paginering
    int total = await query.CountAsync();

    // Tilføj ORDER BY
    query = (sortBy.ToLower(), ascending) switch
    {
        ("price", true)   => query.OrderBy(p => p.Price),
        ("price", false)  => query.OrderByDescending(p => p.Price),
        ("name", true)    => query.OrderBy(p => p.Name),
        ("name", false)   => query.OrderByDescending(p => p.Name),
        _                 => query.OrderBy(p => p.Id)
    };

    // SKIP/TAKE = SQL OFFSET/FETCH (paginering i databasen)
    var items = await query
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();

    return (items, total);
    // Genereret SQL: SELECT TOP 12 * FROM Products WHERE Price <= 5000 ORDER BY Price
}
```

---

## Vue.js `watch` — Lyt på data-ændringer

```javascript
watch: {
    // Simpel watch — kører når søgeTerm ændres
    søgeTerm(nyVærdi, gammelVærdi) {
        console.log(`Søgterm ændret: ${gammelVærdi} -> ${nyVærdi}`);
        this.søgMedDebounce();
    },

    // Deep watch — observer ændringer i objekt/array
    filter: {
        handler(nyVærdi) {
            this.hentProdukter();
        },
        deep: true  // Overvåg alle properties i objektet
    },

    // Immediate — kør straks ved opstart
    søgeTerm: {
        handler(nyVærdi) {
            this.søgMedDebounce();
        },
        immediate: false  // Kør ikke ved mount (vi håndterer det i mounted())
    }
}
```

---

## Opsummering

| Aspekt                   | Frontend                          | Backend                             |
|--------------------------|-----------------------------------|-------------------------------------|
| Implementeringskompleksitet | Lav — kun JavaScript           | Højere — query params + controller  |
| Performance ved mange data | Dårlig                          | God — database-indeks               |
| Netværkstrafik           | Én stor request                   | Mange små requests                  |
| Brugeroplevelse          | Øjeblikkelig (ingen API-kald)     | Lille forsinkelse (loading-spinner) |
| Skalering                | Dårlig (browser-limit)            | God (server/database)               |
| Paginering               | Svær at implementere korrekt      | Naturlig del af backend-løsningen   |
| Anbefaling               | < 500-1000 rækker, simpel data    | > 1000 rækker, kompleks filtrering  |

---

## Opgaver til selvstudium

1. Implementer frontend-sortering i din Vue.js app med computed properties
2. Tilføj query parameter-understøttelse til dit ASP.NET Core API (search, sortBy, ascending)
3. Implementer backend-sortering med Vue.js watch og Axios
4. Tilføj paginering til backend-tilgangen
5. Mål forskellen: brug browser-devtools til at sammenligne request-størrelse og -antal
