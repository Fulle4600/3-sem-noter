# Uge 10 — JavaScript & Vue.js

Denne uge introducerer JavaScript som programmeringssprog i browseren og Vue.js som det reaktive frontend-framework vi bruger i resten af semestret. JavaScript giver os mulighed for at lave dynamiske websider der reagerer på brugerhandlinger, og Vue.js gør det struktureret og overskueligt. Det er en stor uge med mange nye begreber — tag det et trin ad gangen.

---

## JavaScript Grundlæggende

JavaScript (JS) er det primære programmeringssprog i browseren. Det kører direkte i browseren uden at behøve kompilering.

### Datatyper i JavaScript

```javascript
// String — tekststrenge
let navn = "Anna";
let titel = 'Produktchef';
let besked = `Hej ${navn}, du er ${titel}`;  // Template literal

// Number — alle tal (ingen int/float skelnen)
let alder = 25;
let pris = 299.99;

// Boolean
let erLoggetInd = true;
let harVarer = false;

// Null — eksplicit "ingen værdi"
let kundeId = null;

// Undefined — variabel deklareret men ikke sat
let resultat;
console.log(resultat);  // undefined

// Array — liste af værdier
let farver = ["rød", "grøn", "blå"];
let tal = [1, 2, 3, 4, 5];
let blandet = [1, "to", true, null];

// Object — nøgle/værdi-par
let produkt = {
    id: 1,
    navn: "Laptop",
    pris: 8999,
    erPåLager: true
};
```

### `var`, `let` og `const`

```javascript
// var — undgå! Gammel syntaks, function-scoped
var x = 10;

// let — brug til variabler der ændres
let tæller = 0;
tæller = 1;  // OK

// const — brug til værdier der ikke ændres
const PI = 3.14159;
const API_URL = "https://api.example.com";

// BEMÆRK: const til objekter/arrays tillader stadig ændring af indhold
const person = { navn: "Lars" };
person.navn = "Per";  // OK — vi ændrer ikke referencen
// person = {};        // FEJL — vi må ikke tildele en ny reference
```

---

## Funktioner i JavaScript

Funktioner er genbrugelige blokke af kode der udfører en bestemt opgave. I stedet for at skrive den samme kode mange gange, pakker du den ind i en funktion og kalder den. JavaScript har flere måder at definere funktioner på — den moderne og mest udbredte er **arrow functions** (ES6).

```javascript
// Klassisk function declaration
function hilsen(navn) {
    return `Hej ${navn}!`;
}

// Function expression
const hilsen2 = function(navn) {
    return `Hej ${navn}!`;
};

// Arrow function (ES6) — mest brugt i moderne kode
const hilsen3 = (navn) => `Hej ${navn}!`;

// Arrow function med krop
const beregnMoms = (pris) => {
    const moms = pris * 0.25;
    return pris + moms;
};

// Funktion med default parameter
const opretBruger = (navn, rolle = "bruger") => {
    return { navn, rolle };  // Shorthand: { navn: navn, rolle: rolle }
};

console.log(opretBruger("Anna"));           // { navn: "Anna", rolle: "bruger" }
console.log(opretBruger("Lars", "admin"));  // { navn: "Lars", rolle: "admin" }
```

---

## Arrays og Array-metoder

Arrays er lister af værdier — og JavaScript har en række kraftfulde indbyggede metoder til at arbejde med dem. I stedet for manuel iteration med for-løkker, bruger vi funktionelle metoder som `map`, `filter`, og `reduce` der er kortere, mere læsbare og nemmere at teste. Disse metoder bruges meget i Vue.js til at transformere og filtrere data.

```javascript
const produkter = [
    { id: 1, navn: "Laptop", pris: 8999 },
    { id: 2, navn: "Mus", pris: 299 },
    { id: 3, navn: "Tastatur", pris: 499 },
    { id: 4, navn: "Skærm", pris: 3999 }
];

// forEach — iterér over hvert element
produkter.forEach(p => console.log(p.navn));

// map — transformér hvert element, returner nyt array
const navne = produkter.map(p => p.navn);
// ["Laptop", "Mus", "Tastatur", "Skærm"]

const priserMedMoms = produkter.map(p => ({
    ...p,               // Kopier alle properties
    prisMedMoms: p.pris * 1.25
}));

// filter — behold kun elementer der matcher betingelse
const dyreProdukter = produkter.filter(p => p.pris > 500);
// [{ id: 1, navn: "Laptop"... }, { id: 4, navn: "Skærm"... }]

// find — returner første match
const laptop = produkter.find(p => p.navn === "Laptop");

// some — returner true hvis MINDST ét match
const harBilligeProdukter = produkter.some(p => p.pris < 300);  // true

// every — returner true hvis ALLE matcher
const alleUnderlager = produkter.every(p => p.pris < 10000);    // true

// sort — sortér (OBS: modificerer originalt array!)
const sorteret = [...produkter].sort((a, b) => a.pris - b.pris);
// Billigste først

// reduce — aggregér til én værdi
const totalPris = produkter.reduce((sum, p) => sum + p.pris, 0);
// 13796
```

---

## DOM Manipulation

DOM (Document Object Model) er browserens repræsentation af HTML-siden som et træ af objekter. JavaScript kan tilgå og ændre DOM'en — det er præcis det der gør websider dynamiske. Du kan vælge elementer, ændre tekst og styling, og oprette/slette elementer uden at genindlæse siden. **Bemærk:** I Vue.js manipulerer vi sjældent DOM'en direkte — Vue gør det for os via direktiver som `v-if` og `v-for`.

```javascript
// Vælg elementer
const titel = document.getElementById("titel");
const knapper = document.querySelectorAll(".btn");
const første = document.querySelector(".produkt-card");

// Ændr indhold
titel.textContent = "Ny tekst";
titel.innerHTML = "<strong>Fed tekst</strong>";

// Ændr styling
første.style.backgroundColor = "lightblue";
første.classList.add("aktiv");
første.classList.remove("skjult");
første.classList.toggle("highlight");

// Opret nyt element
const nyKnap = document.createElement("button");
nyKnap.textContent = "Klik mig";
nyKnap.className = "btn btn-primary";
document.body.appendChild(nyKnap);
```

---

## Events

Events er handlinger der sker i browseren — brugerens klik på en knap, tekst i et inputfelt, submit af en formular. `addEventListener` registrerer en funktion der kaldes automatisk når et bestemt event sker. Event-objektet (`event` / `e`) indeholder information om hvad der skete, fx hvilket element der blev klikket. `e.preventDefault()` forhindrer browserens standardadfærd — fx at en formular genindlæser siden ved submit.

```javascript
const knap = document.getElementById("minKnap");

// addEventListener er den anbefalede metode
knap.addEventListener("click", function(event) {
    console.log("Knap klikket!", event);
    alert("Du klikkede!");
});

// Med arrow function
knap.addEventListener("click", (e) => {
    e.preventDefault();  // Forhindrer standard handling (f.eks. form submit)
    console.log("Klikket på:", e.target);
});

// Input event
const søgefelt = document.getElementById("søgefelt");
søgefelt.addEventListener("input", (e) => {
    console.log("Søger på:", e.target.value);
});

// Form submit
const form = document.getElementById("minForm");
form.addEventListener("submit", (e) => {
    e.preventDefault();  // VIGTIGT — forhindrer sideopdatering
    const data = new FormData(form);
    console.log(data.get("navn"));
});
```

---

## Vue.js — Introduktion

**Vue.js** er et progressivt JavaScript-framework til at bygge brugergrænseflader. Det er reaktivt: når data ændres, opdateres UI'en automatisk.

### Inkludér Vue.js via CDN (simpelt setup)

```html
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
```

### Grundlæggende Vue.js app

```html
<!DOCTYPE html>
<html lang="da">
<head>
    <meta charset="UTF-8">
    <title>Min Vue App</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div id="app" class="container mt-4">
    <h1>{{ besked }}</h1>
    <p>Tæller: {{ tæller }}</p>
    <button class="btn btn-primary" @click="forøg">Forøg</button>
    <button class="btn btn-danger ms-2" @click="nulstil">Nulstil</button>
</div>

<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
const { createApp } = Vue;

createApp({
    data() {
        return {
            besked: "Hej Vue.js!",
            tæller: 0
        };
    },
    methods: {
        forøg() {
            this.tæller++;
        },
        nulstil() {
            this.tæller = 0;
        }
    }
}).mount("#app");
</script>
</body>
</html>
```

---

## Vue.js Reaktivitet

Reaktivitet er kernen i Vue.js. Når en `data()`-property ændres, opdateres alle steder i HTML'en der bruger den automatisk.

```javascript
createApp({
    data() {
        return {
            // Alle disse er reaktive
            brugernavn: "",
            erLoggetInd: false,
            produkter: [],
            valgtProdukt: null
        };
    }
}).mount("#app");
```

---

## v-for — Generer lister

```html
<div id="app">
    <ul>
        <!-- v-for itererer over et array -->
        <li v-for="produkt in produkter" :key="produkt.id">
            {{ produkt.navn }} — {{ produkt.pris }} kr.
        </li>
    </ul>

    <!-- Med Bootstrap cards -->
    <div class="row">
        <div class="col-md-4" v-for="p in produkter" :key="p.id">
            <div class="card mb-4">
                <div class="card-body">
                    <h5 class="card-title">{{ p.navn }}</h5>
                    <p class="card-text text-success fw-bold">{{ p.pris }} kr.</p>
                    <button class="btn btn-primary" @click="vælgProdukt(p)">
                        Se detaljer
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
createApp({
    data() {
        return {
            produkter: [
                { id: 1, navn: "Laptop", pris: 8999 },
                { id: 2, navn: "Mus", pris: 299 },
                { id: 3, navn: "Tastatur", pris: 499 }
            ]
        };
    }
}).mount("#app");
</script>
```

**Vigtigt:** `:key` er obligatorisk i `v-for` — det hjælper Vue med effektivt at opdatere listen. Brug altid en unik værdi som id.

---

## v-on (@ ) — Event handling

`v-on` (forkortet `@`) bruges til at lytte på browser-events direkte i Vue-templates. I stedet for `addEventListener` i JavaScript-koden, skriver du blot `@click="minMetode"` på elementet i HTML'en. Vue kalder automatisk din metode når eventet sker. Event-modifikatorer som `.prevent` og `.stop` er Vue's genvej til `e.preventDefault()` og `e.stopPropagation()`.

```html
<div id="app">
    <!-- v-on:click er det samme som @click -->
    <button v-on:click="klik">Klik</button>
    <button @click="klik">Klik (shorthand)</button>

    <!-- Inline handler -->
    <button @click="tæller++">Forøg</button>
    <button @click="tæller = 0">Nulstil</button>

    <!-- Med argument -->
    <button @click="slet(produkt.id)">Slet</button>

    <!-- Event-modifikatorer -->
    <form @submit.prevent="send">  <!-- Forhindrer standard form submit -->
        <input @keyup.enter="søg">  <!-- Kun ved Enter-tast -->
    </form>

    <!-- Mouseover / blur / focus -->
    <input @blur="valider" @focus="rydFejl">
</div>
```

---

## v-model — Two-way data binding

`v-model` binder en formular-input til en data-property i BEGGE retninger:
- Data → Input: inputfeltet viser data-værdien
- Input → Data: når brugeren skriver, opdateres data automatisk

```html
<div id="app">
    <!-- Tekstfelt -->
    <input v-model="brugernavn" placeholder="Dit navn">
    <p>Du har skrevet: {{ brugernavn }}</p>

    <!-- Checkbox -->
    <input type="checkbox" v-model="accepterer">
    <span>{{ accepterer ? 'Accepteret' : 'Ikke accepteret' }}</span>

    <!-- Select dropdown -->
    <select v-model="valgtKategori">
        <option value="">-- Vælg --</option>
        <option value="elektronik">Elektronik</option>
        <option value="tøj">Tøj</option>
    </select>

    <!-- Søgefelt der filtrerer listen i realtid -->
    <input v-model="søgeTerm" placeholder="Søg produkter...">

    <ul>
        <li v-for="p in filtreredeProdukter" :key="p.id">
            {{ p.navn }}
        </li>
    </ul>
</div>

<script>
createApp({
    data() {
        return {
            brugernavn: "",
            accepterer: false,
            valgtKategori: "",
            søgeTerm: "",
            produkter: [
                { id: 1, navn: "Apple MacBook" },
                { id: 2, navn: "Dell Laptop" },
                { id: 3, navn: "Logitech Mus" }
            ]
        };
    },
    computed: {
        // Beregnet property — opdateres automatisk når søgeTerm ændres
        filtreredeProdukter() {
            return this.produkter.filter(p =>
                p.navn.toLowerCase().includes(this.søgeTerm.toLowerCase())
            );
        }
    }
}).mount("#app");
</script>
```

---

## v-bind (: ) — Bind attributter dynamisk

`v-bind` (forkortet `:`) bruges til at gøre HTML-attributter dynamiske — altså at sætte dem til en værdi fra Vue's data i stedet for en fast tekst. Uden `v-bind` er alle HTML-attributter statiske strenge. Med `v-bind` kan du fx lade et billedes `src` komme fra data, lave knapper der er `disabled` baseret på en betingelse, eller skifte CSS-klasser dynamisk baseret på applikationens tilstand.

```html
<div id="app">
    <!-- v-bind:href er det samme som :href -->
    <a v-bind:href="linkUrl">Klik her</a>
    <a :href="linkUrl">Klik her (shorthand)</a>

    <!-- Dynamisk class-binding -->
    <div :class="{ 'alert-danger': harFejl, 'alert-success': !harFejl }">
        Status besked
    </div>

    <!-- Dynamisk style -->
    <div :style="{ backgroundColor: farve, fontSize: størrelse + 'px' }">
        Dynamisk stil
    </div>

    <!-- Knap der er disabled baseret på data -->
    <button :disabled="kurv.length === 0" class="btn btn-primary">
        Gå til kassen ({{ kurv.length }} varer)
    </button>

    <!-- Billede med dynamisk src -->
    <img :src="produkt.billedUrl" :alt="produkt.navn">
</div>
```

---

## Computed Properties

Computed properties beregnes fra andre data-properties og caches. De genberegnes kun, når deres afhængigheder ændres.

```javascript
createApp({
    data() {
        return {
            kurv: [
                { id: 1, navn: "Laptop", pris: 8999, antal: 1 },
                { id: 2, navn: "Mus", pris: 299, antal: 2 }
            ]
        };
    },
    computed: {
        // Totalbeløb uden moms
        subtotal() {
            return this.kurv.reduce((sum, item) => sum + item.pris * item.antal, 0);
        },
        // Med moms
        totalMedMoms() {
            return this.subtotal * 1.25;
        },
        // Antal varer i alt
        antalVarer() {
            return this.kurv.reduce((sum, item) => sum + item.antal, 0);
        }
    }
}).mount("#app");
```

```html
<div id="app">
    <p>Subtotal: {{ subtotal.toFixed(2) }} kr.</p>
    <p>Moms (25%): {{ (subtotal * 0.25).toFixed(2) }} kr.</p>
    <p><strong>I alt: {{ totalMedMoms.toFixed(2) }} kr.</strong></p>
    <p>Antal varer: {{ antalVarer }}</p>
</div>
```

---

## v-if og v-show — Betinget rendering

```html
<div id="app">
    <!-- v-if fjerner elementet fra DOM -->
    <div v-if="erLoggetInd">
        <p>Velkommen, {{ brugernavn }}!</p>
        <button @click="logUd">Log ud</button>
    </div>
    <div v-else>
        <p>Du er ikke logget ind.</p>
        <button @click="logInd">Log ind</button>
    </div>

    <!-- v-else-if -->
    <div v-if="status === 'loading'">Indlæser...</div>
    <div v-else-if="status === 'error'">Der opstod en fejl!</div>
    <div v-else>Data hentet!</div>

    <!-- v-show skjuler med CSS (display:none) — elementet er stadig i DOM -->
    <!-- Brug v-show til elementer der skiftes hurtigt -->
    <div v-show="visBesked">Denne besked kan skjules og vises hurtigt</div>
</div>
```

---

## Opsummering

| Direktiv / Koncept | Beskrivelse                                                  |
|--------------------|--------------------------------------------------------------|
| `data()`           | Reaktive data-properties i Vue-applikationen                 |
| `methods`          | Funktioner der kan kaldes fra template                       |
| `computed`         | Beregnede værdier der caches — genberegnes kun ved ændringer |
| `v-for`            | Iterér over array og generer HTML-elementer                  |
| `v-on` / `@`       | Lyt på browser-events (click, submit, keyup osv.)            |
| `v-model`          | Two-way data binding til form-inputs                         |
| `v-bind` / `:`     | Bind HTML-attributter til data-properties                    |
| `v-if` / `v-else`  | Betinget rendering — fjerner/tilføjer fra DOM                |
| `v-show`           | Betinget synlighed — skjuler med CSS                         |
| `:key`             | Unik nøgle i `v-for` — hjælper Vue med at opdatere effektivt |

---

## Opgaver til selvstudium

1. Byg en simpel shoppingliste-app med Vue.js: tilføj/fjern varer, vis antal
2. Tilføj et søgefelt med `v-model` der filtrerer listen i realtid (brug computed)
3. Brug `v-if` til at vise en "Kurven er tom"-besked når listen er tom
4. Tilføj Bootstrap-styling til din app
5. Eksperimentér med `v-bind:class` til at fremhæve valgte varer
