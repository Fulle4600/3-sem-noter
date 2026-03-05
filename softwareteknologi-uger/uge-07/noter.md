# Uge 7 — Softwarekvalitet, Definition of Done & Metodesammenligning

Denne uge dykker vi naermere ned i, hvad det egentlig vil sige at software er af "god kvalitet", og hvordan vi sikrer det i praksis. Vi introducerer begrebet Definition of Done (DoD), gennemgar flere XP-principper og sammenligner forskellige systemudviklingsmetoder — fra den helt planbaserede vandfaldsmodel til ekstrem agil udvikling.

---

## 1. Hvad er Softwarekvalitet?

Softwarekvalitet er et bredt begreb, der daekker mange aspekter af, hvad der giver software vaerdi. Det er ikke kun "virker koden?" men ogsa "er koden vedligeholdbar?", "er den hurtig nok?", "er den sikker?".

### To perspektiver pa kvalitet

**Ekstern kvalitet** — det brugeren ser og oplever:
- Fungerer applikationen korrekt?
- Er den hurtig?
- Er den nem at bruge?
- Kraesher den?

**Intern kvalitet** — det udviklerne arbejder med:
- Er koden laesbar og forstaelig?
- Er den nem at aendre?
- Er den testbar?
- Er der faerre bugs?

En af XPs grundtanker er, at **hoej intern kvalitet forer til hoej ekstern kvalitet** over tid. Darlig kode (teknisk gaeld) giver flere bugs og gere det svaerere at tilfoeje ny funktionalitet.

### ISO 25010 Kvalitetsmodel (overfladisk)

ISO 25010 definerer 8 overordnede kvalitetskarakteristika for software:

| Karakteristika | Forklaring |
|---------------|-----------|
| Functional Suitability | Software goer hvad det skal |
| Performance Efficiency | Ressourceforbrug og responstid |
| Compatibility | Fungerer med andre systemer |
| Usability | Nem at bruge |
| Reliability | Stabilt og fejltolerант |
| Security | Beskyttet mod angreb og uautoriseret adgang |
| Maintainability | Nem at vedligeholde og aendre |
| Portability | Kan flyttes til andre miljoer |

---

## 2. Definition of Done (DoD)

**Definition of Done (DoD)** er en faelles forstaelse i teamet af, hvornaar en opgave er **faerdig**. Det er IKKE en checkliste per User Story — det er en generel standard for alt arbejde teamet laver.

### Hvorfor er DoD vigtig?

Uden en klar DoD kan "faerdig" betyde vidt forskelligt for forskellige teammedlemmer:
- Udvikler A: "Jeg har skrevet koden, den kompilerer"
- Udvikler B: "Koden er skrevet, testet og reviewet"
- Udvikler C: "Alt er deployed til produktion"

Uklart definition af "done" forer til:
- Skjult teknisk gaeld
- Overraskelser ved sprint review
- Manglende tests
- Darlig kode der sender i produktion

### Typisk indhold i en DoD

En god DoD for et softwareudviklingsteam kunne indeholde:

```
Definition of Done — Team Alpha

Kode:
[ ] Koden er implementeret og opfylder acceptance criteria
[ ] Koden er reviewet af mindst eet andet teammedlem
[ ] Koden folger vores kodningsstandard
[ ] Ingen hardcoded vaerdier (brug config/environment variables)

Test:
[ ] Unit tests er skrevet for ny kode
[ ] Alle eksisterende tests bestaar
[ ] Kode coverage er ikke faldet under 80%
[ ] Manuelt testet i testmiljoet

Dokumentation:
[ ] Offentlige metoder/klasser har XML-kommentarer
[ ] README er opdateret hvis noedvendigt
[ ] API dokumentation er opdateret

Deployment:
[ ] Koden er merget til main/develop branch
[ ] CI/CD pipeline er groen
[ ] Deployed til staging-miljoet
[ ] Produktejer har godkendt
```

### DoD vs Acceptance Criteria

| | DoD | Acceptance Criteria |
|-|-----|---------------------|
| **Hvad det er** | Generel standard for alt arbejde | Specifikke krav til en enkelt story |
| **Hvem definerer** | Hele teamet (inkl. Scrum Master) | Produktejer og teamet |
| **Naer det bruges** | Alle opgaver | Specifik story/feature |
| **Eksempel** | "Alle tests skal bestaa" | "Brugeren kan se ordrelisten med pagination" |

---

## 3. XP Principper: Failure, Quality, Improvement, Negotiated Scope Contract, Baby Steps

### Failure (Fejl som laering)

XP-princippet om **Failure** handler om at se fejl som en laeringsmulighed snarere end noget at skjule eller bestraffe.

> "Failure is not a waste if we learn from it."

Praktiske implikationer:
- Nar noget gaar galt, hold en **retrospektiv** og analyseer hvad der skete
- Post-mortems skal vaere blameless (uden bebrejdelse)
- Fejl i tests er GODE — de betyder at vi har opdaget et problem tidligt
- TDD: Skriv en fejlende test FORST (roed) — dette er en bevidst fejl der driver udviklingen

**Eksempel**: Et team deployede og det foraarsagede et nedbrud. I stedet for at bestraffe nogen, analyserede de: Hvad manglede vi i vores deployment-proces? -> De tilfoejede automatisk smoke-test efter deployment.

### Quality (Kvalitet)

XP's syn pa kvalitet er radikalt:

> "Increasing quality increases velocity. Decreasing quality decreases velocity."

Dette lyder kontraintuitivt — "men vi sparer tid nar vi skipper tests". Nej! Darlig kvalitet koester mere pa lang sigt:
- Bugs tager tid at finde og rette
- Svaer kode bremser ny funktionalitet
- Darlig kode fremmaner frygten for at aendre noget

**Praktisk**: Unit tests, code review, refactoring og pair programming er alle kvalitetspraksisser i XP. De koster tid nu, men sparer tid fremover.

### Improvement (Forbedring)

Intet er perfekt. XP opfordrer til loebende forbedring af:
- Kode (refactoring)
- Processer (retrospektiver)
- Faerdigheder (laering, parprogrammering)

> "Best practices are called 'best' not because they are perfect, but because they are better than the alternatives."

Noeglevaerktoej: **Sprint Retrospektive** — efter hvert sprint reflekterer teamet:
- Hvad gik godt?
- Hvad kan forbedres?
- Hvad goer vi anderledes naeste sprint?

### Negotiated Scope Contract (Forhandlet Omfang)

I traditionelle projekter er kontrakten fast: "Vi leverer disse funktioner til den pris pa den dato." Problemet er, at dette er umuligt at overholde i softwareudvikling, fordi krav altid aendrer sig.

XP's loesning er **Negotiated Scope Contract**:
- Tid og pris er FAST (budgettet og deadlinen)
- Scope er FLEKSIBELT (hvad vi leverer kan aendres)

```
Traditionel kontrakt:     XP kontrakt:
+----------+             +----------+
| Scope    | FAST        | Scope    | FLEKSIBEL
| Tid      | FAST        | Tid      | FAST
| Pris     | FAST        | Pris     | FAST
+----------+             +----------+
```

Dette krever at kunden:
1. Prioriterer features (mest vaerdifulde forst)
2. Accepterer at ikke alt kan naas inden for tid og budget
3. Loebende kommunikerer med teamet om prioriteter

### Baby Steps (Sma Skridt)

> "A journey of a thousand miles begins with a single step."

Baby Steps handler om at tage sma, sikre skridt fremad i stedet for store, risikable spring. Princippet galder pa alle niveauer:

**I kode**: Refaktorer lidt ad gangen, kore tests efter hvert lille trin
**I TDD**: Skriv een test ad gangen, implementer minimalt for at fa den til at bestaa
**I deployment**: Deploy hyppigt med sma aendringer (lettere at finde fejl)
**I arkitektur**: Evoluer arkitekturen incrementalt (Incremental Design)

**Eksempel pa Baby Steps i praksis**:
```
FORKERT (Big Bang):
Refaktorer hele authentication-systemet pa een gang
-> Stor risiko for at bryde noget

RIGTIGT (Baby Steps):
1. Ekstraheer password-validering til en metode
2. Kore alle tests - passes? Ja -> fortsaet
3. Flyt metoden til en separat klasse
4. Kore alle tests - passes? Ja -> fortsaet
5. Skriv unit tests for den nye klasse
6. Kore alle tests - passes? Ja -> commit
```

---

## 4. Systemudviklingsmetoder

### Vandfaldsmodellen

Den klassiske vandfaldsmodel (Winston Royce, 1970) bestar af sekventielle faser:

```
Krav -> Design -> Implementation -> Test -> Deployment -> Vedligeholdelse
```

**Fordele:**
- Nem at forsta og styre
- Klar dokumentation i hver fase
- Velegnet nar krav er stabile og veldefinerede (f.eks. NASA-raketsoftware)

**Ulemper:**
- Krav kan ikke aendres let undervejs
- Test sker sent — fejl opdages sent og er dyre
- Kunden ser ikke produktet foer sent i processen
- Overkompleks for de fleste projekter

### Scrum

Scrum er den mest udbredte agile metode. Nagleelementer:
- **Product Backlog**: Liste over alle features
- **Sprint**: 1-4 ugers iteration
- **Daily Scrum**: 15-minutters dagligt standup
- **Sprint Review**: Demo til interessenter
- **Sprint Retrospective**: Teamets refleksion

Roller:
- **Product Owner**: Ejer backloggen, prioriterer
- **Scrum Master**: Facilitator, fjerner blokeringer
- **Development Team**: Laver arbejdet

### XP (Extreme Programming)

XP fokuserer mere pa tekniske praksisser end Scrum:
- Test-Driven Development (TDD)
- Pair Programming
- Continuous Integration
- Refactoring
- Simple Design

Typisk brugt i kombinaton med Scrum (Scrum for processen, XP for de tekniske praksisser).

### Kanban

Kanban er en flydende metode uden sprints:
- Tavle med kolonner (To Do, In Progress, Done)
- **WIP-limits**: Max antal opgaver "in progress"
- Kontinuerlig levering i stedet for sprint-leverancer

Velegnet til support-teams og vedligeholdelsesopgaver.

---

## 5. Metode-Kontinuum

Systemudviklingsmetoder kan placeres pa et kontinuum fra **helt planbaseret** til **helt agilt**:

```
Planbaseret                                              Agilt
|-------|-------|-------|-------|-------|-------|-------|
Vandfald  Spiral  RUP    DSDM   Scrum   XP    Crystal

<-- Mere dokumentation                  Mere kode -->
<-- Mere forudsigelighed              Mere fleksibilitet -->
<-- Bedre for stabile krav     Bedre for ustabile krav -->
```

### Karakteristika for yderpunkterne

| | Planbaseret | Agilt |
|-|-------------|-------|
| **Kravhåndtering** | Faeste krav fra start | Krav kan aendres |
| **Dokumentation** | Encompassing | Minimal, behovsbaseret |
| **Leverancer** | En stor levering til sidst | Hyppige sma leverancer |
| **Teamstruktur** | Hierarkisk | Selvorganiserende |
| **Kundeinteraktion** | I start og slut | Loebende |
| **Risikohaandtering** | Planlaeggingsbaseret | Feedbackbaseret |

---

## 6. Boehm & Turner Matrix

Barry Boehm og Richard Turner udviklede en model for at hjaelpe teams med at vaelge den rigtige metode. Modellen er baseret pa fem faktorer:

### De 5 Faktorer

#### 1. Teamstorrelse
- **Smat team (< 10)**: Agile metoder fungerer godt — nem kommunikation
- **Stort team (> 50)**: Planbaserede metoder fungerer bedre — behov for koordination

#### 2. Kritikalitet (hvad sker nar det fejler?)
- **Lavt** (tab af komfort): Agile OK
- **Medium** (tab af penge): Begge kan bruges
- **Hoejt** (tab af menneskeliv): Planbaseret foretraekkes

#### 3. Dynamik i krav
- **Lav dynamik** (stabile krav): Planbaseret
- **Hoj dynamik** (aendrende krav): Agilt

#### 4. Kulturer (Amabile & Gryskiewicz model)
- **Chaos-tolerant kultur**: Agile fungerer godt
- **Order-seeking kultur**: Planbaseret fungerer bedre

#### 5. Personlig kompetence (Cockburn-skala)
- **Level 1B**: Kompetent, men skadet; behoever hjaelp
- **Level 1**: Folger instruktioner
- **Level 2**: Tilpasser til kontekst
- **Level 3**: Vejleder andre
- **Level 3+**: Innoverer

Agile metoder kraever typisk Level 2+ kompetence.

### Boehm & Turner Diagram

```
         Chaos
          ^
          |  [Agile zone]
          |
Dynamik   |         [Hybrid zone]
i krav    |
          |                    [Plan-driven zone]
          |
         Order
          +-------------------------->
          Sma teams              Store teams
          (< 10)                 (> 50)
```

### Nar bruger man hvilken metode?

| Scenarie | Anbefalet metode |
|---------|-----------------|
| Startup med ustabile krav og lille team | Agil (XP/Scrum) |
| Banksystem med stabile krav og 100+ udviklere | Planbaseret (RUP/SAFe) |
| Medicinsk udstyrssoftware | Planbaseret + FDA compliance |
| E-commerce webshop | Agil (Scrum) |
| Forsvarssystem | Planbaseret |
| SaaS-produkt i hurtig vækst | Agil med skaleringsramme |

---

## 7. Opsummering og Eksamensrelevante Punkter

### Centrale begreber

| Begreb | Definition |
|--------|-----------|
| DoD | Faelles teamstandard for hvornaar arbejde er faerdigt |
| Softwarekvalitet | Intern (kode) og ekstern (brugeroplevelse) kvalitet |
| Vandfaldsmodellen | Sekventiel metode — krav -> design -> kode -> test |
| Metode-kontinuum | Skala fra planbaseret til agilt |
| Boehm & Turner | Model for metodevalg baseret pa 5 faktorer |
| Negotiated Scope | Fast tid/pris, fleksibelt scope |
| Baby Steps | Sma, sikre trin fremfor store risikable spring |

### Typiske eksamenssporgsmaal

1. **Hvad er en Definition of Done, og hvad bor den indeholde?**
   - Forklar med eksempel pa konkrete punkter

2. **Hvornaar er vandfaldsmodellen hensigtsmassig vs. agile metoder?**
   - Brug Boehm & Turner-faktorer

3. **Forklar XP-princippet "Quality" og argumenter for det**
   - "Increasing quality increases velocity"

4. **Hvad er et Negotiated Scope Contract?**
   - Hvad er fast? Hvad er fleksibelt? Hvad krever det af kunden?

5. **Hvad er Baby Steps i XP?**
   - Giv eksempel pa Baby Steps i TDD

### Husk til eksamen
- DoD galder for ALLE opgaver, mens acceptance criteria er specifikke for en enkelt story
- Boehm & Turner siger ikke at en metode er bedst — det afhaenger af konteksten
- XP's syn pa kvalitet er modsat intuitivt: hoejere kvalitet = hojere hastighed pa lang sigt
- Vandfaldsmodellen er ikke altid "darlig" — den er vaerdifuld i rette kontekst (stabile krav, kritiske systemer)
