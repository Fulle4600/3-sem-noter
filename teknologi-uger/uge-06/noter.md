# Uge 6 — TCP/IP Modellen & JSON

I denne uge introduceres den grundlæggende netværksmodel som bruges i moderne internetkommunikation: TCP/IP-modellen med 5 lag. Vi ser også på forskellen mellem TCP og UDP, Client/Server-modellen og JSON-formatet som bruges til dataudveksling.

---

## 1. TCP/IP Netværksmodellen (5 lag)

TCP/IP-modellen beskriver, hvordan data bevæger sig fra ét program på én computer til et program på en anden computer over et netværk. Modellen er opdelt i 5 lag, hvor hvert lag har sit eget ansvarsområde.

```
+-----------------------------+
|  5. Application Layer       |  <-- HTTP, FTP, DNS, SMTP
+-----------------------------+
|  4. Transport Layer         |  <-- TCP, UDP
+-----------------------------+
|  3. Network Layer           |  <-- IP, ICMP, routing
+-----------------------------+
|  2. Data Link Layer         |  <-- Ethernet, WiFi (MAC-adresser)
+-----------------------------+
|  1. Physical Layer          |  <-- Kabler, signaler, bits
+-----------------------------+
```

### Lag 1: Physical Layer (Fysisk lag)
- Håndterer den **fysiske transmission** af rå bits over et medie (kabel, fiber, trådløst).
- Definerer spænding, frekvens, stik-typer, kabeltyper.
- Eksempler: Ethernet-kabler (Cat5e, Cat6), fiber, WiFi-radiofrekvenser.
- Ingen "forståelse" af data — kun 0 og 1.

### Lag 2: Data Link Layer (Datalink-lag)
- Sender **frames** (datapakker med header og trailer) mellem to direkte forbundne noder.
- Bruger **MAC-adresser** (Media Access Control) — 48-bit hardware-adresser (f.eks. `AA:BB:CC:DD:EE:FF`).
- Håndterer fejldetektion på lavt niveau.
- Eksempler: Ethernet (IEEE 802.3), WiFi (IEEE 802.11).
- Switch opererer på dette lag.

### Lag 3: Network Layer (Netværkslag)
- Håndterer **routing** — sender pakker fra kilde til destination på tværs af netværk.
- Bruger **IP-adresser** (IPv4 / IPv6) til adressering.
- Router opererer på dette lag.
- Protokoller: IP (Internet Protocol), ICMP (ping), ARP.
- Ingen garanti for levering (best-effort delivery).

### Lag 4: Transport Layer (Transportlag)
- Ansvarlig for **end-to-end kommunikation** mellem processer.
- Tilføjer **portnumre** (0–65535) så vi ved, hvilket program pakken tilhører.
- To vigtige protokoller: **TCP** og **UDP** (se næste afsnit).
- Opdeler store beskeder i segmenter og genopretter dem hos modtageren.

### Lag 5: Application Layer (Applikationslag)
- Det lag som programmer (applikationer) kommunikerer på.
- Definerer **protokoller for specifikke tjenester**.
- Eksempler:
  - **HTTP/HTTPS** — webbrowsing
  - **FTP** — filoverførsel
  - **SMTP/IMAP** — e-mail
  - **DNS** — domænenavn til IP-opslag
  - **SSH** — sikker fjernbetjening

### Sammenligning: TCP/IP vs OSI-modellen

| TCP/IP (5 lag) | OSI-modellen (7 lag)         |
|----------------|------------------------------|
| Application    | Application + Presentation + Session |
| Transport      | Transport                    |
| Network        | Network                      |
| Data Link      | Data Link                    |
| Physical       | Physical                     |

> Bemærk: I praksis bruges TCP/IP-modellen. OSI bruges mest til undervisning og fejlsøgning som referencemodel.

---

## 2. TCP vs UDP

Begge er transportlagsprotokoller, men de har meget forskellige egenskaber.

### TCP (Transmission Control Protocol)

TCP er en **forbindelsesorienteret** protokol. Inden data sendes, oprettes en forbindelse via et **3-way handshake**:

```
Klient                    Server
  |  -- SYN -->             |
  |  <-- SYN-ACK --         |
  |  -- ACK -->             |
  |   (forbundet)           |
```

**Egenskaber:**
- **Pålidelig levering** — alle pakker ankommer, eller sendes igen.
- **Korrekt rækkefølge** — pakker nummereres og sættes i orden.
- **Flow control** — forhindrer at afsender overvælder modtager.
- **Congestion control** — tilpasser hastighed til netværkets kapacitet.
- Langsommere end UDP pga. overhead.

**Bruges til:**
- HTTP/HTTPS (webbrowsing)
- E-mail (SMTP, IMAP)
- FTP (filoverførsel)
- SSH

### UDP (User Datagram Protocol)

UDP er en **forbindelsesløs** protokol — sender bare pakker afsted uden forberedelse.

**Egenskaber:**
- **Ingen garanti** for levering.
- **Ingen rækkefølge** — pakker kan ankomme i forkert orden.
- **Ingen connection setup** — hurtigere opstart.
- Meget lavere overhead.
- Applikationen selv må håndtere tab, hvis nødvendigt.

**Bruges til:**
- DNS-opslag
- Live video/audio streaming
- Online spil
- VoIP (IP-telefoni)

### TCP vs UDP — Sammenligningstabel

| Egenskab            | TCP                   | UDP                  |
|---------------------|-----------------------|----------------------|
| Forbindelsestype    | Forbindelsesorienteret | Forbindelseslos      |
| Pålidelig levering  | Ja                    | Nej                  |
| Rækkefølge          | Garanteret            | Ikke garanteret      |
| Hastighed           | Langsommere           | Hurtigere            |
| Overhead            | Stor                  | Lille                |
| Brug                | HTTP, FTP, SSH        | DNS, streaming, spil |

---

## 3. Client/Server-modellen

Client/Server er det dominerende kommunikationsmønster på internettet.

```
  +--------+   Request (HTTP GET)    +---------+
  | Client | ----------------------> | Server  |
  |        | <---------------------- |         |
  +--------+   Response (200 OK)     +---------+
```

### Klient (Client)
- Initierer **altid** kommunikationen.
- Sender en **request** (forespørgsel) til serveren.
- Venter på et **svar (response)**.
- Eksempler: webbrowser, mobilapp, Postman, Python-script.

### Server
- Lytter passivt på et **portnummer** for indkommende forbindelser.
- Modtager requests og sender responses.
- Kan betjene mange klienter samtidigt.
- Kører typisk kontinuerligt (24/7).

### Vigtige portnumre

| Port | Protokol | Brug                    |
|------|----------|-------------------------|
| 80   | HTTP     | Usikret webtrafik       |
| 443  | HTTPS    | Krypteret webtrafik     |
| 22   | SSH      | Sikker fjernbetjening   |
| 21   | FTP      | Filoverførsel           |
| 25   | SMTP     | E-mail afsendelse       |
| 53   | DNS      | Domæneopslag            |
| 3306 | MySQL    | Databaseforbindelser    |

### Peer-to-Peer (P2P) — alternativ model
I modsætning til Client/Server kan alle noder i et P2P-netværk både være klient og server.
Eksempler: BitTorrent, blockchain-netværk.

---

## 4. JSON (JavaScript Object Notation)

JSON er et **letvægts dataformat** til udveksling af struktureret data. Det er menneskelæseligt og bruges overalt i moderne web-APIs.

### JSON Grundregler
- Data er organiseret som **nøgle-værdi-par**.
- Nøgler SKAL være **strenge i dobbelte anführselstegn**.
- Værdier kan have 6 forskellige typer.
- Ingen kommentarer er tilladt i JSON.

### JSON Datatyper

| Type      | Eksempel                           | Beskrivelse                          |
|-----------|------------------------------------|--------------------------------------|
| String    | `"Hello, verden"`                  | Tekst i dobbelte anf.tegn            |
| Number    | `42`, `3.14`, `-7`                 | Tal (heltal eller decimal)           |
| Boolean   | `true`, `false`                    | Sand/falsk                           |
| Null      | `null`                             | Ingen/tomværdi                      |
| Object    | `{ "navn": "Anna" }`               | Samling af nøgle-værdi-par           |
| Array     | `[1, 2, 3]`                        | Ordnet liste af værdier              |

### JSON Eksempel — En bruger

```json
{
  "id": 1,
  "navn": "Anna Larsen",
  "alder": 22,
  "aktiv": true,
  "email": null,
  "adresse": {
    "gade": "Nørregade 12",
    "by": "København",
    "postnummer": "1165"
  },
  "hobbyer": ["programmering", "løb", "musik"]
}
```

### JSON Array eksempel

```json
[
  { "id": 1, "navn": "Anna" },
  { "id": 2, "navn": "Bo" },
  { "id": 3, "navn": "Cecilie" }
]
```

### JSON i Java (parsing)

```java
// Med biblioteket org.json
import org.json.JSONObject;

String jsonString = "{\"navn\": \"Anna\", \"alder\": 22}";
JSONObject obj = new JSONObject(jsonString);

String navn = obj.getString("navn");  // "Anna"
int alder = obj.getInt("alder");      // 22
```

### JSON i JavaScript

```javascript
// Parse JSON-streng til objekt
const jsonStreng = '{"navn": "Anna", "alder": 22}';
const obj = JSON.parse(jsonStreng);
console.log(obj.navn); // "Anna"

// Konverter objekt til JSON-streng
const jsonOutput = JSON.stringify(obj);
console.log(jsonOutput); // '{"navn":"Anna","alder":22}'
```

### JSON vs XML

| Aspekt         | JSON                          | XML                            |
|----------------|-------------------------------|--------------------------------|
| Syntaks        | Kompakt, let at læse          | Verbose, tags åbner og lukker  |
| Datamasser     | Lettere                       | Tungere                        |
| Kommentarer    | Ikke understøttet             | Understøttet                   |
| Bruges i       | REST APIs, JavaScript         | SOAP APIs, konfigurationsfiler |

---

## 5. Opsummering

| Emne            | Nogleinformation                                              |
|-----------------|---------------------------------------------------------------|
| TCP/IP model    | 5 lag: Physical, Data Link, Network, Transport, Application   |
| TCP             | Forbindelsesorienteret, pålidelig, 3-way handshake            |
| UDP             | Forbindelseslos, hurtig, ingen garanti                        |
| Client/Server   | Klient initierer, server svarer, portnumre identificerer tjenester |
| JSON            | 6 datatyper, nøgle-værdi-par, bruges i REST APIs              |

---

## Opgaver til selvstudium

1. Tegn TCP/IP-modellens 5 lag uden at kigge, og skriv et eksempel på en protokol/teknologi for hvert lag.
2. Forklar med egne ord forskellen på TCP og UDP. Hvornår ville du vælge det ene frem for det andet?
3. Skriv et JSON-objekt der repræsenterer en studerende med: navn, studienummer, semester, liste af fag, og en adresse som et indlejret objekt.
4. Hvad er portnummeret for HTTPS? Hvad bruges port 53 til?
