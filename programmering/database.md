# Database Begreber

## Indholdsfortegnelse

- [SQL — Structured Query Language](#sql--structured-query-language)
- [ORM — Object-Relational Mapper](#orm--object-relational-mapper)
- [Entity Framework Core (EF)](#entity-framework-core-ef)
- [LINQ — Language Integrated Query](#linq--language-integrated-query)
- [Database Normalisering](#database-normalisering)
- [SQL Injection](#sql-injection)
- [Async/Await i Database Kald](#asyncawait-i-database-kald)

---

## SQL — Structured Query Language

Standardsprog til at kommunikere med relationsdatabaser (MySQL, SQL Server, PostgreSQL).
Alle relationsdatabaser forstår SQL — lær det én gang, virker overalt.

### CRUD operationer

```sql
-- CREATE
INSERT INTO users (name, email) VALUES ('Anna', 'anna@test.dk');

-- READ
SELECT name, email FROM users WHERE age > 18 ORDER BY name;

-- UPDATE
UPDATE users SET email = 'ny@test.dk' WHERE id = 42;

-- DELETE
DELETE FROM users WHERE id = 42;

-- JOIN
SELECT u.name, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
```

---

## ORM — Object-Relational Mapper

Lag der automatisk oversætter mellem C# klasser (objekter) og database tabeller (rows).
Du skriver C# kode — ORM'en genererer SQL. Undgår SQL injection, type-safe, reducerer boilerplate.

**Konceptet:** C# klasse `User` = database tabel `users`. C# object = database row.

**NET ORM:** Entity Framework Core (EF Core) — den mest brugte i .NET.

---

## Entity Framework Core (EF)

Microsofts ORM til .NET. Lader dig arbejde med databasedata som C# objekter.
Skriv C# — EF genererer SQL. Skifte database (SQL Server → PostgreSQL) = minimal kodeændring.

- **Code First:** Definer C# modeller → EF genererer databasestrukturen.
- **DbContext:** Klassen der repræsenterer databaseforbindelsen. Har `DbSet<T>` per tabel.

```csharp
var user = await db.Users.FirstOrDefaultAsync(u => u.Id == 42);
```

### EF Migrations

Sporer databaseændringer i kode — som **git for din database**.
Hold database-struktur synkroniseret med kode på tværs af udviklere og miljøer.

```bash
dotnet ef migrations add NyFunktion  # opret migration
dotnet ef database update             # anvend migration på database
```

---

## LINQ — Language Integrated Query

Forespørgselssprog integreret direkte i C# — virker med lister, databaser, XML.
Type-safe forespørgsler direkte i koden — ingen raw SQL strenge.

```csharp
var adults = users
    .Where(u => u.Age > 18)
    .OrderBy(u => u.Name)
    .ToList();
```

Med EF oversættes LINQ automatisk til SQL før det sendes til databasen.

---

## Database Normalisering

Processen med at organisere data for at reducere redundans og forbedre dataintegritet.
Undgå dataduplicering (samme by-navn 1000 steder = problem ved opdatering).

| Normalform | Krav |
|---|---|
| **1NF** | Alle felter er atomare (ikke lister i én celle) |
| **2NF** | Alle ikke-nøgle felter afhænger af den HELE primærnøgle |
| **3NF** | Ingen transitiv afhængighed (felt afhænger af felt der afhænger af nøgle) |

---

## SQL Injection

Sikkerhedsangreb hvor ondsindet SQL indsættes i et input-felt og køres mod databasen.

```sql
-- Bruger indtaster dette som brugernavn:
' OR 1=1; DROP TABLE users; --
-- → Sletter hele tabellen!
```

**Løsning:** Brug ALTID parameteriserede forespørgsler eller ORM.
ALDRIG string-konkatenering med brugerinput!

> EF Core er som standard beskyttet mod SQL injection (parameteriserer automatisk).

---

## Async/Await i Database Kald

Asynkron programmering der lader programmet lave andet mens det venter på databasesvar.
Database-kald tager tid (ms-sekunder). Synkront = tråd er blokeret = dårlig performance.

```csharp
public async Task<User> GetUser(int id) => await db.Users.FindAsync(id);
```

Alle EF Core metoder har en Async-version: `FindAsync`, `ToListAsync`, `SaveChangesAsync`.

---

## Opsummering

| Begreb                  | Beskrivelse                                                                 |
|-------------------------|-----------------------------------------------------------------------------|
| SQL                     | Standardsprog til relationsdatabaser — CRUD: SELECT, INSERT, UPDATE, DELETE |
| ORM                     | Oversætter automatisk mellem C#-objekter og databasetabeller                |
| EF Core                 | Microsofts ORM til .NET — skriv C#, EF genererer SQL                        |
| Code First              | Definer C#-modeller → EF genererer databasestrukturen                        |
| DbContext               | Repræsenterer databaseforbindelsen — har `DbSet<T>` for hver tabel           |
| Migrationer             | Versionsstyring af databaseskemaet — "git for din database"                 |
| LINQ                    | Forespørgselssprog i C# — oversættes til SQL af EF ved databasekald         |
| Normalisering           | Organiser data for at undgå redundans — 1NF, 2NF, 3NF                       |
| SQL Injection           | Angreb via ondsindet SQL i brugerinput — undgås med parameteriserede queries |
| Async/Await             | Asynkrone databasekald frigiver tråden mens den venter — bedre performance  |

---

## Opgaver til selvstudium

1. Skriv SQL-forespørgsler til at: hente alle produkter over 500 kr., opdatere et produktnavn, slette et produkt
2. Forklar forskellen på `INNER JOIN` og `LEFT JOIN` — giv et eksempel med to tabeller
3. Opret et EF Core-projekt med en `Product`-model og kør `dotnet ef migrations add InitialCreate`
4. Skriv en LINQ-forespørgsel der filtrerer produkter på pris, sorterer dem og returnerer kun navn og pris
5. Forklar SQL Injection med et konkret eksempel — og forklar hvordan parameteriserede forespørgsler forhindrer det
6. Hvad er forskellen på 1NF, 2NF og 3NF? Giv et eksempel på en tabel der bryder 2NF og vis hvordan du normaliserer den
