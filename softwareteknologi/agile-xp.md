# Agile & XP Begreber

## Indholdsfortegnelse

- [Det Agile Manifest (2001)](#det-agile-manifest-2001)
- [XP — Extreme Programming](#xp--extreme-programming)
- [User Stories](#user-stories)
- [Definition of Done (DoD)](#definition-of-done-dod)
- [Burn Down Chart](#burn-down-chart)
- [Scope Creep](#scope-creep)
- [Vandfald vs Agile](#vandfald-vs-agile)

---

## Det Agile Manifest (2001)

Dokument af 17 software-pionerer der definerer kerneprincipper for moderne softwareudvikling.
Reaktion mod tunge Vandfald-processer. **Agilitet = evne til at reagere hurtigt på ændringer.**

### 4 Kerneprincipper — vi prioriterer VENSTRE over højre:

| Prioriteret | Over |
|---|---|
| Individer og interaktion | Processer og værktøjer |
| Fungerende software | Udtømmende dokumentation |
| Samarbejde med kunden | Kontraktforhandling |
| Reaktion på forandring | At følge en plan |

> Det til højre er stadig vigtigt — men venstre prioriteres højere!

### De 12 Agile Principper

1. Kundetilfredshed via tidlig og løbende levering af værdifuld software
2. Velkom ændringer — selv sent i projektet. Agilitet udnytter forandring til kundens fordel
3. Lever fungerende software hyppigt (fra uger til måneder — jo kortere jo bedre)
4. Forretning og udviklere arbejder **dagligt** sammen hele vejen
5. Byg projekter op om motiverede individer — giv dem miljø og tillid
6. Face-to-face kommunikation er mest effektiv (også i et team)
7. Fungerende **software** er det primære mål for fremgang (ikke dokumenter!)
8. Bæredygtig udvikling — konstant tempo der kan opretholdes uendeligt
9. Kontinuerlig opmærksomhed på teknisk excellence øger agilitet
10. Enkelhed — kunsten at maksimere mængden af arbejde der **ikke** gøres — er afgørende
11. De bedste arkitekturer opstår fra selvorganiserende teams
12. Med jævne mellemrum reflekterer teamet og justerer adfærd (**Retrospective**)

---

## XP — Extreme Programming

Agil udviklingsmetode med fokus på teknisk excellence. Skabt af Kent Beck (~1999).
Traditionelle metoder skaber teknisk gæld og fejl. XP insisterer på **høj kvalitet** hele vejen.

### XP's 5 Værdier

| Værdi | Beskrivelse |
|---|---|
| **Communication** | Åben kommunikation. Tal sammen frem for at skrive lange dokumenter. |
| **Simplicity** | Gør simplest mulige løsning der virker. YAGNI = *You Ain't Gonna Need It*. |
| **Feedback** | Hurtig feedback fra tests, kunden og teamet. Jo hurtigere jo bedre. |
| **Courage** | Tør lave ændringer, refaktorere, sige hvad man mener til kunden. |
| **Respect** | Respekter kollegers arbejde, tid og ekspertise. |

### XP Praksisser (det man gør)

| Praksis | Beskrivelse |
|---|---|
| **Pair Programming** | 2 udviklere, 1 computer. Driver skriver, Navigator reviewer. → Færre fejl, vidensdeling. |
| **Test-First (TDD)** | Skriv test FØR kode. Rød-Grøn-Refaktorer cyklus. → Al kode er testet, tvinger godt design. |
| **Continuous Integration** | Merge og test kode løbende. → Find konflikter tidligt, ikke ved projektslut. |
| **Refactoring** | Forbedring af kode UDEN at ændre adfærd. → Bekæmper teknisk gæld. |
| **Small Releases** | Lever fungerende software hyppigt. → Hurtig feedback-cyklus, mindre risiko. |
| **Collective Code Ownership** | Alle ejer al kode, alle kan ændre alt. → Ingen flaskehalse, bedre videndeling. |
| **Coding Standards** | Fælles kodestil. → Kode ser ens ud uanset hvem der skrev det. |
| **Simple Design** | Gør kun det nødvendige nu. → Undgå over-engineering. |
| **Incremental Design** | Start med simpelt design og forbedr løbende efterhånden som du lærer mere. Undgå Big Design Up Front (BDUF). → Design vokser med koden, ikke før den. |
| **Baby Steps** | Lav den mindst mulige ændring ad gangen. → Reducerer risiko — hvis noget går galt, er der kun lidt at fortryde. Virker synergistisk med TDD. |
| **Ten-Minute Build** | Hele systemet skal kunne bygge og alle tests køre på under 10 minutter. → Langsomme builds = udviklere springer dem over. Hurtig build = hurtig feedback. |
| **Daily Deployment** | Deploy til produktion/staging hver dag. → Små hyppige deployments er langt mindre risikable end store sjældne. |
| **Negotiated Scope Contract** | Kontrakter specificerer tid, kvalitet og pris — IKKE scope. Scope forhandles løbende med kunden. → Modsætning til fastpris-fastscope kontrakter. |
| **Real Customer Involvement** | En rigtig kunde (ikke en repræsentant) er en del af teamet. → Reducerer misforståelser om krav drastisk. |
| **Whole Team** | Alle der er nødvendige for projektets succes sidder sammen: udviklere, testere, kunder, ledelse. → Nedbryder siloer mellem roller. |
| **On-Site Customer** | Kunden tilgængelig dagligt til spørgsmål. → Hurtige svar, ingen ventetid. |
| **40-Hour Week** | Bæredygtigt tempo. → Overarbejde skaber fejl og burnout. |

### XP Principper (det man tror på)

| Princip | Beskrivelse |
|---|---|
| **Humanity** | Software laves af mennesker — respekter menneskelige behov (tryghed, tilhørsforhold, vækst). |
| **Economics** | Løs de mest værdifulde problemer først. Undgå at bruge tid på ting der ikke giver forretningsværdi. |
| **Mutual Benefit** | Valg skal gavne nu OG fremtiden (fx tests gavner dig nu + fremtidige udviklere). Undgå løsninger der kun gavner én part. |
| **Failure** | Fejl er en læringsmulighed. Eksperimenter koster lidt i XP pga. korte iterationer og hurtig feedback. *"If it's hard to do, do it more often."* |
| **Quality** | Kvalitet er ikke noget man ofrer for hastighed — dårlig kvalitet gør dig langsommere på sigt. |
| **Improvement** | Gør dit bedste i dag, og forbedr dig i morgen. Stræb efter excellence, ikke perfektion. |

---

## User Stories

Kort beskrivelse af en funktion set fra **brugerens** perspektiv. Fokus på HVAD og HVORFOR, ikke HOW.
Holder fokus på brugerens behov. Let at estimere og planlægge enkeltvis.

**Format:** `"Som [bruger] vil jeg [handling] så jeg kan [formål/værdi]"`

> Eksempel: "Som kunde vil jeg tilføje varer til kurv så jeg kan samle mine køb ét sted."

### INVEST-kriterier (god User Story)

| Bogstav | Betydning |
|---|---|
| **I** — Independent | Kan stå alene, afhænger ikke af andre stories |
| **N** — Negotiable | Detaljer kan forhandles med kunden |
| **V** — Valuable | Giver reel værdi for brugeren (ikke kun teknisk) |
| **E** — Estimable | Du kan estimere tid/størrelse |
| **S** — Small | Lille nok til at afslutte i én iteration |
| **T** — Testable | Du kan verificere om den er opfyldt |

---

## Definition of Done (DoD)

Klar fælles aftale om hvornår en opgave er "færdig". Ingen subjektivitet.
Undgår "90% færdig" syndromet. Alle ved hvad færdig betyder.

**Typisk DoD indeholder:** kode skrevet + reviewed, unit tests består, CI grøn, manuel test ok.

---

## Burn Down Chart

Diagram der viser hvor meget arbejde der er **tilbage** over tid i en sprint.
Giver øjebliksbillede af om teamet er på sporet til at nå sprint-målet.

- **Y-akse:** Arbejde tilbage (story points)
- **X-akse:** Tid (dage)
- **Ideal linje:** Ret linje fra start til 0. Over ideal = bagud. Under = foran.

---

## Scope Creep

Gradvis ukontrolleret udvidelse af et projekts omfang uden tilsvarende justering af tid, budget eller ressourcer.
Projektet vokser sig større og større uden at nogen besluttede det bevidst — og slutter for sent og over budget.

**Typiske årsager:**
- Kunden tilføjer "bare én lille ting" løbende
- Uklare krav fra starten
- Manglende change control proces
- Teamet siger ja uden at vurdere konsekvensen

**Forebyggelse i agile projekter:**
- Tydelige User Stories med INVEST-kriterier
- Prioriter backlog aktivt — ny feature IN = anden feature OUT
- Timeboxed iterationer — scope er fast, tid er fast
- Synlig Burn Down Chart der viser konsekvens af scope-tilføjelser

> **Vandfald-paradoks:** Scope Creep er *værre* i vandfald fordi kunden ikke ser resultater løbende og samler ønsker op til slutningen.

---

## Vandfald vs Agile

| | Vandfald | Agile |
|---|---|---|
| **Tilgang** | Lineært — ALLE krav → HELE design → AL kode → AL test → levering | Iterativt — lever noget fungerende tidligt, få feedback, tilpas |
| **Problem** | Kunden ser produktet sidst. Dyrt at ændre krav. Ikke fleksibelt. | — |
| **Fordel** | Dokumentation kan være lovpåkrævet | Fejl opdages tidligt. Kunden er med hele vejen. |
| **Hvornår?** | Stabile krav, kritisk software (medicin, raketter) | Ustabile krav, tæt kundekontakt |

---

## Opgaver til selvstudium

1. Læs de 4 værdier i det Agile Manifest — forklar med dine egne ord hvad hvert par betyder i praksis
2. Hvad er de 12 XP-praksisser? Vælg tre og beskriv konkret hvad de kræver af et team
3. Hvad er forskellen på en User Story og et krav i en kravspecifikation? Hvad gør en User Story god?
4. Hvad er Acceptance Criteria, og hvornår er en User Story "færdig" ifølge Definition of Done?
5. Sammenlign Vandfald og Agil: hvornår er Vandfald det bedste valg? Hvornår er Agil bedre?
6. Hvad er MoSCoW-prioritering? Brug det på 6 hypotetiske features til en webshop
