# Uge 8 — Test & Kvalitetssikring

Denne uge handler om systematisk softwaretest og kvalitetssikring. Vi gennemgar testpyramiden fra unit tests til end-to-end tests, laerer at bruge Mock/Stub/Fake objekter, og arbejder med AAA-monstret til strukturering af tests. Test er en central del af XP og agil udvikling generelt — det er fundamentet for at kunne levere kvalitetssoftware med tillid.

---

## 1. Hvad er Softwarekvalitet? (Testtilgang)

Softwarekvalitet handler om at sikre, at software opfylder:
- **Funktionelle krav**: Goer det hvad det skal?
- **Non-funktionelle krav**: Er det hurtigt, sikkert, tilgaengeligt?

Test er det primaere vaerktoej til at verificere begge typer krav.

### Hvorfor teste?

1. **Fejl er billigst at finde tidligt**: En fejl fundet i kravfasen koster 10x mindre at rette end en fejl fundet i produktion
2. **Dokumentation**: Tests dokumenterer forventet adfaerd
3. **Tillid til aendringer**: Med tests kan man refaktorere trygt
4. **Kommunikation**: Tests kommunikerer kravene til naeste udvikler

### Begreber

| Begreb | Definition |
|--------|-----------|
| Test | En procedure der verificerer om koden opfylder forventet adfaerd |
| Testcase | Et specifikt scenarie der testes |
| Test Suite | En samling af relaterede tests |
| Test Coverage | Procentdel af kode der daekkes af tests |
| Regression | En ny aendring der bryder eksisterende funktionalitet |
| Regression test | Test der sikrer at eksisterende funktionalitet stadig virker |

---

## 2. Test Pyramiden

Testpyramiden (Mike Cohn) er en model for, hvordan man bor fordele sine tests:

```
          /\
         /  \
        / E2E\          <- Faa, langsomme, dyre
       /------\
      /  Integ. \       <- Middel
     /------------\
    /   Unit Tests  \   <- Mange, hurtige, billige
   /________________\
```

### Layer 1: Unit Tests (Bunden)

**Hvad**: Test af den mindste enhed i koden — typisk en enkelt metode eller klasse.

**Karakteristika**:
- Korer meget hurtigt (millisekunder)
- Isolerede — ingen afhaengigheder af database, filsystem, netvaerk
- Mange af dem (typisk 70-80% af alle tests)
- Nemme at skrive og vedligeholde

**Eksempel i C#**:
```csharp
[TestClass]
public class KalkulatorTests
{
    [TestMethod]
    public void Laeg_Til_TalOgTal_ReturnererKorrektSum()
    {
        // Arrange
        var kalkulator = new Kalkulator();

        // Act
        int resultat = kalkulator.LaegtTil(3, 5);

        // Assert
        Assert.AreEqual(8, resultat);
    }

    [TestMethod]
    public void LaegtTil_NegativeTal_ReturnererKorrektSum()
    {
        var kalkulator = new Kalkulator();
        int resultat = kalkulator.LaegtTil(-3, -5);
        Assert.AreEqual(-8, resultat);
    }

    [TestMethod]
    [ExpectedException(typeof(DivideByZeroException))]
    public void Divider_MedNul_KasterException()
    {
        var kalkulator = new Kalkulator();
        kalkulator.Divider(10, 0);
    }
}
```

### Layer 2: Integrationstests (Midten)

**Hvad**: Test af samspillet mellem to eller flere komponenter. F.eks. service + database, controller + service.

**Karakteristika**:
- Langsommere end unit tests (sekunder)
- Kraever ofte database, filsystem eller andre services
- Faerre end unit tests (typisk 15-20%)
- Tester at komponenter samarbejder korrekt

**Eksempel — test af repository med database**:
```csharp
[TestClass]
public class BrugerRepositoryIntegrationTests
{
    private AppDbContext _dbContext;
    private BrugerRepository _repository;

    [TestInitialize]
    public void Setup()
    {
        // Brug in-memory database til tests
        var options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
            .Build();
        _dbContext = new AppDbContext(options);
        _repository = new BrugerRepository(_dbContext);
    }

    [TestMethod]
    public async Task HentBruger_EksisterendeBrugerId_ReturnererBruger()
    {
        // Arrange
        var bruger = new Bruger { Id = 1, Navn = "Anders Andersen", Email = "anders@example.com" };
        _dbContext.Brugere.Add(bruger);
        await _dbContext.SaveChangesAsync();

        // Act
        var hentetBruger = await _repository.HentBrugerAsync(1);

        // Assert
        Assert.IsNotNull(hentetBruger);
        Assert.AreEqual("Anders Andersen", hentetBruger.Navn);
    }

    [TestCleanup]
    public void Cleanup()
    {
        _dbContext.Dispose();
    }
}
```

### Layer 3: End-to-End Tests / UI Tests (Toppen)

**Hvad**: Test af hele systemet fra brugerens perspektiv — typisk via browseren.

**Karakteristika**:
- Langsome (minutter)
- Skroebrele og svaere at vedligeholde
- Meget faa (typisk 5-10%)
- Bruger vaerktojer som Selenium, Playwright, Cypress

**Eksempel (Playwright):**
```csharp
// End-to-End test med Playwright
[Test]
public async Task Login_GyldigeBrugerkredentialer_OmstillerTilDashboard()
{
    await Page.GotoAsync("https://app.example.com/login");

    await Page.FillAsync("#username", "bruger@example.com");
    await Page.FillAsync("#password", "hemmeligt123");
    await Page.ClickAsync("#login-button");

    await Page.WaitForURLAsync("**/dashboard");

    var velkomstbesked = await Page.TextContentAsync("h1");
    Assert.That(velkomstbesked, Does.Contain("Velkommen"));
}
```

### Sammenligning af testtyper

| | Unit Test | Integrationstest | E2E Test |
|-|-----------|-----------------|---------|
| **Hastighed** | Meget hurtig | Medium | Langsom |
| **Paalidelighed** | Meget palidelig | Palidelig | Skroebelig |
| **Isolation** | Fuld isolation | Delvis | Ingen |
| **Scope** | En metode/klasse | To+ komponenter | Hele systemet |
| **Antal** | Mange (70-80%) | Middel (15-20%) | Faa (5-10%) |
| **Vedligehold** | Let | Medium | Svaert |

---

## 3. Mock, Stub og Fake

Nar vi skriver unit tests, vil vi isolere den kode vi tester fra dens afhaengigheder (database, netvaerk, andre tjenester). Til det bruges **Test Doubles**:

### Terminologi

| Type | Hvad det er | Hvornaar bruges det |
|------|------------|---------------------|
| **Dummy** | Et objekt der sendes rundt men aldrig rigtig bruges | Nar en parameter kreves men ikke bruges i den test |
| **Stub** | Returnerer foruddefinerede vaerdier | Nar du vil kontrollere hvad en afhaengighed returnerer |
| **Mock** | Verificerer at specifikke metoder blev kaldt | Nar du vil teste at din kode kalder afhaengigheder korrekt |
| **Fake** | En rigtig implementering — bare simplere | F.eks. in-memory database |
| **Spy** | En stub der ogsa registrerer kald | Sjeldent brugt |

### Stub Eksempel

En stub returnerer praedefinerede data uden at goere noget rigtigt:

```csharp
// Interface
public interface IVejrService
{
    decimal HentTemperatur(string by);
}

// Stub implementering
public class VejrServiceStub : IVejrService
{
    private decimal _temperatur;

    public VejrServiceStub(decimal temperatur)
    {
        _temperatur = temperatur;
    }

    public decimal HentTemperatur(string by)
    {
        return _temperatur; // Returnerer altid den forudindstillede vaerdi
    }
}

// Test der bruger stub
[TestMethod]
public void HentPaaklaidningsanbefaling_Varmt_AnbefalerTShirt()
{
    // Arrange
    var vejrStub = new VejrServiceStub(25.0m); // Simulerer 25 graders vejr
    var service = new PaaklaidningsService(vejrStub);

    // Act
    string anbefaling = service.HentAnbefaling("Kobenhavn");

    // Assert
    Assert.AreEqual("T-shirt og shorts", anbefaling);
}
```

### Mock Eksempel (med Moq bibliotek)

En mock verificerer at bestemte metoder blivet kaldt:

```csharp
using Moq;

[TestMethod]
public void OpretBruger_GyldigBruger_KalderEmailService()
{
    // Arrange
    var mockEmailService = new Mock<IEmailService>();
    var brugerService = new BrugerService(mockEmailService.Object);
    var nyBruger = new Bruger { Navn = "Maja Jensen", Email = "maja@example.com" };

    // Act
    brugerService.OpretBruger(nyBruger);

    // Assert - verificer at SendVelkomstEmail blev kaldt een gang
    mockEmailService.Verify(
        e => e.SendVelkomstEmail(It.IsAny<string>()),
        Times.Once
    );
}

[TestMethod]
public void OpretBruger_GyldigBruger_SenderEmailTilKorrektAdresse()
{
    // Arrange
    var mockEmailService = new Mock<IEmailService>();
    mockEmailService
        .Setup(e => e.SendVelkomstEmail("maja@example.com"))
        .Returns(true);

    var brugerService = new BrugerService(mockEmailService.Object);
    var nyBruger = new Bruger { Navn = "Maja Jensen", Email = "maja@example.com" };

    // Act
    brugerService.OpretBruger(nyBruger);

    // Assert
    mockEmailService.Verify(
        e => e.SendVelkomstEmail("maja@example.com"),
        Times.Once
    );
}
```

### Fake Eksempel

En fake er en simplificeret men funktionel implementering:

```csharp
// Rigtig repository bruger SQL Server
public class BrugerRepository : IBrugerRepository
{
    private readonly AppDbContext _db;
    // ... SQL Server implementering
}

// Fake repository bruger in-memory Dictionary
public class FakeBrugerRepository : IBrugerRepository
{
    private readonly Dictionary<int, Bruger> _brugere = new();
    private int _nextId = 1;

    public void Tilfoej(Bruger bruger)
    {
        bruger.Id = _nextId++;
        _brugere[bruger.Id] = bruger;
    }

    public Bruger HentById(int id)
    {
        return _brugere.ContainsKey(id) ? _brugere[id] : null;
    }

    public IEnumerable<Bruger> HentAlle()
    {
        return _brugere.Values;
    }
}
```

---

## 4. AAA Pattern (Arrange, Act, Assert)

**AAA-monstret** er en standard maade at strukturere test pa:

- **Arrange**: Opsaet de noedvendige objekter, data og forudsaetninger
- **Act**: Udfore den handling der testes
- **Assert**: Verificer at resultatet er som forventet

### Eksempel

```csharp
[TestMethod]
public void BeregneRabat_PlatinumKunde_GirFemtiProcentsRabat()
{
    // ARRANGE - saet scenen
    var kunde = new Kunde
    {
        Navn = "Viggo Vestergaard",
        KundeType = KundeType.Platinum,
        TotalKoeb = 50000m
    };
    var rabatService = new RabatService();
    decimal originalPris = 1000m;

    // ACT - udfore handlingen
    decimal rabatteret = rabatService.BeregnPris(originalPris, kunde);

    // ASSERT - verificer resultatet
    Assert.AreEqual(500m, rabatteret, "Platinum kunder skal have 50% rabat");
}
```

### Fordele ved AAA
1. **Laesbarhed**: Klart adskilt hvad der saettes op, hvad der testes, hvad der forventes
2. **Struktur**: Alle tests folger samme monstre — lettere for naeste udvikler
3. **Debug**: Nar en test fejler, er det nemt at se i hvilken fase fejlen opstod

### Given-When-Then (BDD alternativ)

I Behaviour-Driven Development (BDD) bruges en lignende struktur:
```
Given [kontekst/forforudsaetning]
When  [handlingen]
Then  [det forventede resultat]
```

---

## 5. Test af Non-Functional Requirements

Non-functional requirements (NFR) kan og bor ogsa testes — men det kraever andre vaerktojer end unit tests.

### Performance Test

```csharp
[TestMethod]
public void HentAlleOrdrer_Under200ms()
{
    // Arrange
    var stopwatch = new Stopwatch();
    var orderService = new OrderService(new TestDbContext());

    // Act
    stopwatch.Start();
    var ordrer = orderService.HentAlleOrdrer();
    stopwatch.Stop();

    // Assert
    Assert.IsTrue(stopwatch.ElapsedMilliseconds < 200,
        $"HentAlleOrdrer tog {stopwatch.ElapsedMilliseconds}ms — forventet < 200ms");
}
```

### Load Testing (vaerktojer)
- **JMeter**: Open source load testing tool
- **k6**: Moderne load testing med JavaScript
- **Azure Load Testing**: Cloud-baseret load testing

### Sikkerheds Test
- **OWASP ZAP**: Automatisk scanning for sikkerhedshuller
- **Penetration testing**: Manuel test af sikkerhed
- **Static Analysis**: SonarQube scanner kode for sikkerhedsprobler

### Tilgaengeligheds Test
- **Lighthouse** (Chrome DevTools): Automatisk tilgaengeligheds-rapport
- **axe**: Browser extension til tilgaengeligheds-test
- **Manuelt**: Test med skaermlaesere (NVDA, VoiceOver)

---

## 6. God Testkode — Principper

### FIRST-principper for gode unit tests

| Bogstav | Princip | Forklaring |
|---------|---------|-----------|
| **F** | Fast | Tests skal kore hurtigt — ellers korer folk dem ikke |
| **I** | Isolated | Tests skal vaere uafhaengige af hinanden |
| **R** | Repeatable | Tests skal give samme resultat hver gang |
| **S** | Self-validating | Tests skal sige pass/fail automatisk |
| **T** | Timely (eller Thorough) | Skriv tests tidligt; daek alle edge cases |

### Test-naming konventioner

Gode testnavne beskriver:
1. **Hvad der testes** (metode/funktion)
2. **Under hvilke betingelser** (input/scenarie)
3. **Hvad det forventede resultat er**

```csharp
// GODT - beskrivende navn
[TestMethod]
public void LaegtTil_ToPositiveTal_ReturnererKorrektSum() { ... }

[TestMethod]
public void Login_ForkertPassword_KasterUnauthorizedException() { ... }

[TestMethod]
public void HentBruger_BrugerIkkeEksisterer_ReturnererNull() { ... }

// DÅRLIGT - ubeskrivenede navne
[TestMethod]
public void Test1() { ... }

[TestMethod]
public void TestLogin() { ... }
```

### En assertion per test (princip)

Generelt: Hold tests fokuserede med een logisk assertion per test. Det giver bedre fejlmeddelelser nar tests fejler.

```csharp
// PROBLEMATISK - hvad fejlede?
[TestMethod]
public void OpretBruger_GemmerAlleData()
{
    var service = new BrugerService();
    var bruger = service.OpretBruger("Anders", "anders@test.dk");

    Assert.AreEqual("Anders", bruger.Navn);
    Assert.AreEqual("anders@test.dk", bruger.Email);
    Assert.IsNotNull(bruger.Id);
    Assert.IsNotNull(bruger.OprettetDato);
}

// BEDRE - een logisk assertion (dog kan grouped assertions vaere OK)
[TestMethod]
public void OpretBruger_GemmerNavn()
{
    var service = new BrugerService();
    var bruger = service.OpretBruger("Anders", "anders@test.dk");
    Assert.AreEqual("Anders", bruger.Navn);
}
```

---

## 7. Testframeworks i C#

### MSTest (Microsofts eget)
```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;

[TestClass]
public class MinTest
{
    [TestInitialize] // Kores foer HVER test
    public void Setup() { ... }

    [TestCleanup]    // Kores EFTER hver test
    public void Teardown() { ... }

    [TestMethod]     // En test
    public void MinTest_Betingelse_ForventetResultat() { ... }

    [DataRow(1, 2, 3)]  // Parameterised test
    [DataRow(0, 0, 0)]
    [DataTestMethod]
    public void LaegtTil_Parameteriseret(int a, int b, int forventet) { ... }
}
```

### NUnit (populaert alternativ)
```csharp
using NUnit.Framework;

[TestFixture]
public class MinTest
{
    [SetUp]
    public void Setup() { ... }

    [TearDown]
    public void Teardown() { ... }

    [Test]
    public void MinTest_Betingelse_ForventetResultat() { ... }

    [TestCase(1, 2, 3)]
    [TestCase(0, 0, 0)]
    public void LaegtTil_Parameteriseret(int a, int b, int forventet) { ... }
}
```

### xUnit (moderne og populaert i .NET)
```csharp
using Xunit;

public class MinTest
{
    // Constructor bruges til Setup
    public MinTest() { ... }

    [Fact]
    public void MinTest_Betingelse_ForventetResultat() { ... }

    [Theory]
    [InlineData(1, 2, 3)]
    [InlineData(0, 0, 0)]
    public void LaegtTil_Parameteriseret(int a, int b, int forventet) { ... }
}
```

---

## 8. XP: Quality og Improvement i Testperspektiv

### Quality (Kvalitet) og tests

XP siger: **"Add testing until fear is transformed into boredom."**

Det betyder:
- Skriv tests til du er tryg ved at deploye
- Ikke til du har 100% coverage (det er et middel, ikke et maal)
- Fokuser pa de kritiske dele af systemet

### Improvement (Forbedring) og tests

Forbedre din testsuite loebende:
- Nar du finder en bug — skriv en test der beviser buggen FOER du retter den
- Refaktorer tests ligesom du refaktorer kode
- Slet tests der ikke giver vaerdi
- Tilfoej tests for edge cases du opdager undervejs

---

## 9. Opsummering og Eksamensrelevante Punkter

### Centrale begreber

| Begreb | Definition |
|--------|-----------|
| Unit Test | Isoleret test af een metode/klasse |
| Integrationstest | Test af samspil mellem to+ komponenter |
| E2E Test | Test af hele systemet via UI |
| Mock | Test double der verificerer metodekald |
| Stub | Test double der returnerer praedefinerede vaerdier |
| Fake | Forenklet men funktionel implementering |
| AAA Pattern | Arrange, Act, Assert — standard teststruktur |
| Testpyramide | Mange unit tests, faa E2E tests |

### Typiske eksamenssporgsmaal

1. **Forklar testpyramiden og begrundelsen for dens form**
   - Hurtige unit tests i bunden, sakte E2E tests i toppen

2. **Hvad er forskellen pa en mock og en stub?**
   - Stub: kontrollerer input/output. Mock: verificerer at metoder kaldes

3. **Vis AAA-monstret med et konkret eksempel**
   - Skriv en simpel C# test med tydeligt Arrange, Act, Assert

4. **Hvornaar bruger man integrationstests vs unit tests?**
   - Unit tests: isoleret logik. Integrationstests: samspil med database/service

5. **Hvordan tester man non-functional requirements?**
   - Performance: stopwatch/load testing. Sikkerhed: OWASP. Tilgaengelighed: axe/Lighthouse

### Husk til eksamen
- Tests er dokumentation — et godt testnavn forklarer hvad koden goer
- Test Doubles (mock/stub/fake) bruges til at isolere den kode der testes
- Testpyramiden: Mange hurtige unit tests er bedre end faa langsomme E2E tests
- AAA er standard — brug det konsekvent
