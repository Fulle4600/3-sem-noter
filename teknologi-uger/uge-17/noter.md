# Uge 17 — Sikkerhed & Kryptering

Denne uge handler om kryptografi og de generelle sikkerhedsmål som kryptografi er designet til at opfylde. Kryptografi alene er ikke et mål — det er et *værktøj* til at realisere fire fundamentale sikkerhedsegenskaber: **confidentiality**, **message integrity**, **end-point authentication** og **operational security**.

```
Kryptografi som fundament:
  ┌─────────────────────────────────────────────────────┐
  │              Generelle sikkerhedsmål                │
  │  Confidentiality │ Integrity │ Authentication │ OpSec│
  └──────────────────────────────────────────────────────┘
              ↑ realiseres af ↑
  ┌─────────────────────────────────────────────────────┐
  │         Kryptografiske mekanismer                   │
  │    AES · RSA · SHA-256 · HMAC · Certifikater        │
  └─────────────────────────────────────────────────────┘
```

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

### Hvad er det?

**Samme nøgle bruges til at kryptere og dekryptere.** Afsender og modtager deler en hemmelig nøgle på forhånd — den der krypterer med nøglen, og den der dekrypterer, er de eneste der kan læse indholdet.

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
- Ekstremt hurtig — kan kryptere gigabytes pr. sekund
- Bruges til at kryptere selve data-indholdet i HTTPS, filer, databaser

### Hvornår bruges det?

Symmetrisk kryptering bruges når du skal kryptere **store mængder data hurtigt**:
- Al selve datatransmissionen i HTTPS (efter handshake)
- Kryptering af filer og databaser (BitLocker, VeraCrypt)
- VPN-tunneler (OpenVPN, WireGuard)
- Streaming og video (krypteret indhold på Netflix e.l.)

### Hvem bruger det?

| Aktør | Brug |
|---|---|
| Webbrowsere og servere | Krypterer HTTP-trafik efter TLS handshake |
| Operativsystemer | BitLocker/FileVault krypterer harddisken med AES |
| Udviklere | Krypterer følsomme data i databaser |
| VPN-udbydere | Krypterer al netværkstrafik i tunnelen |

### Hvorfor — og hvad er problemet?

**Fordel:** Ekstremt hurtig — egnet til store datamængder.

**Problem — nøgleudveksling:** Begge parter skal have *samme* nøgle. Hvordan deler du nøglen sikkert? Sender du den ukrypteret, kan en angriber opfange den og derefter dekryptere al fremtidig trafik.

```
Problem:
  Alice vil sende nøglen til Bob...
  ...men hvis hun sender den over internettet, kan Eve opfange den!
  → Eve kan nu dekryptere al kommunikation mellem Alice og Bob.
```

Dette løses ved at bruge **asymmetrisk kryptering** til at udveksle nøglen sikkert.

---

## Asymmetrisk Kryptering (Public Key Cryptography)

### Hvad er det?

**To matematisk sammenkoblede nøgler: en public key og en private key.** Det der krypteres med den ene nøgle kan *kun* dekrypteres med den anden.

```
Nøglepar:   Public Key  ←→  Private Key
             (Del med alle)    (Hemmelighed — forlad aldrig din enhed)
```

**De to regler:**
1. Krypteret med **public key** → kun **private key** kan dekryptere
2. Signeret med **private key** → **public key** kan verificere signaturen

```
KRYPTERING (fortrolighed):
  Afsender: "Hej Bank!" + Banks public key → krypteret besked
  Bank:     krypteret besked + Banks private key → "Hej Bank!"
  Eve opfanger krypteret besked men har ikke private key → læser ingenting

DIGITAL SIGNATUR (autenticitet):
  Server: hash(data) + servers private key → signatur
  Klient: signatur + servers public key → verificerer at serveren signerede det
  (Alle med public key kan verificere — men kun serveren kunne skabe signaturen)
```

**Eksempel — RSA algoritmen:**
- Baseret på at det er nemt at gange to store primtal, men næsten umuligt at faktorisere produktet
- 2048-bit eller 4096-bit nøglelængde
- Langsom — bruges KUN til nøgleudveksling og signaturer, ikke til bulk-kryptering

**Moderne alternativ — ECDH (Elliptic Curve Diffie-Hellman):**
- Bruges i TLS 1.3 og moderne systemer
- Samme sikkerhed som RSA men med meget kortere nøgler → hurtigere

### Hvornår bruges det?

Asymmetrisk kryptering bruges når to parter **ikke deler en hemmelig nøgle på forhånd**:
- **TLS handshake** — browser og server etablerer en fælles session key
- **Digitale signaturer** — software-signing, certifikater, e-mail (PGP/GPG)
- **SSH** — login på servere uden password via key pair
- **Krypterede e-mails** — PGP krypterer med modtagerens public key

### Hvem bruger det?

| Aktør | Brug |
|---|---|
| Webbrowsere | Verificerer serverens certifikat under TLS handshake |
| Servere | Har et certifikat (public key) signeret af en CA |
| Udviklere | SSH key pairs til at logge ind på servere |
| Software-udgivere | Signerer releases med private key — brugere verificerer med public key |
| Certificate Authorities | Signerer servercertifikater med deres private key |

### Hvorfor — og hvad er begrænsningen?

**Fordel:** Løser nøgleudvekslingsproblemet — du kan offentliggøre din public key frit, og alle kan sende dig krypterede beskeder uden at I har mødt hinanden før.

**Begrænsning:** Meget langsommere end symmetrisk kryptering — uegnet til at kryptere store datamængder direkte.

### Hybrid kryptering — det bedste fra begge verdener

I praksis kombineres de to typer. HTTPS er et eksempel:

```
1. Asymmetrisk fase (handshake):
   Klient og server bruger RSA/ECDH til at aftale en tilfældig session key
   → Session key overføres sikkert fordi den er krypteret med public key

2. Symmetrisk fase (datatransmission):
   Al efterfølgende kommunikation krypteres med den hurtige AES session key
   → Hastighed: gigabytes kan krypteres pr. sekund
```

| | Symmetrisk (AES) | Asymmetrisk (RSA/ECDH) |
|---|---|---|
| **Nøgler** | 1 delt hemmelig nøgle | Public key + Private key par |
| **Hastighed** | Meget hurtig (GB/s) | Langsom (KB/s) |
| **Nøgleudveksling** | Problem — kræver sikker kanal | Løst — public key er offentlig |
| **Hvad bruges det til** | Kryptere data (bulk) | Nøgleudveksling, signaturer |
| **Eksempel-algoritme** | AES-256 | RSA-2048, ECDH |
| **Hvornår** | Efter handshake — hele sessionen | Under handshake og signering |

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

## Confidentiality (Fortrolighed)

**Kun autoriserede parter kan læse data** — selv hvis en angriber opfanger beskeden, kan de ikke forstå indholdet.

### Hvordan opnås det?
Kryptering. Data omformes til ulæselig ciffertekst der kun kan dekrypteres med den rigtige nøgle.

```
Angriber opfanger:  a3f9c1d7e4b2...  ← Betyder ingenting uden nøglen
Modtager dekrypterer:  "Overfor 5000 kr. til konto 1234"
```

### Trusler mod confidentiality
- **Eavesdropping (aflytning)** — angriber opfanger pakker på netværket (f.eks. åbent WiFi)
- **Man-in-the-Middle (MitM)** — angriber placerer sig mellem klient og server
- **Data leaks** — ukrypterede databaser, logs der indeholder passwords

### Nøglemekanismer
| Mekanisme | Bruges til |
|---|---|
| AES (symmetrisk) | Kryptere selve data-indholdet — hurtig |
| RSA / ECDH (asymmetrisk) | Sikker nøgleudveksling |
| TLS | Krypterer al transport-lag kommunikation (HTTPS) |

> **Vigtig pointe:** Confidentiality siger intet om at data er korrekt eller at du taler med den rigtige — det er integrity og authentication's job.

---

## Message Integrity (Beskedintegritet)

**Garanti for at en besked ikke er blevet ændret undervejs** — hverken ved en fejl eller af en angriber.

### Hash-funktioner

En hash-funktion tager et input (data) og producerer et fast-størrelses fingeraftryk (hash/digest).

```
SHA-256("Hej verden")  →  a591a6d40bf420...  (256 bit / 64 hex-tegn)
SHA-256("Hej Verden")  →  6b86b273ff3424...  (totalt anderledes — et enkelt tegn ændrer alt)
```

**Egenskaber ved en kryptografisk hash:**
- **Envejs** — umuligt at gå fra hash tilbage til originaldata
- **Deterministisk** — samme input giver altid samme output
- **Avalanche effect** — én bit ændring → fuldstændig anderledes hash
- **Kollisionsresistent** — ekstremt svært at finde to inputs med samme hash

### HMAC (Hash-based Message Authentication Code)

En hash alene beviser ikke hvem der sendte beskeden — en angriber kan ændre data *og* genberegne hashen.

Løsningen er **HMAC**: hash beregnet med en hemmelig nøgle.

```
HMAC(nøgle, besked) → autentificeringskode

Afsender:  HMAC(K, "betal 100 kr") → a3f9...
Modtager:  HMAC(K, "betal 100 kr") → a3f9...  ✓ Match — autentisk og uændret

Angriber ændrer til "betal 10000 kr":
Modtager:  HMAC(K, "betal 10000 kr") → 7b2c... ✗ Mismatch — AFVIST!
```

Begge parter skal kende nøglen K — ellers kan ingen beregne den korrekte HMAC.

### Digital Signatur vs. HMAC

| | HMAC | Digital Signatur |
|---|---|---|
| **Nøgle** | Delt hemmelighed (symmetrisk) | Private key (asymmetrisk) |
| **Hvem kan verificere** | Kun dem med nøglen | Alle med public key |
| **Non-repudiation** | Nej | Ja — afsenderen kan ikke benægte |
| **Brug** | API-sikkerhed, TLS | Certifikater, software-signing |

### TLS og integrity

TLS bruger en **MAC (Message Authentication Code)** på hver pakke. Enhver ændring i transit opdages øjeblikkeligt og forbindelsen afbrydes.

---

## End-point Authentication (Endpoint-autentificering)

**Verificering af at du faktisk kommunikerer med den part du tror** — ikke en impostor eller MitM-angriber.

### Problemet

Kryptering alene er ikke nok. Hvis du opsætter en krypteret forbindelse til en angriber der udgiver sig for at være din bank, er din krypterede kommunikation stadig kompromitteret.

```
MitM-angreb uden authentication:
  Du  ←──krypteret──→  [Angriber]  ←──krypteret──→  Bank
  Du tror du taler med banken — men alt går igennem angriberen!
```

### Challenge-Response Authentication

En klassisk metode: bevis at du kender en hemmelighed uden at afsløre den.

```
1. Server sender en tilfældig nonce (tal kun brugt én gang): R = 42819
2. Klient beregner: HMAC(password, R) → f7a3c2...
3. Server beregner selv: HMAC(password, R) → f7a3c2...
4. Match! → Klienten kendte password-et
```

Noncen forhindrer **replay attacks** — en optaget besked fra tidligere kan ikke genbruges.

### Certifikat-baseret Authentication (TLS/HTTPS)

Serveren beviser sin identitet med et digitalt certifikat:

```
1. Server sender certifikat: { domæne: "mybank.dk", public_key: ... }
2. Certifikatet er signeret af en CA (Certificate Authority)
3. Browser tjekker: Er CA'en i min betroede liste? Er certifikatet gyldigt (ikke udløbet)?
4. Browser verificerer signatur med CA's public key
5. Ja → Du taler med mybank.dk
```

**Certificate chain (tillids-kæde):**
```
Root CA (selv-signeret, forudinstalleret i OS/browser)
  └── Intermediate CA (signeret af Root CA)
        └── Serverens certifikat (signeret af Intermediate CA)
```

### Mutual TLS (mTLS)

Normalt er det kun *serveren* der autentificerer sig. I **mTLS** skal *begge* parter bevise deres identitet:

```
Server → Klient:  Her er mit certifikat
Klient → Server:  Her er MIT certifikat (klientens eget)
```

Bruges i: microservice-kommunikation, banking-APIs, zero-trust netværk.

### Andre autentificeringsmetoder

| Metode | Princip | Eksempel |
|---|---|---|
| **Password** | Noget du ved | Login-formular |
| **Certifikat** | Noget du har (private key) | TLS client cert |
| **Token** | Noget du har (OTP) | Google Authenticator |
| **Biometri** | Noget du er | Fingeraftryk, Face ID |
| **2FA/MFA** | Kombination | Password + SMS-kode |

> **End-point authentication** løser "hvem er jeg ved at tale med?". Integrity løser "er beskeden ændret?". Confidentiality løser "kan nogen læse det?". Alle tre er nødvendige.

---

## Operational Security (OpSec)

**OpSec handler om de menneskelige og proceduremæssige aspekter af sikkerhed** — teknisk kryptering hjælper ikke hvis brugerne opfører sig usikkert.

### Kerneprincippet: Mindste Privilegium (Least Privilege)

Giv kun adgang til præcis det der er nødvendigt for at udføre opgaven — intet mere.

```
Dårligt:  Alle udviklere har fuld adgang til produktionsdatabasen
Godt:     Kun deploy-systemet har skriveadgang — udviklere har læseadgang til logs
```

### Angrebsoverflade (Attack Surface)

Jo mere du eksponerer, jo større er risikoen. Reducer angrebsoverfladen:
- Luk porte der ikke bruges
- Deaktiver features der ikke er i brug
- Fjern ubrugte brugerkonti og API-nøgler

### Defense in Depth (Lagdelt forsvar)

Ét sikkerhedslag er ikke nok — brug *mange* lag så én fejl ikke kompromitterer hele systemet.

```
Lag 1:  Firewall — bloker uønsket trafik
Lag 2:  Authentication — verificer identitet
Lag 3:  Authorization — tjek rettigheder
Lag 4:  Kryptering — beskyt data i transit og hvile
Lag 5:  Logging/monitoring — opdage angreb
Lag 6:  Backup — gendannelse ved kompromittering
```

### Praktiske OpSec-vaner

- **Stærke, unikke passwords** — brug en password manager (Bitwarden, 1Password)
- **To-faktor autentificering (2FA)** — noget du ved + noget du har; foretrækker authenticator-app over SMS
- **Opdatér software** — patches lukker kendte CVE'er (Common Vulnerabilities and Exposures)
- **Phishing-bevidsthed** — verificer afsender, hover over links, ring op ved tvivl
- **Kryptér følsomme data i hvile** — ikke kun i transit (fuld-disk kryptering, krypterede backups)
- **Incident response plan** — hvad gør du NÅR (ikke hvis) du bliver kompromitteret?

### Social Engineering

Den svageste del af ethvert sikkerhedssystem er mennesket. Angribere udnytter dette:

| Teknik | Beskrivelse |
|---|---|
| **Phishing** | Falsk email der ligner en betroet afsender |
| **Spear phishing** | Målrettet phishing mod en specifik person |
| **Pretexting** | Angriber udgiver sig for at være IT-support e.l. |
| **Baiting** | USB-stik efterladt på en parkeringsplads |

> **Konklusion:** OpSec er ikke kun IT's ansvar — det er alle i organisationens ansvar. En god sikkerhedskultur er vigtigere end det bedste tekniske setup.

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

### De fire generelle sikkerhedsmål (kryptografiens formål)

| Sikkerhedsmål | Spørgsmålet det besvarer | Primær mekanisme |
|---|---|---|
| **Confidentiality** | Kan nogen aflytte og læse vores kommunikation? | Kryptering (AES, TLS) |
| **Message Integrity** | Er beskeden ændret undervejs? | Hash, HMAC, digital signatur |
| **End-point Authentication** | Taler jeg med den rigtige? | Certifikater, challenge-response, 2FA |
| **Operational Security** | Er vores processer og adfærd sikre? | Policies, mindste privilegium, awareness |

### Kryptografiske byggesten

| Begreb                     | Beskrivelse                                                                          |
|----------------------------|--------------------------------------------------------------------------------------|
| CIA-triaden                | De tre grundpilarer: Fortrolighed, Integritet, Tilgængelighed                        |
| Symmetrisk kryptering      | Én delt nøgle til kryptering og dekryptering — hurtig, men nøgleudveksling er problemet |
| AES                        | Industristandard for symmetrisk kryptering — bruges til selve datastrømmen i HTTPS  |
| Asymmetrisk kryptering     | Public/private key-par — kun private key kan dekryptere hvad public key krypterede  |
| RSA                        | Industristandard for asymmetrisk kryptering — langsom, bruges til nøgleudveksling   |
| SHA-256                    | Kryptografisk hash-funktion — producerer 256-bit fingeraftryk af data               |
| HMAC                       | Hash med hemmelig nøgle — beviser at besked er autentisk og uændret                 |
| Digital signatur           | Bevis på autenticitet — skabt med private key, verificeres med public key           |
| TLS Handshake              | Bruger asymmetrisk kryptering til at aftale en symmetrisk session key sikkert       |
| HTTPS                      | HTTP + TLS — opfylder confidentiality, integrity og authentication i ét             |
| Certifikat + CA            | Serverens public key signeret af betroet tredjepart — grundlaget for web-authentication |
| Nonce                      | Tilfældig engangskode — forhindrer replay attacks i authentication-protokoller      |
| mTLS                       | Mutual TLS — begge parter autentificerer sig med certifikater                       |
| 2FA                        | To-faktor: noget du ved + noget du har — forstærker end-point authentication        |
| WireShark                  | Netværksanalyseværktøj — synliggør HTTP (klar tekst) vs. HTTPS (krypteret)          |

---

## Opgaver til selvstudium

1. Beskriv de fire generelle sikkerhedsmål (confidentiality, integrity, authentication, OpSec) og giv et eksempel på et angreb som hvert mål beskytter mod.
2. Forklar med egne ord: Hvad er forskellen på HMAC og en digital signatur — hvornår bruger man hvad?
3. Forklar challenge-response authentication: Hvad er en nonce, og hvorfor forhindrer den replay attacks?
4. Hvorfor er kryptering alene ikke tilstrækkeligt til sikker kommunikation? Hvad mangler?
5. Hvad er mTLS og i hvilke situationer er det relevant?
6. Beskriv et scenarie hvor god OpSec kan afbøde et angreb som kryptering ikke kan forhindre.
