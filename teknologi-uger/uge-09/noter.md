# Uge 9 — HTTP Status Koder, CORS & Swagger

Denne uge bygger videre på HTTP og REST ved at se på status koder (hvad betyder alle de numre?), CORS-mekanismen der er afgørende for webapplikationer, og Swagger/OpenAPI som er standard for API-dokumentation.

---

## 1. HTTP Status Koder

HTTP status koder er **3-cifrede tal** der fortæller klienten, **hvad der skete** med dens request. De er grupperet i 5 kategorier baseret på det første ciffer.

```
1xx → Informational  (request modtaget, fortsæt)
2xx → Success        (request lykkedes)
3xx → Redirection    (klient skal gøre noget mere)
4xx → Client Error   (klienten lavede en fejl)
5xx → Server Error   (serveren lavede en fejl)
```

---

## 2. 1xx — Informational

Bruges sjældent i hverdagen. Fortæller klienten, at processen er i gang.

| Kode | Tekst       | Forklaring                                    |
|------|-------------|-----------------------------------------------|
| 100  | Continue    | Server bekræfter at klient kan fortsætte med at sende request body |
| 101  | Switching Protocols | Server skifter protokol (f.eks. til WebSocket) |

---

## 3. 2xx — Success

Disse koder betyder, at requesten lykkedes.

| Kode | Tekst                   | Hvornår bruges det                                        |
|------|-------------------------|-----------------------------------------------------------|
| 200  | OK                      | Standard succes. GET/PUT/PATCH returnerer typisk dette    |
| 201  | Created                 | Ny ressource oprettet (typisk efter POST)                 |
| 202  | Accepted                | Request modtaget men behandles asynkront                  |
| 204  | No Content              | Succes, men ingen body (typisk efter DELETE)              |

### Eksempler i praksis

```
GET /api/brugere/1  →  200 OK           (bruger fundet og returneret)
POST /api/brugere   →  201 Created      (ny bruger oprettet)
DELETE /api/brugere/1 → 204 No Content  (slettet, ingen body)
```

---

## 4. 3xx — Redirection

Klienten skal lave endnu en handling for at fuldføre requesten.

| Kode | Tekst              | Forklaring                                           |
|------|--------------------|------------------------------------------------------|
| 301  | Moved Permanently  | Ressourcen er permanent flyttet til ny URL           |
| 302  | Found              | Ressourcen er midlertidigt på anden URL              |
| 304  | Not Modified       | Ressourcen er ikke ændret siden sidst (caching)      |
| 307  | Temporary Redirect | Midlertidig redirect, bevar HTTP-metode              |
| 308  | Permanent Redirect | Permanent redirect, bevar HTTP-metode                |

### 301 eksempel
```
GET /gammelt-endpoint  →  301 Moved Permanently
                          Location: /nyt-endpoint

Klient sender automatisk:
GET /nyt-endpoint  →  200 OK
```

---

## 5. 4xx — Client Errors

Disse koder betyder, at **klienten** har lavet en fejl. Serveren forstod requesten, men kan ikke opfylde den.

| Kode | Tekst                    | Typisk årsag                                             |
|------|--------------------------|----------------------------------------------------------|
| 400  | Bad Request              | Ugyldig syntax, manglende felter i body                  |
| 401  | Unauthorized             | Mangler autentificering (log ind!)                       |
| 403  | Forbidden                | Autentificeret, men mangler tilladelse                   |
| 404  | Not Found                | Ressourcen eksisterer ikke                               |
| 405  | Method Not Allowed       | HTTP-metode ikke tilladt på denne URL                    |
| 409  | Conflict                 | Konflikt med eksisterende data (f.eks. dubleret email)   |
| 422  | Unprocessable Entity     | Syntaks OK men semantisk fejl (valideringsfejl)          |
| 429  | Too Many Requests        | Rate limiting — for mange requests på kort tid           |

### 401 vs 403 — vigtig forskel!

```
401 Unauthorized → "Hvem er du? Log ind først!"
403 Forbidden    → "Jeg ved hvem du er, men du må ikke."

Eksempel:
- Anonym bruger forsøger at se private data:  401 Unauthorized
- Bruger forsøger at slette en andens data:   403 Forbidden
- Admin forsøger at se en andens data:        200 OK
```

### 404 vs 400
```
GET /api/brugere/999 → 404 Not Found (bruger eksisterer ikke)
GET /api/bruger^^    → 400 Bad Request (ugyldig URL-syntaks)
```

---

## 6. 5xx — Server Errors

Disse koder betyder, at **serveren** har lavet en fejl. Requesten kan have været korrekt.

| Kode | Tekst                    | Typisk årsag                                           |
|------|--------------------------|--------------------------------------------------------|
| 500  | Internal Server Error    | Generel fejl i serverens kode (uncaught exception)    |
| 501  | Not Implemented          | Serveren understøtter ikke den anmodede funktionalitet |
| 502  | Bad Gateway              | Proxy modtog ugyldigt svar fra upstream server         |
| 503  | Service Unavailable      | Serveren er nede eller overbelastet                    |
| 504  | Gateway Timeout          | Proxy fik ikke svar fra upstream server i tide         |

### 500 fejl — typisk årsag
```java
// Et uncaught exception i Java Spring Boot → 500 Internal Server Error
@GetMapping("/brugere/{id}")
public Bruger getBruger(@PathVariable Long id) {
    // Hvis databasen er nede, kastes exception → 500
    return brugerRepository.findById(id).get(); // NoSuchElementException!
}

// Bedre:
    return brugerRepository.findById(id)
        .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
// → 404 Not Found
```

---

## 7. Status koder i REST API design

```
CRUD Operation          HTTP Method   Success Code   Fejl-koder
──────────────────────────────────────────────────────────────
Hent alle               GET           200            500
Hent en                 GET           200            404, 500
Opret                   POST          201            400, 409, 422
Erstat hel              PUT           200            400, 404, 422
Delvis opdater          PATCH         200            400, 404, 422
Slet                    DELETE        204            404, 500
Uautoriseret adgang     GET/POST/...  —              401, 403
```

---

## 8. CORS — Cross-Origin Resource Sharing

### Problemet: Same-Origin Policy

Af sikkerhedshensyn håndhæver browsere **Same-Origin Policy (SOP)**:
- En webside må kun lave AJAX-requests til **samme oprindelse** (same origin).
- Origin = protokol + domæne + port.

```
https://frontend.dk:3000  →  https://api.backend.dk:8080
                              BLOKERET af browser! (forskellig origin)

https://frontend.dk:3000  →  https://frontend.dk:3000/api
                              OK! (samme origin)
```

### Hvad er CORS?

**CORS (Cross-Origin Resource Sharing)** er en mekanisme der giver servere mulighed for at **tillade** requests fra andre origins.

Serveren kommunikerer via HTTP-headers, hvem der må kalde den.

### CORS i praksis — Simple requests

```
Klient (frontend.dk:3000)                 Server (api.backend.dk:8080)
    |                                            |
    |  GET /api/brugere                          |
    |  Origin: http://frontend.dk:3000           |
    |  ---------------------------------------->|
    |                                            |
    |  HTTP/1.1 200 OK                           |
    |  Access-Control-Allow-Origin: *            |
    |<------------------------------------------|
```

`Access-Control-Allow-Origin: *` betyder alle origins er tilladt.

### CORS Preflight (OPTIONS request)

For "komplekse" requests (f.eks. POST med JSON) sender browseren først en **preflight request**:

```
1. Browser sender OPTIONS til server:
   OPTIONS /api/brugere HTTP/1.1
   Origin: http://frontend.dk:3000
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type, Authorization

2. Server svarer med CORS-tilladelser:
   HTTP/1.1 204 No Content
   Access-Control-Allow-Origin: http://frontend.dk:3000
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Content-Type, Authorization
   Access-Control-Max-Age: 86400

3. Hvis tilladelse er givet, sender browser den faktiske POST request.
```

### CORS Headers oversigt

| Header                           | Forklaring                                    |
|----------------------------------|-----------------------------------------------|
| `Access-Control-Allow-Origin`    | Hvilke origins må kalde API'et                |
| `Access-Control-Allow-Methods`   | Hvilke HTTP-metoder er tilladt                |
| `Access-Control-Allow-Headers`   | Hvilke request-headers er tilladt             |
| `Access-Control-Allow-Credentials` | Om cookies/credentials må sendes          |
| `Access-Control-Max-Age`         | Hvor længe preflight-svar kan caches (sekunder)|

### CORS i Spring Boot (Java)

```java
// Global CORS konfiguration
@Configuration
public class CorsConfig {
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of("http://localhost:3000", "https://frontend.dk"));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("Content-Type", "Authorization"));
        config.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return source;
    }
}

// Eller på controller-niveau:
@CrossOrigin(origins = "http://localhost:3000")
@RestController
public class BrugerController { ... }
```

---

## 9. Swagger / OpenAPI

### Hvad er OpenAPI?

**OpenAPI Specification (OAS)** er en standard for at **dokumentere REST APIs** i et maskinlæseligt format (YAML eller JSON). Den beskriver:
- Alle endpoints (URL'er og metoder)
- Request-parametre og bodies
- Response-formater og statuskoder
- Autentificeringsmetoder

**Swagger** er det originale navn og er nu et sæt af værktøjer bygget på OpenAPI.

### Swagger UI

Swagger UI er et **interaktivt browser-interface** der genereres automatisk fra OpenAPI-specifikationen. Det lader udviklere:
- Se alle API-endpoints
- Læse beskrivelser og parametre
- Teste endpoints direkte i browseren

```
http://localhost:8080/swagger-ui.html
http://localhost:8080/v3/api-docs  (JSON-specifikation)
```

### Swagger i Spring Boot (springdoc)

Tilføj dependency i `pom.xml`:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.1.0</version>
</dependency>
```

Annotér din kode:

```java
@RestController
@RequestMapping("/api/brugere")
@Tag(name = "Brugere", description = "API til brugerstyring")
public class BrugerController {

    @Operation(
        summary = "Hent alle brugere",
        description = "Returnerer en liste af alle brugere i systemet"
    )
    @ApiResponse(responseCode = "200", description = "Liste af brugere returneret")
    @GetMapping
    public List<Bruger> getAlle() { ... }

    @Operation(summary = "Opret ny bruger")
    @ApiResponse(responseCode = "201", description = "Bruger oprettet")
    @ApiResponse(responseCode = "400", description = "Ugyldig input")
    @PostMapping
    public ResponseEntity<Bruger> opret(@RequestBody @Valid BrugerDTO dto) { ... }
}
```

### OpenAPI YAML Eksempel

```yaml
openapi: 3.0.3
info:
  title: Bruger API
  description: API til styring af brugere
  version: 1.0.0

paths:
  /api/brugere:
    get:
      summary: Hent alle brugere
      responses:
        '200':
          description: Liste af brugere
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Bruger'
    post:
      summary: Opret ny bruger
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BrugerInput'
      responses:
        '201':
          description: Bruger oprettet
        '400':
          description: Ugyldig input

components:
  schemas:
    Bruger:
      type: object
      properties:
        id:
          type: integer
        navn:
          type: string
        email:
          type: string
    BrugerInput:
      type: object
      required:
        - navn
        - email
      properties:
        navn:
          type: string
        email:
          type: string
```

### Swagger UI Udseende

```
+---------------------------------------------+
|  Bruger API  v1.0.0                         |
|  API til styring af brugere                 |
+---------------------------------------------+
|  Brugere                                    |
|  +------------------------------------------+
|  | GET  /api/brugere  Hent alle brugere  v  |
|  | POST /api/brugere  Opret ny bruger    v  |
|  | GET  /api/brugere/{id}  Hent bruger   v  |
|  | PUT  /api/brugere/{id}  Opdater bruger v  |
|  | DELETE /api/brugere/{id}  Slet bruger v  |
|  +------------------------------------------+
+---------------------------------------------+
```

---

## 10. Opsummering

| Emne           | Nogleinformation                                               |
|----------------|----------------------------------------------------------------|
| 2xx            | Success: 200 OK, 201 Created, 204 No Content                  |
| 4xx            | Client error: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx            | Server error: 500 Internal Server Error, 503 Service Unavailable |
| 401 vs 403     | 401 = ikke logget ind, 403 = logget ind men ikke tilladelse    |
| CORS           | Browser-sikkerhed, server tillader cross-origin via headers    |
| Preflight      | OPTIONS request sendes automatisk af browser ved komplekse requests |
| Swagger/OpenAPI| Standard API-dokumentation, interaktiv UI til test            |

---

## Opgaver til selvstudium

1. Hvilken statuskode returneres ved: a) bruger ikke fundet, b) server fejl, c) ressource oprettet, d) ingen adgang uden login?
2. Forklar Same-Origin Policy med egne ord. Giv et eksempel på en situation der kræver CORS.
3. Hvad er forskellen på `401 Unauthorized` og `403 Forbidden`? Hvornår bruges hvert?
4. Konfigurér Swagger/springdoc i et Spring Boot projekt og beskriv, hvad du ser på `/swagger-ui.html`.
5. Skriv CORS-konfigurationen i Spring Boot der kun tillader requests fra `http://localhost:4200` med metoderne GET og POST.
