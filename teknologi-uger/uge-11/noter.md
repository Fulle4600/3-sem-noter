# Uge 11 — JWT Authentication

Denne uge handler om autentificering i REST APIs med JWT (JSON Web Tokens). Vi ser på tokensstruktur, hvordan tokens skabes og valideres, og det overordnede flow for sikker adgangsstyring i moderne webapplikationer.

---

## 1. Autentificering vs. Autorisering

Disse to begreber forveksles ofte:

| Begreb           | Spørgsmål            | Eksempel                                    |
|------------------|----------------------|---------------------------------------------|
| **Autentificering** | Hvem er du?       | Log ind med brugernavn + password           |
| **Autorisering**    | Hvad må du?       | Bruger må se egne data, admin må se alle    |

```
1. Autentificering: "Jeg er Anna, her er mit password"
   → Server verificerer: JA, det er Anna (201 token udstedt)

2. Autorisering: "Jeg vil se bruger id=5 data"
   → Server tjekker: Er Anna tilladt dette? (kan Anna se id=5?)
```

---

## 2. Sessionbaseret vs. Tokenbaseret Authentication

### Sessionbaseret (gammeldags)

```
1. Bruger logger ind (username + password)
2. Server opretter session i database med sessionId
3. Server sender sessionId som cookie til klienten
4. Klienten sender cookie med i hvert request
5. Server slår sessionId op i database for hvert request
```

**Problem:** Serveren skal gemme sessions — svært at skalere (load balancing).

### Tokenbaseret (JWT)

```
1. Bruger logger ind (username + password)
2. Server genererer JWT med brugerdata og underskriver det
3. Server sender JWT til klienten
4. Klienten gemmer token og sender det med i Authorization-header
5. Server validerer token's signatur — INGEN database opslag nødvendigt!
```

**Fordel:** Serveren er stateless — nemt at skalere!

---

## 3. JWT Struktur

Et JWT (JSON Web Token) er en **streng opdelt i 3 dele** adskilt af punktummer:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmF2biI6IkFubmEiLCJyb2xlIjoiVVNFUiIsImV4cCI6MTcwODAwMDAwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
^                                      ^                                                                                             ^
|                                      |                                                                                             |
HEADER (Base64)                        PAYLOAD (Base64)                                                                          SIGNATURE
```

### Del 1: Header

Indeholder metadata om token-typen og signeringsalgoritmen.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg`: Algoritme til signatur (HS256 = HMAC SHA-256, RS256 = RSA SHA-256)
- `typ`: Type = JWT

### Del 2: Payload (Claims)

Indeholder **claims** — påstande om brugeren og tokenet.

```json
{
  "sub": "1234567890",
  "navn": "Anna Larsen",
  "email": "anna@example.com",
  "role": "USER",
  "iat": 1708000000,
  "exp": 1708003600
}
```

**Standardiserede claims (Registered Claims):**

| Claim | Navn              | Beskrivelse                                  |
|-------|-------------------|----------------------------------------------|
| `sub` | Subject           | Hvem tokenet handler om (typisk bruger-ID)   |
| `iss` | Issuer            | Hvem har udstedt tokenet                     |
| `aud` | Audience          | Hvem tokenet er tiltænkt                     |
| `exp` | Expiration Time   | Hvornår tokenet udløber (Unix timestamp)     |
| `iat` | Issued At         | Hvornår tokenet blev udstedt                 |
| `nbf` | Not Before        | Tokenet er ikke gyldigt før dette tidspunkt  |
| `jti` | JWT ID            | Unikt ID for dette token                     |

**Custom claims** — du kan tilføje dine egne:
```json
{
  "bruger_id": 42,
  "rolle": "ADMIN",
  "organisation": "DFI"
}
```

> VIGTIGT: Payload er kun Base64-encoded, IKKE krypteret! Enhver kan dekode og læse indholdet. Læg ALDRIG passwords eller fortrolige data i payload.

### Del 3: Signature

Signaturen sikrer, at tokenet ikke er blevet manipuleret.

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  hemmelighed
)
```

Kun serveren kender `hemmelighed` — derfor kan kun serveren lave og validere signatures.

---

## 4. Base64 og Base64URL Encoding

JWT bruger **Base64URL** encoding (en variant af Base64 der er URL-sikker).

### Hvad er Base64?

Base64 konverterer binære data til en streng der kun indeholder ASCII-tegn (A-Z, a-z, 0-9, +, /). Det bruges til at transportere binære data som tekst.

```
Input:  { "alg": "HS256" }
Output: eyJhbGciOiJIUzI1NiJ9
```

**Base64URL** erstatter `+` med `-` og `/` med `_` for at gøre det URL-sikkert.

### Dekod et JWT

Du kan dekode JWT-payload manuelt på jwt.io:
1. Gå til https://jwt.io
2. Indsæt dit token
3. Se decoded header, payload og signaturverifikation

---

## 5. JWT Authentication Flow

```
+----------+                        +----------+                    +----------+
|  Klient  |                        |  Server  |                    |    DB    |
+----------+                        +----------+                    +----------+
     |                                   |                               |
     | POST /auth/login                  |                               |
     | { email, password }               |                               |
     |---------------------------------->|                               |
     |                                   | SELECT bruger WHERE email=... |
     |                                   |------------------------------>|
     |                                   | Bruger fundet                 |
     |                                   |<------------------------------|
     |                                   |                               |
     |                                   | Verificer password (bcrypt)   |
     |                                   |                               |
     | 200 OK                            |                               |
     | { accessToken, refreshToken }     |                               |
     |<----------------------------------|                               |
     |                                   |                               |
     | GET /api/brugere/profil           |                               |
     | Authorization: Bearer <token>     |                               |
     |---------------------------------->|                               |
     |                                   | Valider JWT signatur          |
     |                                   | Tjek exp ikke udløbet         |
     |                                   | Udtræk bruger-ID fra claims   |
     |                                   |                               |
     | 200 OK                            |                               |
     | { profil data }                   |                               |
     |<----------------------------------|                               |
```

### Authorization Header format

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
               ^^^^^^
               Prefix (altid "Bearer")
```

---

## 6. Refresh Tokens

### Problemet med token udløb

Access tokens er kortlivede (f.eks. 15 minutter til 1 time) af sikkerhedshensyn. Men det er irriterende at brugeren skal logge ind igen hvert 15. minut.

### Løsning: Refresh Tokens

```
Access Token:   Kortlivet (15 min - 1 time), bruges til API-requests
Refresh Token:  Langlivet (7-30 dage), bruges KUN til at hente nyt access token
```

**Flow med refresh:**

```
1. Login → modtag access_token (1t) + refresh_token (30d)
2. Brug access_token til API-requests
3. access_token udløber (401 Unauthorized)
4. POST /auth/refresh med refresh_token i body
5. Server validerer refresh_token
6. Ny access_token returneres
7. Fortsæt med API-requests
```

### Token storage

| Sted                  | Fordele                        | Ulemper                            |
|-----------------------|--------------------------------|------------------------------------|
| `localStorage`        | Nem adgang fra JavaScript      | Sårbar over for XSS-angreb         |
| `sessionStorage`      | Slettes ved browser-luk        | Sårbar over for XSS-angreb         |
| HttpOnly Cookie       | Ikke tilgængeligt via JS       | Kræver CSRF-beskyttelse            |
| Memory (variabel)     | Meget sikker                   | Mistes ved page reload             |

**Anbefaling:** HttpOnly cookies til refresh token, memory/localStorage til access token.

---

## 7. JWT i Spring Boot

### Dependency

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
</dependency>
```

### JWT Utility klasse

```java
@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration}")
    private long expiration; // millisekunder

    private Key getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secret);
        return Keys.hmacShaKeyFor(keyBytes);
    }

    // Generér token
    public String generateToken(String email, String rolle) {
        return Jwts.builder()
            .setSubject(email)
            .claim("rolle", rolle)
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(getSigningKey(), SignatureAlgorithm.HS256)
            .compact();
    }

    // Udtræk email fra token
    public String getEmailFromToken(String token) {
        return Jwts.parserBuilder()
            .setSigningKey(getSigningKey())
            .build()
            .parseClaimsJws(token)
            .getBody()
            .getSubject();
    }

    // Validér token
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }
}
```

### Auth Controller

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest req) {
        // Verificer brugernavn og password
        Bruger bruger = brugerService.authenticate(req.getEmail(), req.getPassword());
        if (bruger == null) {
            return ResponseEntity.status(401).build();
        }

        String token = jwtUtil.generateToken(bruger.getEmail(), bruger.getRolle());
        return ResponseEntity.ok(new TokenResponse(token));
    }
}
```

### JWT Filter (Security)

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws IOException, ServletException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7); // Fjern "Bearer "

            if (jwtUtil.validateToken(token)) {
                String email = jwtUtil.getEmailFromToken(token);
                // Sæt authentication i SecurityContext
                UsernamePasswordAuthenticationToken auth =
                    new UsernamePasswordAuthenticationToken(email, null, List.of());
                SecurityContextHolder.getContext().setAuthentication(auth);
            }
        }

        chain.doFilter(request, response);
    }
}
```

---

## 8. Sikkerhedsovervejelser

### Hvad kan gå galt?

| Angreb              | Beskrivelse                                    | Forebyggelse                               |
|---------------------|------------------------------------------------|--------------------------------------------|
| **Token stjålet**   | Angriber bruger dit access token               | Korte tokens levetid (15 min)              |
| **XSS**             | Script stjæler token fra localStorage          | HttpOnly cookies                           |
| **CSRF**            | Cross-Site Request Forgery                     | CSRF-tokens, SameSite cookies              |
| **Svag hemmelighed**| Hemmelighed gættes, tokens forfalskes          | Lang, tilfældig hemmelighed (256+ bit)     |
| **Algorithm None**  | JWT accepterer "none" som algoritme            | Afvis tokens med alg=none                  |

### Best practices

1. **Brug HTTPS** — al JWT-trafik skal krypteres
2. **Korte access tokens** — 15 min til 1 time maksimum
3. **Verificer altid signatur** — afvis tokens med ugyldig signatur
4. **Inkludér `exp` claim** — aldrig tokens uden udløbstid
5. **Brug stærk hemmelighed** — minimum 256-bit tilfældig streng
6. **Valider algoritme** — afvis uventede algoritmmer i header
7. **Gem ikke password i payload** — payload er ikke krypteret!

---

## 9. Opsummering

| Emne                 | Nogleinformation                                              |
|----------------------|---------------------------------------------------------------|
| JWT                  | 3 dele: Header.Payload.Signature, Base64URL-encoded          |
| Header               | Algoritme og token-type                                      |
| Payload              | Claims (sub, exp, iat + custom claims)                       |
| Signature            | Kryptografisk signatur der forhindrer manipulation           |
| Auth flow            | Login → token → bearer i Authorization header               |
| Refresh tokens       | Langlivede tokens til at forny access tokens                 |
| Sikkerhed            | HTTPS, korte levetider, stærk hemmelighed                    |
| 401 vs 403           | 401 = mangler token, 403 = forkerte rettigheder              |

---

## Opgaver til selvstudium

1. Gå til jwt.io og dekod følgende token. Hvad er brugerens rolle og hvornår udløber tokenet?
   `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyQGV4YW1wbGUuY29tIiwicm9sbGUiOiJVU0VSIiwiaWF0IjoxNzA4MDAwMDAwLCJleHAiOjE3MDgwMDM2MDB9.xxx`
2. Forklar hvorfor payload ikke er sikkert til at gemme passwords.
3. Hvad er forskellen på et access token og et refresh token? Hvorfor har vi brug for begge?
4. Hvilken HTTP-header sendes JWT i? Hvad er det nøjagtige format?
5. Beskriv 3 sikkerhedsrisici ved JWT og hvordan de afbødes.
