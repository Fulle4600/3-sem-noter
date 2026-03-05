# Uge 9 — HTML, CSS & Bootstrap

Denne uge skifter vi fokus fra backend til frontend og lærer grundlaget for webudvikling: HTML-strukturen der beskriver indhold, CSS der styler det, og Bootstrap 5 som giver os et responsivt designsystem ud af boksen. Vi afslutter med at se på, hvordan man deployer en statisk webside til Azure. Disse færdigheder danner fundamentet for alt frontend-arbejde i resten af semestret.

---

## HTML — Hypertext Markup Language

HTML er sproget vi bruger til at beskrive strukturen og indholdet af en webside. Det består af **elementer** og **tags**.

### Grundlæggende HTML-struktur

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <!-- Meta-information om siden (vises ikke på siden) -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Min Webshop</title>

    <!-- Link til CSS -->
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <!-- Sidens synlige indhold -->
    <header>
        <nav>
            <a href="/">Hjem</a>
            <a href="/produkter">Produkter</a>
        </nav>
    </header>

    <main>
        <h1>Velkommen til Min Webshop</h1>
        <p>Her finder du de bedste priser.</p>
    </main>

    <footer>
        <p>&copy; 2024 Min Webshop</p>
    </footer>

    <!-- JavaScript placeres typisk sidst i body -->
    <script src="app.js"></script>
</body>
</html>
```

---

## Vigtige HTML-tags

### Tekstindhold

```html
<h1>Overskrift niveau 1</h1>
<h2>Overskrift niveau 2</h2>
<h3>Overskrift niveau 3</h3>
<p>Afsnit med tekst. Kan indeholde <strong>fed tekst</strong> og <em>kursiv</em>.</p>
<br>  <!-- Linjeskift -->
<hr>  <!-- Vandret linje -->

<!-- Uordnet liste (punkter) -->
<ul>
    <li>Punkt 1</li>
    <li>Punkt 2</li>
</ul>

<!-- Ordnet liste (nummereret) -->
<ol>
    <li>Første trin</li>
    <li>Andet trin</li>
</ol>
```

### Links og billeder

```html
<!-- Eksternt link -->
<a href="https://www.google.com" target="_blank">Åbn Google</a>

<!-- Internt link -->
<a href="/kontakt">Kontakt os</a>

<!-- Billede -->
<img src="billede.jpg" alt="Beskrivelse af billedet" width="300">
```

### Formularer

```html
<form action="/submit" method="POST">
    <!-- Tekstfelt -->
    <label for="navn">Navn:</label>
    <input type="text" id="navn" name="navn" placeholder="Dit navn" required>

    <!-- Email -->
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>

    <!-- Tal -->
    <label for="antal">Antal:</label>
    <input type="number" id="antal" name="antal" min="1" max="100" value="1">

    <!-- Dropdown -->
    <label for="kategori">Kategori:</label>
    <select id="kategori" name="kategori">
        <option value="">-- Vælg kategori --</option>
        <option value="elektronik">Elektronik</option>
        <option value="toej">Tøj</option>
    </select>

    <!-- Textarea -->
    <label for="besked">Besked:</label>
    <textarea id="besked" name="besked" rows="4" cols="50"></textarea>

    <!-- Checkboks -->
    <input type="checkbox" id="accept" name="accept">
    <label for="accept">Jeg accepterer betingelserne</label>

    <!-- Submit-knap -->
    <button type="submit">Send</button>
</form>
```

### Semantiske tags (HTML5)

```html
<header>   <!-- Sidens eller sektionens hoved -->
<nav>      <!-- Navigationsmenu -->
<main>     <!-- Sidens primære indhold -->
<article>  <!-- Selvstændigt indhold (blogpost, nyhedsartikel) -->
<section>  <!-- Tematisk gruppering -->
<aside>    <!-- Sidebjælke / relateret indhold -->
<footer>   <!-- Sidens eller sektionens fod -->
<figure>   <!-- Billede med billedtekst -->
<figcaption> <!-- Billedtekst -->
```

---

## DOM — Document Object Model

**DOM** er en træstruktur, der repræsenterer HTML-dokumentet i hukommelsen. JavaScript kan manipulere DOM'en for at ændre indholdet dynamisk.

```
document
└── html
    ├── head
    │   ├── title
    │   └── meta
    └── body
        ├── header
        │   └── nav
        ├── main
        │   ├── h1
        │   └── p
        └── footer
```

DOM bruges primært med JavaScript — mere om det i uge 10.

---

## CSS — Cascading Style Sheets

CSS styler HTML-elementerne: farver, skrifttyper, størrelse, placering osv.

### De tre måder at inkludere CSS

**1. Inline CSS** (direkte på elementet — undgå generelt)

```html
<p style="color: red; font-size: 18px;">Rød tekst</p>
```

**2. Intern CSS** (i `<style>`-tag i `<head>`)

```html
<head>
    <style>
        p {
            color: blue;
            font-size: 16px;
        }
    </style>
</head>
```

**3. Ekstern CSS** (anbefalet — separat `.css`-fil)

```html
<!-- I HTML: -->
<link rel="stylesheet" href="style.css">
```

```css
/* I style.css: */
p {
    color: blue;
    font-size: 16px;
}
```

---

## CSS Selektorer

```css
/* Element-selektor — vælger alle <p>-elementer */
p {
    color: darkblue;
}

/* Klasse-selektor — vælger elementer med class="highlight" */
.highlight {
    background-color: yellow;
}

/* ID-selektor — vælger elementet med id="header" */
#header {
    background-color: #333;
    color: white;
}

/* Kombineret selektor */
.card p {          /* <p> inde i et element med class="card" */
    font-size: 14px;
}

/* Pseudo-klasse */
a:hover {          /* Link når musen er over */
    color: orange;
    text-decoration: underline;
}

button:disabled {  /* Deaktiveret knap */
    opacity: 0.5;
}
```

---

## CSS Box Model

Hvert HTML-element er en boks med fire lag:

```
┌─────────────────────────────────┐
│           MARGIN                │  ← Afstand til andre elementer
│  ┌───────────────────────────┐  │
│  │         BORDER            │  │  ← Kant
│  │  ┌─────────────────────┐  │  │
│  │  │       PADDING       │  │  │  ← Indre afstand til kanten
│  │  │  ┌───────────────┐  │  │  │
│  │  │  │    CONTENT    │  │  │  │  ← Selve indholdet
│  │  │  └───────────────┘  │  │  │
│  │  └─────────────────────┘  │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

```css
.box {
    /* Indhold */
    width: 300px;
    height: 200px;

    /* Padding — indre afstand */
    padding: 20px;             /* Alle sider */
    padding: 10px 20px;        /* Top/bund  venstre/højre */
    padding-top: 10px;

    /* Border */
    border: 1px solid #ccc;
    border-radius: 8px;        /* Afrundede hjørner */

    /* Margin — ydre afstand */
    margin: 10px;
    margin: 0 auto;            /* Centrer horisontalt */
}
```

---

## Bootstrap 5

**Bootstrap 5** er et CSS-framework der giver færdige, responsive komponenter. Det sparer enorm tid i frontend-udvikling.

### Inkludér Bootstrap via CDN

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bootstrap Eksempel</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

    <!-- Bootstrap JavaScript (til interaktive komponenter) -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## Bootstrap Grid System

Bootstrap bruger et 12-kolonne grid-system. Du opdeler en row i op til 12 kolonner.

```html
<div class="container">
    <!-- En row med tre lige brede kolonner (4+4+4 = 12) -->
    <div class="row">
        <div class="col-4">Kolonne 1</div>
        <div class="col-4">Kolonne 2</div>
        <div class="col-4">Kolonne 3</div>
    </div>

    <!-- Responsivt layout: 12 kolonner på mobil, 6 på tablet, 4 på desktop -->
    <div class="row">
        <div class="col-12 col-md-6 col-lg-4">Produkt 1</div>
        <div class="col-12 col-md-6 col-lg-4">Produkt 2</div>
        <div class="col-12 col-md-6 col-lg-4">Produkt 3</div>
    </div>
</div>
```

### Breakpoints

| Præfiks | Skærmstørrelse | Typisk enhed     |
|---------|----------------|------------------|
| (ingen) | < 576px        | Mobil (portrait) |
| `sm`    | >= 576px       | Mobil (landscape)|
| `md`    | >= 768px       | Tablet           |
| `lg`    | >= 992px       | Laptop           |
| `xl`    | >= 1200px      | Desktop          |
| `xxl`   | >= 1400px      | Stor desktop     |

---

## Bootstrap Cards

Cards er en af de mest brugte Bootstrap-komponenter — perfekt til at vise produkter, artikler osv.

```html
<div class="container mt-4">
    <div class="row row-cols-1 row-cols-md-3 g-4">

        <!-- Produkt-card -->
        <div class="col">
            <div class="card h-100 shadow-sm">
                <img src="laptop.jpg" class="card-img-top" alt="Laptop">
                <div class="card-body">
                    <h5 class="card-title">Apple MacBook Air</h5>
                    <p class="card-text">
                        Ultratyndt og lettere end nogensinde.
                        Med M2-chip og hele 18 timers batteritid.
                    </p>
                    <p class="card-text">
                        <strong class="text-success fs-5">14.999 kr.</strong>
                    </p>
                </div>
                <div class="card-footer">
                    <button class="btn btn-primary w-100">Læg i kurv</button>
                </div>
            </div>
        </div>

    </div>
</div>
```

---

## Bootstrap Utility Classes

Bootstrap har hundredvis af utility-klasser til hurtigt styling:

```html
<!-- Tekst -->
<p class="text-center">Centeret tekst</p>
<p class="text-primary">Blå tekst</p>
<p class="fw-bold">Fed tekst</p>
<p class="fs-4">Stor tekst</p>

<!-- Farver og baggrunde -->
<div class="bg-primary text-white p-3">Blå boks med hvid tekst</div>
<div class="bg-light border rounded p-3">Lys grå boks</div>

<!-- Spacing (m = margin, p = padding, t/b/s/e/x/y = top/bottom/start/end/x-axis/y-axis) -->
<div class="mt-3 mb-2 px-4 py-3">Spacing på alle sider</div>

<!-- Display -->
<div class="d-flex justify-content-between align-items-center">
    <span>Venstre</span>
    <span>Højre</span>
</div>

<!-- Knapper -->
<button class="btn btn-primary">Primær</button>
<button class="btn btn-secondary">Sekundær</button>
<button class="btn btn-danger">Slet</button>
<button class="btn btn-outline-primary">Outline</button>
<button class="btn btn-sm btn-success">Lille grøn</button>
```

---

## Bootstrap Navbar

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <!-- Logo/Brand -->
        <a class="navbar-brand" href="/">Min Webshop</a>

        <!-- Hamburger-menu til mobil -->
        <button class="navbar-toggler" type="button"
                data-bs-toggle="collapse"
                data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>

        <!-- Nav-links -->
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav ms-auto">
                <li class="nav-item">
                    <a class="nav-link active" href="/">Hjem</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="/produkter">Produkter</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="/kontakt">Kontakt</a>
                </li>
            </ul>
        </div>
    </div>
</nav>
```

---

## Azure Deployment — Statisk webside

Azure Static Web Apps giver gratis hosting til statiske hjemmesider.

### Trin-for-trin

1. Opret konto på portal.azure.com
2. Søg efter "Static Web Apps" og klik Create
3. Vælg subscription og resource group
4. Giv appen et navn
5. Vælg "Free" plan
6. Forbind til GitHub repository
7. Angiv build-konfiguration (typisk `/ ` som app location)
8. Klik "Review + create"

Azure opretter automatisk en GitHub Actions workflow der deployer ved hvert push til main.

---

## Komplet HTML/Bootstrap eksempel — Produktside

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Produkter — Min Webshop</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        .card:hover { transform: translateY(-4px); transition: transform 0.2s; }
        .price { color: #28a745; font-size: 1.3rem; font-weight: bold; }
    </style>
</head>
<body class="bg-light">

    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-4">
        <div class="container">
            <a class="navbar-brand fw-bold" href="/">WebShop</a>
        </div>
    </nav>

    <main class="container">
        <h1 class="mb-4">Vores Produkter</h1>

        <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4" id="produkter">

            <div class="col">
                <div class="card h-100 shadow-sm">
                    <div class="card-body">
                        <h5 class="card-title">Apple MacBook Air</h5>
                        <p class="card-text text-muted">M2-chip, 8GB RAM, 256GB SSD</p>
                        <p class="price">14.999 kr.</p>
                    </div>
                    <div class="card-footer bg-white border-0">
                        <button class="btn btn-primary w-100">Læg i kurv</button>
                    </div>
                </div>
            </div>

            <div class="col">
                <div class="card h-100 shadow-sm">
                    <div class="card-body">
                        <h5 class="card-title">Logitech MX Master 3</h5>
                        <p class="card-text text-muted">Ergonomisk mus til professionelle</p>
                        <p class="price">899 kr.</p>
                    </div>
                    <div class="card-footer bg-white border-0">
                        <button class="btn btn-primary w-100">Læg i kurv</button>
                    </div>
                </div>
            </div>

        </div>
    </main>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## Opsummering

| Begreb           | Beskrivelse                                                      |
|------------------|------------------------------------------------------------------|
| HTML             | Markup-sprog der beskriver sidens struktur og indhold            |
| Semantiske tags  | Tags med mening (header, nav, main, article osv.)                |
| DOM              | Træstruktur af HTML-dokumentet i hukommelsen                     |
| CSS              | Stylesheet-sprog der styler HTML-elementerne                     |
| Box model        | Content, padding, border, margin                                 |
| Bootstrap 5      | CSS-framework med grid, komponenter og utility-klasser           |
| Grid system      | 12-kolonne system til responsive layouts                         |
| Cards            | Bootstrap-komponent til at vise indhold i bokse                  |
| Azure Static Web | Gratis hosting af statiske websider                              |

---

## Opgaver til selvstudium

1. Byg en simpel HTML-side med en navbar, en header og mindst 4 produktkort
2. Brug Bootstrap grid til at gøre siden responsiv
3. Tilføj et kontaktformular med Bootstrap-styling
4. Eksperimentér med Bootstrap utility-klasser (farver, spacing, tekst)
5. Deploy siden til Azure Static Web Apps eller GitHub Pages
