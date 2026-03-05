# Uge 10 — UI/UX Design

Denne uge handler om brugeroplevelse og design af brugergraenseflade. Vi gennemgar centrale designprincipper fra Nielsen og Shneiderman, CRAP-principperne for visuel design, og ser pa vaerktojer som personas, Think Aloud testing og prototyping. Godt UI/UX design er ikke kun aestetik — det handler om at lave systemer som brugere faktisk kan og vil bruge.

---

## 1. Grundlaeggende om UX og UI

### Hvad er forskellen?

| | UX (User Experience) | UI (User Interface) |
|-|---------------------|---------------------|
| **Fokus** | Hele brugeroplevelsen | Den visuelle graeenseflade |
| **Sporgsmal** | "Loser systemet brugerens problem?" | "Er knappen det rigtige sted?" |
| **Faerdighedssaet** | Research, information architecture, flow | Visuel design, typografi, farver |
| **Analogi** | En bils indretning og funktionalitet | Bilens design og udseende |

### Maal for godt UI/UX

1. **Brugbarhed (Usability)**: Kan brugerne udfore opgaverne?
2. **Effektivitet**: Kan de udfore dem hurtigt?
3. **Tilfredshed**: Oplevede de det positivt?
4. **Tilgaengelighed**: Kan alle bruge det, inkl. mennesker med handicap?
5. **Aerlighed**: Manipulerer graesenfladen ikke brugerne (dark patterns)?

---

## 2. Nielsens 10 Usability Heuristikker

Jakob Nielsen formulerede i 1994 10 heuristikker for UI-design. De er stadig relevante og bruges bredt til evaluering af brugergraenseflade.

### Heuristik 1: Synlighed af Systemstatus (Visibility of System Status)

Systemet skal altid informere brugeren om, hvad der sker, via passende feedback inden for en rimelig tid.

**Eksempler:**
- En loading-spinner nar siden henter data
- "Fil uploades: 67%..." med en progressbar
- Gron hak nar en formular er indsendt korrekt
- "Dit koeb er gennemfort — Ordrenummer: #12345"

**Modeksempel (darligt):**
- En knap der ikke giver feedback efter klik — brugeren ved ikke om det virkede
- En lang operation uden nogen indikation af fremskridt

### Heuristik 2: Match Mellem System og Virkelighed (Match Between System and the Real World)

Systemet skal tale brugerens sprog. Brug ord, saetninger og koncepter der er kendte for brugeren — ikke systemtermer.

**Eksempler:**
- "Indkoebskurv" (ikke "Session-baseret ordre-buffer")
- Mappe-ikoner der ligner rigtige mapper
- Slet-handling med skaraldesikon
- "Gem" i stedet for "Kommit til database"

### Heuristik 3: Brugerkontrol og Frihed (User Control and Freedom)

Brugere vager forkert valg. Giv dem altid en tydelig "emergency exit" — fortryd og annuller.

**Eksempler:**
- Ctrl+Z (Undo) i alle editorer
- "Annuller"-knap i dialogbokse
- Bekraeftelse foer sletning: "Er du sikker pa at du vil slette?"
- "Fortryd"-funktion i Gmail (nar en email er sendt)

### Heuristik 4: Konsistens og Standarder (Consistency and Standards)

Brugere skal ikke skulle spekulere over, om forskellige ord, situationer eller handlinger betyder det samme.

**Eksempler:**
- Alle primaerae knapper er blaa i hele applikationen
- "Gem" er altid øverst til hoejre
- Back-knappen opfoerer sig konsistent
- Brug platformkonventioner (iOS vs Android vs Web har forskellige standarder)

**Platform-konventioner:**
```
Mobil (iOS):    [Annuller] [OK]  <- Negativt til venstre
Mobil (Android): [OK]      <- En handling
Web:            [Annuller] [Gem] <- Varierer
```

### Heuristik 5: Fejlforebyggelse (Error Prevention)

Endnu bedre end gode fejlmeddelelser er et design der forhindrer fejl i at opstaa.

**Eksempler:**
- Deaktiver "Bestil"-knap til formular er valideret
- Bekraeftelsesdialog foer destruktive handlinger
- Autocomplete der forebygger stavefejl
- Dato-vaelger i stedet for fritekstfelt (forhindrer "13/2026/04" fejl)
- Vis adgangskodestyrke i realtid

### Heuristik 6: Genkendelse frem for Erindring (Recognition Rather Than Recall)

Minimer brugerens behoev for at huske information. Synliggor objekter, handlinger og muligheder. Brugeren bor ikke behoeve at huske information fra een del af graesenfladen til en anden.

**Eksempler:**
- Autofyld i formularer
- Nylig besoegte sider i browsers history
- "Seneste dokuments" i Word
- Knapper med ikoner OG tekst (ikke kun ikoner)
- Kontekstmenu viser relevante handlinger

### Heuristik 7: Fleksibilitet og Effektivitet (Flexibility and Efficiency of Use)

Acceleratorer — usynlige for noevne brugere — kan ofte fremskynde interaktion for den erfarne bruger, sa systemet kan betjene bade uerfarne og erfarne brugere.

**Eksempler:**
- Tastaturgenveje (Ctrl+S, Ctrl+C, etc.)
- Macro-optagelse for gentagne handlinger
- Avancerede sogefiltre for power users
- Keyboard navigation som alternativ til mus
- "Favoritter" eller "Bogmaerker" funktionalitet

### Heuristik 8: AEstetisk og Minimalistisk Design (Aesthetic and Minimalist Design)

Dialoger bor ikke indeholde irrelevant eller sjaeldent behoevet information. Hver ekstra enhed af information i en dialog konkurrerer med relevante informationsenheder og forringer deres relative synlighed.

**Eksempler:**
- Fjern unodvendige felter fra formularer
- Vis kun relevante muligheder
- Undgaa dekorative elementer der ikke tilforer vaerdi
- Whitespace er dit ven — brug det

**Eksempel pa overlaesset vs. minimalistisk:**
```
OVERLAESSET FORMULAR:
Fornavn: [____] Mellemnavn: [____] Efternavn: [____]
Primaar email: [____] Sekundaer email: [____]
Primaar telefon: [____] Sekundaer telefon: [____]
...

MINIMALISTISK (kun det noedvendige):
Navn: [____]
Email: [____]
[Opret konto]
```

### Heuristik 9: Hjaelp Brugere med at Genkende, Diagnosticere og Genoprette Fejl (Help Users Recognize, Diagnose, and Recover from Errors)

Fejlmeddelelser skal vaere udtrykt pa alment sprog (ingen koder), praecierende beskrive problemet, og konstruktivt forsla en loesning.

**Darligt:**
```
Fejl: Error 403 - Access Denied
SQL Exception: Violation of UNIQUE constraint 'IX_Users_Email'
```

**Godt:**
```
Din emailadresse er allerede registreret.
Log ind i stedet, eller brug en anden emailadresse.
[Log ind] [Brug anden email]
```

### Heuristik 10: Hjaelp og Dokumentation (Help and Documentation)

Selvom det er bedst, hvis systemet kan bruges uden dokumentation, kan det vaere noedvendigt at levere hjaelp. Saadan hjaelp bor vaere nem at soge i, fokuseret pa brugerens opgave og liste konkrete trin.

**Eksempler:**
- In-app tooltips og onboarding tours
- Contextual help (?) ikoner ved komplicerede felter
- Soegbar hjelpedokumentation
- Video-tutorials for komplekse opgaver

---

## 3. Ben Shneidermans 8 Golden Rules

Ben Shneiderman formulerede 8 regler for interface-design i bogen "Designing the User Interface" (1987). De overlapper delvist med Nielsen, men har et andet fokus.

### Regel 1: Strab efter Konsistens

Konsistente sekvenser af handlinger bor kraeves i lignende situationer. Terminologi bor vaere konsistent i prompter, menueer og hjelpeskaerme.

### Regel 2: Soeg Universel Brugbarhed

Design for alle brugere — begyndere og eksperter. Tilbud tastaturgenveje, tutorials og avancerede muligheder.

### Regel 3: Tilbyd Informativ Feedback

For alle brugerhandlinger bor der vaere systemmassig feedback. For hyppige og mindre handlinger kan svaret vaere beskedent, mens det for sjaeldne og store handlinger bor vaere mere omfattende.

### Regel 4: Design Dialoger til at Yield Closures

Sekvenser af handlinger bor organiseres i grupper med start, midte og slut. Den informative feedback ved afslutning giver brugeren tilfredshed og en indikation af, at vejen er klar til at forberede den naeste gruppe af handlinger.

**Eksempel: Checkout-flow:**
```
Trin 1: Kurv
Trin 2: Levering
Trin 3: Betaling
Trin 4: Bekraeftelse -> "Din ordre er modtaget!" (CLOSURE)
```

### Regel 5: Forebyg Fejl

Design systemet sa brugere ikke kan begaa alvorlige fejl. Hvis der begaas en fejl, bor systemet opdage fejlen og tilbyde enkle, forstaelige instrukser til gendannelse.

### Regel 6: Tillad Nem Tilbagefoersel af Handlinger

Denne regel letter angsten, da brugeren ved at fejl kan fortrydes, og tilskynder til udforskning af ukende muligheder.

**Eksempler:**
- Undo/Redo (Ctrl+Z / Ctrl+Y)
- "Papirkurv" i stedet for permanent sletning
- 30-sekunders "fortryd" i Gmail

### Regel 7: Stoet Intern Locus of Control

Erfarne brugere oensker staerek fornemmelse af at de styrer graesenfladen og at graesenfladen reagerer pa deres handlinger. Design graesenfladen sa brugere er initiativtagere af handlinger, ikke responderende.

**Modeksempel**: Automatiske pop-ups der afbryder arbejdsflowet.

### Regel 8: Reduser Korttidshukommelsesbelastning

Begransningen af menneskelig informationsbehandling i korttidshukommelsen kraever, at displays holdes enkle, flersides displays konsolideres og traening i vindues-bevaegelses-sekvenser reduceres.

**Eksempel**: Vis relevant info pa den aktuelle side i stedet for at kraeve at brugeren husker info fra en tidligere side.

---

## 4. CRAP Principper for Visuel Design

CRAP er et akronym for fire visuelle designprincipper formuleret af Robin Williams i "The Non-Designer's Design Book":

### C — Contrast (Kontrast)

Hvis to elementer IKKE er ens, goer dem MEGET forskellige. Kontrast skaber visuel interesse og hjaelper brugeren med at navigere.

**Typer af kontrast:**
- Farvekontrast (sort tekst pa hvid baggrund)
- Stoerrelsekontrast (overskrift vs. brodtekst)
- Skrifttypekontrast (serif vs. sans-serif)
- Formskontrast (runde vs. kantede elementer)

**Tilgaengelighed:** WCAG 2.1 kraever minimum 4.5:1 kontrastforhold for normal tekst.

```
GOD KONTRAST:
#000000 (sort) pa #FFFFFF (hvid) = 21:1 ✓

DARLIG KONTRAST (tilgaengelighedsproblem):
#767676 (gra) pa #FFFFFF (hvid) = 4.48:1 (graensevaeardi)
#AAAAAA (lys gra) pa #FFFFFF (hvid) = 2.32:1 ✗
```

### R — Repetition (Gentagelse)

Gentag visuelle elementer pa tvaers af designet. Gentagelse skaber konsistens og styrker det visuelle look.

**Eksempler:**
- Alle primaerae knapper har samme farveskema og form
- Overskrifter bruger altid samme skrifttype og stoerrelse
- Ens margin og padding pa tvaers af sider
- Logo pa alle sider pa samme placering

**I kode — CSS Variabler:**
```css
:root {
  --primary-color: #2563eb;
  --secondary-color: #64748b;
  --border-radius: 8px;
  --spacing-md: 16px;
  --font-heading: 'Inter', sans-serif;
}

.button-primary {
  background-color: var(--primary-color);
  border-radius: var(--border-radius);
  padding: var(--spacing-md);
}
```

### A — Alignment (Justering)

Ingen element bor placeres vilkaarligt. Hvert element bor have en visuel forbindels med et andet element pa siden. Dette skaber orden og kohaerans.

**Typer:**
- Venstreorientering (mest laesbart for venstre-til-hoejre sprog)
- Centrering (til overskrifter og helte-sektioner)
- Hoejrejustering (taeller, priser)
- Grid-baseret layout (9-kolonne, 12-kolonne grid)

**Eksempel pa god vs. darlig justering:**
```
DARLIG (vilkaarlig placering):
  Overskrift
         Undertekst her
    Knap her       Anden knap

GODT (konsistent venstrejustering):
Overskrift
Undertekst her
[Knap 1] [Knap 2]
```

### P — Proximity (Naerhed)

Grupper relaterede elementer sammen. Hold ikke-relaterede elementer adskilt. Nærhed signalerer relation.

**Eksempel pa formular-design:**
```
DARLIG PROXIMITY (alle elementer ens afstand):
Navn: [___]
Email: [___]
--------
Adresse: [___]
By: [___]
Postnummer: [___]
[Send]

GODT PROXIMITY (relaterede felter grupperet):
[Personlige oplysninger]
  Navn: [___]
  Email: [___]

[Adresse]
  Adresse: [___]
  By: [___]   Postnummer: [___]

[Send]
```

---

## 5. Personas

En **persona** er en fiktiv repraesentation af en typisk bruger baseret pa rigtig research. Det er IKKE en rigtig person — det er en arketyp skabt fra observationer og interviews.

### Hvad er en god persona?

En god persona indeholder:
1. **Navn og foto** (goer det personligt og konkret)
2. **Demografisk info** (alder, job, uddannelse)
3. **Maal og motivationer** (hvad vil de opna?)
4. **Frustrationer og smerter** (hvad er svart?)
5. **Adfaerd og teknologibrug**
6. **Et citat** der faenger personens attitude

### Eksempel pa en Persona

```
+------------------------------------------+
|  [FOTO]  Anna Andersen, 34 ar            |
|          Regnskabsassistent              |
|          Aarhus                          |
+------------------------------------------+
|                                          |
| MAAL:                                    |
| - Haandtere firmaets fakturaer effektivt |
| - Holde styr pa udgifter og budgetter    |
| - Minimere tid brugt pa administration  |
|                                          |
| FRUSTRATIONER:                           |
| - Systemet kraesher nar hun eksporterer  |
|   store rapporter                        |
| - Det tager for mange klik at finde      |
|   specifikke fakturaer                  |
| - Mangel pa mobilapp                    |
|                                          |
| TEKNOLOGIBRUG:                           |
| - Bruger Excel dagligt                  |
| - Smartphone (iOS)                      |
| - Komfortabel med teknologi             |
|                                          |
| CITAT:                                   |
| "Jeg vil gerne have systemet til at      |
|  arbejde for mig, ikke omvendt."         |
+------------------------------------------+
```

### Persona vs. Proto-Persona

| | Persona | Proto-persona |
|-|---------|--------------|
| **Baseret pa** | Rigtig user research | Antagelser og gisk |
| **Tid** | Dage/uger at skabe | Timer at skabe |
| **Noejagtig** | Hoej | Lav |
| **Naer bruges** | Nar man har tid og ressourcer | Nar man hurtigt skal i gang |

---

## 6. Think Aloud Testing

**Think Aloud testing** er en usability-testmetode, hvor deltageren siger hoejt, hvad de taenker, imens de bruger systemet.

### Fremgangsmade

1. **Rekrutter deltagere**: 5 deltagere afslorer typisk 85% af usability-problemer (Nielsens regel)
2. **Forbered opgaver**: Skriv konkrete, realistiske opgaver (ikke "find noget pa siden")
3. **Briefing**: Forklar at du tester systemet — IKKE deltageren
4. **Think Aloud instruktion**: "Sig alt hvad du taenker"
5. **Observer og noteer**: Skriv hvad de goer og siger
6. **Analyser**: Identificer tilbagevendende problemer

### Godt opgaveeksempel

```
DARLIG opgave:
"Naviger til checkout-siden"
(For teknisk, afslorer svaret)

GOD opgave:
"Du har set en sort t-shirt i str. M du gerne vil koebe.
 Vis mig, hvad du ville goere."
```

### Hvad man ser efter

| Observation | Hvad det kan indikere |
|-------------|----------------------|
| Bruger klikker pa forkert element | Darlig visuel hierarki |
| Bruger laenlaes siden gentagne gange | Information er svaer at finde |
| Bruger siger "hmm, hvad sker der?" | Manglende feedback |
| Bruger bruger browser-back i stedet for in-app nav | Navigation er forvirrende |
| Bruger stopper og kigger hjaeloest | Kritisk usability-problem |

### Fordele og ulemper

| Fordele | Ulemper |
|---------|---------|
| Afslorer det faktiske tankeprocessen | Kunstigt — folk taenker normalt ikke hoejt |
| Nem at gennemfore | Koster tid |
| Billig | Kan vaere svaert at rekruttere raette deltagere |
| Finder reelle problemer, ikke antagne | Observer-effekt: folk opforer sig anderledes |

---

## 7. Prototyping

En **prototype** er en tidlig model af et system, brugt til at udforske og validere design.

### Low-Fidelity Prototyper

**Karakteristika:**
- Papir og blyant, sticky notes, whiteboard
- Hurtig at lave (timer, ikke dage)
- Billig
- Nem at aendre

**Naer bruges:**
- Tidlig i designprocessen
- Nar man vil teste flow og koncepter
- Nar krav er usikre

**Eksempel:**
```
[Papirskitser af skaerme]
    +-------------+
    | LOGO        |
    |             |
    | [Soeg...]   |
    |             |
    | Resultat 1  |
    | Resultat 2  |
    |             |
    | [Laes mere] |
    +-------------+
```

### High-Fidelity Prototyper

**Karakteristika:**
- Lavet i Figma, Adobe XD, Sketch
- Ser ud som det faerdige produkt
- Interaktive (klikbare links)
- Mere tidskraevende at lave

**Naer bruges:**
- Later i designprocessen
- Foer implementering
- Til brugertest der kraever realisme
- Til stakeholder-godkendelse

### Wireframes

Wireframes er skeletstrukturer af en side — de viser layout og indhold uden visual design:

```
+---------------------------+
| [LOGO]     [Nav] [Nav]    |
+---------------------------+
|   Hero Image              |
|   Overskrift H1           |
|   Brodb tekst...          |
|   [CTA Knap]              |
+---------------------------+
| [Feature 1] [Feature 2]   |
| [tekst]     [tekst]       |
+---------------------------+
| Footer                    |
+---------------------------+
```

---

## 8. Design Sprint (Jake Knapp)

Design Sprint er en 5-dages proces udviklet af Jake Knapp hos Google Ventures til at loese store designudfordringer hurtigt.

### De 5 Dage

#### Dag 1: Understand (Forsta)
- Definer problemet og maalet
- Interview eksperter
- Skab et kort over brugerens journey
- Vaelg et specifikt ma at fokusere pa

#### Dag 2: Diverge (Diverger)
- Lightning Demos: find inspiration fra andre produkter
- Individuelle skitser af loesninger
- Crazy 8s: 8 ideer pa 8 minutter
- Detaljeret loesningsskitse

#### Dag 3: Decide (Beslut)
- Hver deltager gennemgar sin loesning stille
- Varme kritik og afstemning (med dots/stickers)
- Teamet vaelger den bedste loesning (eller kombinerer)
- Storyboard den valgte loesning i detaljer

#### Dag 4: Prototype (Prototyper)
- Byg en realistisk prototype
- Uddel roller: maker, collector, stitcher, writer, interviewer
- Det handler om SPEED — ikke perfekthed

#### Dag 5: Test (Test)
- Kore 5 brugerinterviews med prototypen
- Teamet ser interviews live via videostream
- Noteer observationer
- Se monstre og laer

### Hvornaar er Design Sprint vaerdifuldt?

- Nar man er usikker pa retningen
- Foer man starter en stor implementering
- Nar teamet er i tvivl om brugernes beehov
- Nar man vil validere en ide hurtigt

---

## 9. JWT — JSON Web Token

JWT bruges til at håndtere autentificering i stateless systemer som REST APIs — relevant i forbindelse med brugerinddragelse og systemers sikkerhed overfor rigtige brugere.

### Hvad er JWT?

Et JWT er et kompakt, URL-sikkert token der beviser din identitet. Det sendes med hvert API-kald i stedet for at gemme sessioner på serveren.

**Struktur:** `Header.Payload.Signature` — tre dele adskilt af punktummer, alle Base64-kodet.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjM0NTYiLCJuYW1lIjoiQW5uYSIsInJvbGUiOiJ1c2VyIn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
   ^-- Header                ^-- Payload                                               ^-- Signature
```

### De tre dele

**Header** — hvilken algoritme bruges:
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload** — data om brugeren (kallas *claims*):
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
HMACSHA256(
  base64(header) + "." + base64(payload),
  hemmelig_nøgle
)
```

### Autentificeringsflow

```
1. Bruger logger ind med email + password
   POST /api/auth/login  {"email": "...", "password": "..."}

2. Server verificerer credentials → genererer JWT
   Server → Klient: {"token": "eyJhbGciOi..."}

3. Klient gemmer tokenet (localStorage eller cookie)

4. Klient sender tokenet med hvert efterfølgende request
   GET /api/orders
   Authorization: Bearer eyJhbGciOi...

5. Server verificerer signaturen → kender brugerens identitet
   → ingen database-opslag nødvendig!
```

### Vigtige sikkerhedsregler

> **JWT er signeret — IKKE krypteret!** Payloaden kan læses af alle (Base64 er ikke kryptering). Gem aldrig passwords, kreditkortnumre eller følsomme data i payload.

- Sæt altid en udløbstid (`exp`) — f.eks. 1 time
- Brug HTTPS — så tokenet ikke kan opfanges under transport
- Gem tokenet sikkert — `localStorage` er sårbart over for XSS-angreb; `httpOnly cookie` er sikrere

### JWT vs. Sessions

| | JWT (Stateless) | Session (Stateful) |
|---|---|---|
| **Serveren gemmer** | Intet | Session-data i database/memory |
| **Klienten sender** | Token med hvert request | Session-ID cookie |
| **Skalerbarhed** | Nem (alle servere kan verificere) | Sværere (alle servere skal kende sessionen) |
| **Logout** | Svært — tokenet er gyldigt til det udløber | Nemt — slet sessionen |

---

## 10. Query Parameters & [FromQuery]

Når din Vue-frontend henter data fra en REST API, sender den ofte ekstra info via URL'en — fx hvad brugeren har søgt på, eller hvordan listen skal sorteres. Det sker med **query parameters**.

### Hvad er query parameters?

Alt efter `?` i en URL er query parameters — nøgle=værdi par adskilt med `&`:

```
GET /api/products                              → Hent alle produkter
GET /api/products?search=laptop                → Søg efter "laptop"
GET /api/products?sort=price                   → Sortér efter pris
GET /api/products?search=laptop&sort=price     → Kombiner begge
```

### `[FromQuery]` i ASP.NET Controller

`[FromQuery]` fortæller din C# controller at hente værdien fra URL'en — ikke fra body'en:

```csharp
[HttpGet]
public IActionResult GetProducts(
    [FromQuery] string? search,
    [FromQuery] string? sort)
{
    var products = _repo.GetAll();

    // Filtrér hvis search er angivet
    if (!string.IsNullOrEmpty(search))
        products = products.Where(p =>
            p.Name.Contains(search, StringComparison.OrdinalIgnoreCase));

    // Sortér hvis sort er angivet
    if (sort == "price")
        products = products.OrderBy(p => p.Price);

    return Ok(products);
}
```

### Fra Vue til API — hele flowet

```
Bruger skriver "laptop" i søgefelt
          ↓
Vue sender: GET /api/products?search=laptop
          ↓
ASP.NET: [FromQuery] string? search = "laptop"
          ↓
Kode filtrerer listen
          ↓
JSON sendes tilbage til Vue
          ↓
Vue viser de filtrerede resultater
```

**Vue.js eksempel — send query parameter med Axios:**
```javascript
// Simpel søgning
const response = await axios.get('/api/products', {
  params: {
    search: this.searchText,
    sort: this.sortBy
  }
});
// Axios bygger automatisk URL: /api/products?search=laptop&sort=price
```

### `[FromQuery]` vs `[FromRoute]` vs `[FromBody]`

| Attribut | Hvor | Eksempel URL / request |
|---|---|---|
| `[FromQuery]` | Efter `?` i URL | `/api/products?sort=price` |
| `[FromRoute]` | Del af URL-stien | `/api/products/42` |
| `[FromBody]` | JSON i request body | `POST /api/products` med `{"name": "Laptop"}` |

---

## 11. Opsummering og Eksamensrelevante Punkter

### Centrale begreber

| Begreb | Definition |
|--------|-----------|
| Usability Heuristikker | Nielsens 10 principper for god UI |
| Shneidermans 8 regler | 8 regler for interface design |
| CRAP | Contrast, Repetition, Alignment, Proximity |
| Persona | Fiktiv bruger baseret pa research |
| Think Aloud | Testmetode hvor brugere siger hvad de taenker |
| Lo-Fi Prototype | Papir/sketches — hurtig og billig |
| Hi-Fi Prototype | Digital og interaktiv — realistisk |
| Design Sprint | 5-dages designproces |

### Nielsens 10 Heuristikker — Oversigt

| # | Navn | Kortversion |
|---|------|------------|
| 1 | Visibility of Status | Fortael brugeren hvad der sker |
| 2 | Real World Match | Brug brugerens sprog |
| 3 | User Control | Tillad fortryd og annuller |
| 4 | Consistency | Vaer konsistent i design og sprog |
| 5 | Error Prevention | Forbyg fejl foer de sker |
| 6 | Recognition | Vis muligheder, krav ikke hukommelse |
| 7 | Flexibility | Hjaelp bade begyndere og eksperter |
| 8 | Minimalism | Vis kun relevant information |
| 9 | Error Recovery | Hjaelp brugere med at komme sig efter fejl |
| 10 | Help & Docs | Tilbyd hjaelp nar det behoeves |

### Typiske eksamenssporgsmaal

1. **Naevn og forklar 3 af Nielsens heuristikker med eksempler**
   - Vaelg dem du husker bedst og giv konkrete eksempler

2. **Hvad er CRAP-principperne?**
   - Forklar alle 4 med eksempler

3. **Hvad er en persona, og hvad bor den indeholde?**
   - Definition + eksempel pa godt indhold

4. **Beskriv Think Aloud testing**
   - Fremgangsmade, hvad man kigger efter

5. **Hvad er forskellen pa Lo-Fi og Hi-Fi prototyper?**
   - Nar bruger man hvad?

### Husk til eksamen
- Personas er baseret pa RIGTIG research — ikke gisninger
- Nielsens regel: 5 deltagere i Think Aloud afsloerer 85% af usability-problemer
- CRAP er ikke kun for designere — det er relevant nar du laver rapporter, prasentationer, etc.
- Design Sprint er 5 dage — dag 1 forsta, dag 2 diverger, dag 3 beslut, dag 4 prototyper, dag 5 test
