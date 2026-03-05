# Uge 12 — Vue.js + Axios CRUD

Denne uge implementerer vi komplet CRUD (Create, Read, Update, Delete) ved at kombinere Vue.js frontend med vores ASP.NET Core REST API via Axios. Vi dækker alle fire HTTP-metoder, formhåndtering i Vue.js, og hvordan man holder UI'en synkroniseret med backend-data. Efter denne uge har du alle byggeklodserne til at lave en fuldt funktionel webapplikation.

---

## CRUD Overblik

| Operation | HTTP-metode | URL                  | Status-kode   |
|-----------|-------------|----------------------|---------------|
| Read all  | GET         | `/api/products`      | 200 OK        |
| Read one  | GET         | `/api/products/{id}` | 200 OK / 404  |
| Create    | POST        | `/api/products`      | 201 Created   |
| Update    | PUT         | `/api/products/{id}` | 200 OK / 404  |
| Delete    | DELETE      | `/api/products/{id}` | 204 NoContent |

---

## Backend — Komplet ASP.NET Core Controller

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

    // GET /api/products
    [HttpGet]
    public async Task<ActionResult<List<Product>>> GetAll()
    {
        return Ok(await _repo.GetAllAsync());
    }

    // GET /api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _repo.GetByIdAsync(id);
        if (product == null) return NotFound();
        return Ok(product);
    }

    // POST /api/products
    [HttpPost]
    public async Task<ActionResult<Product>> Create([FromBody] Product product)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        var created = await _repo.AddAsync(product);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }

    // PUT /api/products/5
    [HttpPut("{id}")]
    public async Task<ActionResult<Product>> Update(int id, [FromBody] Product product)
    {
        if (!ModelState.IsValid) return BadRequest(ModelState);
        var updated = await _repo.UpdateAsync(id, product);
        if (updated == null) return NotFound();
        return Ok(updated);
    }

    // DELETE /api/products/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> Delete(int id)
    {
        var deleted = await _repo.DeleteAsync(id);
        if (!deleted) return NotFound();
        return NoContent();  // 204 — succes, ingen body
    }
}
```

---

## Axios POST — Opret nyt produkt

```javascript
async opretProdukt() {
    try {
        const respons = await axios.post(`${API_BASE}/products`, {
            name: this.nytProdukt.name,
            price: this.nytProdukt.price,
            stockQuantity: this.nytProdukt.stockQuantity,
            description: this.nytProdukt.description
        });

        // 201 Created — tilføj det nye produkt til listen
        this.produkter.push(respons.data);

        // Nulstil formularen
        this.nulstilForm();

        // Vis succesbesked
        this.visSucces("Produktet er oprettet!");

    } catch (e) {
        this.visFejl("Kunne ikke oprette produktet.");
        console.error(e);
    }
}
```

---

## Axios PUT — Opdater eksisterende produkt

```javascript
async opdaterProdukt() {
    try {
        const respons = await axios.put(
            `${API_BASE}/products/${this.redigereProdukt.id}`,
            this.redigereProdukt
        );

        // Find og erstat i listen
        const index = this.produkter.findIndex(p => p.id === respons.data.id);
        if (index !== -1) {
            this.produkter[index] = respons.data;
        }

        this.redigereProdukt = null;
        this.visModus = "liste";
        this.visSucces("Produktet er opdateret!");

    } catch (e) {
        this.visFejl("Kunne ikke opdatere produktet.");
    }
}
```

---

## Axios DELETE — Slet produkt

```javascript
async sletProdukt(id) {
    // Bekræft sletning hos brugeren
    if (!confirm("Er du sikker på at du vil slette dette produkt?")) return;

    try {
        await axios.delete(`${API_BASE}/products/${id}`);

        // Fjern fra listen
        this.produkter = this.produkter.filter(p => p.id !== id);
        this.visSucces("Produktet er slettet.");

    } catch (e) {
        if (e.response?.status === 404) {
            this.visFejl("Produktet findes ikke.");
        } else {
            this.visFejl("Kunne ikke slette produktet.");
        }
    }
}
```

---

## Komplet CRUD Vue.js Applikation

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Produkt Administration</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">

<div id="app" class="container py-4">

    <h1 class="mb-4">Produkt Administration</h1>

    <!-- Succesbesked -->
    <div v-if="successBesked" class="alert alert-success alert-dismissible fade show">
        {{ successBesked }}
        <button type="button" class="btn-close" @click="successBesked = null"></button>
    </div>

    <!-- Fejlbesked -->
    <div v-if="fejlBesked" class="alert alert-danger alert-dismissible fade show">
        {{ fejlBesked }}
        <button type="button" class="btn-close" @click="fejlBesked = null"></button>
    </div>

    <!-- ============================================================ -->
    <!-- OPRET PRODUKT FORMULAR -->
    <!-- ============================================================ -->
    <div class="card shadow-sm mb-4">
        <div class="card-header bg-primary text-white">
            <h5 class="mb-0">
                {{ redigeringsModus ? 'Redigér Produkt' : 'Opret Nyt Produkt' }}
            </h5>
        </div>
        <div class="card-body">
            <form @submit.prevent="redigeringsModus ? opdaterProdukt() : opretProdukt()">
                <div class="row g-3">

                    <div class="col-md-6">
                        <label class="form-label">Navn *</label>
                        <input
                            v-model="form.name"
                            type="text"
                            class="form-control"
                            :class="{ 'is-invalid': formFejl.name }"
                            placeholder="Produktnavn"
                            required
                        >
                        <div class="invalid-feedback">{{ formFejl.name }}</div>
                    </div>

                    <div class="col-md-3">
                        <label class="form-label">Pris (kr.) *</label>
                        <input
                            v-model.number="form.price"
                            type="number"
                            class="form-control"
                            :class="{ 'is-invalid': formFejl.price }"
                            min="0"
                            step="0.01"
                            required
                        >
                        <div class="invalid-feedback">{{ formFejl.price }}</div>
                    </div>

                    <div class="col-md-3">
                        <label class="form-label">Lagerantal *</label>
                        <input
                            v-model.number="form.stockQuantity"
                            type="number"
                            class="form-control"
                            min="0"
                            required
                        >
                    </div>

                    <div class="col-12">
                        <label class="form-label">Beskrivelse</label>
                        <textarea
                            v-model="form.description"
                            class="form-control"
                            rows="2"
                            placeholder="Valgfri beskrivelse..."
                        ></textarea>
                    </div>

                </div>

                <div class="mt-3 d-flex gap-2">
                    <button type="submit" class="btn btn-primary" :disabled="gemmer">
                        <span v-if="gemmer" class="spinner-border spinner-border-sm me-2"></span>
                        {{ redigeringsModus ? 'Gem ændringer' : 'Opret produkt' }}
                    </button>
                    <button
                        v-if="redigeringsModus"
                        type="button"
                        class="btn btn-secondary"
                        @click="annullerRedigering"
                    >
                        Annuller
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- ============================================================ -->
    <!-- PRODUKTLISTE -->
    <!-- ============================================================ -->

    <!-- Loading -->
    <div v-if="indlæser" class="text-center py-4">
        <div class="spinner-border text-primary"></div>
        <p class="mt-2">Henter produkter...</p>
    </div>

    <!-- Tom liste -->
    <div v-else-if="produkter.length === 0" class="alert alert-info">
        Ingen produkter endnu. Opret det første produkt ovenfor!
    </div>

    <!-- Produkttabel -->
    <div v-else class="card shadow-sm">
        <div class="card-header d-flex justify-content-between align-items-center">
            <h5 class="mb-0">Produkter ({{ produkter.length }})</h5>
            <button class="btn btn-sm btn-outline-secondary" @click="hentProdukter">
                Opdatér
            </button>
        </div>
        <div class="table-responsive">
            <table class="table table-hover mb-0">
                <thead class="table-dark">
                    <tr>
                        <th>ID</th>
                        <th>Navn</th>
                        <th>Beskrivelse</th>
                        <th>Pris</th>
                        <th>Lager</th>
                        <th>Handlinger</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="produkt in produkter" :key="produkt.id"
                        :class="{ 'table-warning': produkt.stockQuantity < 5 }">
                        <td>{{ produkt.id }}</td>
                        <td><strong>{{ produkt.name }}</strong></td>
                        <td class="text-muted">{{ produkt.description || '—' }}</td>
                        <td>{{ formatPris(produkt.price) }}</td>
                        <td>
                            <span
                                class="badge"
                                :class="produkt.stockQuantity < 5 ? 'bg-danger' : 'bg-success'"
                            >
                                {{ produkt.stockQuantity }}
                            </span>
                        </td>
                        <td>
                            <button
                                class="btn btn-sm btn-outline-primary me-1"
                                @click="startRedigering(produkt)"
                            >
                                Redigér
                            </button>
                            <button
                                class="btn btn-sm btn-outline-danger"
                                @click="sletProdukt(produkt.id)"
                                :disabled="sletterIds.includes(produkt.id)"
                            >
                                <span v-if="sletterIds.includes(produkt.id)"
                                      class="spinner-border spinner-border-sm"></span>
                                <span v-else>Slet</span>
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>

</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script>

const API_BASE = "https://localhost:7001/api";
const { createApp } = Vue;

createApp({
    data() {
        return {
            produkter: [],
            form: this.tomtForm(),
            formFejl: {},
            redigeringsModus: false,
            redigereId: null,
            indlæser: false,
            gemmer: false,
            sletterIds: [],
            successBesked: null,
            fejlBesked: null
        };
    },

    async mounted() {
        await this.hentProdukter();
    },

    methods: {
        tomtForm() {
            return {
                name: "",
                price: null,
                stockQuantity: 0,
                description: ""
            };
        },

        // ---- READ ----
        async hentProdukter() {
            this.indlæser = true;
            try {
                const res = await axios.get(`${API_BASE}/products`);
                this.produkter = res.data;
            } catch (e) {
                this.visFejl("Kunne ikke hente produkter fra serveren.");
            } finally {
                this.indlæser = false;
            }
        },

        // ---- CREATE ----
        async opretProdukt() {
            if (!this.validerForm()) return;
            this.gemmer = true;
            try {
                const res = await axios.post(`${API_BASE}/products`, this.form);
                this.produkter.push(res.data);
                this.form = this.tomtForm();
                this.visSucces(`"${res.data.name}" er oprettet!`);
            } catch (e) {
                this.visFejl("Kunne ikke oprette produktet.");
            } finally {
                this.gemmer = false;
            }
        },

        // ---- UPDATE ----
        startRedigering(produkt) {
            this.redigeringsModus = true;
            this.redigereId = produkt.id;
            // Kopier — undgå at ændre originalen direkte
            this.form = { ...produkt };
            window.scrollTo({ top: 0, behavior: "smooth" });
        },

        annullerRedigering() {
            this.redigeringsModus = false;
            this.redigereId = null;
            this.form = this.tomtForm();
            this.formFejl = {};
        },

        async opdaterProdukt() {
            if (!this.validerForm()) return;
            this.gemmer = true;
            try {
                const res = await axios.put(`${API_BASE}/products/${this.redigereId}`, this.form);
                const idx = this.produkter.findIndex(p => p.id === this.redigereId);
                if (idx !== -1) {
                    // Vue 3 — brug splice for reaktiv opdatering
                    this.produkter.splice(idx, 1, res.data);
                }
                this.annullerRedigering();
                this.visSucces("Produktet er opdateret!");
            } catch (e) {
                this.visFejl("Kunne ikke opdatere produktet.");
            } finally {
                this.gemmer = false;
            }
        },

        // ---- DELETE ----
        async sletProdukt(id) {
            if (!confirm("Er du sikker på at du vil slette dette produkt?")) return;
            this.sletterIds.push(id);
            try {
                await axios.delete(`${API_BASE}/products/${id}`);
                this.produkter = this.produkter.filter(p => p.id !== id);
                this.visSucces("Produktet er slettet.");
            } catch (e) {
                this.visFejl("Kunne ikke slette produktet.");
            } finally {
                this.sletterIds = this.sletterIds.filter(x => x !== id);
            }
        },

        // ---- VALIDERING ----
        validerForm() {
            this.formFejl = {};
            if (!this.form.name?.trim()) {
                this.formFejl.name = "Navn er påkrævet";
            }
            if (this.form.price == null || this.form.price < 0) {
                this.formFejl.price = "Pris skal være 0 eller mere";
            }
            return Object.keys(this.formFejl).length === 0;
        },

        // ---- HJÆLPEMETODER ----
        visSucces(besked) {
            this.successBesked = besked;
            this.fejlBesked = null;
            setTimeout(() => this.successBesked = null, 3000);
        },

        visFejl(besked) {
            this.fejlBesked = besked;
            this.successBesked = null;
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

## v-model Modifiers

```html
<!-- .number — konvertér input-streng til tal automatisk -->
<input v-model.number="form.price" type="number">

<!-- .trim — fjern whitespace fra begge sider -->
<input v-model.trim="form.name" type="text">

<!-- .lazy — opdater kun ved blur (ikke ved hver taste-tryk) -->
<input v-model.lazy="form.description" type="text">

<!-- Kombiner:-->
<input v-model.number.lazy="form.price" type="number">
```

---

## Formhåndtering med validation

```javascript
// Avanceret validering
validerForm() {
    const fejl = {};

    if (!this.form.name || this.form.name.trim().length < 2) {
        fejl.name = "Navn skal være mindst 2 tegn";
    }
    if (this.form.name && this.form.name.length > 200) {
        fejl.name = "Navn må ikke overstige 200 tegn";
    }
    if (this.form.price == null || isNaN(this.form.price)) {
        fejl.price = "Pris er påkrævet";
    } else if (this.form.price < 0) {
        fejl.price = "Pris kan ikke være negativ";
    } else if (this.form.price > 1000000) {
        fejl.price = "Pris virker urealistisk";
    }
    if (this.form.stockQuantity < 0) {
        fejl.stockQuantity = "Lagerantal kan ikke være negativt";
    }

    this.formFejl = fejl;
    return Object.keys(fejl).length === 0;  // true = ingen fejl
}
```

---

## Reaktive opdateringer — splice vs direkte assignment

I Vue 3 er arrays reaktive, men du bør bruge `splice` til at opdatere elementer for sikker reaktivitet:

```javascript
// KORREKT — Vue registrerer ændringen
this.produkter.splice(index, 1, nytProdukt);     // Erstat ét element
this.produkter.splice(index, 1);                  // Slet ét element
this.produkter.push(nytProdukt);                  // Tilføj
this.produkter = this.produkter.filter(p => ...); // Erstat hele array

// I Vue 3 virker dette dog også (via Proxy):
this.produkter[index] = nytProdukt;  // OK i Vue 3
```

---

## Opsummering

| Begreb                  | Beskrivelse                                                         |
|-------------------------|---------------------------------------------------------------------|
| axios.post(url, data)   | HTTP POST — sender data i request body (JSON)                       |
| axios.put(url, data)    | HTTP PUT — opdaterer eksisterende ressource                         |
| axios.delete(url)       | HTTP DELETE — sletter ressource                                     |
| v-model.number          | Konverterer input til tal automatisk                                |
| v-model.trim            | Fjerner whitespace fra input-streng                                 |
| Formvalidering          | Tjek at data er korrekt INDEN det sendes til API                    |
| splice()                | Reaktiv array-opdatering i Vue                                      |
| Optimistisk opdatering  | Opdater UI straks — rul tilbage ved fejl                            |
| Spinner/disabled        | Vis feedback mens en operation er i gang                            |

---

## Opgaver til selvstudium

1. Implementer komplet CRUD i din Vue.js + Axios app mod dit eget API
2. Tilføj formvalidering for alle felter med brugervenlinge fejlbeskeder
3. Tilføj loading-spinners på knapper under asynkrone operationer
4. Implementer en bekræftelsesdialog FØR sletning
5. Brug `v-bind:class` til at farve rækker i tabellen baseret på lager (f.eks. rød ved under 5)
