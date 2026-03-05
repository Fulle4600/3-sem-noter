# Uge 18 — DNS, DHCP & Begrebsopsamling

Denne uge runder vi de sidste netværksprotokoller af: DNS (der oversætter domænenavne til IP-adresser) og DHCP (der automatisk konfigurerer enheder på et netværk). Vi bruger også ugen til at samle alle semesterets begreber.

---

## DNS — Domain Name System

### Hvad er DNS?

DNS er internettets "telefonbog". Det oversætter menneskevenlige domænenavne som `google.com` til IP-adresser som `142.250.185.46`, som computere faktisk bruger til at finde hinanden.

**Uden DNS:** Du skulle huske IP-adressen til hvert website du vil besøge.
**Med DNS:** Du skriver `google.com` — DNS finder IP-adressen for dig.

---

### DNS Hierarki

DNS er organiseret som et hierarkisk træ:

```
                    . (Root)
                   /    \    \
                .com    .dk   .org
               /    \      \
          google    apple  zealand
            /
          mail (mail.google.com)
```

| Niveau | Eksempel | Hvem styrer det |
|---|---|---|
| **Root** | `.` (usynlig) | ICANN / root server operatører |
| **Top-Level Domain (TLD)** | `.com`, `.dk`, `.org` | Registries (Verisign for .com, DK Hostmaster for .dk) |
| **Second-Level Domain** | `google`, `zealand` | Den der har registreret domænet |
| **Subdomain** | `mail`, `www`, `api` | Domæneejeren |

---

### DNS Opslag — step by step

Hvad sker der når du skriver `www.zealand.dk` i browseren?

```
Browser → DNS Resolver (din router/ISP) → Root Server → TLD Server → Authoritative Server

1. Browser: "Hvad er IP for www.zealand.dk?"
2. DNS Resolver: Tjekker sin cache → ikke fundet
3. DNS Resolver → Root Server: "Hvem kender .dk?"
   Root Server: "Spørg .dk TLD serveren på 193.163.102.1"
4. DNS Resolver → .dk TLD Server: "Hvem kender zealand.dk?"
   TLD Server: "Spørg Zealands authoritative DNS server på 185.14.48.1"
5. DNS Resolver → Zealands DNS Server: "Hvad er IP for www.zealand.dk?"
   Zealands DNS: "Det er 185.14.48.10"
6. DNS Resolver: Gem svaret i cache (TTL: 3600 sekunder)
7. DNS Resolver → Browser: "www.zealand.dk = 185.14.48.10"
8. Browser forbinder til 185.14.48.10
```

**TTL (Time To Live):** Hvor længe svaret caches. Lav TTL = ændringer slår hurtigt igennem. Høj TTL = færre DNS-opslag, men langsommere propagering ved ændringer.

---

### DNS Record Typer

| Record | Bruges til | Eksempel |
|---|---|---|
| **A** | Domæne → IPv4 adresse | `google.com → 142.250.185.46` |
| **AAAA** | Domæne → IPv6 adresse | `google.com → 2a00:1450:400f:...` |
| **CNAME** | Alias til et andet domæne | `www.example.com → example.com` |
| **MX** | Mail server for domænet | `example.com → mail.example.com` |
| **TXT** | Fritekst (bruges til verifikation, SPF) | `"v=spf1 include:..."` |
| **NS** | Hvilke navneservere er autoritative | `example.com NS → ns1.example.com` |

---

### DNS i praksis

```bash
# Slå IP op for et domæne (Windows/Linux)
nslookup google.com
ping google.com

# Se alle DNS records
nslookup -type=MX gmail.com
```

**Hosts-filen:** Lokal override af DNS på din computer (`C:\Windows\System32\drivers\etc\hosts`). Bruges fx til at blokere websites eller teste lokalt:
```
127.0.0.1   localhost
127.0.0.1   myapp.local   # Peger myapp.local på din lokale maskine
```

---

## DHCP — Dynamic Host Configuration Protocol

### Hvad er DHCP?

DHCP er protokollen der **automatisk tildeler IP-konfiguration** til enheder der forbinder sig til et netværk.

**Uden DHCP:** Hver enhed skal manuelt konfigureres med IP, subnet mask, gateway og DNS. Upraktisk og fejlbehæftet.
**Med DHCP:** Tilslut til WiFi → får automatisk al nødvendig konfiguration på få sekunder.

---

### Hvad DHCP tildeler

| Konfiguration | Eksempel | Beskrivelse |
|---|---|---|
| **IP-adresse** | 192.168.1.105 | Din unikke adresse på det lokale netværk |
| **Subnet Mask** | 255.255.255.0 | Definerer netværkets størrelse |
| **Default Gateway** | 192.168.1.1 | Routerens IP — vejen ud til internettet |
| **DNS Server** | 8.8.8.8 | Hvilken DNS server du skal bruge |
| **Lease Time** | 86400 sekunder | Hvor længe du "låner" IP-adressen |

---

### DHCP DORA — den 4-trins proces

```
Klient (ny enhed)                    DHCP Server (router)
      |                                      |
      |----DISCOVER (broadcast) ----------→  |
      |    "Er der en DHCP server? Jeg       |
      |     har brug for en IP-adresse"      |
      |                                      |
      |  ←------OFFER ----------------------|
      |    "Jeg tilbyder dig 192.168.1.105   |
      |     (gyldig i 24 timer)"             |
      |                                      |
      |----REQUEST (broadcast) ----------→   |
      |    "Jeg vil gerne have               |
      |     192.168.1.105"                   |
      |                                      |
      |  ←------ACKNOWLEDGE ----------------|
      |    "Bekræftet! 192.168.1.105 er din  |
      |     i 86400 sekunder"                |
```

**D**iscover → **O**ffer → **R**equest → **A**cknowledge = **DORA**

---

### DHCP Lease Renewal

IP-adressen er ikke permanent — den "lejes" for en periode (lease time):

- Halvvejs igennem lease-perioden forsøger klienten at forny
- Hvis serveren ikke svarer, forsøger klienten igen ved 87,5% af lejeperioden
- Udløber lejen helt uden fornyelse, mister klienten IP-adressen

**Praktisk konsekvens:** Din laptop kan få en ny IP-adresse næste gang den forbinder til samme WiFi.

---

### DHCP Reservation (Statisk DHCP)

Du kan konfigurere DHCP til *altid* at give en specifik enhed den samme IP-adresse, baseret på dens MAC-adresse:

```
MAC: AA:BB:CC:DD:EE:FF  →  Altid IP: 192.168.1.50
```

Bruges til: printere, NAS, servere — enheder andre skal kunne nå på en fast adresse.

---

## Begrebsopsamling — Hele Semesteret

### Netværkslag og protokoller

| Lag | Protokol | Funktion |
|---|---|---|
| Application (5) | HTTP, HTTPS, DNS, FTP | Din applikation kommunikerer her |
| Transport (4) | TCP, UDP | Pålidelig/hurtig levering |
| Network (3) | IP | Routing og adressering |
| Data Link (2) | Ethernet, WiFi | Lokal netværksforbindelse |
| Physical (1) | Kabler, radio | Fysiske bits |

### REST API oversigt

```
GET    /api/products         → Hent alle
POST   /api/products         → Opret ny
GET    /api/products/42      → Hent én
PUT    /api/products/42      → Erstat helt
PATCH  /api/products/42      → Opdater delvist
DELETE /api/products/42      → Slet
```

### HTTP Status koder

| Kode | Kategori | Typisk årsag |
|---|---|---|
| 200 OK | Success | GET lykkes |
| 201 Created | Success | POST lykkes |
| 204 No Content | Success | DELETE lykkes |
| 400 Bad Request | Client fejl | Forkert input |
| 401 Unauthorized | Client fejl | Ikke logget ind |
| 403 Forbidden | Client fejl | Ikke adgang |
| 404 Not Found | Client fejl | Ressource findes ikke |
| 500 Internal Server Error | Server fejl | Din kode crashede |

### Sikkerhed quick-reference

| Koncept | Huskeregel |
|---|---|
| **Symmetrisk** | Én nøgle, hurtig, problem: nøgleudveksling |
| **Asymmetrisk** | Public/private par, langsom, løser nøgleudveksling |
| **TLS** | Kombination: asymmetrisk til nøgleudveksling, symmetrisk til data |
| **JWT** | Header.Payload.Signature — signeret men IKKE krypteret |
| **HTTPS** | HTTP + TLS = krypteret + autentificeret |

---

## Opgaver til selvstudium

1. Brug `nslookup` (Windows) til at slå IP-adressen op for `zealand.dk` — hvilken IP returneres, og hvilken type record er det?
2. Forklar DORA-processen med egne ord: hvad sender klient og server i hvert trin, og hvad opnår de?
3. Hvad er forskellen på en DHCP Lease og en DHCP Reservation? Hvornår ville du bruge en Reservation?
4. Hvad er TTL i DNS-sammenhæng? Hvad sker der hvis du ændrer en DNS A-record, men TTL er sat til 86400 sekunder?
5. Kig i hosts-filen på din computer (`C:\Windows\System32\drivers\etc\hosts`) — hvad indeholder den, og hvad bruges den til?
6. Forklar med egne ord: Hvad er forskellen på DNS og DHCP? Hvad ville gå i stykker hvis din DHCP-server fejlede — og hvad hvis din DNS-server fejlede?
