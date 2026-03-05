# Uge 12 — Teknisk Gæld, Code Review & Bæredygtighed

Denne uge binder vi en masse sammen: vi kigger på hvad der sker når kode gradvist bliver dårligere (teknisk gæld), hvordan vi bekæmper det med refactoring og code reviews, og vi slutter med bæredygtighed i IT — hvad har software med verden at gøre?

---

## Refactoring

**Refactoring = forbedring af kodens indre struktur uden at ændre dens ydre adfærd.**

Outputtet er det samme — men koden er renere, mere læsbar og nemmere at ændre.

### Hvornår refaktorerer man?

- Når du ser duplikeret kode (Copy-Paste antipattern)
- Når en metode er for lang (over ~20 linjer er et tegn)
- Når et navn ikke beskriver hvad koden faktisk gør
- Når en klasse har for mange ansvarsområder (SRP-brud)
- **Altid:** Før du tilføjer ny funktionalitet til rodet kode

### Eksempler på refactoring

**Extract Method** — Træk kode ud i en separat navngivet metode:
```csharp
// FØR — svær at forstå
public decimal CalculateTotal(Order order) {
    decimal subtotal = 0;
    foreach (var item in order.Items) subtotal += item.Price * item.Quantity;
    decimal tax = subtotal * 0.25m;
    decimal discount = order.Customer.IsMember ? subtotal * 0.1m : 0;
    return subtotal + tax - discount;
}

// EFTER — selvforklarende
public decimal CalculateTotal(Order order) {
    decimal subtotal = CalculateSubtotal(order.Items);
    decimal tax = CalculateTax(subtotal);
    decimal discount = CalculateDiscount(subtotal, order.Customer);
    return subtotal + tax - discount;
}

private decimal CalculateSubtotal(IEnumerable<OrderItem> items) =>
    items.Sum(i => i.Price * i.Quantity);
```

**Rename** — Giv variabler og metoder præcise navne:
```csharp
// FØR
int x = GetData(id);
// EFTER
int orderCount = GetOrderCountForCustomer(customerId);
```

**Replace Magic Number** — Erstat tal med navngivne konstanter:
```csharp
// FØR
if (age > 67) ApplyRetirementDiscount();
// EFTER
const int RetirementAge = 67;
if (age > RetirementAge) ApplyRetirementDiscount();
```

> **Forudsætning:** Du skal have tests der bekræfter adfærden FØR du refaktorerer, så du ved at du ikke brød noget.

---

## Teknisk Gæld (Technical Debt)

**Teknisk gæld = den akkumulerede pris af dårlige beslutninger i kodebasen.**

Som finansiel gæld: du "låner" tid nu (skriver hurtig, rodet kode) og betaler renter senere (fremtidige ændringer tager længere og længere tid).

### Årsager til teknisk gæld

| Årsag | Eksempel |
|---|---|
| **Tidspres** | "Vi skal nå deadline — gør det bare" |
| **Manglende erfaring** | Juniorudviklere kender ikke bedre mønstre |
| **"Det virker jo"** | Ingen refaktorerer velfungerende rodet kode |
| **Manglende tests** | Ingen turde ændre noget fordi det kan gå i stykker |
| **Scope Creep** | Features tilføjet hurtigt uden at tænke design |

### Symptomer på teknisk gæld

- **Frygtkulturen:** "Rør ikke den klasse — den bryder alt"
- Nye features tager dobbelt så lang tid som forventet
- Bugs popper op uventede steder når man retter én bug
- Ingen forstår hvad koden gør — ikke engang dem der skrev den

### Konsekvens: Velocity kurven

```
Hastighed
  ^
  |████
  |████████
  |████████████
  |████████████████
  |████████████████████____
  |████████████████████________
  +---------------------------------> Tid

Uden gæld: konstant hastighed
Med gæld:  hastighed falder gradvist mod nul
```

### Løsninger

1. **Refactoring** — aktivt arbejde med at forbedre koden
2. **Tests** — gør det trygt at ændre kode
3. **Code Reviews** — fang dårlige mønstre tidligt
4. **XP-praksisser** — TDD, Pair Programming, Continuous Integration forebygger gæld

> "Det tager altid lidt kortere tid at gøre det rigtigt end at forklare hvorfor du gjorde det forkert." — Martin Fowler

---

## Code Review

**Code Review = en anden udvikler gennemser din kode systematisk *før* den merges til main.**

### Formål

- **Find bugs** der overses af den der skrev koden
- **Sikre arkitekturkvalitet** (SOLID, patterns, navngivning)
- **Vidensdeling** — alle lærer af hinandens kode
- **Kollektivt ejerskab** — hele teamet forstår hele kodebasen

### Fokuspunkter i et code review

| Kategori | Spørgsmål |
|---|---|
| **Korrekthed** | Gør koden det den skal? Er der edge cases der ikke håndteres? |
| **Læsbarhed** | Kan en ny udvikler forstå koden uden forklaring? |
| **Arkitektur** | Følger koden SOLID? Er der unødvendig kompleksitet? |
| **Tests** | Er der tests? Dækker de de vigtige cases? |
| **Sikkerhed** | Er der SQL injection, XSS eller andre sikkerhedshuller? |
| **Performance** | Er der åbenlyse performance-problemer (fx N+1 queries)? |

### Pull Request (PR) workflow

```
1. Udvikler laver en feature branch: git checkout -b feature/tilfoej-login
2. Koder og committer løbende
3. Pusher branch til GitHub
4. Åbner en Pull Request mod main
5. Kollegaer reviewer — kommenterer, stiller spørgsmål, foreslår ændringer
6. Udvikler retter baseret på feedback og pusher igen
7. Reviewer godkender (approve)
8. Merge til main → CI pipeline kører
9. Branch slettes
```

### God vs dårlig review-feedback

```
// DÅRLIG feedback — subjektiv og usaglig
"Dette er grimt kode."

// GOD feedback — specifik og konstruktiv
"Denne metode gør tre ting (beregner, gemmer og sender mail).
 Overvej at opdele den for at følge SRP.
 Se linje 47-52."
```

### Husk: Reviews er ikke personlige angreb

Koden kritiseres — ikke personen. Et godt team har en kultur hvor feedback er velkommen og læring sker begge veje.

---

## Git Branches & Workflow

### Branch-strategier

**Feature Branch Workflow:**
```
main ─────────────────────────────────────►
         \                        /
          feature/login ─────────
```

- `main` er altid produktionsklar kode
- Al ny kode sker i feature branches
- Merge sker KUN via Pull Request + review + grøn CI

**Navngivning:**
```
feature/bruger-login
feature/produkt-soegning
bugfix/beregn-total-fejl
hotfix/kritisk-sikkerhedshul
```

### Merge Conflicts

Opstår når to branches har ændret den SAMME linje i den SAMME fil.

```
<<<<<<< HEAD (din branch)
return priceWithTax + shippingCost;
=======
return price + tax;
>>>>>>> feature/refactor-price
```

Du skal manuelt vælge hvilken version der er korrekt (eller kombinere dem) og derefter:
```bash
git add .
git commit -m "Resolve merge conflict in PriceCalculator"
```

**Forebyggelse:** Merge fra main ind i din feature branch ofte — så er dine konflikter små.

---

## Karlskrona Manifest — Bæredygtighed i IT

Manifest fra 2015 der erklærer at software-udviklere har et ansvar for bæredygtighed — langt ud over det rent tekniske.

### 5 Dimensioner af bæredygtighed

| Dimension | Hvad handler det om? | Eksempel i IT |
|---|---|---|
| **Miljø** | Energi, CO₂, ressourcer, e-affald | Serveroptimering reducerer strømforbrug |
| **Social** | Fællesskaber, lighed, tilgængelighed | App designet til alle — ikke kun unge med iPhone |
| **Økonomisk** | Langsigtet økonomi, fair fordeling | Open source der ikke låser kunder inde |
| **Individuel** | Helbred, privatliv, velvære | App der ikke skaber afhængighed eller misbruger data |
| **Teknisk** | Systemets evne til at overleve tid | Kode der kan vedligeholdes og videreudvikles |

### Dual Sfærer

Software har to typer påvirkning:

1. **Direkte:** Systemet selv — energiforbrug på serverne, privacy i appen
2. **Indirekte:** Hvad systemet *muliggør* — en ride-sharing app ændrer bytrafik

En GPS-app bruger måske lidt strøm (direkte) men ændrer måske hele transportmønstre i en by (indirekte).

### Teknisk Bæredygtighed

Kode der ikke er bæredygtig teknisk:
- Kan ikke ændres uden at alt går i stykker
- Ingen tør refaktorere den
- Umulig at teste
- Kun én person forstår den

**Teknisk bæredygtighed = XP-praksisser:** Tests, Refactoring, Collective Code Ownership, Simple Design — alt det vi har lært dette semester er teknisk bæredygtighed i praksis.

---

## Boehm & Turner — Metodevalg

Ramme til at vælge den rette udviklingsmetode til et givent projekt.

| Faktor | Agil | Planbaseret (Vandfald) |
|---|---|---|
| **Krav** | Ustabile, ændrer sig | Veldefinerede og stabile |
| **Team** | Lille (5-10 personer) | Stort, distribueret |
| **Kundekontakt** | Daglig, tæt | Begrænset (kontrakt-styret) |
| **Kritikalitet** | Lav (forretningsapp) | Høj (flystyring, medicin) |
| **Erfaring** | Erfarent, selvstyrende team | Varierende, behøver struktur |

**Kontinuum:** De fleste projekter er *et sted imellem* — rent vandfald eller ren XP er sjældent optimalt. Find den rette balance til dit konkrete projekt.

> "Der er ingen sølvkugle i softwareudvikling." — Fred Brooks

---

## Opsummering

| Begreb                      | Beskrivelse                                                                          |
|-----------------------------|--------------------------------------------------------------------------------------|
| Refactoring                 | Forbedring af kodens indre struktur uden at ændre dens ydre adfærd                  |
| Extract Method              | Refactoring-teknik: træk et stykke kode ud i en navngivet metode                    |
| Magic Number                | Ubeskrevet tal i koden — erstat med en navngivet konstant for læsbarhed             |
| Teknisk gæld                | Akkumuleret pris af dårlige designbeslutninger — "lånte" tid der betales med renter |
| Velocity-kurven             | Uden gæld: konstant hastighed. Med gæld: hastighed falder gradvist mod nul          |
| Code Review                 | Systematisk gennemgang af kode af en anden udvikler inden merge til main            |
| Pull Request                | Formel anmodning om merge — åbner for code review og diskussion                     |
| Feature Branch Workflow     | Al ny kode i feature branches — merge KUN via PR + review + grøn CI               |
| Merge Conflict              | To branches har ændret samme linje — skal løses manuelt                             |
| Karlskrona Manifest         | 5 dimensioner af bæredygtighed: Miljø, Social, Økonomisk, Individuel, Teknisk       |
| Dual Sfærer                 | Direkte påvirkning (appens eget energiforbrug) vs. indirekte (hvad appen muliggør)  |
| Teknisk bæredygtighed       | Kode der kan ændres, testes og forstås over tid — opnås via XP-praksisser           |
| Boehm & Turner              | Ramme til metodevalg: agil vs. planbaseret baseret på krav, team, kritikalitet osv. |

---

## Opgaver til selvstudium

1. Find et stykke kode du har skrevet i semester — hvilke tegn på teknisk gæld kan du identificere? Hvad ville du refaktorere?
2. Anvend "Extract Method" refactoring på et eksempel: tag en lang metode og opdel den i mindre, navngivne dele
3. Beskriv Karlskrona Manifestets 5 dimensioner med egne ord og giv et konkret IT-eksempel for hver dimension
4. Hvad er forskellen på Continuous Delivery og Continuous Deployment? Hvornår ville man foretrække det ene over det andet?
5. Brug Boehm & Turner-rammen til at vurdere dit eget semesterprojekt: hvilken udviklingsmetode passer bedst — og hvorfor?
6. Skriv to kommentarer til en kodestump (én dårlig review-kommentar og én god) — hvad gør den gode kommentar anderledes?
