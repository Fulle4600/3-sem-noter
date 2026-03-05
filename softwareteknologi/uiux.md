# UI/UX Begreber

## Indholdsfortegnelse

- [Jakob Nielsens 10 Usability Heuristikker](#jakob-nielsens-10-usability-heuristikker)
- [Ben Shneidermans 8 Golden Rules](#ben-shneidermans-8-golden-rules)
- [CRAP Principper (David Travis)](#crap-principper-david-travis)
- [Persona](#persona)
- [Think Aloud Testing (Tænk-højt test)](#think-aloud-testing-tænk-højt-test)

---

## Jakob Nielsens 10 Usability Heuristikker

10 tommelfingerregler til ekspert-evaluering af brugergrænsefladens brugervenlighed.
Find UX-problemer **hurtigt** uden at skulle bruge rigtige testpersoner (billigere end brugertest).

### Heuristikkerne

**1. Visibility of System Status** — Hold brugeren informeret
Systemet skal altid vise hvad det laver via passende feedback inden for rimelig tid.
Brugeren skal forstå systemets tilstand for at tage næste skridt korrekt.
> Eksempel: Loading spinner, upload-fremskridt bar (45%), bekræftelsesbesked efter gem, aktivt menu-item markeret.

**2. Match Between System and Real World** — Tal brugerens sprog
Brug ord og koncepter kendte for brugeren — ikke intern teknisk jargon.
Brugeren lærer intuitivt når der er naturlig sammenhæng med den virkelige verden.
> Eksempel: Folderikon ligner en rigtig mappe. "Papirkurv" frem for "delete buffer". Knapper der ligner fysiske knapper.

**3. User Control and Freedom** — Nødudgang til fejl
Brugere laver fejl — giv dem en tydelig exit UDEN lang process.
Reducerer angst og opfordrer til at udforske systemet trygt.
> Eksempel: UNDO (Ctrl+Z), ANNULLER-knap, "Fortryd sletning" besked i 5 sekunder, tydelig TILBAGE-knap.

**4. Consistency and Standards** — Vær konsistent
Brugere skal ikke spekulere på om forskellige ord/situationer/handlinger betyder det samme.
Konsistens reducerer kognitiv belastning — man behøver ikke lære nyt på hvert skærmbillede.
> Eksempel: Ctrl+C = kopier OVERALT. "Gem" altid øverst til venstre. Rød-farve = fare/sletning overalt.

**5. Error Prevention** — Forebyg fejl
Godt design forhindrer problemer FØR de opstår — bedre end gode fejlbeskeder bagefter.
En fejl der aldrig sker er bedre end en fejl der håndteres godt.
> Eksempel: Grå/deaktiverede knapper, "Er du sikker?" bekræftelse ved irreversible handlinger, datovælger frem for fri tekst.

**6. Recognition Rather than Recall** — Vis frem for at kræve hukommelse
Minimer hukommelsesbyrden ved at gøre muligheder synlige.
Genkendelse er langt lettere end at huske fra bunden — reducerer fejl.
> Eksempel: Dropdown frem for blank tekstboks. "Nylig brugte dokumenter" i Word. Autocomplete i søgefelt.

**7. Flexibility and Efficiency of Use** — Tilpas til alle brugere
Genveje for erfarne brugere — nybegyndere bruger lang vej, eksperter bruger shortcuts.
Systemet skal servicere BÅDE nybegyndere og eksperter effektivt.
> Eksempel: Keyboard shortcuts (Ctrl+Z, Ctrl+S), højreklik-menu, brugertilpasning, makroer.

**8. Aesthetic and Minimalist Design** — Hold det simpelt
Fjern irrelevant information der konkurrerer med essentielt indhold.
Al ekstra information svækker vigtig informations synlighed og øger kognitiv belastning.
> Eksempel: Google-forsiden = kun søgefelt og logo. Undgå visual clutter og unødvendige knapper.

**9. Help Users Recognize, Diagnose, and Recover from Errors**
Fejlbeskeder på almindeligt sprog (IKKE fejlkoder) med præcis problemidentifikation og konstruktiv løsning.
Brugeren skal forstå hvad gik galt og hvad de skal gøre — ikke være forvirret.
> **GODT:** "Password skal være mindst 8 tegn. Dit har 5."
> **DÅRLIGT:** "Error 403: Invalid credentials."

**10. Help and Documentation** — Tilgængelig hjælp
Søgbar, opgavefokuseret dokumentation med konkrete trin til handling.
Selv det bedste system har brugere der har brug for hjælp — det skal være nemt at finde.
> Eksempel: Kontekst-sensitiv hjælp (? ikon nær feltet), søgbar FAQ, tooltips, onboarding flow.

---

## Ben Shneidermans 8 Golden Rules

8 principper for godt interface-design fra bogen *"Designing the User Interface"*.
Retningslinjer for at skabe interfaces der er effektive og tilfredsstillende at bruge.

| # | Regel | Beskrivelse |
|---|---|---|
| 1 | **Strive for Consistency** | Samme handlingssekvenser i lignende situationer. Identisk terminologi overalt. |
| 2 | **Seek Universal Usability** | Design for ALLE — nybegyndere, eksperter, unge, ældre, handicappede, internationale brugere. |
| 3 | **Offer Informative Feedback** | Feedback på HVER handling. Svarstørrelse matcher handlingens størrelse. |
| 4 | **Design Dialogs to Yield Closure** | Klart start, midte og SLUT. Brugeren ved HVORNÅR de er færdige. Eksempel: checkout-flow med tydelige trin. |
| 5 | **Prevent Errors** | Begræns interface så alvorlige fejl er umulige. Giv konstruktiv vejledning nær fejl. |
| 6 | **Permit Easy Reversal of Actions** | UNDO reducerer angst — brugere opfordres til at udforske systemet. |
| 7 | **Keep Users in Control** | Erfarne brugere føler sig som initiativtagere. Systemet svarer forudsigeligt. Ingen overraskelser. |
| 8 | **Reduce Short-Term Memory Load** | Kræv ikke at brugeren husker info fra ét skærmbillede til et andet. Vis det de behøver. |

---

## CRAP Principper (David Travis)

Visuelle designprincipper for layout:

| Bogstav | Princip | Beskrivelse |
|---|---|---|
| **C** | Contrast | Elementer der IKKE er ens skal se TYDELIGT forskellige ud (farve, størrelse, form). |
| **R** | Repetition | Gentag visuelle elementer for konsistens (samme knap-stil overalt). |
| **A** | Alignment | Elementer aligner til noget på siden — giver ro og orden. |
| **P** | Proximity | Relaterede elementer grupperes NÆR hinanden. Urelated elementer adskilles. |

---

## Persona

En fiktiv, men realistisk repræsentant for en brugergruppe baseret på rigtig forskning.
Hjælper teamet at designe for **rigtige** brugere med rigtige behov — ikke "den gennemsnitlige bruger".

**Indeholder:** navn, alder, job, mål, frustrationer, teknologiniveau, citater fra interviews.

> Eksempel: "Mette, 52, regnskabschef. Bruger Excel dagligt. Frygter ny software. Vil have ting til at virke fra dag 1."

---

## Think Aloud Testing (Tænk-højt test)

Testpersoner siger **højt** hvad de tænker når de bruger systemet.
Afslører brugerens mentale model og forvirringspunkter — du ser HVAD de gør OG HVORFOR.

> Regel #1 i UX Research ifølge Nielsen Norman Group. Simpelt, billigt og magtfuldt.

---

## Opgaver til selvstudium

1. Vælg en app eller hjemmeside du bruger dagligt og identificer 3 brud på Nielsens 10 usability heuristikker
2. Hvad er forskellen på en wireframe og en prototype? Hvornår bruger man lavfidelity vs. højfidelity?
3. Forklar "Thinking Aloud" som testmetode — hvad afslører det, og hvad er den typiske fejl testlederen gør?
4. Hvad er forskellen på UX og UI? Giv et eksempel på god UI men dårlig UX
5. Hvad er WCAG, og hvorfor er tilgængelighed vigtigt i softwareudvikling?
6. Design en brugertestplan for dit semesterprojekt: hvad vil du teste, hvem er dine testpersoner, og hvilke opgaver vil du bede dem løse?
