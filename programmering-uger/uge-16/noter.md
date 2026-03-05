# Uge 16 — Storeopgave: User Stories, PR & Branching

Denne uge er en storeopgave (obligatorisk studieaktivitet) der kombinerer alt vi har lært: User Stories til at planlægge, Pair Programming til at kode, Unit Tests til at verificere, og GitHub Pull Requests + branches til at samarbejde professionelt.

---

## User Stories i Praksis

User Stories er mere end bare et format — de er et kommunikationsværktøj mellem kunden og teamet.

### Format

```
"Som [brugertype] vil jeg [handling] så jeg kan [formål/værdi]"
```

### Fra krav til User Story

**Krav (vagt):** "Brugere skal kunne logge ind"

**User Story (konkret):**
```
Som registreret bruger
vil jeg logge ind med email og password
så jeg kan tilgå mine personlige data sikkert
```

### Acceptance Criteria

Til hver User Story hører konkrete kriterier der definerer hvornår den er *færdig*:

```
User Story: Som kunde vil jeg se mine ordrer så jeg kan følge min leveringstatus

Acceptance Criteria:
✓ Jeg kan se en liste over alle mine ordrer
✓ Hver ordre viser ordrenummer, dato, status og samlet beløb
✓ Jeg kan klikke på en ordre for at se detaljer
✓ Ordrer sorteres med nyeste øverst
✓ Hvis jeg ingen ordrer har, vises en venlig besked
```

### Prioritering — MoSCoW

Når du har mange User Stories skal de prioriteres:

| Prioritet | Betydning |
|---|---|
| **M**ust Have | Kernen — uden dette er systemet ubrugeligt |
| **S**hould Have | Vigtigt men ikke kritisk til første release |
| **C**ould Have | Nice-to-have hvis der er tid |
| **W**on't Have | Ikke i dette iteration (men måske senere) |

---

## Pair Programming i Praksis

**To udviklere, én computer** — men det handler om meget mere end at dele skærm.

### Roller

| Rolle | Hvad du gør |
|---|---|
| **Driver** | Skriver kode. Fokuserer på den aktuelle linje og syntaks. |
| **Navigator** | Tænker på det store billede. Ser efter bugs, foreslår forbedringer, stiller spørgsmål. |

### Skift roller

- Skift driver/navigator jævnligt — f.eks. hvert 20. minut (Pomodoro-stil)
- Driver skal forklare hvad de gør — navigator skal forstå og udfordre

### Fordele

- **Færre bugs:** Fire øjne ser mere end to
- **Vidensdeling:** Begge lærer af hinanden
- **Bedre design:** Navigator holder styr på helheden mens driver fokuserer på detaljer
- **Ingen "lone wolf" kode:** Koden tilhører begge fra starten

### Remote Pair Programming

Med VS Code Live Share kan to udviklere arbejde på den samme fil i realtid:
```
VS Code → Extensions → "Live Share" → Share → Del linket
```

---

## GitHub Pull Request Workflow

### Hvad er en Pull Request?

En Pull Request (PR) er en formel anmodning om at merge din branch ind i main. Den åbner mulighed for:
- Code review inden merge
- Diskussion om implementeringen
- Automatiske tests (CI) kører inden godkendelse

### Workflow step by step

```bash
# 1. Hent seneste version af main
git checkout main
git pull origin main

# 2. Opret feature branch
git checkout -b feature/bruger-login

# 3. Kode og commit løbende
git add src/LoginController.cs
git commit -m "Add login endpoint with JWT validation"

git add tests/LoginControllerTests.cs
git commit -m "Add unit tests for login endpoint"

# 4. Push branch til GitHub
git push origin feature/bruger-login

# 5. Åbn Pull Request på GitHub (i browseren)
# GitHub foreslår automatisk at åbne en PR når du pusher en ny branch

# 6. Teammedlemmer reviewer og godkender

# 7. Merge til main (på GitHub)
# Valgmuligheder: Merge commit / Squash and merge / Rebase and merge

# 8. Slet branch (ryd op)
git branch -d feature/bruger-login
```

### En god PR-beskrivelse

```markdown
## Hvad gør denne PR?
Tilføjer login-funktionalitet med email og password via JWT tokens.

## Hvorfor?
User Story: "Som bruger vil jeg logge ind så jeg kan tilgå mine data"

## Test plan
- [x] Unit tests for LoginController
- [x] Integration test: login med gyldige credentials
- [x] Integration test: login med ugyldige credentials → 401
- [ ] Manuel test: test i Swagger UI

## Skærmbilleder (hvis UI-ændringer)
[...]
```

---

## GitHub Branching Strategier

### Feature Branch Workflow (mest brugt på uddannelsen)

```
main ──────────────────────────────────────────►
      \                /    \               /
       feature/login──       feature/cart──
```

- **main** er altid produktionsklar og stabil
- Al ny kode sker på feature branches
- Feature branch → PR → Review → Merge til main
- **Branch protection:** main kræver mindst 1 godkendelse + grøn CI

### Git Flow (større projekter)

```
main      ─────────────────────────────────► (kun releases)
develop   ──────────────────────────────────►
           \           /     \          /
            feature/x ─       feature/y ─
```

- `develop` er integration branch (dagligt arbejde)
- `main` opdateres kun ved releases
- Mere struktur, men mere overhead

### Trunk-Based Development (CI/CD)

```
main ────────────────────────────────────────►
       ↑ ↑ ↑ ↑ ↑ (mange små commits direkte)
```

- Alle committer direkte til main (eller meget kortlivede branches)
- Kræver stærke automated tests
- Bruges i teams med moden CI/CD pipeline

### Commit-beskeder — konventioner

```
// GOD: imperativ form, forklarer hvad og evt. hvorfor
feat: add user authentication with JWT
fix: correct total calculation when cart is empty
test: add unit tests for discount calculator
refactor: extract price calculation into separate method

// DÅRLIG: vag og ikke handlings-orienteret
updated stuff
fixed bug
wip
```

---

## Unit Tests — opsamling

### Hvornår skriver du tests?

I XP: **FØR** du skriver implementeringen (Test-First / TDD).
I praksis: Skriv i hvert fald tests som en del af den samme commit som koden.

### xUnit eksempel — komplet

```csharp
public class ShoppingCartTests {

    [Fact]
    public void AddItem_IncreasesTotalPrice() {
        // Arrange
        var cart = new ShoppingCart();
        var item = new CartItem { Name = "Bog", Price = 199.95m, Quantity = 2 };

        // Act
        cart.AddItem(item);

        // Assert
        Assert.Equal(399.90m, cart.Total);
    }

    [Fact]
    public void AddItem_EmptyCart_CountIsOne() {
        var cart = new ShoppingCart();
        cart.AddItem(new CartItem { Name = "Test", Price = 10m, Quantity = 1 });
        Assert.Single(cart.Items);
    }

    [Theory]
    [InlineData(0)]
    [InlineData(-1)]
    [InlineData(-100)]
    public void AddItem_NegativeOrZeroQuantity_ThrowsException(int quantity) {
        // Arrange
        var cart = new ShoppingCart();
        var item = new CartItem { Price = 10m, Quantity = quantity };

        // Act & Assert
        Assert.Throws<ArgumentException>(() => cart.AddItem(item));
    }
}
```

**`[Fact]`** — en test der altid er sand
**`[Theory]`** + **`[InlineData]`** — samme test med forskellige input (parametriseret)
