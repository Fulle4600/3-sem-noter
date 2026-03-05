# Netværk & API Begreber

## Indholdsfortegnelse

- [HTTP — HyperText Transfer Protocol](#http--hypertext-transfer-protocol)
- [REST — Representational State Transfer](#rest--representational-state-transfer)
- [JSON — JavaScript Object Notation](#json--javascript-object-notation)
- [TCP/IP Model (5 lag)](#tcpip-model-5-lag)
- [CORS — Cross-Origin Resource Sharing](#cors--cross-origin-resource-sharing)
- [JWT — JSON Web Token](#jwt--json-web-token)
- [Swagger / OpenAPI](#swagger--openapi)
- [IP-Adresser](#ip-adresser)
- [DNS og DHCP](#dns-og-dhcp)
- [HTTPS / TLS / SSL](#https--tls--ssl)
- [Kryptering — Symmetrisk vs Asymmetrisk](#kryptering--symmetrisk-vs-asymmetrisk)
- [Docker og Containerisering](#docker-og-containerisering)
- [DevOps](#devops)

---

## HTTP — HyperText Transfer Protocol

Protokol (regler) til kommunikation mellem klient (browser) og server på web.
Definerer HVORDAN data sendes — hvilken format request og response har.

**Stateless:** Serveren husker INTET om forrige requests. Hver request er uafhængig.

### HTTP Metoder (verber)

| Metode | Beskrivelse | Eksempel |
|---|---|---|
| **GET** | Hent data (ingen sideeffekter, kan caches) | `GET /api/users` |
| **POST** | Opret ny ressource | `POST /api/users` med body `{navn: "Anna"}` |
| **PUT** | Erstat en hel ressource | `PUT /api/users/42` med komplet bruger-objekt |
| **PATCH** | Delvis opdatering | `PATCH /api/users/42` med kun `{navn: "Anna B."}` |
| **DELETE** | Slet ressource | `DELETE /api/users/42` |

### HTTP Status Koder

| Kode | Betydning |
|---|---|
| **200** OK | Alt gik godt |
| **201** Created | Ressource oprettet (svar på POST) |
| **204** No Content | Success men ingen body (svar på DELETE) |
| **400** Bad Request | Fejl i request (fx manglende felt, forkert format) |
| **401** Unauthorized | Ikke logget ind / intet gyldigt token |
| **403** Forbidden | Logget ind men ingen rettighed til DETTE |
| **404** Not Found | Ressourcen findes ikke |
| **500** Internal Server Error | Noget gik galt på SERVEREN (din kode fejlede) |

---

## REST — Representational State Transfer

Arkitekturprincip (**ikke** en protokol!) for API design. Definerer HOW et godt API ser ud.
Skaber ensartede, forståelige APIs. Alle der kender REST kan bruge dit API intuitivt.

**Ressource-baseret:** URLs repræsenterer ressourcer (IKKE handlinger!)

### REST URL konventioner

```
GET    /api/customers         → Hent alle kunder
POST   /api/customers         → Opret ny kunde
GET    /api/customers/42      → Hent specifik kunde
PUT    /api/customers/42      → Opdater kunde
DELETE /api/customers/42      → Slet kunde
GET    /api/customers/42/orders → Ordrer for kunde 42
```

| | Eksempel |
|---|---|
| **FORKERT** (ikke REST) | `/api/getCustomer`, `/api/deleteUser?id=5` |
| **RIGTIGT** (REST) | `GET /api/customers/5`, `DELETE /api/users/5` |

---

## JSON — JavaScript Object Notation

Let, tekstbaseret format til strukturerede data. Standard for dataudveksling på web.
Lettere at læse og parse end XML. Virker naturligt med JavaScript.

**Datatyper:** `string "tekst"`, `number 42`, `boolean true/false`, `null`, `array [1,2,3]`, `object {key: value}`

```csharp
// C#
JsonSerializer.Serialize(obj)
JsonSerializer.Deserialize<T>(json)
```

```javascript
// JavaScript
JSON.stringify(obj)
JSON.parse(jsonString)
```

---

## TCP/IP Model (5 lag)

Model der beskriver hvordan data rejser fra én computer til en anden over netværket.
Opdeler kommunikation i lag — hvert lag håndterer sin del, kender ikke de andre.

| Lag | Navn | Protokoller/eksempler |
|---|---|---|
| **5** | Application | HTTP, DNS, FTP (din app kommunikerer her) |
| **4** | Transport | TCP / UDP (pålidelig/hurtig levering) |
| **3** | Network | IP-adressering og routing (finder vejen) |
| **2** | Data Link | Ethernet, WiFi, MAC-adresser (lokal netværksforbindelse) |
| **1** | Physical | Kabler, radio, lys (de fysiske bits) |

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| **Egenskaber** | Pålidelig, garanteret levering, korrekt rækkefølge | Hurtig, ingen garanti, ingen rækkefølge |
| **Bruges til** | HTTP, email, filer | Streaming, gaming, DNS |
| **Prioriterer** | Pålidelighed | Hastighed |

---

## CORS — Cross-Origin Resource Sharing

Browserens sikkerhedsmekanisme der blokerer requests til **andre domæner** end den aktuelle side.
Forhindrer at onde hjemmesider laver requests til din netbank og stjæler data.

**Problem:** Din Vue-app på `localhost:3000` kalder ASP.NET API på `localhost:5000` = CORS fejl!

**Løsning:** Server tilføjer header: `Access-Control-Allow-Origin: http://localhost:3000`

```csharp
// ASP.NET
builder.Services.AddCors(...)
app.UseCors(...)
```

---

## JWT — JSON Web Token

Standard til at bevise din identitet i stateless systemer. Token sendes med hvert request.
Serveren behøver ikke slå op i database for at verificere hvem du er — alt er i tokenet.

**Struktur:** `Header.Payload.Signature` (alle tre er base64 encodet, adskilt med punktum)

**Payload indeholder:** bruger-ID, roller, udløbstid (`exp`), udstedelsestidspunkt (`iat`)

**Sendes som:** `Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...`

> **VIGTIGT:** JWT er IKKE krypteret (kun signeret) — læg ALDRIG kodeord eller kort i payload!

---

## Swagger / OpenAPI

Standard til at dokumentere REST APIs. Genererer interaktiv API-dokumentation automatisk.
Udviklere kan se OG teste alle endpoints direkte i browseren — ingen Postman nødvendig.

```csharp
// ASP.NET — tilgængelig på /swagger
builder.Services.AddSwaggerGen()
```

---

## IP-Adresser

| Type | Format | Eksempel |
|---|---|---|
| **IPv4** | 4 tal 0-255 adskilt med punktum | `192.168.1.100` |
| **IPv6** | 128-bit hex adresse | `2001:0db8:85a3::8a2e:0370:7334` |

**Private adresser** (kun lokalt netværk): `192.168.x.x`, `10.x.x.x`, `172.16.x.x`
**`127.0.0.1`** = localhost = din egen maskine
**DNS:** Oversætter domænenavn (`google.com`) til IP-adresse (`142.250.185.46`)

---

## DNS og DHCP

### DNS — Domain Name System

Internettets "telefonbog" — oversætter menneskevenlige domænenavne til IP-adresser.
Uden DNS skulle du huske IP-adresser i stedet for navne som `google.com`.

**Sådan virker det:** Browser spørger DNS-server → DNS svarer med IP → Browser forbinder til IP.

**Hierarki:** Root → Top-Level Domain (`.com`, `.dk`) → Domain (`google.com`) → Subdomain (`mail.google.com`)

### DHCP — Dynamic Host Configuration Protocol

Protokol der automatisk tildeler IP-adresser til enheder på et netværk.
Uden DHCP skulle alle enheder have manuelt konfigurerede IP-adresser (upraktisk og fejlbehæftet).

**DHCP tildeler:** IP-adresse, subnet mask, default gateway, DNS-serveradresse.

**DHCP Lease:** IP-adressen "lejes" for en periode. Når lejen udløber kan en ny adresse tildeles.

> Typisk scenarie: Du tilslutter din laptop til WiFi → router (DHCP-server) sender automatisk IP-konfiguration.

---

## HTTPS / TLS / SSL

HTTPS = HTTP + krypteret forbindelse via TLS (Transport Layer Security).
Sikrer at data ikke kan aflyttes eller manipuleres under transport (man-in-the-middle angreb).

**Forskel:**
- **HTTP** — data sendes i klar tekst. Alle der lytter kan læse det.
- **HTTPS** — data krypteres. Kun afsender og modtager kan læse det.

**TLS Handshake (forenklet):**
1. Klient siger "hej" og hvilke krypteringsalgoritmer den understøtter
2. Server sender sit **certifikat** (beviser serverens identitet) + vælger algoritme
3. Klient verificerer certifikatet (signeret af betroet CA — Certificate Authority)
4. Begge parter aftaler en **session key** og al videre kommunikation krypteres

**Port:** HTTP = 80, HTTPS = 443

> **SSL** er den ældre term (SSL 3.0 → TLS 1.0 → TLS 1.3). SSL er deprecated — i dag bruges TLS, men folk siger stadig "SSL".

---

## Kryptering — Symmetrisk vs Asymmetrisk

### Symmetrisk kryptering

Samme nøgle bruges til at **kryptere og dekryptere**.
Hurtig og effektiv til store datamængder. Problem: Hvordan deler man nøglen sikkert?

> Eksempel: AES (Advanced Encryption Standard). Bruges til kryptere selve data i HTTPS.

### Asymmetrisk kryptering (Public Key Cryptography)

To nøgler: **Public key** (alle må se den) og **Private key** (kun ejeren har den).
- Data krypteret med public key kan **kun** dekrypteres med private key.
- Data signeret med private key kan **verificeres** med public key.

Løser nøgle-udvekslingsproblemet — du kan sende din public key til alle uden risiko.

> Eksempel: RSA. Bruges i TLS Handshake til sikkert at aftale en symmetrisk session key.

| | Symmetrisk | Asymmetrisk |
|---|---|---|
| **Nøgler** | 1 delt nøgle | Public + Private key par |
| **Hastighed** | Hurtig | Langsom |
| **Brug** | Kryptere data (bulk) | Nøgleudveksling, signaturer |
| **Eksempel** | AES | RSA |

---

## Docker og Containerisering

### Virtualisering vs Containerisering

| | Virtuel Maskine (VM) | Container |
|---|---|---|
| **Inkluderer** | Fuldt OS + app | Kun app + afhængigheder |
| **Størrelse** | GB | MB |
| **Opstartstid** | Minutter | Sekunder |
| **Isolation** | Stærk (eget OS) | Svagere (deler host OS kernel) |

### Docker

Platform til at bygge, sende og køre containers.
*"It works on my machine"* — med Docker virker det **overalt** fordi container inkluderer alle afhængigheder.

**Centrale begreber:**

| Begreb | Beskrivelse |
|---|---|
| **Image** | Skabelon/blueprint for en container. Uforanderlig. Bygges fra Dockerfile. |
| **Container** | En kørende instans af et image. Kan startes, stoppes, slettes. |
| **Dockerfile** | Opskrift på hvordan et image bygges (base image, copy filer, run kommandoer). |
| **Docker Hub** | Registry til at dele og hente images (som GitHub for Docker images). |

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0
COPY . /app
WORKDIR /app
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

```bash
docker build -t min-app .      # Byg image
docker run -p 8080:80 min-app  # Start container
```

---

## DevOps

Kultur og praksis der **forener** development (Dev) og operations (Ops) for hurtigere og mere pålidelig softwarelevering.
Traditionelt: Dev-teamet koder → kaster det "over muren" til Ops → Ops deployer. DevOps fjerner muren.

**CI/CD Pipeline:**

| Fase | Beskrivelse |
|---|---|
| **CI** — Continuous Integration | Kode merges og testes automatisk ved hvert push |
| **CD** — Continuous Delivery | Software er altid klar til deployment (kræver manuel godkendelse) |
| **CD** — Continuous Deployment | Hvert grønt build deployes automatisk til produktion |

**DevOps nøgleprincipper:**
- Automatiser alt der kan automatiseres (build, test, deploy)
- Infrastruktur som kode (Infrastructure as Code)
- Overvågning og feedback fra produktion
- Kort feedback-loop fra kode til bruger

> **Relation til XP:** CI er en XP-praksis. DevOps er dens naturlige udvidelse ind i operations.

---

## Opgaver til selvstudium

1. Forklar forskellen på HTTP `PUT` og `PATCH` — hvornår ville du bruge den ene frem for den anden?
2. Design RESTful URL-endpoints for en simpel biblioteksapplikation med bøger og lånere
3. Forklar hvad JWT er, og hvad der er i de tre dele (header, payload, signature) — og hvad man ALDRIG må lægge i payload
4. Hvad er CORS, og hvorfor opstår CORS-fejl? Hvad skal tilføjes i ASP.NET Core for at tillade en specifik frontend-origin?
5. Forklar TLS Handshake i egne ord — hvad er formålet, og hvad sker der step-by-step?
6. Hvad er forskellen på en Docker Image og en Docker Container? Hvad er en Dockerfile, og hvad indeholder den?
