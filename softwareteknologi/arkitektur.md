# Arkitektur Begreber

## Indholdsfortegnelse

- [SOLID Principper](#solid-principper)
- [Repository Pattern](#repository-pattern)
- [MVC — Model View Controller](#mvc--model-view-controller)
- [Dependency Injection (DI)](#dependency-injection-di)
- [Design Patterns (Designmønstre)](#design-patterns-designmønstre)
- [Code Review](#code-review)
- [Refactoring](#refactoring)
- [Teknisk Gæld (Technical Debt)](#teknisk-gæld-technical-debt)

---

## SOLID Principper

5 principper for objektorienteret design der giver maintainable og udvidebar kode.
Kode der følger SOLID er lettere at ændre, teste og forstå.

### S — Single Responsibility Principle
**En klasse = ét ansvar / én grund til at ændre den.**

```csharp
// FORKERT: UserService der håndterer auth, email OG database
// RIGTIGT:
class AuthService { ... }
class EmailService { ... }
class UserRepository { ... }
```

### O — Open/Closed Principle
**Klasser skal være ÅBNE for udvidelse men LUKKEDE for ændring.**

Udvid via inheritance/interfaces — ændr ikke eksisterende kode der virker.

### L — Liskov Substitution Principle
**En subklasse skal ALTID kunne erstatte sin superklasse uden at programmet går i stykker.**

> Hvis `Duck extends Bird`, og `Bird.fly()` virker, så skal `Duck.fly()` OGSÅ virke.

### I — Interface Segregation Principle
**Mange specifikke interfaces er bedre end ét stort generelt interface.**

Klasser bør IKKE tvinges til at implementere metoder de ikke behøver.

### D — Dependency Inversion Principle
**Afhæng af ABSTRAKTIONER (interfaces), ikke konkrete implementationer.**

```csharp
// FORKERT — hårdt koblet:
class UserService { var repo = new SqlUserRepository(); }

// RIGTIGT — fleksibelt:
class UserService(IUserRepository repo) { this.repo = repo; }
```

→ Let at skifte implementering (SQL til MongoDB). Let at mocke i tests.

---

## Repository Pattern

Abstraktionslag mellem business logic og data access layer.
Skift database uden at ændre business logic. Let at mocke i unit tests.

```csharp
// Interface
interface IUserRepository {
    GetById(int id), GetAll(), Create(User u), Update(User u), Delete(int id)
}

// Implementeringer
class SqlUserRepository : IUserRepository { ... }
class MongoUserRepository : IUserRepository { ... }
class InMemoryUserRepository : IUserRepository { ... }  // til tests
```

Business logic kalder ALTID interfacet — ved aldrig hvilken database der er bag.

---

## MVC — Model View Controller

Arkitekturmønster der opdeler applikation i 3 ansvarsområder.
**Separation of concerns** — UI-ændringer påvirker ikke business logic og omvendt.

| Del | Ansvar |
|---|---|
| **Model** | Data og business logic. Kender INTET til UI. |
| **View** | Præsentation — hvad brugeren ser. Kender INTET til business logic. |
| **Controller** | Broker — modtager requests, kalder Model, returnerer View. |

> Eksempel: ASP.NET Core MVC, Vue.js (MVVM)

---

## Dependency Injection (DI)

Klassen **modtager** sine afhængigheder udefra i stedet for at oprette dem selv.
Testbarhed (inject mock), fleksibilitet (skift implementering), loose coupling.

```csharp
// ASP.NET — registrer i Program.cs
builder.Services.AddScoped<IUserRepository, SqlUserRepository>();
```

---

## Design Patterns (Designmønstre)

Genanvendelige løsninger på kendte softwareproblemer. Ikke kode — men skabeloner.
Undgå at opfinde hjulet igen. Fælles sprog: *"brug et Singleton her"*.

| Kategori | Mønstre |
|---|---|
| **Creational** (objekt-oprettelse) | Singleton, Factory, Builder |
| **Structural** (sammensætning) | Adapter, Decorator, Facade |
| **Behavioral** (kommunikation) | Observer, Strategy, Command |

---

## Code Review

En anden udvikler gennemser din kode systematisk **før** den merges til main.
Find bugs, sikre kodekvalitet, videndeling i teamet, fælles ejerskab.

**Fokuspunkter:** Logik og korrekthed, arkitektur og mønstre, sikkerhed, læsbarhed, testdækning.

**Pull Request:** GitHub/GitLab mekanisme til at foreslå ændringer og anmode om review.

---

## Refactoring

Forbedring af kode **UDEN** at ændre dens ydre adfærd (output og sideeffekter er samme).
Bekæmper teknisk gæld. Gør kode lettere at læse, teste og ændre.

**Eksempler:** Rename variabel, extract method, remove duplication, simplify condition.

> **Forudsætning:** Du skal have tests der bekræfter adfærden FØR du refaktorerer!

---

## Teknisk Gæld (Technical Debt)

Akkumuleret "gæld" af dårlig kode, hurtige fixes og manglende tests som skal betales tilbage.

**Opstår pga.:** Tidspres, manglende erfaring, "det virker jo" mentalitet.

**Konsekvens:** Nye features tager længere og længere tid. Bugs stiger. Udviklere hader kodebasen.

**Løsning:** Refactoring, tests, code reviews. XP's praksisser forebygger teknisk gæld.

---

## Opgaver til selvstudium

1. Forklar de 5 SOLID-principper med egne ord og giv et konkret eksempel på et brud for hvert princip
2. Hvad er Repository Pattern, og hvad er fordelen ved at bruge et interface i stedet for en konkret klasse?
3. Tegn et MVC-diagram for en simpel webshop og annotér hvilken del der håndterer databasekald, HTTP-requests og HTML-rendering
4. Forklar Dependency Injection: hvad er forskellen på at en klasse "opretter" sin afhængighed vs. at den "modtager" den?
5. Hvad er et Design Pattern? Hvad er forskellen på Singleton, Factory og Observer — og hvornår bruges de?
6. Hvad er forskellen på `AddSingleton`, `AddScoped` og `AddTransient` i ASP.NET Core's DI-container?
