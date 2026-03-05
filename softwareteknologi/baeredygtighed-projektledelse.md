# Bæredygtighed & Projektledelse Begreber

## Indholdsfortegnelse

- [Karlskrona Manifest for Sustainability Design (2015)](#karlskrona-manifest-for-sustainability-design-2015)
- [Prototyping](#prototyping)
- [Design Sprint (Jake Knapp / Google Ventures)](#design-sprint-jake-knapp--google-ventures)
- [Boehm & Turner Matrix (Metodevalg)](#boehm--turner-matrix-metodevalg)
- [Projektperioden — Cycles og Overdragelser](#projektperioden--cycles-og-overdragelser)
- [Git Branches og Workflow](#git-branches-og-workflow)

---

## Karlskrona Manifest for Sustainability Design (2015)

Manifest der erklærer at software-udviklere har ansvar for bæredygtighed — ikke kun environmentale aspekter.
Software påvirker samfund, miljø, individer og økonomi. **Vi har et ansvar.**

### 5 Dimensioner af bæredygtighed i IT

| Dimension | Beskrivelse |
|---|---|
| **Miljø** | Energiforbrug, CO₂-udledning, e-affald, ressourceforbrug |
| **Social** | Påvirkning på fællesskaber, lighed, arbejdsvilkår, tilgængelighed |
| **Økonomisk** | Langtidsøkonomisk bæredygtighed, fair fordeling af ressourcer |
| **Individuel** | Sundhed, læring, privatliv, velvære for enkeltpersoner |
| **Teknisk** | Systemets evne til at være vedligeholdt, ændret og udvidet over tid |

### 9 Principper fra Karlskrona Manifestet

1. **Systemisk natur** — Bæredygtighed er et sammenkædet system — kan ikke løses isoleret.
2. **Multi-dimensionel** — 5 dimensioner — alle er vigtige, ikke kun miljø.
3. **Tværfaglig** — Kræver samarbejde på tværs af fagområder.
4. **Universel applicabilitet** — Gælder ALLE systemer uanset primært formål.
5. **Dual sfærer** — Design skal adressere både systemet SELV og dets bredere konsekvenser.
6. **Langsigtet perspektiv** — Vurder konsekvenser over lange tidshorisonter.
7. **Nutid-fremtid balance** — Innovation kan adskille nutidige behov fra fremtidige.
8. **Systemsynlighed** — Gennemsigtighed på alle niveauer.
9. **Opfordring til handling** — Integrer principper i dagligt arbejde, forskning og uddannelse.

---

## Prototyping

Hurtig opbygning af en simplificeret version af produktet til at afprøve ideer.
Test antagelser **billigt og tidligt** inden man bruger måneder på at kode noget forkert.

| Type | Beskrivelse |
|---|---|
| **Low-fidelity** | Papirprototype, skitser — hurtig og billig. Tester KONCEPTET. |
| **High-fidelity** | Interaktiv prototype i Figma/Adobe XD. Tester INTERAKTIONEN og UI. |
| **Throwaway** | Smid prototypen væk — kun til at lære. |
| **Evolutionary** | Byg videre på prototypen til det færdige produkt. |

---

## Design Sprint (Jake Knapp / Google Ventures)

5-dages intensiv proces til at besvare kritiske forretningsmæssige spørgsmål via design og test.
Spar måneder af arbejde ved at teste ideer på kun 5 dage.

| Dag | Aktivitet |
|---|---|
| **Dag 1 — Map** | Forstå problemet, vælg et mål |
| **Dag 2 — Sketch** | Individuelle skitser til løsninger |
| **Dag 3 — Decide** | Vælg den bedste løsning |
| **Dag 4 — Prototype** | Byg en realistisk prototype på 1 dag |
| **Dag 5 — Test** | Test på 5 rigtige brugere — lær enormt meget |

---

## Boehm & Turner Matrix (Metodevalg)

Ramme til at vælge hvilken udviklingsmetode (agil vs planbaseret) der passer bedst til et projekt.
Ingen metode er perfekt til alle projekter — vælg bevidst.

**Faktorer:** Stabilitet af krav, team-størrelse, risiko ved fejl, kundens involvering.

| | Agil | Planbaseret |
|---|---|---|
| **Krav** | Ustabile | Stabile |
| **Team** | Lille | Stort, distribueret |
| **Kundekontakt** | Tæt, dagligt | Begrænset |
| **Kritikalitet** | Lav | Høj (sikkerhed, medicin) |

---

## Projektperioden — Cycles og Overdragelser

| Begreb | Beskrivelse |
|---|---|
| **Cycle** | En iteration (sprint) i projektperioden. Typisk 1-2 ugers varighed. |
| **Overdragelse** | Demonstration og levering af fungerende software til kunde/vejleder ved cycle-slut. |
| **Cycle 0** | Inception — opsæt workspace, repository, planlæg Cycle 1. Ingen kode. |
| **Spike** | Tidsbegrænset undersøgelse for at reducere usikkerhed (teknisk proof-of-concept). |
| **Code Freeze** | Tidspunkt hvor ingen ny kode tilføjes. Kun bugfixes. Forberedelse til demo. |

---

## Git Branches og Workflow

Git branches lader flere udviklere arbejde parallelt uden at forstyrre hinanden.
Isoler ny funktionalitet, eksperimenter sikkert, let at rulle tilbage.

| Branch | Beskrivelse |
|---|---|
| **main/master** | Produktionsklar kode. Merge kun via Pull Request efter review + grøn CI. |
| **feature/xxx** | Branch for ny funktion. Navngivning: `feature/bruger-login`. |

**Pull Request (PR):** Formel anmodning om at merge din branch til main. Muliggør code review.

**Merge conflict:** Opstår når to branches har ændret den SAMME linje. Skal resolves manuelt.

---

## Opgaver til selvstudium

1. Beskriv de 5 dimensioner i Karlskrona Manifestet og giv et konkret IT-eksempel for hver
2. Hvad er "teknisk bæredygtighed"? Hvilke XP-praksisser bidrager til det, og hvordan?
3. Forklar "Dual Sfærer" — hvad er den direkte og den indirekte påvirkning af en GPS-app?
4. Hvad er teknisk gæld? Beskriv et scenario hvor teknisk gæld opstår og konsekvensen over tid
5. Hvornår ville du vælge Feature Branch Workflow frem for Trunk-Based Development?
6. Hvad er en god Pull Request-beskrivelse? Skriv en skabelon til en PR der tilføjer login til en webapp
