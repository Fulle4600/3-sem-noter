# Uge 7 — IP Adresser & Protokoller

I denne uge dykker vi ned i IP-adresser — den fundamentale mekanisme der muliggør kommunikation på internettet. Vi ser på IPv4 og IPv6, forskellen på offentlige og private adresser, NAT-teknologien og hvad et protokolbegreb egentlig dækker over.

---

## 1. IPv4 — Internet Protocol version 4

### Hvad er en IP-adresse?
En IP-adresse er en **unik numerisk identifikation** for en enhed på et netværk. Det er som en postadresse — den fortæller, hvor pakker skal sendes hen.

### IPv4 Format
IPv4-adressen er **32 bits** lang og skrives som 4 tal (oktetter) adskilt af punktummer:

```
192  .  168  .  1  .  100
 ^       ^      ^     ^
 8 bit   8 bit  8 bit 8 bit  =  32 bit i alt
```

Hvert oktet kan have en værdi fra **0 til 255** (2^8 = 256 mulige værdier).

**Eksempler:**
- `192.168.1.1` — typisk hjemmeruter
- `10.0.0.1` — privat netværk
- `8.8.8.8` — Googles DNS-server (offentlig)
- `127.0.0.1` — loopback (localhost, din egen maskine)

### IPv4 Adresseklasser (klassisk opdeling)

Historisk set var IPv4-adresser opdelt i klasser:

| Klasse | Første oktet | Eksempel       | Netværk-del | Host-del | Antal hosts   |
|--------|--------------|----------------|-------------|----------|---------------|
| A      | 1 – 126      | 10.0.0.1       | 8 bit       | 24 bit   | ~16 millioner |
| B      | 128 – 191    | 172.16.0.1     | 16 bit      | 16 bit   | ~65.000       |
| C      | 192 – 223    | 192.168.1.1    | 24 bit      | 8 bit    | 254           |
| D      | 224 – 239    | (multicast)    | —           | —        | —             |
| E      | 240 – 255    | (reserveret)   | —           | —        | —             |

> Bemærk: Klassebaseret adressering bruges ikke længere i praksis — i dag bruges CIDR (Classless Inter-Domain Routing).

### Subnet Mask og CIDR

En **subnet mask** bestemmer, hvilken del af IP-adressen der er netværksdel og hvilken der er hostdel.

```
IP-adresse:   192.168.1.100
Subnet mask:  255.255.255.0
```

I CIDR-notation skrives det som: `192.168.1.100/24`

`/24` betyder at de første 24 bits er netværksdelen (svarende til `255.255.255.0`).

**Almindelige CIDR-masker:**

| CIDR | Subnet Mask     | Antal hosts |
|------|-----------------|-------------|
| /8   | 255.0.0.0       | 16.777.214  |
| /16  | 255.255.0.0     | 65.534      |
| /24  | 255.255.255.0   | 254         |
| /25  | 255.255.255.128 | 126         |
| /30  | 255.255.255.252 | 2           |

### Specielle IPv4-adresser

| Adresse        | Betydning                                      |
|----------------|------------------------------------------------|
| `127.0.0.1`    | Loopback — din egen maskine                    |
| `0.0.0.0`      | Uspecificeret / alle grænseflader              |
| `255.255.255.255` | Broadcast — alle på netværket              |
| `169.254.x.x`  | APIPA — automatisk tildelt hvis DHCP fejler    |

---

## 2. Private vs. Offentlige IP-adresser

### Private adresserum (RFC 1918)

Private adresser bruges **internt** i et netværk og kan ikke rutes direkte på internettet.

| Klasse | Adresserum                   | Eksempel        |
|--------|------------------------------|-----------------|
| A      | `10.0.0.0` – `10.255.255.255`| `10.0.0.1`      |
| B      | `172.16.0.0` – `172.31.255.255` | `172.16.0.1` |
| C      | `192.168.0.0` – `192.168.255.255` | `192.168.1.1` |

**Brug:**
- Hjemmenetværk: `192.168.1.x`
- Virksomhedsnetværk: `10.x.x.x`

### Offentlige adresser

Offentlige adresser er **unikt tildelt på internettet** via IANA (Internet Assigned Numbers Authority) og regionale registre (RIPE i Europa).

Eksempler på offentlige adresser: `8.8.8.8`, `1.1.1.1`, `93.184.216.34`.

---

## 3. NAT — Network Address Translation

### Problem
Der er kun ca. 4,3 milliarder IPv4-adresser (2^32). Dette er ikke nok til alle enheder i verden.

### Løsning: NAT
NAT gør det muligt for mange enheder med **private adresser** at dele én **offentlig IP-adresse**.

```
[Laptop 192.168.1.10]  \
[Telefon 192.168.1.11] ---> [Router NAT] ---> Internet (87.45.12.33)
[Tablet 192.168.1.12]  /
```

### Sådan virker NAT

1. Laptop sender request til `8.8.8.8` fra `192.168.1.10:54321`.
2. Router erstatter source-IP med sin offentlige IP: `87.45.12.33:54321`.
3. Router gemmer mapping i NAT-tabel.
4. Svar returneres til `87.45.12.33:54321`.
5. Router slår op i NAT-tabel og videresender til `192.168.1.10`.

### NAT-tabel eksempel

| Intern IP:Port       | Ekstern IP:Port       | Destination         |
|----------------------|-----------------------|---------------------|
| 192.168.1.10:54321   | 87.45.12.33:54321     | 8.8.8.8:80          |
| 192.168.1.11:12345   | 87.45.12.33:12345     | 93.184.216.34:443   |

**Fordele ved NAT:**
- Sparer IPv4-adresser
- Tilbyder et lag af sikkerhed (interne IP'er skjules)

**Ulemper ved NAT:**
- Komplicerer indkommende forbindelser (port forwarding nødvendigt)
- Bryder end-to-end forbindelsesmodellen

---

## 4. IPv6 — Internet Protocol version 6

### Hvorfor IPv6?
IPv4 har ca. 4,3 milliarder adresser — ikke nok til fremtidens behov. IPv6 løser dette.

### IPv6 Format
IPv6-adressen er **128 bits** lang og skrives som 8 grupper af 4 hexadecimale cifre, adskilt af koloner:

```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

**Forkortningsregler:**
1. Ledende nuller i en gruppe kan udelades: `0db8` → `db8`
2. En eller flere grupper af kun nuller kan erstattes med `::` (kun én gang!)

```
Fuld:        2001:0db8:0000:0000:0000:0000:0000:0001
Forkortet:   2001:db8::1
```

### IPv6 Adressetyper

| Type       | Beskrivelse                               | Eksempel          |
|------------|-------------------------------------------|-------------------|
| Unicast    | En specifik enhed                         | `2001:db8::1`     |
| Multicast  | Gruppe af enheder                         | `ff02::1`         |
| Anycast    | Nærmeste enhed i en gruppe                | —                 |
| Loopback   | Din egen maskine                          | `::1`             |
| Link-local | Kun på lokalt netværk                     | `fe80::1`         |

### IPv4 vs IPv6 sammenligning

| Egenskab         | IPv4                   | IPv6                          |
|------------------|------------------------|-------------------------------|
| Adresselængde    | 32 bit                 | 128 bit                       |
| Antal adresser   | ~4,3 milliarder        | ~340 undecillioner            |
| Notation         | Decimal, punktum       | Hexadecimal, kolon            |
| NAT              | Nødvendigt             | Ikke nødvendigt               |
| Header           | 20 bytes minimum       | 40 bytes (simplere format)    |
| Konfiguration    | Manuel eller DHCP      | SLAAC (automatisk)            |
| Sikkerhed        | Valgfrit (IPsec)       | Indbygget understøttelse      |

---

## 5. Protokol-begrebet

### Hvad er en protokol?
En **protokol** er et sæt af **regler og standarder** der definerer, hvordan to parter kommunikerer. Ligesom der er regler for, hvordan man skriver et brev (adresse øverst, hilsen, underskrift), er der protokoller for netværkskommunikation.

En protokol definerer typisk:
- **Syntaks** — formatet af beskeder (f.eks. bytes i en bestemt rækkefølge)
- **Semantik** — betydningen af hvert felt
- **Timing** — hvornår og hvor hurtigt der sendes

### Eksempler på protokoller

| Lag             | Protokol    | Formål                              |
|-----------------|-------------|-------------------------------------|
| Application     | HTTP        | Webbrowsing                         |
| Application     | SMTP        | E-mail afsendelse                   |
| Application     | DNS         | Domænenavn-opslag                   |
| Transport       | TCP         | Pålidelig levering                  |
| Transport       | UDP         | Hurtig, upålidelig levering         |
| Network         | IP          | Routing af pakker                   |
| Network         | ICMP        | Fejlmeddelelser (ping)              |
| Data Link       | ARP         | IP til MAC-adresse opslag           |
| Data Link       | Ethernet    | Lokal netværkskommunikation         |

### Protokolstakken i praksis

Når du besøger `www.google.com`:

```
Browser         → HTTP GET request
HTTP            → Indkapsler i TCP segment (port 443)
TCP             → Indkapsler i IP pakke (til Googles IP)
IP              → Indkapsler i Ethernet frame (til router MAC)
Ethernet        → Sender bits over kabel/WiFi
```

Hos Google sker det omvendte — "afpakning" lag for lag.

---

## 6. Routing Basics

### Hvad er routing?
Routing er processen, hvor **routere** beslutter, hvilken vej en datapakke skal tage for at nå sin destination.

### Default Gateway
Din computers **default gateway** er typisk IP-adressen på din hjemmeruter. Alle pakker til adresser uden for dit lokale netværk sendes til default gateway.

```
Din PC: 192.168.1.10
Default gateway: 192.168.1.1 (routeren)

Sender du til 8.8.8.8?
→ 8.8.8.8 er IKKE i 192.168.1.0/24
→ Send til default gateway 192.168.1.1
→ Routeren finder videre vej til 8.8.8.8
```

### Routing Table
Routere har en **routing tabel** der viser, hvilke netværk de kender til og via hvilken "next hop":

```
Destination      Gateway         Interface
0.0.0.0/0        87.45.12.1      eth0 (default route)
192.168.1.0/24   directly        eth1 (lokalt netværk)
10.0.0.0/8       10.1.1.1        eth2
```

### Traceroute
`traceroute` (Windows: `tracert`) viser stien pakker tager over netværket:

```bash
tracert www.google.com
```

Output viser hvert "hop" (router) med IP-adresse og svartid.

---

## 7. ICMP og Ping

**ICMP (Internet Control Message Protocol)** bruges til netværksdiagnostik.

**ping** sender ICMP Echo Request og venter på Echo Reply:

```bash
ping 8.8.8.8
# Output:
# Reply from 8.8.8.8: bytes=32 time=14ms TTL=57
```

**TTL (Time To Live):** Et tal der decrementeres ved hvert hop. Nul → pakken kasseres. Forhindrer pakker i at cirkulere evigt.

---

## 8. Opsummering

| Emne           | Nogleinformation                                              |
|----------------|---------------------------------------------------------------|
| IPv4           | 32-bit, 4 oktetter, ~4,3 mia. adresser                       |
| Subnet mask    | Definerer netværks- og host-del, CIDR-notation (/24 osv.)    |
| Private IP     | 10.x, 172.16-31.x, 192.168.x — ikke rutbare på internet      |
| NAT            | Mange private IP'er deler én offentlig IP                    |
| IPv6           | 128-bit, hexadecimal, enormt adresserum, ingen NAT nødvendig |
| Protokol       | Regler for kommunikation — syntax, semantik, timing          |
| Routing        | Routere videresender pakker, default gateway = din ruter      |

---

## Opgaver til selvstudium

1. Hvad er din maskines IP-adresse, subnet mask og default gateway? (Brug `ipconfig` på Windows)
2. Er din IP-adresse privat eller offentlig? Hvad er din offentlige IP? (Besøg whatismyip.com)
3. Beregn: Hvor mange hosts kan der være på et /26 netværk?
4. Skriv den fulde IPv6-adresse `2001:db8::1` ud som den fuldstændige version med alle nuller.
5. Kør `ping 8.8.8.8` og forklar hvad TTL-værdien i svaret betyder.
