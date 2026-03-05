# Uge 8 — HTTP & REST

Denne uge handler om HTTP-protokollen — rygraden i al kommunikation på World Wide Web — og REST-arkitekturen, der er den dominerende måde at designe web-APIs på. Vi ser på request/response-strukturen, HTTP-metoder og principper for godt REST-design.

---

## 1. HTTP — HyperText Transfer Protocol

HTTP er en **applikationslagsprotokol** (lag 5 i TCP/IP-modellen) der bruges til at overføre data på World Wide Web. Det er en **tekst-baseret protokol** der bygger på Client/Server-modellen.

**HTTPS** = HTTP + TLS (krypteret version af HTTP via port 443).

### HTTP's natur

| Egenskab         | Forklaring                                                    |
|------------------|---------------------------------------------------------------|
| Stateless        | Serveren husker IKKE tidligere requests — hver request er uafhængig |
| Text-baseret     | Requests og responses er menneskelig-læselige tekst          |
| Request/Response | Klient sender request, server sender response                 |
| Port             | 80 (HTTP), 443 (HTTPS)                                       |
| Protokol         | Kører over TCP                                               |

---

## 2. HTTP Request Struktur

Hvert HTTP request består af:
1. **Request Line** — metode, URL, og HTTP-version
2. **Headers** — metadata om requesten
3. **Blank linje** — adskiller headers fra body
4. **Body** (valgfri) — data der sendes med (ved POST/PUT/PATCH)

```
GET /api/brugere/1 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{
  "navn": "Anna"
}
```

### Request Line
```
GET /api/brugere/1 HTTP/1.1
^    ^              ^
|    |              |
metode  URL-sti    HTTP-version
```

### Vigtige Request Headers

| Header            | Eksempel                            | Formål                                       |
|-------------------|-------------------------------------|----------------------------------------------|
| `Host`            | `api.example.com`                   | Domænet der sendes til (obligatorisk i HTTP/1.1) |
| `Content-Type`    | `application/json`                  | Format på body-data                          |
| `Accept`          | `application/json`                  | Hvad klienten kan modtage                    |
| `Authorization`   | `Bearer <token>`                    | Autentificeringstoken                        |
| `Content-Length`  | `125`                               | Størrelse af body i bytes                    |
| `User-Agent`      | `Mozilla/5.0 ...`                   | Hvilken klient der sender                    |

---

## 3. HTTP Response Struktur

Et HTTP response fra serveren indeholder:
1. **Status Line** — HTTP-version og statuskode
2. **Headers** — metadata om responsen
3. **Blank linje**
4. **Body** — svardataene (JSON, HTML, osv.)

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 85
Date: Mon, 10 Feb 2025 12:00:00 GMT

{
  "id": 1,
  "navn": "Anna Larsen",
  "email": "anna@example.com"
}
```

### Status Line
```
HTTP/1.1 200 OK
^        ^   ^
|        |   |
version  kode tekst
```

### Vigtige Response Headers

| Header             | Eksempel                   | Formål                                   |
|--------------------|----------------------------|------------------------------------------|
| `Content-Type`     | `application/json`         | Format på body-data                      |
| `Content-Length`   | `85`                       | Størrelse af body                        |
| `Location`         | `/api/brugere/1`           | Bruges ved 201 Created og 301 redirect   |
| `Cache-Control`    | `max-age=3600`             | Cachingstyring                           |
| `Access-Control-Allow-Origin` | `*`         | CORS (se uge 9)                          |

---

## 4. HTTP Metoder

HTTP-metoder (også kaldet "verbs") angiver, **hvilken handling** der ønskes udført.

### De 5 primære metoder

#### GET — Hent data
```
GET /api/brugere        → Hent alle brugere
GET /api/brugere/1      → Hent bruger med id 1
```
- Sender **ingen body**.
- Skal aldrig ændre data på serveren (**idempotent og safe**).
- Resultater kan caches.

#### POST — Opret ny ressource
```
POST /api/brugere
Body: { "navn": "Anna", "email": "anna@example.com" }
```
- Sender **body** med data til den nye ressource.
- Serveren tildeler ID og returnerer typisk `201 Created` med `Location`-header.
- **Ikke idempotent** — to identiske POST-requests opretter to ressourcer.

#### PUT — Erstat hel ressource
```
PUT /api/brugere/1
Body: { "navn": "Anna Larsen", "email": "ny@email.com", "alder": 23 }
```
- Erstatter **hele** ressourcen.
- Alle felter skal sendes med.
- **Idempotent** — samme request giver samme resultat.

#### PATCH — Delvis opdatering
```
PATCH /api/brugere/1
Body: { "email": "nyemail@example.com" }
```
- Opdaterer kun de **angivne felter**.
- Alle andre felter bevares som de er.
- Mere effektivt end PUT ved store ressourcer.

#### DELETE — Slet ressource
```
DELETE /api/brugere/1
```
- Sletter ressourcen.
- Sender typisk ingen body.
- Returnerer `204 No Content` eller `200 OK`.

### Metodeoversigt

| Metode | Body i request? | Idempotent? | Safe? | Typisk statuskode |
|--------|-----------------|-------------|-------|-------------------|
| GET    | Nej             | Ja          | Ja    | 200 OK            |
| POST   | Ja              | Nej         | Nej   | 201 Created       |
| PUT    | Ja              | Ja          | Nej   | 200 OK            |
| PATCH  | Ja              | Nej*        | Nej   | 200 OK            |
| DELETE | Nej (oftest)    | Ja          | Nej   | 204 No Content    |

**Idempotent** = gentagne identiske requests giver samme resultat.
**Safe** = requesten ændrer ikke data på serveren.

---

## 5. Statelessness

HTTP er **stateless** — serveren gemmer **ingen information** om tidligere requests.

```
Request 1: GET /api/produkter
→ Server svarer, husker INTET

Request 2: GET /api/kurv
→ Server ved IKKE hvem du er!
```

### Konsekvens
Klienten skal sende al nødvendig information med i **hvert enkelt request**, typisk:
- Session cookies
- JWT-tokens i Authorization-headeren
- API-nøgler

### Fordele ved statelessness
- Serveren behøver ikke huske tilstand → nemmere at skalere
- Enhver server kan betjene enhver request (load balancing)
- Fejlhåndtering er simplere

---

## 6. REST — Representational State Transfer

REST er en **arkitekturstil** (ikke en protokol) der beskriver, hvordan man designer web-APIs. Det er opfundet af Roy Fielding i hans ph.d.-afhandling fra 2000.

Et API der følger REST-principperne kaldes et **RESTful API**.

### De 6 REST-principper (constraints)

1. **Client-Server** — adskil klient og server. Klienten kender ikke serverens interne implementering.

2. **Stateless** — serveren gemmer ingen tilstand. Hvert request er selvstændigt.

3. **Cacheable** — responses skal indikere om de kan caches, for at reducere serverbelastning.

4. **Uniform Interface** — ensartet måde at interagere med ressourcer på (HTTP-metoder + URL-struktur).

5. **Layered System** — klienten ved ikke om den taler med en server, proxy eller load balancer.

6. **Code on Demand** (valgfri) — serveren kan sende eksekverbar kode (JavaScript) til klienten.

---

## 7. REST URL Konventioner

Gode REST-URL'er er **ressource-orienterede** — de refererer til ting, ikke handlinger.

### Regler for RESTful URLs

| Regel                         | Godt eksempel              | Dårligt eksempel              |
|-------------------------------|----------------------------|-------------------------------|
| Brug navneord, ikke verber    | `/api/brugere`             | `/api/getBrugere`             |
| Brug flertal for samlinger    | `/api/brugere`             | `/api/bruger`                 |
| Brug id for specifik ressource| `/api/brugere/1`           | `/api/brugerMedId?id=1`       |
| Nested ressourcer             | `/api/brugere/1/ordrer`    | `/api/getOrdreForBruger?uid=1`|
| Kleine bogstaver og bindestreg| `/api/produkt-kategorier`  | `/api/ProduktKategorier`      |

### CRUD-mapping

| Operation       | HTTP Metode | URL                   | Beskrivelse                  |
|-----------------|-------------|-----------------------|------------------------------|
| Hent alle       | GET         | `/api/brugere`        | Returnerer liste af brugere  |
| Hent en         | GET         | `/api/brugere/{id}`   | Returnerer én bruger         |
| Opret           | POST        | `/api/brugere`        | Opretter ny bruger           |
| Opdater hel     | PUT         | `/api/brugere/{id}`   | Erstatter hel bruger         |
| Opdater delvist | PATCH       | `/api/brugere/{id}`   | Opdaterer specifikke felter  |
| Slet            | DELETE      | `/api/brugere/{id}`   | Sletter bruger               |

### Nested ressourcer
```
GET /api/brugere/1/ordrer          → Alle ordrer for bruger 1
GET /api/brugere/1/ordrer/42       → Ordre 42 tilhørende bruger 1
POST /api/brugere/1/ordrer         → Opret ny ordre for bruger 1
```

---

## 8. REST i praksis — Java/Spring Boot eksempel

```java
@RestController
@RequestMapping("/api/brugere")
public class BrugerController {

    @GetMapping
    public List<Bruger> getAlleBrugere() {
        return brugerService.findAll();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Bruger> getBrugerById(@PathVariable Long id) {
        return brugerService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }

    @PostMapping
    public ResponseEntity<Bruger> opretBruger(@RequestBody Bruger bruger) {
        Bruger gemt = brugerService.save(bruger);
        URI location = URI.create("/api/brugere/" + gemt.getId());
        return ResponseEntity.created(location).body(gemt);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Bruger> opdaterBruger(
            @PathVariable Long id,
            @RequestBody Bruger bruger) {
        return ResponseEntity.ok(brugerService.update(id, bruger));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> sletBruger(@PathVariable Long id) {
        brugerService.deleteById(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 9. Postman — Introduktion

**Postman** er et GUI-værktøj til at sende HTTP-requests og teste APIs uden at skrive kode.

### Postman interface

```
+--------------------------------------------------+
|  Method  |  URL                        | [Send]  |
|  [GET v] |  http://api.example.com/... |         |
+--------------------------------------------------+
| Headers | Body | Auth | Params                   |
|----------------------------------------          |
| Key              | Value                          |
| Content-Type     | application/json               |
| Authorization    | Bearer eyJhbGci...             |
+--------------------------------------------------+
|  Response  |  Status: 200 OK  |  Time: 45ms      |
|  Body | Headers | ...                            |
|  {                                               |
|    "id": 1,                                      |
|    "navn": "Anna"                                |
|  }                                               |
+--------------------------------------------------+
```

### Brug af Postman

1. Vælg HTTP-metode (GET, POST, PUT, PATCH, DELETE)
2. Skriv URL
3. Tilføj headers (f.eks. `Content-Type: application/json`)
4. Tilføj body (ved POST/PUT/PATCH) — vælg "raw" og "JSON"
5. Klik Send
6. Inspicer response (statuskode, body, headers)

### Eksempel: POST request i Postman
```
Method: POST
URL: http://localhost:8080/api/brugere
Headers:
  Content-Type: application/json
Body (raw JSON):
{
  "navn": "Anna Larsen",
  "email": "anna@example.com",
  "alder": 22
}
```

---

## 10. Opsummering

| Emne              | Nogleinformation                                                |
|-------------------|-----------------------------------------------------------------|
| HTTP              | Applikationsprotokol, request/response, stateless              |
| HTTP Request      | Request line + Headers + (Body)                                |
| HTTP Response     | Status line + Headers + Body                                   |
| GET               | Hent data, ingen body, safe og idempotent                      |
| POST              | Opret ny, sender body, ikke idempotent                         |
| PUT               | Erstat hel ressource, idempotent                               |
| PATCH             | Delvis opdatering                                              |
| DELETE            | Slet ressource                                                 |
| REST              | Arkitekturstil med 6 principper, ressource-orienterede URLs    |
| URL-konventioner  | Navneord, flertal, hierarki med /                              |

---

## Opgaver til selvstudium

1. Hvad er forskellen på `PUT` og `PATCH`? Giv et konkret eksempel.
2. Forklar begrebet "stateless" med egne ord. Hvorfor er det en fordel?
3. Design REST-endpoints for en online boghandel med ressourcerne: bøger, forfattere, ordrer.
4. Installer Postman og send en GET-request til `https://jsonplaceholder.typicode.com/users`.
5. Hvad returnerer `POST /api/brugere` typisk som HTTP-statuskode? Hvad sender serveren i Location-headeren?
