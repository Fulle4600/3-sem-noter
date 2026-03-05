# Uge 13 — Selenium UI Testing

Denne uge introducerer End-to-End (E2E) testing med Selenium WebDriver. Hvor unit tests tester isolerede kodedele og integration tests tester samspillet mellem klasser, tester E2E tests hele applikationen — fra brugerens klik i browseren til databasen og tilbage igen. Selenium automatiserer en rigtig browser og simulerer brugernes handlinger. Det er et kraftfuldt redskab til at verificere at din applikation fungerer korrekt set fra brugerens perspektiv.

---

## Hvad er End-to-End Testing?

### Testpyramiden

```
        /\
       /  \      E2E Tests (Selenium)
      /    \     - Tester hele systemet
     /------\    - Langsom, men realistisk
    /        \
   /          \  Integrationstests
  /            \ - Tester samspil mellem lag
 /--------------\
/                \ Unit Tests (xUnit)
/                 \ - Tester isolerede enheder
/------------------\ - Hurtig, mange tests
```

**Mange unit tests, færre integrationstests, få men vigtige E2E tests.**

### Hvornår bruger man Selenium?

- Test af kritiske brugerflows (login, checkout, opret produkt)
- Verificering af UI fungerer korrekt i en rigtig browser
- Regression testing — sikre at eksisterende funktionalitet ikke brydes
- Cross-browser testing

---

## Selenium WebDriver Setup

### NuGet Pakker

```bash
dotnet add package Selenium.WebDriver
dotnet add package Selenium.WebDriver.ChromeDriver
dotnet add package Selenium.Support
```

Eller via NuGet Package Manager i Visual Studio.

### Alternativt med xUnit og Selenium

```bash
dotnet new xunit -n WebshopApp.UITests
dotnet add WebshopApp.UITests/WebshopApp.UITests.csproj package Selenium.WebDriver
dotnet add WebshopApp.UITests/WebshopApp.UITests.csproj package Selenium.WebDriver.ChromeDriver
```

---

## ChromeDriver — Grundlæggende Setup

```csharp
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
using Xunit;

namespace WebshopApp.UITests
{
    public class ProduktUITests : IDisposable
    {
        private readonly IWebDriver _driver;
        private readonly WebDriverWait _wait;
        private const string BaseUrl = "http://localhost:5500";

        public ProduktUITests()
        {
            // Konfigurér ChromeOptions
            var options = new ChromeOptions();
            // options.AddArgument("--headless");  // Kør uden GUI (til CI/CD)
            options.AddArgument("--no-sandbox");
            options.AddArgument("--disable-dev-shm-usage");
            options.AddArgument("--window-size=1920,1080");

            // Start Chrome
            _driver = new ChromeDriver(options);
            _driver.Manage().Window.Maximize();

            // Implicite timeouts
            _driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(10);

            // WebDriverWait til eksplicitte waits
            _wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(15));
        }

        // Dispose kører efter HVER test
        public void Dispose()
        {
            _driver.Quit();    // Luk browseren
            _driver.Dispose();
        }
    }
}
```

---

## Lokalisering af elementer — Selektorer

Selenium bruger `By`-klassen til at finde HTML-elementer:

```csharp
// Find ét element
IWebElement element;

// Via ID (mest præcis og hurtigst)
element = _driver.FindElement(By.Id("søgefelt"));

// Via CSS-selektor (fleksibel og kraftfuld)
element = _driver.FindElement(By.CssSelector(".card-title"));
element = _driver.FindElement(By.CssSelector("#app button[type='submit']"));
element = _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));

// Via XPath (kraftfuld men svær at læse)
element = _driver.FindElement(By.XPath("//h5[@class='card-title']"));
element = _driver.FindElement(By.XPath("//button[text()='Opret produkt']"));

// Via tag-navn
element = _driver.FindElement(By.TagName("h1"));

// Via link-tekst
element = _driver.FindElement(By.LinkText("Se detaljer"));
element = _driver.FindElement(By.PartialLinkText("detaljer"));

// Find ALLE matchende elementer
IReadOnlyList<IWebElement> alleKort = _driver.FindElements(By.CssSelector(".card"));
```

---

## Interaktion med elementer

```csharp
// Klik
knap.Click();

// Skriv tekst i inputfelt
inputFelt.Clear();                    // Ryd feltet først
inputFelt.SendKeys("Laptop");         // Skriv tekst
inputFelt.SendKeys(Keys.Enter);       // Tryk Enter
inputFelt.SendKeys(Keys.Tab);         // Tryk Tab

// Læs indhold
string tekst = element.Text;          // Synlig tekst
string værdi = inputFelt.GetAttribute("value");  // Input-feltets værdi
string klasser = element.GetAttribute("class");  // CSS-klasser

// Tjek tilstand
bool erSynlig = element.Displayed;
bool erAktiv = element.Enabled;
bool erValgt = checkbox.Selected;

// Dropdown (Select)
var dropdown = new SelectElement(_driver.FindElement(By.Id("kategori")));
dropdown.SelectByText("Elektronik");
dropdown.SelectByValue("elektronik");
dropdown.SelectByIndex(2);
```

---

## Eksplicitte Waits — Vent på tilstand

Webapps er asynkrone — Selenium skal vente på at elementer er klar:

```csharp
var wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(10));

// Vent på at element er synligt
var element = wait.Until(ExpectedConditions.ElementIsVisible(
    By.CssSelector(".alert-success")
));

// Vent på at element er klikbart
var knap = wait.Until(ExpectedConditions.ElementToBeClickable(
    By.Id("submitKnap")
));

// Vent på at tekst er til stede
wait.Until(ExpectedConditions.TextToBePresentInElement(
    _driver.FindElement(By.Id("status")),
    "Produktet er oprettet"
));

// Vent på at URL indeholder bestemt tekst
wait.Until(ExpectedConditions.UrlContains("/produkter"));

// Vent på at element IKKE eksisterer
wait.Until(driver => {
    var elementer = driver.FindElements(By.CssSelector(".spinner-border"));
    return elementer.Count == 0;
});
```

---

## Komplet Selenium UI Test Suite

```csharp
using OpenQA.Selenium;
using OpenQA.Selenium.Chrome;
using OpenQA.Selenium.Support.UI;
using SeleniumExtras.WaitHelpers;
using Xunit;

namespace WebshopApp.UITests
{
    public class ProduktUITests : IDisposable
    {
        private readonly IWebDriver _driver;
        private readonly WebDriverWait _wait;
        private const string BaseUrl = "http://localhost:5500";

        public ProduktUITests()
        {
            var options = new ChromeOptions();
            _driver = new ChromeDriver(options);
            _driver.Manage().Window.Maximize();
            _driver.Manage().Timeouts().ImplicitWait = TimeSpan.FromSeconds(5);
            _wait = new WebDriverWait(_driver, TimeSpan.FromSeconds(10));
        }

        // =============================================
        // HJÆLPEMETODER
        // =============================================

        private void NavigerTilApp()
        {
            _driver.Navigate().GoToUrl(BaseUrl + "/index.html");
        }

        private void UdfyldOgSubmitForm(string navn, decimal pris, int lager, string? beskrivelse = null)
        {
            var navnFelt = _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
            navnFelt.Clear();
            navnFelt.SendKeys(navn);

            var prisFelt = _driver.FindElement(By.CssSelector("input[type='number']"));
            prisFelt.Clear();
            prisFelt.SendKeys(pris.ToString("F2"));

            var lagerFelt = _driver.FindElements(By.CssSelector("input[type='number']"))[1];
            lagerFelt.Clear();
            lagerFelt.SendKeys(lager.ToString());

            if (beskrivelse != null)
            {
                var beskrivelsesFelt = _driver.FindElement(By.TagName("textarea"));
                beskrivelsesFelt.Clear();
                beskrivelsesFelt.SendKeys(beskrivelse);
            }

            var submitKnap = _driver.FindElement(By.CssSelector("button[type='submit']"));
            submitKnap.Click();
        }

        private void VentPåSuccesAlert()
        {
            _wait.Until(ExpectedConditions.ElementIsVisible(
                By.CssSelector(".alert-success")
            ));
        }

        // =============================================
        // TESTS: SIDE INDLÆSNING
        // =============================================

        [Fact]
        public void Forside_HarKorrektTitel()
        {
            // Arrange & Act
            NavigerTilApp();

            // Assert
            Assert.Equal("Produkt Administration", _driver.Title);
        }

        [Fact]
        public void Forside_HarOpretProduktFormular()
        {
            // Arrange & Act
            NavigerTilApp();

            // Assert — tjek at formularen eksisterer
            var form = _driver.FindElement(By.TagName("form"));
            Assert.NotNull(form);

            var navnFelt = _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
            Assert.True(navnFelt.Displayed);
        }

        // =============================================
        // TESTS: OPRET PRODUKT
        // =============================================

        [Fact]
        public void OpretProdukt_GyldigeData_ViserSuccesAlert()
        {
            // Arrange
            NavigerTilApp();

            // Act
            UdfyldOgSubmitForm("Selenium Test Laptop", 9999m, 10, "Test beskrivelse");

            // Assert
            VentPåSuccesAlert();
            var alert = _driver.FindElement(By.CssSelector(".alert-success"));
            Assert.Contains("oprettet", alert.Text.ToLower());
        }

        [Fact]
        public void OpretProdukt_NytProdukt_VisesPåSiden()
        {
            // Arrange
            NavigerTilApp();
            var uniktNavn = $"Test Produkt {DateTime.Now.Ticks}";

            // Act
            UdfyldOgSubmitForm(uniktNavn, 499m, 5);
            VentPåSuccesAlert();

            // Assert — verificer at produktet vises i listen
            var produktNavne = _driver.FindElements(By.CssSelector("td strong"));
            var navne = produktNavne.Select(el => el.Text).ToList();
            Assert.Contains(uniktNavn, navne);
        }

        [Fact]
        public void OpretProdukt_TomtNavn_ViserValidationsfejl()
        {
            // Arrange
            NavigerTilApp();

            // Act — submit uden at udfylde navn
            var submitKnap = _driver.FindElement(By.CssSelector("button[type='submit']"));
            submitKnap.Click();

            // Assert — tjek HTML5 validation eller vores fejlbesked
            var navnFelt = _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
            // HTML5 required-attribut forhindrer submit
            Assert.True(navnFelt.GetAttribute("required") != null);
        }

        // =============================================
        // TESTS: REDIGÉR PRODUKT
        // =============================================

        [Fact]
        public void RedigerProdukt_ÆndrerNavn_OpdatererIListen()
        {
            // Arrange — sørg for at der er mindst ét produkt
            NavigerTilApp();
            string originaltNavn = $"Redigér Mig {DateTime.Now.Ticks}";
            UdfyldOgSubmitForm(originaltNavn, 100m, 1);
            VentPåSuccesAlert();

            // Act — find og klik Redigér-knap
            var redigérKnapper = _driver.FindElements(By.CssSelector("button.btn-outline-primary"));
            redigérKnapper.Last().Click();

            // Vent på at formularen er udfyldt
            _wait.Until(d => {
                var felt = d.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
                return felt.GetAttribute("value") == originaltNavn;
            });

            // Opdater navn
            var navnFelt = _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
            navnFelt.Clear();
            string nytNavn = originaltNavn + " (opdateret)";
            navnFelt.SendKeys(nytNavn);

            var submitKnap = _driver.FindElement(By.CssSelector("button[type='submit']"));
            submitKnap.Click();

            // Assert
            VentPåSuccesAlert();
            var alert = _driver.FindElement(By.CssSelector(".alert-success"));
            Assert.Contains("opdateret", alert.Text.ToLower());
        }

        // =============================================
        // TESTS: SLET PRODUKT
        // =============================================

        [Fact]
        public void SletProdukt_BeknkræfterSletning_FjernesFragListen()
        {
            // Arrange
            NavigerTilApp();
            string produktNavn = $"Slet Mig {DateTime.Now.Ticks}";
            UdfyldOgSubmitForm(produktNavn, 50m, 1);
            VentPåSuccesAlert();

            // Tæl produkter FØR sletning
            int antalFør = _driver.FindElements(By.CssSelector("tbody tr")).Count;

            // Act — klik Slet og accepter confirm-dialog
            var sletKnapper = _driver.FindElements(By.CssSelector("button.btn-outline-danger"));
            sletKnapper.Last().Click();

            // Håndter browser confirm-dialog
            var alert = _driver.SwitchTo().Alert();
            alert.Accept();  // Klik "OK"

            // Vent på succes
            VentPåSuccesAlert();

            // Assert
            int antalEfter = _driver.FindElements(By.CssSelector("tbody tr")).Count;
            Assert.Equal(antalFør - 1, antalEfter);
        }

        // =============================================
        // TESTS: SØGNING
        // =============================================

        [Fact]
        public void Søgefelt_Søgning_FilterererResultater()
        {
            // Arrange
            NavigerTilApp();

            // Act
            var søgefelt = _driver.FindElement(By.CssSelector("input[placeholder*='Søg']"));
            søgefelt.SendKeys("Apple");

            // Vent på at resultater opdateres (debounce ~ 300ms)
            System.Threading.Thread.Sleep(500);

            // Assert — alle synlige produktnavne indeholder "Apple"
            var produktNavne = _driver.FindElements(By.CssSelector("td strong"));
            foreach (var el in produktNavne)
            {
                Assert.Contains("apple", el.Text.ToLower());
            }
        }

        public void Dispose()
        {
            _driver.Quit();
            _driver.Dispose();
        }
    }
}
```

---

## Page Object Model (POM)

Page Object Model er et designmønster der samler sidespecifik Selenium-logik i egne klasser. Det gør tests mere vedligeholdbare.

```csharp
// Page Object for Produktsiden
public class ProduktPage
{
    private readonly IWebDriver _driver;
    private readonly WebDriverWait _wait;

    public ProduktPage(IWebDriver driver)
    {
        _driver = driver;
        _wait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
    }

    // Selektorer som properties
    private IWebElement NavnFelt => _driver.FindElement(By.CssSelector("input[placeholder='Produktnavn']"));
    private IWebElement PrisFelt => _driver.FindElements(By.CssSelector("input[type='number']"))[0];
    private IWebElement SubmitKnap => _driver.FindElement(By.CssSelector("button[type='submit']"));

    // Handlinger som metoder
    public void OpretProdukt(string navn, decimal pris, int lager)
    {
        NavnFelt.Clear();
        NavnFelt.SendKeys(navn);
        PrisFelt.Clear();
        PrisFelt.SendKeys(pris.ToString());
        SubmitKnap.Click();
    }

    public bool HarSuccesAlert()
    {
        try {
            _wait.Until(ExpectedConditions.ElementIsVisible(By.CssSelector(".alert-success")));
            return true;
        } catch {
            return false;
        }
    }

    public List<string> HentAlleProduktNavne()
    {
        return _driver.FindElements(By.CssSelector("td strong"))
                      .Select(el => el.Text)
                      .ToList();
    }
}

// Test med Page Object
[Fact]
public void OpretProdukt_MedPageObject_ViserSucces()
{
    _driver.Navigate().GoToUrl(BaseUrl);
    var side = new ProduktPage(_driver);

    side.OpretProdukt("POM Test Produkt", 999m, 5);

    Assert.True(side.HarSuccesAlert());
}
```

---

## Kørsel af Selenium Tests

### Krav

1. Applikationen KØREr lokalt (backend API + frontend)
2. Chrome browser er installeret
3. ChromeDriver-versionen matcher din Chrome-version

### Via terminal

```bash
cd WebshopApp.UITests
dotnet test
```

### Via Visual Studio Test Explorer

- Åbn Test Explorer
- Kør UI-tests (de tager længere tid end unit tests)

---

## Opsummering

| Begreb               | Beskrivelse                                                      |
|----------------------|------------------------------------------------------------------|
| Selenium WebDriver   | Automatiserer browserinteraktioner til E2E testing              |
| ChromeDriver         | Driver der styrer Chrome-browseren                               |
| By.Id / By.CssSelector | Metoder til at lokalisere HTML-elementer                        |
| WebDriverWait        | Explicit wait — vent på bestemt tilstand                         |
| ImplicitWait         | Global timeout for at finde elementer                            |
| IDisposable          | Interface der sikrer browser lukkes efter hver test              |
| Page Object Model    | Designmønster der samler side-logik i egne klasser               |
| Headless Chrome      | Chrome kører uden GUI — nyttigt til CI/CD pipelines              |

---

## Opgaver til selvstudium

1. Opsæt et Selenium xUnit-testprojekt i din solution
2. Skriv en test der verificerer at din forside indlæser korrekt
3. Skriv en test der opretter et produkt og verificerer at det vises i listen
4. Skriv en test der sletter et produkt og verificerer at antal reduceres
5. Refaktorer dine tests til at bruge Page Object Model
6. Eksperimentér med at køre Chrome headless (`--headless`-argument)
