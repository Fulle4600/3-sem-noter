# Uge 17 — Sikkerhed & Kryptering

Denne uge handler om informationssikkerhed — hvad det betyder, hvordan kryptering virker, og hvordan HTTPS beskytter dine data. Vi kigger også på WireShark som et værktøj til at se hvad der faktisk sker på netværket.

---

## Informationssikkerhed — CIA-triaden

Al informationssikkerhed handler om at beskytte tre ting, samlet kaldet **CIA**:

| Bogstav | Princip | Betyder |
|---|---|---|
| **C** | Confidentiality (Fortrolighed) | Kun autoriserede kan læse data |
| **I** | Integrity (Integritet) | Data er ikke blevet ændret undervejs |
| **A** | Availability (Tilgængelighed) | Systemet er tilgængeligt når det behøves |

**Eksempel — netbank:**
- **C:** Kun du kan se din kontosaldo (ikke nogen der aflytter netværket)
- **I:** En overførsel på 100 kr. ankommer som 100 kr. — ikke ændret til 10.000 kr.
- **A:** Banken er tilgængelig 24/7 — ikke nede pga. DDoS angreb

---

## Symmetrisk Kryptering

**Samme nøgle bruges til at kryptere og dekryptere.**

```
Afsender:  Klartekst + Nøgle → Krypteringsalgoritme → Krypteret tekst
Modtager:  Krypteret tekst + Nøgle → Dekrypteringsalgoritme → Klartekst
```

**Eksempel — Caesar Cipher (simpel, ikke sikker):**
```
Klartekst:    H E L L O
Nøgle:        +3 (skift 3 bogstaver frem)
Krypteret:    K H O O R
```

**I virkeligheden bruges AES (Advanced Encryption Standard):**
- 128-bit eller 256-bit nøgle
- Ekstremt hurtig — bruges til at kryptere store datamængder
- Bruges til at kryptere selve data-indholdet i HTTPS

**Problem med symmetrisk kryptering:**
> Hvordan sender du nøglen sikkert til modtageren? Hvis du sender nøglen usikret, kan en angriber opfange den og dekryptere alt.

---

## Asymmetrisk Kryptering (Public Key Cryptography)

**To nøgler: public key (alle må se) og private key (kun ejeren har).**

```
Nøglepar:   Public Key  ←→  Private Key
             (Del med alle)    (Hemmelighed)
```

**Reglerne:**
1. Data krypteret med **public key** kan KUN dekrypteres med **private key**
2. Data signeret med **private key** kan verificeres med **public key**

```
KRYPTERING:
  Afsender: "Hej Bank!" + Banks public key → krypteret besked
  Bank:     krypteret besked + Banks private key → "Hej Bank!"
  (Ingen andre kan læse beskeden — kun banken har private key)

DIGITAL SIGNATUR:
  Server: data + servers private key → signatur
  Klient: data + signatur + servers public key → "Ja, serveren signerede dette!"
  (Bevis på at data er autentisk og ikke ændret)
```

**Eksempel — RSA algoritmen:**
- Baseret på at det er nemt at gange to store primtal, men næsten umuligt at faktorisere produktet
- 2048-bit eller 4096-bit nøglelængde
- Langsom — bruges KUN til at udveksle nøgler, ikke til at kryptere store datamængder

| | Symmetrisk (AES) | Asymmetrisk (RSA) |
|---|---|---|
| **Nøgler** | 1 delt nøgle | Public + Private key par |
| **Hastighed** | Meget hurtig | Langsom |
| **Problem** | Nøgleudveksling | — |
| **Brug** | Kryptere data (bulk) | Nøgleudveksling, signaturer |

---

## HTTPS / TLS / SSL

**HTTPS = HTTP + TLS-kryptering.** Al data er krypteret — ingen kan aflytte forbindelsen.

### Terminologi

- **SSL** (Secure Sockets Layer) — den gamle protokol (nu deprecated og usikker)
- **TLS** (Transport Layer Security) — den moderne erstatning (TLS 1.2 / TLS 1.3)
- Folk siger stadig "SSL" i hverdagssproget, men de mener TLS

### TLS Handshake — step by step

```
Klient                                          Server
  |                                               |
  |------ ClientHello (jeg understøtter TLS 1.3) →|
  |                                               |
  |← ServerHello + Certifikat (public key) -------|
  |                                               |
  |  Klient: Verificer certifikat mod CA liste    |
  |  (Er certifikatet signeret af en betroet CA?) |
  |                                               |
  |------ Krypteret: nøgleudveksling -----------→ |
  |                                               |
  |  Begge beregner nu den samme session key      |
  |  (Symmetrisk nøgle til resten af sessionen)  |
  |                                               |
  |==== Al videre kommunikation er krypteret ====|
```

**Certifikater og CA:**
- Et **certifikat** indeholder serverens public key + serverens identitet (domænenavn)
- Certifikatet er **signeret** af en **Certificate Authority (CA)** — f.eks. Let's Encrypt, DigiCert
- Din browser har en liste over betroede CA'er
- Hvis certifikatet er signeret af en betroet CA → browseren stoler på serveren

**Hvad HTTPS beskytter:**
- Kryptering: ingen kan aflytte data
- Autentificering: du ved du taler med den rigtige server (ikke en impostor)
- Integritet: data er ikke ændret undervejs

**Hvad HTTPS IKKE beskytter:**
- En sikker forbindelse til et ondsindet website er stadig en sikker forbindelse til et ondsindet website!

---

## Digital Signatur

Bruges til at bevise at et dokument/besked er autentisk og ikke ændret.

```
1. Afsender beregner hash af dokumentet    → f3a9b2...
2. Afsender krypterer hashen med private key → signatur
3. Sender dokument + signatur til modtager

Modtager:
4. Dekrypterer signatur med afsenders public key → f3a9b2...
5. Beregner selv hash af dokumentet           → f3a9b2...
6. Sammenlign: ens? → Autentisk og uændret!
```

---

## Operational Security (OpSec)

Praksis og vaner der beskytter information i hverdagen:

- **Stærke, unikke passwords** — brug en password manager
- **To-faktor autentificering (2FA)** — noget du ved + noget du har
- **Opdatér software** — patches lukker kendte sikkerhedshuller
- **Phishing-bevidsthed** — verificer afsender, klik ikke på mistænkelige links
- **Mindste privilegium** — giv kun adgang til det der er nødvendigt

---

## WireShark

Netværksanalyseværktøj der lader dig *se* al netværkstrafik på dit netværkskort.

**Bruges til:**
- Forstå netværksprotokoller i praksis
- Debugge netværksproblemer
- Se forskel på HTTP (klar tekst) og HTTPS (krypteret)

**Hvad du kan se:**
```
HTTP (ukrypteret):
  GET /api/users HTTP/1.1
  Host: example.com
  Authorization: Bearer eyJhbGciOiJIUzI1...  ← SYNLIG!

HTTPS (krypteret):
  [Encrypted Data]  ← Ingenting kan læses!
```

**Praktisk demonstration:** WireShark viser tydeligt HVORFOR HTTPS er nødvendigt — HTTP-trafik er fuldstændig læsbar for alle på samme netværk.

> WireShark bruges etisk til at forstå dit *eget* netværk. Aflytning af andres netværk uden tilladelse er ulovligt.

---

## Opsummering

| Begreb                     | Beskrivelse                                                                          |
|----------------------------|--------------------------------------------------------------------------------------|
| CIA-triaden                | De tre grundpilarer i sikkerhed: Fortrolighed, Integritet, Tilgængelighed            |
| Symmetrisk kryptering      | Én delt nøgle til kryptering og dekryptering — hurtig, men nøgleudveksling er problemet |
| AES                        | Industristandard for symmetrisk kryptering — bruges til selve datastrømmen i HTTPS  |
| Asymmetrisk kryptering     | Public/private key-par — kun private key kan dekryptere hvad public key krypterede  |
| RSA                        | Industristandard for asymmetrisk kryptering — langsom, bruges til nøgleudveksling   |
| TLS Handshake              | Bruger asymmetrisk kryptering til at aftale en symmetrisk session key sikkert       |
| HTTPS                      | HTTP + TLS — krypterer data, verificerer serverens identitet, sikrer integritet     |
| Certifikat                 | Indeholder serverens public key + identitet, signeret af en betroet CA              |
| CA (Certificate Authority) | Betroet tredjepart der signerer certifikater — fx Let's Encrypt                     |
| Digital signatur           | Bevis på autenticitet — skabt med private key, verificeres med public key           |
| 2FA                        | To-faktor: noget du ved (password) + noget du har (telefon) — dobbelt sikring       |
| WireShark                  | Netværksanalyseværktøj — synliggør HTTP (klar tekst) vs. HTTPS (krypteret)          |

---

## Opgaver til selvstudium

1. Beskriv CIA-triaden med egne ord og giv et konkret eksempel på hvert princip for en webapplikation
2. Forklar med egne ord: Hvorfor bruger HTTPS asymmetrisk kryptering til nøgleudveksling, men symmetrisk til selve datastrømmen?
3. Hvad er en CA (Certificate Authority), og hvad sker der i browseren hvis et certifikat er selvunderskrevet?
4. Forklar step-by-step hvad der sker under TLS Handshake — hvad sender klient og server til hinanden?
5. Hvad er en digital signatur? Hvordan bruges den til at bevise at en besked er autentisk og uændret?
6. Installer WireShark og sammenlign trafik for et HTTP-site vs. et HTTPS-site — hvad kan du aflæse i klartekst?
