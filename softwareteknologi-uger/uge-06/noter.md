# Uge 6 — Introduktion til Agile & Extreme Programming (XP)

Denne uge introducerer vi det agile tankesæt og ser naermere pa, hvad det vil sige at arbejde agilt i praksis. Vi gennemgar Det Agile Manifest med dets 4 vaerdier og 12 principper, og begynder at arbejde med XP (Extreme Programming) som konkret agil metode. Derudover ser vi pa User Stories, Non-functional requirements og Burn Down-Charts som centrale vaerktojer i agil udvikling.

---

## 1. Hvad er agil softwareudvikling?

Agil softwareudvikling er en tilgang til systemudvikling, der laegger vaegt pa **fleksibilitet, samarbejde og loebende levering af vaerdi**. I stedet for at planlaegge alt fra start og derefter eksekvere en lang plan (som i vandfaldsmodellen), arbejder agile teams i korte iterationer og tilpasser sig loebende til ny viden og aendrede krav.

### Det modsatte: Vandfaldsmodellen
I den klassiske vandfaldsmodel gennemgar man faserne sekventielt:
1. Kravindsamling
2. Design
3. Implementering
4. Test
5. Deployment
6. Vedligeholdelse

**Problemet** med vandfald er, at krav aendrer sig, og man opdager fejl sent i processen, hvor de er dyre at rette.

### Agil tilgang
Agile teams arbejder i **sprints** (typisk 1-4 uger), hvor man:
- Planlaegger en lille del af arbejdet
- Implementerer og tester loebende
- Leverer fungerende software
- Indhenter feedback fra kunden
- Tilpasser planen baseret pa feedback

---

## 2. Det Agile Manifest (2001)

Det Agile Manifest blev skrevet i 2001 af 17 softwareudviklere i Snowbird, Utah. Det bestar af **4 vaerdier** og **12 principper**.

### De 4 Vaerdier

| Vaerdsat | Frem for |
|----------|----------|
| **Individer og interaktioner** | Processer og vaerktojer |
| **Fungerende software** | Omfattende dokumentation |
| **Kundesamarbejde** | Kontraktforhandling |
| **Reaktion pa forandring** | At folge en plan |

> "Det vil sige, at selvom der er vaerdi i det, der star til hoejre, vaerdsaetter vi det, der star til venstre, hoejere."

#### Uddybning af vaerdierne:

**Individer og interaktioner frem for processer og vaerktojer**
Det handler om, at et godt team med god kommunikation er vigtigere end de rigtige processer. Vaerktojer og processer er hjaelpemidler, ikke mal i sig selv. Et moede ansigt til ansigt er mere vaerdifuldt end 10 emails.

**Fungerende software frem for omfattende dokumentation**
Dokumentation har vaerdi, men maalet er at levere software der virker. "Working software is the primary measure of progress." Skriv dokumentation nar det er noedvendigt - ikke som ritual.

**Kundesamarbejde frem for kontraktforhandling**
Inkluder kunden i processen. Jo mere kunden er involveret, jo storre er sandsynligheden for at slutproduktet matcher behovene. Kontrakter kan blive barrierer for samarbejde.

**Reaktion pa forandring frem for at folge en plan**
Krav aendrer sig. Markedet aendrer sig. Teknologi aendrer sig. Et agilt team kan reagere hurtigt pa disse aendringer i stedet for blindt at folge en gammel plan.

---

### De 12 Principper

1. **Tilfreds kunde** — Vores hoejeste prioritet er at tilfredsstille kunden gennem tidlig og kontinuerlig levering af vaerdifuld software.

2. **Velkommne aendringer** — Velkommne skiftende krav, selv sent i udviklingen. Agile processer udnytter forandring til kundens konkurrencefordel.

3. **Hyppig levering** — Lever fungerende software hyppigt, fra et par uger til et par maneder, med praeferance for kortere tidsramme.

4. **Samarbejde dagligt** — Forretningsfolk og udviklere skal samarbejde dagligt igennem projektet.

5. **Motiverede individer** — Byg projekter rundt om motiverede individer. Giv dem de omgivelser og den stoette de har brug for, og stol pa at de udforer arbejdet.

6. **Ansigt-til-ansigt kommunikation** — Den mest effektive metode til at formidle information til og inden for et udviklingsteam er samtale ansigt til ansigt.

7. **Fungerende software** — Fungerende software er det primaere mal for fremskridt.

8. **Baeredygtigt tempo** — Agile processer fremmer baeredygtig udvikling. Sponsorerne, udviklerne og brugerne skal kunne opretholde et konstant tempo pa ubestemt tid.

9. **Teknisk ekspertise** — Kontinuerlig opmaerksomhed pa teknisk ekspertise og godt design forbedrer agilitet.

10. **Simplicitet** — Simplicitet — kunsten at maksimere maengden af arbejde der ikke er gjort — er essentiel.

11. **Selvorganiserende teams** — De bedste arkitekturer, krav og designs fremkommer fra selvorganiserende teams.

12. **Refleksion og tilpasning** — Med regelmaessige mellemrum reflekterer teamet over, hvordan det kan blive mere effektivt, og justerer sin adfaerd derefter.

---

## 3. Burn Down-Charts

Et **Burn Down-Chart** er et visuelt vaerktoej der viser, hvor meget arbejde der er tilbage i et sprint eller projekt over tid.

### Laesning af et Burn Down-Chart

```
Story Points
tilbage
  ^
40|  *
35|    *
30|      *  <- faktisk linje
25|        *
20|          *
15|            *   <- ideal linje
10|               *
 5|                  *
 0|____________________*___> Dage
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10
```

- **Y-aksen**: Resterende arbejde (story points eller timer)
- **X-aksen**: Tid (dage i sprintet)
- **Ideal-linjen** (stiplet): En ret linje fra sprint-start til sprint-slut, der viser det ideelle tempo
- **Faktisk linje**: Den rigtige fremgang i teamet

### Hvad kan man laese fra et Burn Down-Chart?

| Scenarie | Hvad det betyder |
|----------|-----------------|
| Faktisk linje over ideal-linjen | Teamet er bagud — der er mere arbejde tilbage end planlagt |
| Faktisk linje under ideal-linjen | Teamet er foran — arbejdet gar hurtigere end planlagt |
| Flad linje i perioder | Teamet er blokeret eller tilfojer ny opgaver |
| Pludselig stigning | Ny scope er tilfojet (scope creep) |
| Flad til slut, ikke nul | Sprint er afsluttet uden at al arbejde er faerdigt |

### Burn Up-Chart (alternativ)
I stedet for at vise hvad der er TILBAGE, viser et Burn Up-Chart hvad der er GJORT. Fordelen er at det ogsa viser, nar scope aendres (linjen for total scope stiger).

---

## 4. User Stories

En **User Story** er en kort, uformel beskrivelse af en funktionalitet set fra slutbrugerens perspektiv. Det er IKKE en teknisk specifikation — det er en kommunikationsvaerktoej.

### Format

```
Som [rolle/bruger]
vil jeg [handling/funktionalitet]
sa jeg kan [forretningsvaerdi/mal]
```

### Eksempler

**Godt eksempel:**
```
Som indlogget kunde
vil jeg kunne se mine tidligere ordrer
sa jeg kan holde styr pa mine koeb og spore leveringer
```

**Andet eksempel:**
```
Som administrator
vil jeg kunne deaktivere brugere
sa jeg kan haandtere misbrug af systemet
```

**Daarligt eksempel (for teknisk):**
```
Som system
skal der vaere en GET /api/users/{id} endpoint
der returnerer JSON med brugerdata
```
Dette er en teknisk implementeringsdetalje, ikke en User Story.

### Acceptance Criteria
Til hver User Story knyttes **acceptance criteria** — konkrete betingelser for hvornaar historien er "done":

```
Som indlogget kunde
vil jeg kunne se mine tidligere ordrer
sa jeg kan holde styr pa mine koeb

Acceptance Criteria:
- [ ] Kunden kan se en liste over alle tidligere ordrer
- [ ] Hver ordre viser ordrenummer, dato og totalbeloeb
- [ ] Kunden kan klikke pa en ordre for at se detaljer
- [ ] Siden viser max 20 ordrer pr. side med pagination
- [ ] Hvis ingen ordrer, vises en venlig besked
```

---

## 5. INVEST-Kriterier for User Stories

INVEST er et akronym for 6 egenskaber en god User Story bor have:

| Bogstav | Engelsk | Dansk | Forklaring |
|---------|---------|-------|------------|
| **I** | Independent | Uafhaengig | Historien skal kunne udviklkes uafhaengigt af andre historier |
| **N** | Negotiable | Forhandlingsbar | Detaljerne kan forhandles mellem team og kunde |
| **V** | Valuable | Vaerdifuld | Historien skal give vaerdi til brugeren eller forretningen |
| **E** | Estimable | Estmerbar | Teamet skal kunne estimere historiens stoerrelse |
| **S** | Small | Lille | Historien skal vaere lille nok til at passe i eet sprint |
| **T** | Testable | Testbar | Det skal vaere muligt at verificere om historien er implementeret korrekt |

### Eksempel pa INVEST-evaluering

**User Story**: "Som bruger vil jeg have et bedre system"

- I: Ja (dog bred)
- N: Ja
- V: Maske (for vag)
- E: **NEJ** — kan ikke estimeres
- S: **NEJ** — alt for stor
- T: **NEJ** — "bedre" kan ikke testes

Denne historie **bestaar ikke INVEST**. Den skal brydes ned i mere specifikke historier.

---

## 6. Non-Functional Requirements (NFR)

Non-functional requirements (ikke-funktionelle krav) beskriver **hvordan** systemet skal fungere, i modsaetning til hvad det skal gere (funktionelle krav).

### Typer af Non-Functional Requirements

#### Performance (Ydeevne)
```
Systemet skal svare pa 95% af alle API-kald inden for 200ms under normal belastning.
Login-siden skal loade inden for 1 sekund pa en 4G forbindelse.
```

#### Sikkerhed
```
Alle passwords skal hashes med BCrypt med minimum salt-rounds 10.
Systemet skal vaere beskyttet mod SQL injection og XSS angreb.
Sessioner skal udlobe efter 30 minutters inaktivitet.
HTTPS skal bruges til alle forbindelser.
```

#### Tilgaengelighed (Accessibility)
```
Systemet skal overholde WCAG 2.1 niveau AA.
Alle billeder skal have alt-tekst.
Systemet skal kunne bruges med kun tastatur.
Farvekontrast skal vaere mindst 4.5:1.
```

#### Skalerbarhed
```
Systemet skal kunne haandtere 10.000 samtidige brugere.
Databasen skal skalere til 1 million poster uden performancenedgang.
```

#### Vedligeholdbarhed
```
Al kode skal vaere kommenteret og dokumenteret.
Kode-coverage for unit tests skal vaere mindst 80%.
```

### NFR vs Funktionelle Krav

| | Funktionelt krav | Non-functional krav |
|-|-----------------|---------------------|
| **Sporgsmaal** | Hvad skal systemet gere? | Hvordan skal systemet gere det? |
| **Eksempel** | Bruger kan logge ind | Login skal ske inden for 1 sekund |
| **Testning** | Ja/Nej (virker det?) | Maalt (performance, usability) |

---

## 7. XP — Extreme Programming: Intro og Principper

**Extreme Programming (XP)** er en agil udviklingsmetode skabt af Kent Beck i slutningen af 1990'erne. XP er baseret pa at "ekstrapolere" god praksis til det ekstreme.

XP er bygget pa:
- **5 vaerdier**
- **14 principper**
- **Konkrete praksisser** (TDD, Pair Programming, CI, etc.)

### XP's 5 Vaerdier

1. **Communication (Kommunikation)** — Alle i teamet skal kommunikere aabent og aerlikt
2. **Simplicity (Enkelthed)** — Lav den enkleste loesning der virker
3. **Feedback** — Hent feedback tidligt og hyppigt
4. **Courage (Mod)** — Tag svare beslutninger, refaktorer kode, sig nej til urealistiske deadlines
5. **Respect (Respekt)** — Alle bidrager, alle er vaerdifulde

### XP Principper (Uge 6 fokus)

#### Accepted Responsibility (Accepteret Ansvar)
Ansvar **tildeles ikke** — det **accepteres**. Du kan ikke tvinge nogen til at vaere ansvarlig for noget, de ikke selv har sagt ja til. Nar et teammedlem accepterer en opgave, accepterer de ogsa ansvaret for dens kvalitet og korrekte udforelse.

Praktisk: Nar du vaelger en User Story fra backloggen, er det dig der har ansvaret for den — ikke din chef.

#### Communication (Kommunikation)
Alle problemer i softwareudvikling kan spores tilbage til kommunikationsproblemer. XP loser dette med:
- Pair programming (loebende kommunikation)
- Standup-moeder (daglig synkronisering)
- Fysisk placering af teamet (osmotisk kommunikation)

#### Humanity (Menneskelighed)
Software laves af mennesker, for mennesker. XP anerkender, at udviklere er mennesker med beehov for:
- Basal sikkerhed (job, indkomst)
- Tilhoersforhold (del af et team)
- Vaekst (learing og udvikling)
- Bidrag (ens arbejde giver mening)

#### Respect (Respekt)
Alle i teamet respekterer hinanden — uanset erfaring, rolle, eller status. Seniorer respekterer juniorer og omvendt. Alle bidrag vaerdsaettes.

#### Feedback
Feedback skal vaere:
- **Hurtigt**: Jo hurtigere du faar feedback, jo billigere er det at rette fejl
- **Hyppigt**: Ikke kun ved sprint-review, men loebende
- **Konkret**: "Din kode bestod ikke testen" er bedre end "Din kode er darlig"

Feedback-loekker i XP:
```
Sekunder: Tests giver feedback (unit tests)
Minutter:  Pair programmer giver feedback
Timer:     Daily standup giver feedback
Dage:      Sprint review giver feedback
Uger:      Release giver feedback fra brugere
```

#### Economics (Oekonomi)
Tid er penge. Softwareudvikling koster penge, og maalet er at levere forretningsvaerdi. XP's praksisser er designet til at maksimere vaerdi og minimere spild:
- Lever det mest vaerdifulde forst
- Undga overingeniering
- Undga gentagelse (DRY)

#### Mutual Benefit (Gensidig fordel)
Alle praksisser i XP skal gavne alle involverede parter:
- **Nu**: Koden er let at forsta og aendre
- **Fremtid**: Koden er vedligeholdbar og testbar
- **Kunden**: Faar vaerdi hurtigt og kan aendre krav

Eksempel: Unit tests er skrevet nu (kosten tidsmassigt), men gavner fremtidige udviklere og reducerer bugs (gensidigt fordel).

---

## 8. Opsummering og Eksamensrelevante Punkter

### Centrale begreber at huske

| Begreb | Definition |
|--------|-----------|
| Agil | Iterativ, inkrementel udviklingsmetode med fokus pa feedback og fleksibilitet |
| Sprint | Kort, tidsbegraenset iteration (1-4 uger) |
| User Story | Kort beskrivelse af funktionalitet fra brugerens perspektiv |
| Burn Down Chart | Visuelt vaerktoej der viser resterende arbejde over tid |
| NFR | Krav til systemets egenskaber (performance, sikkerhed, etc.) |
| XP | Extreme Programming — agil metode med starka praksisser |
| INVEST | Kriterier for gode User Stories |

### Typiske eksamenssporgsmaal

1. **Forklar de 4 vaerdier i Det Agile Manifest**
   - Brug tabellen ovenfor og forklar hvert punkt med egne ord

2. **Hvad er en User Story, og hvad er INVEST-kriterierne?**
   - Giv eksempler pa gode og darlige user stories

3. **Hvad viser et Burn Down-Chart?**
   - Forklar akserne og hvad man kan laese af grafen

4. **Hvad er non-functional requirements? Giv eksempler.**
   - Naevn mindst 4 kategorier med eksempler

5. **Forklar XP-princippet "Mutual Benefit"**
   - Giv et konkret eksempel pa en praksis med gensidig fordel

### Husk til eksamen
- Det Agile Manifest siger IKKE at dokumentation, processer og planer er vaerdiloes — bare at de fire vaerdier til venstre vaegtes hoejere
- User Stories er et **kommunikationsvaerktoej**, ikke en kravsspecifikation
- NFR er ligesom vigtige som funktionelle krav — de pavirker arkitektur og design fra starten
