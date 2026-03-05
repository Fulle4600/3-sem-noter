# Uge 11 — Vue.js + Axios (GET)

Denne uge forbinder vi vores Vue.js frontend med vores ASP.NET Core REST API via Axios — et populært HTTP-klientbibliotek til JavaScript. Vi lærer async/await mønsteret i JavaScript, forstår hvad CORS er og hvorfor det er vigtigt, og bygger en komplet produktliste der henter data fra en rigtig backend. Dette er grundstenen i moderne webapplikationer.

---

## Hvad er Axios?

**Axios** er et promise-baseret HTTP-klientbibliotek til JavaScript. Det bruges til at lave HTTP-requests (GET, POST, PUT, DELETE) fra browseren til en REST API.

### Fordele ved Axios frem for den native `fetch()`

| Funktion              | Axios                      | fetch()                         |
|-----------------------|----------------------------|---------------------------------|
| JSON auto-parsing     | Ja — automatisk            | Nej — kræver `.json()` manuelt  |
| Fejlhåndtering        | Kaster fejl ved 4xx/5xx    | Kaster kun ved netværksfejl     |
| Request/Response      | Interceptors understøttet  | Ikke built-in                   |
| Timeout               | Nem konfiguration          | Kræver AbortController          |
| Browser support       | God                        | God (moderne browsere)          |

### Inkludér Axios via CDN

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

---

## Async/Await i JavaScript

Moderne JavaScript bruger **async/await** til at håndtere asynkrone operationer (som HTTP-requests). Det gør asynkron kode der læses som synkron kode.

### Promises — baggrunden

```javascript
// En Promise repræsenterer en fremtidig værdi
// Den er enten: pending, fulfilled (resolved), eller rejected

const løfte = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve("Data hentet!");  // Succes
        // reject("Noget gik galt!"); // Fejl
    }, 1000);
});

// .then() til at håndtere succes, .catch() til fejl
løfte
    .then(data => console.log(data))
    .catch(fejl => console.error(fejl));
```

### Async/Await — den moderne måde

```javascript
// async markerer at en funktion returnerer en Promise
async function hentData() {
    try {
        // await venter på at Promise'en er resolved
        const respons = await axios.get("https://api.example.com/produkter");
        console.log(respons.data);  // Axios parser JSON automatisk
    } catch (fejl) {
        // Kører hvis request fejler (4xx, 5xx, netværksfejl)
        console.error("Fejl:", fejl.message);
    }
}
```

### Vigtige regler for async/await

1. `await` kan KUN bruges inde i en `async`-funktion
2. `await` venter på en Promise uden at blokere browseren
3. Brug altid `try/catch` til fejlhåndtering
4. En `async`-funktion returnerer altid en Promise

---

## Axios GET — Hent data fra REST API

### Simpel GET-request

```javascript
async function hentAlleProdukter() {
    try {
        const respons = await axios.get("https://localhost:7001/api/products");
        // respons.data indeholder den parsede JSON
        console.log(respons.data);
        return respons.data;
    } catch (fejl) {
        if (fejl.response) {
            // Server svarede med fejlkode
            console.error(`HTTP fejl: ${fejl.response.status}`);
        } else if (fejl.request) {
            // Request blev sendt men ingen svar (netværksfejl)
            console.error("Ingen svar fra serveren");
        } else {
            // Noget andet gik galt
            console.error("Fejl:", fejl.message);
        }
    }
}
```

### GET med query parameters

```javascript
// Hent produkter med søgeterm og sortering
const respons = await axios.get("https://localhost:7001/api/products", {
    params: {
        search: "laptop",
        sortBy: "price",
        ascending: true
    }
});
// Resulterende URL: /api/products?search=laptop&sortBy=price&ascending=true
```

### GET enkelt produkt (by id)

```javascript
async function hentProdukt(id) {
    try {
        const respons = await axios.get(`https://localhost:7001/api/products/${id}`);
        return respons.data;
    } catch (fejl) {
        if (fejl.response?.status === 404) {
            console.warn(`Produkt med id ${id} blev ikke fundet`);
            return null;
        }
        throw fejl;
    }
}
```

---

## CORS — Cross-Origin Resource Sharing

**CORS** er en sikkerhedsmekanisme i browseren der forhindrer JavaScript på `http://siteA.com` i at lave requests til `http://siteB.com` — medmindre siteB eksplicit tillader det.

### Problemet

```
Frontend: http://localhost:5500 (HTML/Vue.js)
Backend:  https://localhost:7001 (ASP.NET Core API)

Browser: "Vent! Din frontend forsøger at hente data fra et andet origin!
          Jeg tillader det kun, hvis API'et siger OK."
```

### Løsning — Konfigurer CORS i ASP.NET Core

```csharp
// Program.cs

var builder = WebApplication.CreateBuilder(args);

// 1. Definer CORS-politik
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy
            .WithOrigins("http://localhost:5500", "http://127.0.0.1:5500")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});

builder.Services.AddControllers();

var app = builder.Build();

// 2. Anvend CORS-politikken (SKAL stå FØR MapControllers)
app.UseCors("AllowFrontend");
app.MapControllers();

app.Run();
```

### Tillad alle origins (kun til udvikling!)

```csharp
// Til lokal udvikling — ALDRIG i produktion!
policy.AllowAnyOrigin().AllowAnyHeader().AllowAnyMethod();
```

---

## Komplet Vue.js + Axios Eksempel

### Backend — ASP.NET Core Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repo;

    public ProductsController(IProductRepository repo)
    {
        _repo = repo;
    }

    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetAll()
    {
        var produkter = await _repo.GetAllAsync();
        return Ok(produkter);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var produkt = await _repo.GetByIdAsync(id);
        if (produkt == null) return NotFound();
        return Ok(produkt);
    }
}
```

### Frontend — HTML + Vue.js + Axios

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Produktliste</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<div id="app" class="container py-4">

    <h1 class="mb-4">Produkter</h1>

    <!-- Søgefelt -->
    <div class="mb-4">
        <input
            v-model="søgeTerm"
            @input="søg"
            class="form-control"
            placeholder="Søg produkter..."
        >
    </div>

    <!-- Loading indikator -->
    <div v-if="indlæser" class="text-center py-5">
        <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">Indlæser...</span>
        </div>
        <p class="mt-2">Henter produkter...</p>
    </div>

    <!-- Fejlbesked -->
    <div v-else-if="fejlBesked" class="alert alert-danger">
        {{ fejlBesked }}
        <button class="btn btn-sm btn-outline-danger ms-2" @click="hentProdukter">
            Prøv igen
        </button>
    </div>

    <!-- Ingen resultater -->
    <div v-else-if="produkter.length === 0" class="alert alert-info">
        Ingen produkter fundet.
    </div>

    <!-- Produktgrid -->
    <div v-else class="row row-cols-1 row-cols-md-3 g-4">
        <div class="col" v-for="produkt in produkter" :key="produkt.id">
            <div class="card h-100 shadow-sm">
                <div class="card-body">
                    <h5 class="card-title">{{ produkt.name }}</h5>
                    <p class="card-text text-muted">{{ produkt.description }}</p>
                    <p class="card-text">
                        <span class="badge bg-secondary">Lager: {{ produkt.stockQuantity }}</span>
                    </p>
                    <p class="text-success fw-bold fs-5">
                        {{ formatPris(produkt.price) }}
                    </p>
                </div>
                <div class="card-footer bg-white border-0 pb-3">
                    <button
                        class="btn btn-primary w-100"
                        @click="visDetaljer(produkt)"
                    >
                        Se detaljer
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Detalje-modal (Bootstrap) -->
    <div class="modal fade" id="detaljeModal" tabindex="-1" ref="modal">
        <div class="modal-dialog">
            <div class="modal-content" v-if="valgtProdukt">
                <div class="modal-header">
                    <h5 class="modal-title">{{ valgtProdukt.name }}</h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
                </div>
                <div class="modal-body">
                    <p>{{ valgtProdukt.description }}</p>
                    <p><strong>Pris:</strong> {{ formatPris(valgtProdukt.price) }}</p>
                    <p><strong>Lager:</strong> {{ valgtProdukt.stockQuantity }} stk.</p>
                </div>
                <div class="modal-footer">
                    <button class="btn btn-secondary" data-bs-dismiss="modal">Luk</button>
                    <button class="btn btn-primary">Læg i kurv</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Status-linje -->
    <div class="mt-4 text-muted">
        <small>Viser {{ produkter.length }} produkt(er)</small>
    </div>

</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
<script>

const API_BASE = "https://localhost:7001/api";

const { createApp } = Vue;

createApp({
    data() {
        return {
            produkter: [],
            valgtProdukt: null,
            søgeTerm: "",
            indlæser: false,
            fejlBesked: null,
            søgeTimeout: null
        };
    },

    // mounted() kører når komponenten er sat ind i DOM — perfekt til initial data-hentning
    async mounted() {
        await this.hentProdukter();
    },

    methods: {
        async hentProdukter(søg = null) {
            this.indlæser = true;
            this.fejlBesked = null;

            try {
                const params = {};
                if (søg) params.search = søg;

                const respons = await axios.get(`${API_BASE}/products`, { params });
                this.produkter = respons.data;
            } catch (fejl) {
                this.fejlBesked = "Kunne ikke hente produkter. Tjek at API'et kører.";
                console.error("API fejl:", fejl);
            } finally {
                // finally kører altid — uanset fejl eller ej
                this.indlæser = false;
            }
        },

        // Debounce søgning — vent 300ms efter bruger stopper med at skrive
        søg() {
            clearTimeout(this.søgeTimeout);
            this.søgeTimeout = setTimeout(() => {
                this.hentProdukter(this.søgeTerm || null);
            }, 300);
        },

        visDetaljer(produkt) {
            this.valgtProdukt = produkt;
            // Åbn Bootstrap modal
            const modalEl = document.getElementById("detaljeModal");
            const modal = new bootstrap.Modal(modalEl);
            modal.show();
        },

        formatPris(pris) {
            return new Intl.NumberFormat("da-DK", {
                style: "currency",
                currency: "DKK"
            }).format(pris);
        }
    }
}).mount("#app");

</script>
</body>
</html>
```

---

## Axios Konfiguration (Base URL)

Til større projekter er det smart at konfigurere en Axios-instans med base URL:

```javascript
// Opret en konfigureret Axios-instans
const api = axios.create({
    baseURL: "https://localhost:7001/api",
    timeout: 10000,  // 10 sekunder timeout
    headers: {
        "Content-Type": "application/json"
    }
});

// Brug den i stedet for global axios
const respons = await api.get("/products");
const respons2 = await api.get("/products/1");
```

### Interceptors — kør kode på alle requests/responses

```javascript
// Request interceptor — kør FØR request sendes
api.interceptors.request.use(config => {
    // Tilføj f.eks. authentication token
    const token = localStorage.getItem("token");
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// Response interceptor — kør ved hvert svar
api.interceptors.response.use(
    response => response,  // Succes — returner uændret
    error => {
        if (error.response?.status === 401) {
            // Uautoriseret — omdiriger til login
            window.location.href = "/login.html";
        }
        return Promise.reject(error);
    }
);
```

---

## Vue.js Lifecycle Hooks

Lifecycle hooks er funktioner der kører på bestemte tidspunkter i en komponents liv:

```javascript
createApp({
    data() {
        return { data: null };
    },

    // Kører FØR komponenten er mounted i DOM
    // Bruges sjældent
    beforeMount() {
        console.log("Komponenten er ved at blive mounted");
    },

    // Kører EFTER komponenten er mounted i DOM
    // BEDST til at hente data fra API
    async mounted() {
        console.log("Komponenten er mounted — henter data");
        await this.hentData();
    },

    // Kører FØR komponenten fjernes fra DOM
    beforeUnmount() {
        // Ryd op — annuller timers, luk forbindelser osv.
        clearInterval(this.timer);
    },

    methods: {
        async hentData() {
            const res = await axios.get("/api/data");
            this.data = res.data;
        }
    }
}).mount("#app");
```

---

## Fejlhåndtering — Best Practices

```javascript
methods: {
    async hentProdukter() {
        this.indlæser = true;
        this.fejl = null;

        try {
            const res = await axios.get(`${API_BASE}/products`);
            this.produkter = res.data;
        } catch (e) {
            // Brugervenllige fejlbeskeder baseret på fejltype
            if (!e.response) {
                this.fejl = "Ingen forbindelse til serveren. Tjek din internetforbindelse.";
            } else if (e.response.status === 404) {
                this.fejl = "Ressourcen blev ikke fundet.";
            } else if (e.response.status === 500) {
                this.fejl = "Der opstod en serverfejl. Prøv igen senere.";
            } else {
                this.fejl = `Fejl: ${e.response.status} ${e.response.statusText}`;
            }
        } finally {
            this.indlæser = false;
        }
    }
}
```

---

## Opsummering

| Begreb             | Beskrivelse                                                       |
|--------------------|-------------------------------------------------------------------|
| Axios              | HTTP-klientbibliotek til at lave requests fra JavaScript          |
| Promise            | Repræsenterer en fremtidig asynkron værdi                         |
| async/await        | Syntaks der gør asynkron kode læsbar som synkron                  |
| CORS               | Sikkerhedsmekanisme — API skal eksplicit tillade frontend-origins |
| mounted()          | Vue lifecycle hook — kører når komponent er sat i DOM             |
| try/catch/finally  | Fejlhåndtering: prøv, fang fejl, kør altid                       |
| respons.data       | Det parsede JSON-svar fra Axios                                   |
| params             | Query parameters i Axios GET-request                              |
| Debounce           | Forsink eksekvering af funktion — undgå for mange API-kald        |

---

## Opgaver til selvstudium

1. Start dit ASP.NET Core API og verificer at det kører på `https://localhost:7001`
2. Konfigurer CORS til at tillade `localhost:5500`
3. Byg en HTML/Vue.js side der henter og viser alle produkter via Axios GET
4. Tilføj en loading-spinner og fejlhåndtering
5. Tilføj debounce-søgning — send søgeterm som query parameter til API'et
6. Brug `Intl.NumberFormat` til at formatere priser i DKK
