# Eksamenstræning — 3. Semester Kontekst

> Smid denne fil ind i en ny Claude Code samtale og skriv: "Læs min kontekstfil og fortsæt eksamenstræningen"

---

## Fag og eksamensformat

### Fag 1: Systemudvikling (Softwareteknologi)
- **Format:** Gruppepræsentation af WeatherDress-projektet → individuel mundtlig forsvar alene
- **Fokus hele semester:** XP (Extreme Programming), Testing, Udviklingsmetoder
- **Lærer:** Streng — kræver præcise forklaringer og forsvar af valg
- **Noter:** `/tmp/3-sem-noter/softwareteknologi/` (klon repo: `https://github.com/Fulle4600/3-sem-noter.git`)

### Fag 2+3: Teknologi & Programmering (fælles eksamen)
- **Format:** 4 timers skriftlig programmering — alle hjælpemidler tilladt inkl. AI
- **Efterfølges af:** 1 times mundtlig evaluering hvor du forklarer dine valg
- **Upload:** Løsning afleveres i Wiseflow

---

## WeatherDress Eksamensprojekt (Systemudvikling)

**Problem:** Forældre har svært ved at vælge tøj til børn om morgenen baseret på vejret  
**Løsning:** Web-app der kombinerer OpenWeatherMap API + Raspberry Pi motorwheel + brugerpræferencer  
**Elevator pitch:** Tøj-anbefalingsplatform til travle forældre med børnevenlig visning

**XP-praksisser I bruger i projektet (XP 2. udgave terminologi):**
- Pair Programming (ved kritiske dele som API- og motor-integration)
- Test-First Programming / TDD på backend-logik — Rød → Grøn → Refaktorer (xUnit)
- Continuous Integration via GitHub Actions — tests ved hver push
- Weekly Cycle (Small Releases) — fungerende inkrementer hver uge
- Shared Code (XP 2. udg. for Collective Code Ownership) — alle kan ændre alt
- Refactoring løbende — ikke som separat fase
- Informative Workspace — taskboard og burndown synligt for hele teamet
- Energized Work — vi arbejder ikke udmattet
- Ten-Minute Build — CI-pipeline under 10 minutter
- Slack — uge 5 er buffer, vi planlægger ikke 100% kapacitet

**XP Principper brugt i projektet:**
- Humanity → software tjener menneskers behov (forældre/børn)
- Economics → løser et reelt problem med reel værdi
- Simplicity → MVP først, udvid derefter
- Redundancy → fallback-mekanismer (cache, konservative anbefalinger)
- Root-Cause Analysis → ved fejl finder vi årsagen, ikke blot symptomet
- Opportunity → risici er læringsmuligheder
- Negotiated Scope Contract → tid og kvalitet er låst, scope er fleksibelt
- Accepted Responsibility → vi accepterer kun opgaver vi kan levere med kvalitet
- Baby Steps → altid fungerende software at vise

**Definition of Done:**
1. Koden skrevet og committed
2. Unit test skrevet og består
3. Koden reviewet af mindst én fra gruppen
4. Product Owner har accepteret funktionen

**5 Weekly Cycles (Size it up):**
- Uge 1: Setup + API + basis algoritme → REST API kørende, OpenWeatherMap integreret
- Uge 2: Frontend + Raspberry Pi → Vue-frontend viser anbefaling, Pi viser vejrkort
- Uge 3: Brugerpræferencer + Test-First → præferencer per barn, TDD på algoritme
- Uge 4: CI + integrationstest + PO-demo → GitHub Actions, daglig build < 10 min
- Uge 5 (Slack): Buffer + GDPR + polering → fejlrettelser, alle DoD verificeret

**Be clear on what's going to give (Negotiated Scope Contract):**
- Must have: vejrdata, tøjanbefaling, brugerpræferencer, frontend
- Should have: Raspberry Pi integration, CI pipeline
- Could have: push-notifikationer, 3+ børn support
- Won't have: native app, AI-anbefalinger, webshop
- Dropprækkefølge hvis vi er bagud: notifikationer → avanceret multi-barn → Pi simuleres

**Tekniske naboer:** OpenWeatherMap API, Raspberry Pi + motor, GitHub CI/CD  
**GitHub:** `https://github.com/Patrick-LD/WeatherDress_3.SemesterEksamen`  
**Fuld Inception Deck v2:** `/Users/Skole/Desktop/WeatherDress_InceptionDeck_v2.md`

---

## Pensum Systemudvikling — Overblik

### Agile & XP (`softwareteknologi/agile-xp.md`)
- Agile Manifest: 4 værdier (Individer > Processer, Fungerende software > Dokumentation, Samarbejde > Kontrakt, Reaktion > Plan)
- 12 Agile principper
- XP 5 værdier: Communication, Simplicity, Feedback, Courage, Respect
- XP ~14 praksisser: Pair Programming, TDD, CI, Refactoring, Small Releases, Collective Code Ownership, Simple Design, Incremental Design, Baby Steps, Ten-Minute Build, Daily Deployment, Negotiated Scope Contract, Real Customer Involvement, Whole Team, On-Site Customer, 40-Hour Week
- User Stories + INVEST-kriterier
- Definition of Done, Burn Down Chart, Scope Creep, Vandfald vs Agile

### XP Cheatsheet — Ny XP 2. udgave (Michele Marchesi / Kent Beck)
**14 principper (vigtige at kende udover de 6 i noter):**
- Humanity, Economics, Mutual Benefit, **Self-Similarity**, Improvement, **Diversity**, **Reflection**, **Flow**, **Opportunity**, **Redundancy**, Failure, Quality, Baby Steps, **Accepted Responsibility**

**24 praksisser (13 primære + 11 korollar):**
- Primære: Stories, Weekly Cycle, Quarterly Cycle, Slack, Sit Together, Whole Team, Informative Workspace, Energized Work, Pair Programming, Incremental Design, Test-First Programming, Ten-Minute Build, Continuous Integration
- Korollar: Real Customer Involvement, Incremental Deployment, Negotiated Scope Contract, Pay-Per-Use, Team Continuity, Shrinking Teams, Root-Cause Analysis, Code and Tests, Shared Code, Single Code Base, Daily Deployment

**Vigtige ændringer i ny XP:** Collective Code Ownership → **Shared Code** | Coding Standards **fjernet** (anses selvfølgeligt)

### Testing (`softwareteknologi/test.md`)
- TDD: Rød → Grøn → Refaktorer
- Test Pyramiden: Unit (mange) → Integration (middel) → E2E (få)
- Unit Tests: AAA-pattern (Arrange, Act, Assert), isolation med mocks/stubs
- Mock vs Stub: Stub returnerer værdier, Mock verificerer interaktioner
- CI Pipeline, Non-functional requirements

### Arkitektur (`softwareteknologi/arkitektur.md`)
- SOLID: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
- Repository Pattern: interface + implementering (SQL, Mongo, InMemory)
- MVC: Model (data/logik), View (UI), Controller (broker)
- Dependency Injection: modtag afhængigheder udefra (ikke opret selv)
- Design Patterns: Creational (Singleton, Factory, Builder), Structural (Adapter, Decorator, Facade), Behavioral (Observer, Strategy, Command)
- Code Review, Refactoring, Teknisk Gæld

### Bæredygtighed & Projektledelse (`softwareteknologi/baeredygtighed-projektledelse.md`)
- Karlskrona Manifest: 5 dimensioner (Miljø, Social, Økonomisk, Individuel, **Teknisk**), 9 principper
- Prototyping: Low/High fidelity, Throwaway/Evolutionary
- Design Sprint: 5 dage (Map, Sketch, Decide, Prototype, Test)
- Boehm & Turner Matrix: Agil vs Planbaseret metodevalg
- Git workflow: feature branches, Pull Requests, merge conflicts

---

## Pensum Teknologi (`teknologi/netvaerk-api.md`)
- HTTP: metoder (GET/POST/PUT/PATCH/DELETE), statuskoder (200/201/204/400/401/403/404/500)
- REST: ressource-baserede URLs, ikke handlinger
- JSON: datatyper, serialize/deserialize
- TCP/IP model: 5 lag (Application, Transport, Network, Data Link, Physical)
- TCP vs UDP: pålidelig vs hurtig
- CORS: browserens sikkerhedsmekanisme, løses med `AddCors()` i ASP.NET
- JWT: Header.Payload.Signature — ALDRIG kodeord i payload
- Swagger/OpenAPI: `/swagger` endpoint
- IP: IPv4/IPv6, private adresser, localhost 127.0.0.1
- DNS + DHCP
- HTTPS/TLS: TLS Handshake (certifikat, CA, session key), port 443
- Kryptering: Symmetrisk (AES, én nøgle) vs Asymmetrisk (RSA, public/private key)
- Docker: Image vs Container, Dockerfile, Docker Hub
- DevOps: CI/CD pipeline (CI → CD Delivery → CD Deployment)

---

## Pensum Programmering (`programmering/database.md`)
- SQL: CRUD (SELECT/INSERT/UPDATE/DELETE), JOIN
- ORM: C# klasse = database tabel
- Entity Framework Core: Code First, DbContext, DbSet\<T\>
- EF Migrations: `dotnet ef migrations add` / `dotnet ef database update`
- LINQ: `.Where().OrderBy().ToList()` — oversættes til SQL af EF
- Normalisering: 1NF (atomare felter), 2NF (afhænger af hel nøgle), 3NF (ingen transitiv afhængighed)
- SQL Injection: brug ALTID parameteriserede queries / ORM
- Async/Await: `FindAsync`, `ToListAsync`, `SaveChangesAsync`

---

## Prøveeksamen Prog/Tek — Opgavestruktur

Eksamen består af **8 opgaver** som bygger ovenpå hinanden:

| # | Opgave | Teknologier |
|---|---|---|
| 1 | Repository med List (CRUD: GetAll, GetById, Add, Delete, Update) | ASP.NET Core Web API, C# |
| 2 | Unit-test af repository | xUnit, AAA-pattern |
| 3 | REST controller med DI + Swagger | HTTP metoder/statuskoder, DI, Swagger |
| 4 | CORS + Azure deploy | CORS config, Azure |
| 5 | Vue.js frontend (vis/tilføj/slet/opdater) | Vue.js, Axios, Bootstrap |
| 6 | Teori: TCP/IP lagene + adressering | Netværksteori (skriftlig note) |
| 7 | Filtrering og sortering i backend | Query parameters, LINQ |
| 8 | Repository med database (erstatter List) | EF Core, unit tests |

**Mønsteret:** Repository → Test → REST → Frontend → Database

---

## Træningsstatus

- [ ] XP mock-forhør (individuel forsvar)
- [ ] SOLID + arkitektur quiz
- [ ] Testing begreber
- [ ] Prog/Tek: Repository → REST mønster
- [ ] Prog/Tek: EF Core + database repository
- [ ] Netværksteori (TCP/IP, DNS, HTTPS)

---

## Sådan fortsætter du

Start en ny samtale i Claude Code og skriv:

> "Læs `/Users/Skole/Desktop/EKSAMEN_KONTEKST.md` og start eksamenstræning. Jeg vil starte med [emne]."
