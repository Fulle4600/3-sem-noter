# Uge 10 — REST Filter, Sortering & Postman Tests

Denne uge ser vi på avancerede REST-mønstre: filtrering og sortering via query parameters. Vi dykker desuden ned i Postman som testværktøj — collections, testscripts og integrationstests, som er en vigtig del af API-udvikling.

---

## 1. Query Parameters i REST

### Hvad er query parameters?

Query parameters er en del af URL'en og bruges til at **modificere** en requests opførsel — typisk til filtrering, sortering og paginering. De skrives efter `?` og adskilles med `&`.

```
https://api.example.com/api/brugere?navn=Anna&aktiv=true&sort=alder&order=asc
                                    ^          ^           ^          ^
                                    filter     filter      sortering  retning
```

Strukturen:
```
URL  ?  nøgle1=værdi1  &  nøgle2=værdi2  &  nøgle3=værdi3
```

---

## 2. Filtrering med Query Parameters

### Grundlæggende filtrering

```
GET /api/brugere?aktiv=true             → Alle aktive brugere
GET /api/brugere?by=Kobenhavn           → Brugere fra København
GET /api/produkter?kategori=elektronik  → Elektronikprodukter
GET /api/brugere?navn=Anna              → Brugere navngivet Anna
```

### Kombination af filtre (AND-logik)

```
GET /api/brugere?aktiv=true&by=Kobenhavn
→ Aktive brugere fra København (begge betingelser skal være opfyldt)
```

### Range-filtrering

```
GET /api/produkter?min_pris=100&max_pris=500
→ Produkter der koster 100-500 kr

GET /api/ordrer?fra=2024-01-01&til=2024-12-31
→ Ordrer fra 2024
```

### Java Spring Boot implementation

```java
@GetMapping
public List<Bruger> getBrugere(
        @RequestParam(required = false) String navn,
        @RequestParam(required = false) Boolean aktiv,
        @RequestParam(required = false) String by) {

    if (navn != null && aktiv != null) {
        return brugerRepo.findByNavnContainingAndAktiv(navn, aktiv);
    } else if (aktiv != null) {
        return brugerRepo.findByAktiv(aktiv);
    } else if (navn != null) {
        return brugerRepo.findByNavnContaining(navn);
    }
    return brugerRepo.findAll();
}
```

### Med Spring Data JPA Specification (mere avanceret)

```java
@GetMapping
public List<Bruger> getBrugere(
        @RequestParam Map<String, String> params) {

    Specification<Bruger> spec = Specification.where(null);

    if (params.containsKey("navn")) {
        spec = spec.and((root, query, cb) ->
            cb.like(root.get("navn"), "%" + params.get("navn") + "%"));
    }
    if (params.containsKey("aktiv")) {
        boolean aktiv = Boolean.parseBoolean(params.get("aktiv"));
        spec = spec.and((root, query, cb) ->
            cb.equal(root.get("aktiv"), aktiv));
    }

    return brugerRepo.findAll(spec);
}
```

---

## 3. Sortering med Query Parameters

### Sorteringskonventioner

Der er ingen officiel REST-standard for sortering, men gængse konventioner er:

```
GET /api/brugere?sort=navn                  → Sorter efter navn (stigende)
GET /api/brugere?sort=navn&order=asc        → Eksplicit stigende
GET /api/brugere?sort=navn&order=desc       → Faldende
GET /api/produkter?sort=pris,desc           → Alternativ notation
GET /api/brugere?sort=efternavn,asc,navn,asc → Multi-kolonne sortering
```

### Spring Boot sortering med Pageable

```java
@GetMapping
public Page<Bruger> getBrugere(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String sortDir) {

    Sort.Direction direction = sortDir.equalsIgnoreCase("desc")
        ? Sort.Direction.DESC
        : Sort.Direction.ASC;

    Pageable pageable = PageRequest.of(page, size, Sort.by(direction, sortBy));
    return brugerRepo.findAll(pageable);
}
```

URL'er der understøttes:
```
GET /api/brugere                             → Side 0, 10 per side, sorteret efter id
GET /api/brugere?page=1&size=5               → Side 1, 5 per side
GET /api/brugere?sortBy=navn&sortDir=desc    → Sorteret efter navn, faldende
```

---

## 4. Paginering

Paginering opdeler store datasæt i sider for at undgå at sende for meget data ad gangen.

### Pagination response format

```json
{
  "content": [
    { "id": 1, "navn": "Anna" },
    { "id": 2, "navn": "Bo" }
  ],
  "pageable": {
    "pageNumber": 0,
    "pageSize": 10
  },
  "totalElements": 247,
  "totalPages": 25,
  "first": true,
  "last": false
}
```

### Link-baseret paginering (HATEOAS-stil)

```json
{
  "data": [...],
  "pagination": {
    "current_page": 2,
    "per_page": 10,
    "total": 247,
    "links": {
      "prev": "/api/brugere?page=1&size=10",
      "next": "/api/brugere?page=3&size=10",
      "first": "/api/brugere?page=0&size=10",
      "last": "/api/brugere?page=24&size=10"
    }
  }
}
```

---

## 5. Postman Collections

### Hvad er en Postman Collection?

En **collection** er en organiseret samling af relaterede HTTP-requests i Postman. Det er som en mappe med alle dine API-tests.

```
Collection: Bruger API
├── Bruger CRUD
│   ├── GET Alle brugere
│   ├── GET Bruger med ID
│   ├── POST Opret bruger
│   ├── PUT Opdater bruger
│   └── DELETE Slet bruger
└── Auth
    ├── POST Login
    └── POST Logout
```

### Postman Environments

**Environments** lader dig gemme **variabler** der kan ændres uden at redigere requests.

```
Environment: "Local Development"
  baseUrl = http://localhost:8080
  token   = (opdateres automatisk ved login)

Environment: "Production"
  baseUrl = https://api.example.com
  token   = (produktions-token)
```

Brug variabler i URL: `{{baseUrl}}/api/brugere`

---

## 6. Postman Tests (JavaScript)

### Hvad er Postman Tests?

Postman lader dig skrive **JavaScript-tests** der kører efter hvert request. Brug fanen "Tests" i Postman.

### Test syntax

```javascript
// Basisprincipper:
pm.test("Beskrivelse af test", function() {
    pm.expect(noget).to.equal(forventet);
});
```

### Test eksempler

```javascript
// Test 1: Tjek statuskode
pm.test("Status code er 200", function() {
    pm.response.to.have.status(200);
});

// Test 2: Tjek response er JSON
pm.test("Response er JSON", function() {
    pm.response.to.be.json;
});

// Test 3: Tjek specifikt felt i response
pm.test("Bruger har korrekt navn", function() {
    const json = pm.response.json();
    pm.expect(json.navn).to.equal("Anna Larsen");
});

// Test 4: Tjek at response er et array
pm.test("Response er et array", function() {
    const json = pm.response.json();
    pm.expect(json).to.be.an("array");
});

// Test 5: Tjek at array ikke er tomt
pm.test("Response array er ikke tomt", function() {
    const json = pm.response.json();
    pm.expect(json.length).to.be.greaterThan(0);
});

// Test 6: Tjek svartid
pm.test("Response tid er under 500ms", function() {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Test 7: Tjek Content-Type header
pm.test("Content-Type er application/json", function() {
    pm.expect(pm.response.headers.get("Content-Type"))
      .to.include("application/json");
});
```

### Gem variabel fra response

```javascript
// Pre-request Script: (kører FØR request)
pm.environment.set("timestamp", Date.now());

// Tests: (kører EFTER request, gem token til næste request)
pm.test("Login lykkedes", function() {
    pm.response.to.have.status(200);
});

const json = pm.response.json();
pm.environment.set("token", json.token);
// Nu kan næste request bruge {{token}}
```

---

## 7. Integrationstests med Postman

### Testscenarie: Komplet CRUD-flow

```javascript
// === Request 1: POST /api/brugere (Opret bruger) ===
// Tests:
pm.test("Status 201 Created", function() {
    pm.response.to.have.status(201);
});

pm.test("Bruger har ID", function() {
    const json = pm.response.json();
    pm.expect(json.id).to.be.a("number");
    // Gem ID til næste request
    pm.environment.set("brugerId", json.id);
});

pm.test("Bruger har korrekt navn", function() {
    const json = pm.response.json();
    pm.expect(json.navn).to.equal("Test Bruger");
});
```

```javascript
// === Request 2: GET /api/brugere/{{brugerId}} (Hent bruger) ===
// Tests:
pm.test("Status 200 OK", function() {
    pm.response.to.have.status(200);
});

pm.test("Korrekt bruger returneret", function() {
    const json = pm.response.json();
    pm.expect(json.id).to.equal(pm.environment.get("brugerId"));
    pm.expect(json.navn).to.equal("Test Bruger");
});
```

```javascript
// === Request 3: PUT /api/brugere/{{brugerId}} (Opdater bruger) ===
// Tests:
pm.test("Status 200 OK", function() {
    pm.response.to.have.status(200);
});

pm.test("Navn er opdateret", function() {
    const json = pm.response.json();
    pm.expect(json.navn).to.equal("Opdateret Bruger");
});
```

```javascript
// === Request 4: DELETE /api/brugere/{{brugerId}} (Slet bruger) ===
// Tests:
pm.test("Status 204 No Content", function() {
    pm.response.to.have.status(204);
});
```

```javascript
// === Request 5: GET /api/brugere/{{brugerId}} (Verificer slettet) ===
// Tests:
pm.test("Status 404 Not Found", function() {
    pm.response.to.have.status(404);
});
```

### Kør collection automatisk

Brug **Collection Runner** i Postman til at køre hele collectionen i rækkefølge:
1. Åbn Collection
2. Klik "Run Collection"
3. Se alle tests køre og resultaterne

Eller via kommandolinje med **Newman** (Postman's CLI):

```bash
# Installer Newman
npm install -g newman

# Kør en collection
newman run BrugerAPI.postman_collection.json \
           -e local.postman_environment.json \
           --reporters cli,json \
           --reporter-json-export resultat.json
```

---

## 8. JWT — Autentificering i REST APIs

JWT (JSON Web Token) bruges til at håndtere autentificering i stateless systemer som REST APIs. I stedet for at gemme sessioner på serveren sender klienten et token med hvert request.

### Struktur

Et JWT består af tre Base64-kodede dele adskilt af punktummer: `Header.Payload.Signature`

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTYiLCJuYW1lIjoiQW5uYSIsInJvbGUiOiJ1c2VyIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
   ^-- Header                ^-- Payload                                               ^-- Signature
```

**Header** — hvilken algoritme bruges:
```json
{ "alg": "HS256", "typ": "JWT" }
```

**Payload** — data om brugeren (*claims*):
```json
{
  "sub": "123456",
  "name": "Anna Andersen",
  "role": "admin",
  "exp": 1735689600,
  "iat": 1735603200
}
```

| Claim | Betyder |
|---|---|
| `sub` | Subject — bruger-ID |
| `exp` | Expiration — hvornår tokenet udløber (Unix timestamp) |
| `iat` | Issued At — hvornår tokenet blev udstedt |
| `role` | Brugerdefineret claim — fx brugerens rolle |

**Signature** — serverens garanti for at tokenet er ægte:
```
HMACSHA256(base64(header) + "." + base64(payload), hemmelig_nøgle)
```

### Autentificeringsflow

```
1. Bruger logger ind: POST /api/auth/login  {"email": "...", "password": "..."}
2. Server verificerer → genererer JWT → sender {"token": "eyJhbGciOi..."}
3. Klient gemmer tokenet (localStorage eller cookie)
4. Klient sender tokenet med hvert request:
   GET /api/orders
   Authorization: Bearer eyJhbGciOi...
5. Server verificerer signaturen → kender brugerens identitet (ingen database-opslag!)
```

### JWT vs. Sessions

| | JWT (Stateless) | Session (Stateful) |
|---|---|---|
| **Serveren gemmer** | Intet | Session-data i database/memory |
| **Klienten sender** | Token med hvert request | Session-ID cookie |
| **Skalerbarhed** | Nem (alle servere kan verificere) | Sværere (alle servere skal kende sessionen) |
| **Logout** | Svært — tokenet er gyldigt til det udløber | Nemt — slet sessionen |

### Vigtige sikkerhedsregler

> **JWT er signeret — IKKE krypteret!** Gem aldrig passwords eller følsomme data i payload.

- Sæt altid en udløbstid (`exp`) — f.eks. 1 time
- Brug HTTPS — så tokenet ikke kan opfanges under transport
- `httpOnly cookie` er sikrere end `localStorage` (modstandsdygtig over for XSS)

---

## 9. Avancerede Query Parameter mønstre

### Søgning

```
GET /api/brugere?q=anna              → Fritekst-søgning
GET /api/produkter?search=laptop     → Søg i navn/beskrivelse
```

### Field selection (sparse fieldsets)

```
GET /api/brugere?fields=id,navn,email
→ Returner kun disse felter (reducerer data)
```

### Embed/include (relations)

```
GET /api/brugere/1?include=ordrer
→ Returner bruger med indlejrede ordrer
```

### Eksempel response med filtrering + sortering + paginering

```
GET /api/produkter?kategori=elektronik&min_pris=500&sort=pris&order=asc&page=0&size=5
```

```json
{
  "filter": {
    "kategori": "elektronik",
    "min_pris": 500
  },
  "sort": { "by": "pris", "order": "asc" },
  "pagination": { "page": 0, "size": 5, "total": 42 },
  "data": [
    { "id": 5, "navn": "Mus", "pris": 549 },
    { "id": 12, "navn": "Tastatur", "pris": 699 },
    { "id": 3, "navn": "Headset", "pris": 799 },
    { "id": 8, "navn": "Webcam", "pris": 899 },
    { "id": 21, "navn": "USB-hub", "pris": 1099 }
  ]
}
```

---

## 10. Opsummering

| Emne                   | Nogleinformation                                              |
|------------------------|---------------------------------------------------------------|
| Query parameters       | Skrives efter `?`, adskilles med `&`, bruges til filter/sort |
| Filtrering             | `?felt=vaerdi` — filtrer på specifikke felter                |
| Sortering              | `?sort=felt&order=asc/desc`                                  |
| Paginering             | `?page=0&size=10` — opdel store datasæt                      |
| Postman Collections    | Organiserede samlinger af requests med variabler             |
| Postman Tests          | JavaScript assertions der kører efter requests               |
| pm.test()              | `pm.test("navn", function() { pm.expect(...).to.equal(...) })` |
| pm.environment.set()   | Gem variabel til brug i næste request                        |
| Newman                 | Kommandolinje-kørsel af Postman collections (CI/CD)          |
| JWT                    | Stateless token-baseret autentificering — Header.Payload.Signature |
| JWT claims             | `sub` (bruger-ID), `exp` (udløb), `iat` (udstedt), `role`   |

---

## Opgaver til selvstudium

1. Skriv URL'er til disse queries for en produktdatabase:
   a) Alle produkter under 1000 kr sorteret efter pris stigende
   b) Side 2 med 20 produkter pr side filtreret på kategori "mobler"
2. Skriv en Postman test der tjekker at status er 200 OG at response JSON indeholder et felt "id" der er et positivt tal.
3. Hvad gør `pm.environment.set("token", json.token)`? Hvornår er det nyttigt?
4. Implementér i Spring Boot: `GET /api/produkter?min_pris=X&max_pris=Y&sort=pris&order=asc`
5. Forklar hvad Newman bruges til. Hvorfor er det nyttigt i en CI/CD pipeline?
