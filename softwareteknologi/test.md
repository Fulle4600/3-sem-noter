# Test Begreber

## Indholdsfortegnelse

- [TDD — Test-Drevet Udvikling](#tdd--test-drevet-udvikling)
- [Test Pyramiden](#test-pyramiden)
- [Unit Tests](#unit-tests)
- [Mock / Stub / Fake](#mock--stub--fake)
- [Integrationstest](#integrationstest)
- [E2E Tests (End-to-End)](#e2e-tests-end-to-end)
- [Continuous Integration (CI)](#continuous-integration-ci)
- [Non-Functional Requirements (Ikke-funktionelle krav)](#non-functional-requirements-ikke-funktionelle-krav)

---

## TDD — Test-Drevet Udvikling

Udviklingsmetode hvor du skriver **TEST** før du skriver implementeringen.
Tvinger dig til at tænke på API/interface før kode. Sikrer at AL kode er testet.

### TDD Cyklus: Rød → Grøn → Refaktorer

```
🔴 RØD       Skriv en test der FEJLER (koden eksisterer endnu ikke!)
🟢 GRØN      Skriv MINIMUM kode til at testen består (ingen over-engineering!)
🔵 REFAKTORER  Forbedr koden uden at bryde testen. Fjern duplikering, forbedre navne.
              Gentag for næste lille funktion.
```

### Fordele ved TDD

- Al kode er testet (100% test-coverage er muligt)
- Tests tjener som **levende dokumentation** af forventet adfærd
- Tvinger simpelt design (kode der er svær at teste er ofte dårligt designet)
- Fejl opdages STRAKS — ikke uger/måneder senere

---

## Test Pyramiden

Model der viser den ideelle fordeling af testtyper.

```
        /\
       /E2E\          ← Få, langsomme, dyre — tester hele brugerflows
      /------\
     /  Inte- \       ← Middel antal — tester at komponenter virker SAMMEN
    / grations \
   /------------\
  /  Unit Tests  \    ← Mange, hurtige, billige — tester enkelt funktion/metode
 /________________\
```

For mange E2E tests = langsomt og skrøbeligt. Pyramideformen giver hurtig, billig og pålidelig test-suite.

---

## Unit Tests

Test af den **mindste** mulige kode-enhed (én metode/funktion) i isolation.
Find bugs tidligt, dokumenterer forventet adfærd, giver confidence til refactoring.

- **Isoleret:** Afhængigheder (database, API) erstattes med mocks/stubs.
- **AAA Pattern:** Arrange (opsæt data) → Act (kald kode) → Assert (verificer resultat).
- **C# værktøjer:** xUnit, NUnit, MSTest

---

## Mock / Stub / Fake

Falske objekter der erstatter rigtige afhængigheder (database, email-service) under test.
Isoler kode fra ydre afhængigheder. Gør tests hurtige og forudsigelige.

| Type | Beskrivelse |
|---|---|
| **Stub** | Returnerer foruddefinerede værdier. *"Når GetUser(42) kaldes, returner denne fake bruger."* |
| **Mock** | Verificerer INTERAKTIONER. *"Kontroller at SendEmail() blev kaldt én gang med dette argument."* |

```csharp
// C# — Moq bibliotek
new Mock<IUserRepository>()
```

---

## Integrationstest

Test at **flere** komponenter/moduler virker **korrekt sammen**.
Unit tests kan bestå men systemet fejler alligevel (integration er knækkede).

> Eksempel: Test at din Controller + Service + Repository + Database fungerer korrekt end-to-end.

Langsommere end unit tests men hurtigere end E2E.

---

## E2E Tests (End-to-End)

Tester **hele brugerflows** fra brugerens perspektiv — simulerer en rigtig bruger.
Finder fejl der kun opstår når ALLE dele kører sammen.

> Eksempel: "Åbn browser, gå til login, indtast credentials, verificer du er logget ind."

**Værktøjer:** Playwright, Cypress, Selenium. Langsomst og dyreste — brug kun til kritiske flows.

---

## Continuous Integration (CI)

Praksis om at merge kode til main branch hyppigt (dagligt eller flere gange om dagen).
Find konflikter og fejl **tidligt** frem for sidst i projektet (katastrofe).

- **CI Pipeline:** Automatisk bygge + test kode ved hvert commit/push.
- **Værktøjer:** GitHub Actions, GitLab CI, Jenkins, Azure DevOps.

> **Golden regel:** Grøn CI pipeline = koden virker. Rød pipeline = STOP alt, ret fejlen.

---

## Non-Functional Requirements (Ikke-funktionelle krav)

Krav der beskriver **HVORDAN** systemet skal være — ikke hvad det skal gøre.

| Type | Eksempel |
|---|---|
| **Functional** (hvad) | "Bruger kan logge ind" |
| **Non-functional** (kvalitet) | "Login tager maks 1 sekund" |

### Typiske non-functional krav

| Kategori | Eksempel |
|---|---|
| **Performance** | Svar inden for 2 sekunder under 1000 samtidige brugere |
| **Sikkerhed** | Passwords skal hashes. SQL injection ikke mulig. |
| **Tilgængelighed** | Systemet oppe 99,9% af tiden = maks 8,7 timer nedetid/år |
| **Skalerbarhed** | Kan håndtere 10x trafik uden redesign |

---

## Opgaver til selvstudium

1. Skriv en unit test for en simpel `ShoppingCart`-klasse med metoderne `AddItem()` og `GetTotal()` — brug AAA-mønsteret
2. Hvad er TDD? Beskriv Red-Green-Refactor cyklussen med et konkret eksempel
3. Hvad er forskellen på unit tests, integrationstests og E2E tests? Hvornår bruger man hvilken?
4. Hvad er et mock/stub? Hvornår og hvorfor erstatter man rigtige afhængigheder med mocks i tests?
5. Hvad er CI (Continuous Integration)? Hvad sker der automatisk i en CI-pipeline?
6. Forklar testpyramiden — hvorfor skal der være mange unit tests, men få E2E tests?
