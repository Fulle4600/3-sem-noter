# Uge 9 — TDD, Continuous Integration & Brugerinddragelse

Denne uge har to hoveddele: Mandag fokuserer vi pa Test-Driven Development (TDD) og Continuous Integration (CI) som tekniske praksisser der sikrer kvalitet i agil udvikling. Torsdag skifter vi fokus til brugerinddragelse — hvorfor det er afgerende at inddrage rigtige brugere i udviklingsprocessen, og hvilke metoder vi kan bruge. Begge dele er centrale i XP-tankegangen.

---

## DEL 1: TDD og Continuous Integration

---

## 1. Test-Driven Development (TDD)

**TDD** er en udviklingsteknik hvor tests skrives **foer** implementeringen. Det lyder bakvendt, men det har mange fordele.

### TDD-cyklussen: Red-Green-Refactor

```
       +--------+
       |        |
  +--> |  ROD   | Skriv en fejlende test
  |    |        |
  |    +--------+
  |         |
  |         v
  |    +--------+
  |    |        |
  |    | GROEN  | Skriv den mindste kode der gor testen groen
  |    |        |
  |    +--------+
  |         |
  |         v
  |    +----------+
  |    |          |
  +--- | REFAKTOR | Forbedre koden uden at aendre adfaerd
       |          |
       +----------+
```

1. **Rod (Red)**: Skriv en test der beskriver den oenskede adfaerd — testen **skal** fejle (roed), fordi koden ikke eksisterer endnu

2. **Groen (Green)**: Skriv den **mindst moeglige kode** for at testen bestaar — bare nok til at ga fra roed til groen. Det er OK at skrive "hacky" kode her.

3. **Refaktorer (Refactor)**: Forbedr koden uden at aendre dens ydre adfaerd. Alle tests skal stadig vaere groenne efter refaktoring.

Gentag cyklussen for naeste funktion.

### TDD Eksempel Trin for Trin

Vi vil implementere en metode der beregner rabat baseret pa kurvens totalbeloeb.

#### Trin 1: Skriv foerste test (ROD)

```csharp
[TestClass]
public class RabatBeregnerTests
{
    [TestMethod]
    public void BeregnRabat_Under500Kr_IngenRabat()
    {
        // Arrange
        var beregner = new RabatBeregner();

        // Act
        decimal rabat = beregner.BeregnRabat(300m);

        // Assert
        Assert.AreEqual(0m, rabat);
    }
}
```

Denne test **fejler** fordi `RabatBeregner` ikke eksisterer endnu. Det er ROD-fasen.

#### Trin 2: Mindste implementering (GROEN)

```csharp
public class RabatBeregner
{
    public decimal BeregnRabat(decimal beloeb)
    {
        return 0m; // Mindste moeglige implementering!
    }
}
```

Testen bestaar nu. Det er GROEN-fasen.

#### Trin 3: Naeste test (ROD igen)

```csharp
[TestMethod]
public void BeregnRabat_500KrEllerMere_10ProcentRabat()
{
    var beregner = new RabatBeregner();
    decimal rabat = beregner.BeregnRabat(500m);
    Assert.AreEqual(50m, rabat); // 10% af 500
}
```

Denne test fejler! Returner stadig 0.

#### Trin 4: Implementering (GROEN)

```csharp
public decimal BeregnRabat(decimal beloeb)
{
    if (beloeb >= 500m)
        return beloeb * 0.10m;

    return 0m;
}
```

Begge tests bestaar.

#### Trin 5: Naeste test

```csharp
[TestMethod]
public void BeregnRabat_1000KrEllerMere_20ProcentRabat()
{
    var beregner = new RabatBeregner();
    decimal rabat = beregner.BeregnRabat(1000m);
    Assert.AreEqual(200m, rabat); // 20% af 1000
}
```

#### Trin 6: Implementering og Refaktoring

```csharp
public decimal BeregnRabat(decimal beloeb)
{
    if (beloeb >= 1000m)
        return beloeb * 0.20m;

    if (beloeb >= 500m)
        return beloeb * 0.10m;

    return 0m;
}

// Refaktoring — gor det mere laesbart:
public decimal BeregnRabat(decimal beloeb)
{
    return beloeb switch
    {
        >= 1000m => beloeb * 0.20m,
        >= 500m  => beloeb * 0.10m,
        _        => 0m
    };
}
```

### Fordele ved TDD

1. **Bedre design**: Naar du skriver tests foer kode, tvinger det dig til at taenke pa snitflader og design
2. **Levende dokumentation**: Tests forklarer hvad koden goer
3. **Tryg refaktoring**: Du kan omstrukturere kode med tillid
4. **Faa bugs**: Bugs fanges med det samme
5. **Fokus**: TDD tvinger dig til at implementere een ting ad gangen

### Baby Steps i TDD

Baby Steps princippet er CENTRALT i TDD:
- Skriv een test ad gangen
- Implementer kun det der er noedvendigt for den aktuelle test
- Refaktorer kun naar alle tests er groenne
- Commit ofte (efter hvert groen-refaktor cyklus)

---

## 2. Continuous Integration (CI)

**Continuous Integration (CI)** er en praksis hvor alle udviklere integrerer (merger) deres kode ind pa en faelles main-branch **hyppigt** — typisk mindst dagligt.

### Problemet CI loser

Forestil dig et team pa 5 udviklere, der alle arbejder pa separate branches i 2 uger:

```
Uge 1:
  Anders  ----Branch A---------+
  Maja    ----Branch B---------+---> MERGE CONFLICT HELVETE!
  Frederik----Branch C---------+
  ...

Uge 2:
  Alle branches divergerer mere og mere
  Merge til sidst tager dage
```

Med CI:
```
Dag 1: Anders committer -> CI korer tests -> GROEN
Dag 1: Maja committer -> CI korer tests -> GROEN
Dag 2: Frederik committer -> CI korer tests -> ROD -> FIX!
```

### CI Principper

1. **Integrer hyppigt**: Mindst dagligt, ideelt set efter hvert faerdigt feature
2. **Autonatisk test**: CI-serveren korer alle tests automatisk
3. **Fix straks**: Nar bygget er roedt, er det teamets hoejeste prioritet at fikse det
4. **Synlighed**: Alle kan se om buildet er groen eller rod

### GitHub Actions — Konkret CI Eksempel

GitHub Actions er et populaert CI/CD vaerktoej. Konfiguration gemmes som YAML:

```yaml
# .github/workflows/ci.yml

name: CI Pipeline

# Korer nar nogen pusher eller laver PR til main
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    # 1. Hent kode fra repository
    - name: Checkout kode
      uses: actions/checkout@v3

    # 2. Installeer .NET 8
    - name: Setup .NET 8
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '8.0.x'

    # 3. Restore NuGet pakker
    - name: Restore dependencies
      run: dotnet restore

    # 4. Byg projektet
    - name: Build
      run: dotnet build --no-restore --configuration Release

    # 5. Kore alle tests
    - name: Test
      run: dotnet test --no-build --verbosity normal --collect:"XPlat Code Coverage"

    # 6. Upload code coverage rapport
    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

### Hvad en god CI Pipeline indeholder

```
Push/PR
   |
   v
+------------------+
| Lint / Format    | <- Sikrer kodestil (StyleCop, Roslyn analyzers)
+------------------+
   |
   v
+------------------+
| Build            | <- Sikrer at koden kompilerer
+------------------+
   |
   v
+------------------+
| Unit Tests       | <- Hurtige tests
+------------------+
   |
   v
+------------------+
| Integration Tests| <- Langsommere tests
+------------------+
   |
   v
+------------------+
| Code Coverage    | <- Minimum threshold (f.eks. 80%)
+------------------+
   |
   v
+------------------+
| Security Scan    | <- OWASP / Dependency check
+------------------+
   |
   v
+------------------+
| Deploy to Staging| <- Automatisk deploy (CD)
+------------------+
```

### Ten-Minute Build

XP's **Ten-Minute Build** princip siger, at en komplet build + test-suite bor kore pa under 10 minutter.

**Hvorfor 10 minutter?**
- Hvis buildet tager 30+ minutter, venter udviklere eller skippper at kore tests
- 10 minutter er en psykologisk granse — det er acceptabelt at vente
- Tvinger teams til at holde tests hurtige (undgaa for mange E2E tests)

**Strategier for hurtig build:**
- Paralleliser tests
- Brug hurtige in-memory databases i tests
- Undgaa unodvendige E2E tests
- Cache dependencies i CI

### Incremental Deployment / Daily Deployment

**Incremental Deployment**: Deploy sma aendringer hyppigt i stedet for store releases sjeldent.

Fordele:
- Nemmere at finde hvad der foraarsagede et problem
- Lavere risiko per deployment
- Hurtigere feedback fra rigtige brugere
- Lettere rollback

**Deployment strategier:**
```
Feature Flags (Feature Toggles):
  - Kode i produktion, men skjult bag et flag
  - Tand/sluk features uden ny deployment

Blue-Green Deployment:
  - To identiske produktionsmiljoer
  - Skift trafik fra Blue til Green
  - Rollback: skift tilbage til Blue

Canary Release:
  - Deploy til 5% af brugere forst
  - Overvag fejl og performance
  - Rul gradvist ud til alle
```

---

## DEL 2: Brugerinddragelse

---

## 3. Hvad er Brugerinddragelse?

**Brugerinddragelse** betyder at inddrage de faktiske slutbrugere i udviklingsprocessen — ikke bare kunden eller produktejeren, men de mennesker der rent faktisk skal bruge systemet.

### Hvorfor er det vigtigt?

1. **Kunden ved ikke alt**: Kunden ved hvad de tror de vil have, men brugerne ved hvad de har brug for
2. **Validering**: Tidlig feedback fra brugere afsloerer misforstaelser
3. **Fordelagtig oplevelse**: Systemer designet MED brugere bruges mere

**Klassisk fejl**: Et team bygger et system i 6 maneder baseret pa kundens krav. Nar det endelig rulles ud, opager brugerne, at det er svart at bruge, fordi ingen spurgte dem.

### XP: Real Customer Involvement

XP-princippet **Real Customer Involvement** er konkret:

- En **rigtig** kunde (ideelt: en faktisk bruger) er del af teamet
- De besvarer sporgsmal dagligt
- De prioriterer User Stories
- De accepterer eller afviser leveret arbejde

> "The customer is not someone who throws requirements over the wall. They are a member of the team."

---

## 4. Incremental Design

**Incremental Design** (ogsaa kaldet Evolutionary Design) er princippet om at design vokser og udvikler sig loebende med koden — i stedet for at blive fastlagt fra start.

### Traditionelt Design vs. Incremental Design

```
Traditionelt (Big Design Up Front - BDUF):
  Krav -> [Lang designfase] -> Implementering -> ...

  Problem: Det "perfekte" design naes aldrig,
  fordi krav altid aendrer sig.

Incremental Design:
  Krav -> [Lille design] -> Implementering -> Test
       -> [Lille design] -> Implementering -> Test
       -> [Lille design] -> ...

  Design tilpasses baseret pa hvad man laerer.
```

### Hvornaar designer man?

- **Inden en story**: Lav en enkel, tilstraeklig design
- **Under implementering**: Refaktorer naar bedre design opstaar
- **Efter levering**: Revurder design baseret pa hvad du larte

**YAGNI-princippet (You Ain't Gonna Need It):**
```csharp
// FORKERT - over-engineering
public class BrugerService
{
    // "Vi kan jo have brug for det en dag..."
    public void EksporterTilXML() { ... }
    public void EksporterTilPDF() { ... }
    public void EksporterTilCSV() { ... }
    public void SyncMedExternalSystem() { ... }
    public void BeregnStatistik() { ... }
}

// RIGTIGT - implementer kun hvad der er brug for NU
public class BrugerService
{
    public Bruger HentBruger(int id) { ... }
    public void OpretBruger(Bruger bruger) { ... }
    public void OpdaterBruger(Bruger bruger) { ... }
}
```

---

## 5. User Research Metoder

### Observation

**Observation** betyder at iagttage brugere, mens de bruger systemet (eller et eksisterende system):

- **Kontekstuel observation**: Observer brugere i deres naturlige miljo (pa arbejdspladsen, hjemme)
- **Laboratorieobservation**: Brugere udforer opgaver i kontrollerede omgivelser

**Hvad man kigger efter:**
- Hvilke opgaver loeser de dagligt?
- Hvor er de frustrerede?
- Hvilke omveje tager de?
- Hvad glemmer de ofte?

**Vigtig regel**: Undga at hjaelpe brugeren, nar de kaeemper. Vent og observer — det er vaerdifuld data!

### Interviews

**Brugerinterviews** giver dybere indsigt i brugerens behov, motivationer og kontekst.

**Typer:**
- **Struktureret**: Faste sporgsmal til alle
- **Semi-struktureret**: Kerne-sporgsmal, men fleksibelt
- **Ustruktureret**: Aaben samtale

**Gode interviewteknikker:**
```
Aabne sporgsmal (GODT):
"Fortael mig om sidst du bestilte noget online?"
"Hvad er det svaereste ved dit arbejde?"
"Hvad ville gere dit liv lettere?"

Lukkede sporgsmal (undgaa):
"Bruger du vores app?"
"Kan du lide den?"
"Er det let at navigere?"

Ledende sporgsmal (undgaa!):
"Er det ikke irriterende, nar systemet er langsomt?"
"Du ville vel gerne have en dark mode?"
```

---

## 6. Scope Creep

**Scope Creep** er faenomenet, hvor et projekts omfang gradvist udvides uden at det reflekteres i tid, pris eller planlagning.

### Tegn pa scope creep

- "Kan vi ikke bare ogsa tilfoeje..."
- "Det er kun en lille aendring..."
- Backloggen vokser hurtigere end teamet kan levere
- Deadlines rykkes gentagne gange

### Arsager til scope creep

1. Uklar kravspecifikation fra start
2. Ingen formaliseret change-management proces
3. Kunden/stakeholders har direkte adgang til udviklere
4. Productejeren er ikke stærk nok til at sige nej

### Hvordan undgaar man scope creep?

1. **Klare DoD og acceptance criteria** for hver story
2. **Product Owner** som gatekeeper — al ny funktionalitet gaar via PO og backlog
3. **Sprint backlog er laast** — nye opgaver i neste sprint, ikke dette
4. **Negotiated Scope Contract** — kunden accepterer at aendre scope koster noget
5. **Loebende prioritering** — PO prioriterer hyppigt og aabent
6. **Burn Down Chart** som synligt vaerktoej — viser konsekvensen af ny scope

### Hvad goer man nar scope creep opstaar?

```
Ny opgave opstaar midt i et sprint:

Scenario 1: Det er kritisk (produktionsbug)
  -> Stop noget andet, lav det, start naeste sprint som planlagt

Scenario 2: Det er en ny feature
  -> Tilfoej til Product Backlog
  -> PO prioriterer den
  -> Den kommer i et fremtidigt sprint

Scenario 3: Det aendrer en eksisterende story
  -> Diskuter med PO om det aendrer estimat
  -> Juster sprint-backloggen (fjern noget andet)
```

---

## 7. Opsummering og Eksamensrelevante Punkter

### Centrale begreber

| Begreb | Definition |
|--------|-----------|
| TDD | Skriv tests foer kode — Red, Green, Refactor |
| CI | Automatisk integration og test ved hvert commit |
| Ten-Minute Build | Build + tests skal kore pa < 10 min |
| Incremental Design | Design vokser evolutionaert med koden |
| Scope Creep | Ukontrolleret udvidelse af projektets omfang |
| Real Customer Involvement | Faktiske brugere er del af teamet |
| YAGNI | You Ain't Gonna Need It — implementer kun hvad der er brug for |
| Feature Flag | Kode i produktion, men deaktiveret bag et flag |

### Typiske eksamenssporgsmaal

1. **Beskriv TDD-cyklussen med et konkret eksempel**
   - Rod -> Groen -> Refaktor med C# kode

2. **Hvad er Continuous Integration, og hvorfor er det vigtigt?**
   - Hyppig integration, automatiske tests, hurtig feedback

3. **Forklar Ten-Minute Build princippet**
   - Kort build-tid = udviklere korer faktisk tests

4. **Hvad er Scope Creep, og hvordan haandterer man det?**
   - Definition, arsager, Product Owner som gatekeeper

5. **Hvad menes med "Real Customer Involvement" i XP?**
   - Rigtige brugere i teamet, ikke bare kunden

6. **Hvad er YAGNI?**
   - Implementer kun det der er brug for nu

### Husk til eksamen
- TDD handler om design, ikke kun om tests — tests driver designet frem
- CI korer automatisk nar nogen pusher kode
- Scope Creep haandteres af Product Owner — ikke af udviklerne
- Incremental Design er IKKE det samme som ingen design — det er evolutionaer design
