# Uge 13 — Python & REST

Denne uge introducerer Python som programmeringssprog og ser på, hvordan Python bruges til at lave HTTP-requests og arbejde med REST APIs. Vi gennemgår Python-syntaks, datatyper, funktioner og det vigtige `requests`-modul. Ugen er også en forberedelse til prøveeksamen.

---

## 1. Python — Introduktion

Python er et **fortolket, dynamisk typet** programmeringssprog der er kendt for sin enkle og læsbare syntaks. Det er ekstremt populært inden for webudvikling, data science, AI og scripting.

### Python vs. Java — Syntaks sammenligning

```python
# Python — ingen semikolon, ingen krølparentes, indrykning definerer blokke
def beregn_sum(a, b):
    resultat = a + b
    return resultat

print(beregn_sum(3, 4))  # 7
```

```java
// Java — semikolon, krølparenteser, statisk typet
public int beregnSum(int a, int b) {
    int resultat = a + b;
    return resultat;
}

System.out.println(beregnSum(3, 4)); // 7
```

---

## 2. Python Grundlæggende Syntaks

### Variabler og typer

```python
# Ingen typeangivelse nødvendig
navn = "Anna"           # str
alder = 22              # int
gns = 8.5               # float
aktiv = True            # bool (stort T/F)
ingen = None            # None (svarer til null i Java)

# Type kan tjekkes
print(type(navn))       # <class 'str'>
print(type(alder))      # <class 'int'>
```

### Datatyper oversigt

| Type   | Eksempel                   | Mutabel? | Java-ækvivalent |
|--------|----------------------------|----------|-----------------|
| `str`  | `"Hej"`, `'verden'`        | Nej      | String          |
| `int`  | `42`, `-7`                 | Nej      | int / Integer   |
| `float`| `3.14`, `-0.5`             | Nej      | double / Double |
| `bool` | `True`, `False`            | Nej      | boolean         |
| `list` | `[1, 2, 3]`                | Ja       | ArrayList       |
| `dict` | `{"navn": "Anna"}`         | Ja       | HashMap         |
| `tuple`| `(1, 2, 3)`                | Nej      | —               |
| `set`  | `{1, 2, 3}`                | Ja       | HashSet         |
| `None` | `None`                     | —        | null            |

---

## 3. Python Kontrolstrukturer

### If/elif/else

```python
alder = 20

if alder >= 18:
    print("Myndig")
elif alder >= 15:
    print("Teenager")
else:
    print("Barn")

# One-liner (ternary)
status = "Myndig" if alder >= 18 else "Mindreaarig"
```

### Løkker

```python
# for-løkke med range
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

# for-løkke over liste
navne = ["Anna", "Bo", "Cecilie"]
for navn in navne:
    print(f"Hej {navn}!")

# while-løkke
tæller = 0
while tæller < 5:
    print(tæller)
    tæller += 1

# enumerate — hent index og værdi
for i, navn in enumerate(navne):
    print(f"{i}: {navn}")
```

---

## 4. Python Funktioner

```python
# Grundlæggende funktion
def hilsen(navn):
    return f"Hej, {navn}!"

# Funktion med standardværdi
def hilsen(navn, sproget="dansk"):
    if sproget == "dansk":
        return f"Hej, {navn}!"
    else:
        return f"Hello, {navn}!"

# Funktion med type hints (Python 3.5+)
def beregn_bmi(vaegt: float, hoejde: float) -> float:
    return vaegt / (hoejde ** 2)

# Lambda (anonym funktion)
kvadrat = lambda x: x ** 2
print(kvadrat(5))  # 25

# Variadisk funktion (*args og **kwargs)
def sum_alle(*tal):
    return sum(tal)

print(sum_alle(1, 2, 3, 4))  # 10
```

---

## 5. Python Lists

```python
# Opret liste
brugere = ["Anna", "Bo", "Cecilie"]

# Tilgå elementer
print(brugere[0])    # "Anna" (0-indekseret)
print(brugere[-1])   # "Cecilie" (fra slutningen)

# Slicing
print(brugere[0:2])  # ["Anna", "Bo"]
print(brugere[1:])   # ["Bo", "Cecilie"]

# Modificér
brugere.append("David")      # Tilføj til slutning
brugere.insert(1, "Mette")   # Indsæt ved index 1
brugere.remove("Bo")         # Fjern første "Bo"
brugere.pop()                # Fjern og returnér sidste

# Useful methods
print(len(brugere))          # Antal elementer
print("Anna" in brugere)     # True/False (exists check)
brugere.sort()               # Sorter in-place
sorteret = sorted(brugere)   # Returnér ny sorteret liste

# List comprehension (kompakt, meget pythonic)
tal = [1, 2, 3, 4, 5]
kvadrater = [x ** 2 for x in tal]         # [1, 4, 9, 16, 25]
lige = [x for x in tal if x % 2 == 0]    # [2, 4]
```

---

## 6. Python Dictionaries

```python
# Opret dict
bruger = {
    "id": 1,
    "navn": "Anna Larsen",
    "email": "anna@example.com",
    "aktiv": True
}

# Tilgå
print(bruger["navn"])              # "Anna Larsen"
print(bruger.get("telefon"))       # None (ingen KeyError)
print(bruger.get("telefon", "—"))  # "—" (standardværdi)

# Tilføj/modificér
bruger["alder"] = 22
bruger["email"] = "ny@email.com"

# Slet
del bruger["aktiv"]
vaerdi = bruger.pop("alder")  # Fjern og returnér

# Iteration
for nogle in bruger.keys():
    print(nogle)

for vaerdi in bruger.values():
    print(vaerdi)

for nogle, vaerdi in bruger.items():
    print(f"{nogle}: {vaerdi}")

# Tjek om nøgle eksisterer
if "navn" in bruger:
    print(bruger["navn"])

# Dict comprehension
kvadrater = {x: x**2 for x in range(1, 6)}
# {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

---

## 7. Python Requests Modulet

`requests` er et tredjeparts Python-bibliotek til at lave HTTP-requests. Det er langt nemmere at bruge end Pythons indbyggede `urllib`.

### Installation

```bash
pip install requests
```

### Grundlæggende brug

```python
import requests

# GET request
response = requests.get("https://api.example.com/brugere")

print(response.status_code)   # 200
print(response.json())         # Python dict/list
print(response.text)           # Råt tekst-svar
print(response.headers)        # Response headers
```

---

## 8. GET Requests

```python
import requests

# Simpel GET
response = requests.get("https://jsonplaceholder.typicode.com/users")

# Tjek statuskode
if response.status_code == 200:
    brugere = response.json()   # Parser JSON til Python liste
    for bruger in brugere:
        print(f"ID: {bruger['id']}, Navn: {bruger['name']}")
else:
    print(f"Fejl: {response.status_code}")

# GET med query parameters
params = {
    "userId": 1,
    "completed": "false"
}
response = requests.get(
    "https://jsonplaceholder.typicode.com/todos",
    params=params
)
# URL bliver: .../todos?userId=1&completed=false
print(response.url)  # Se den fulde URL

# GET med headers
headers = {
    "Authorization": "Bearer eyJhbGci...",
    "Accept": "application/json"
}
response = requests.get(
    "https://api.example.com/profil",
    headers=headers
)
```

---

## 9. POST, PUT, PATCH, DELETE

```python
import requests

BASE_URL = "https://jsonplaceholder.typicode.com"
HEADERS = {"Content-Type": "application/json"}

# POST — opret ny ressource
ny_bruger = {
    "navn": "Anna Larsen",
    "email": "anna@example.com",
    "alder": 22
}
response = requests.post(
    f"{BASE_URL}/users",
    json=ny_bruger,   # Sætter automatisk Content-Type: application/json
    headers=HEADERS
)
print(response.status_code)  # 201
print(response.json())        # Ny bruger med ID

# PUT — erstat hel ressource
opdateret_bruger = {
    "id": 1,
    "navn": "Anna Larsen",
    "email": "ny@email.com",
    "alder": 23
}
response = requests.put(
    f"{BASE_URL}/users/1",
    json=opdateret_bruger,
    headers=HEADERS
)

# PATCH — delvis opdatering
partial_update = {"email": "nyemail@example.com"}
response = requests.patch(
    f"{BASE_URL}/users/1",
    json=partial_update,
    headers=HEADERS
)

# DELETE — slet ressource
response = requests.delete(f"{BASE_URL}/users/1")
print(response.status_code)  # 200 eller 204
```

---

## 10. JSON Parsing i Python

```python
import requests
import json

# JSON fra API til Python
response = requests.get("https://jsonplaceholder.typicode.com/users/1")
data = response.json()  # Dict

print(data["name"])          # "Leanne Graham"
print(data["address"]["city"]) # Indlejret dict

# Python til JSON-streng
import json
bruger = {"navn": "Anna", "alder": 22}
json_streng = json.dumps(bruger, indent=2)
print(json_streng)
# {
#   "navn": "Anna",
#   "alder": 22
# }

# JSON-streng til Python
json_input = '{"navn": "Anna", "alder": 22}'
objekt = json.loads(json_input)
print(objekt["navn"])  # "Anna"

# JSON med ensure_ascii=False (for dansk tekst)
dansk = {"by": "København", "aktivitet": "Løb"}
print(json.dumps(dansk, ensure_ascii=False, indent=2))
```

---

## 11. Komplet REST Client Eksempel

```python
import requests

class BrugerAPIClient:
    def __init__(self, base_url, token=None):
        self.base_url = base_url
        self.headers = {"Content-Type": "application/json"}
        if token:
            self.headers["Authorization"] = f"Bearer {token}"

    def login(self, email, password):
        response = requests.post(
            f"{self.base_url}/auth/login",
            json={"email": email, "password": password}
        )
        if response.status_code == 200:
            token = response.json()["token"]
            self.headers["Authorization"] = f"Bearer {token}"
            return True
        return False

    def hent_alle(self):
        response = requests.get(
            f"{self.base_url}/api/brugere",
            headers=self.headers
        )
        response.raise_for_status()  # Kaster exception ved 4xx/5xx
        return response.json()

    def hent_en(self, bruger_id):
        response = requests.get(
            f"{self.base_url}/api/brugere/{bruger_id}",
            headers=self.headers
        )
        if response.status_code == 404:
            return None
        response.raise_for_status()
        return response.json()

    def opret(self, bruger_data):
        response = requests.post(
            f"{self.base_url}/api/brugere",
            json=bruger_data,
            headers=self.headers
        )
        response.raise_for_status()
        return response.json()

    def slet(self, bruger_id):
        response = requests.delete(
            f"{self.base_url}/api/brugere/{bruger_id}",
            headers=self.headers
        )
        return response.status_code == 204


# Brug
client = BrugerAPIClient("http://localhost:8080")
client.login("admin@example.com", "hemmelighed")

brugere = client.hent_alle()
for b in brugere:
    print(f"{b['id']}: {b['navn']}")

ny = client.opret({"navn": "Test Bruger", "email": "test@example.com"})
print(f"Oprettet bruger med ID: {ny['id']}")
```

---

## 12. Fejlhåndtering i Python

```python
import requests
from requests.exceptions import ConnectionError, Timeout, HTTPError

try:
    response = requests.get(
        "http://api.example.com/data",
        timeout=5  # Sekunder
    )
    response.raise_for_status()  # Kaster HTTPError ved 4xx/5xx
    data = response.json()

except ConnectionError:
    print("Kunne ikke oprette forbindelse til serveren")

except Timeout:
    print("Forbindelsen tog for lang tid")

except HTTPError as e:
    print(f"HTTP fejl: {e.response.status_code}")
    if e.response.status_code == 401:
        print("Du er ikke logget ind")
    elif e.response.status_code == 404:
        print("Ressourcen blev ikke fundet")

except Exception as e:
    print(f"Uventet fejl: {e}")
```

---

## 13. Prøveeksamen Forberedelse

### Typiske opgavetyper i TEK

1. **Forklar en protokol** — TCP vs UDP, HTTP request/response, JWT struktur
2. **Tegn et diagram** — TCP/IP lag, Docker arkitektur, CI/CD pipeline
3. **Læs og forklar kode** — REST endpoints i Spring Boot, Python requests
4. **Skriv simple kode** — Python GET/POST request, Postman test script
5. **Status koder** — hvilken kode returneres i scenario X?

### Hurtig opsummering — alt pensum

| Uge | Emne               | Nøgletal/begreber                              |
|-----|--------------------|------------------------------------------------|
| 6   | TCP/IP & JSON      | 5 lag, TCP vs UDP, JSON datatyper              |
| 7   | IP & Routing       | IPv4 (32 bit), IPv6 (128 bit), NAT, private IP|
| 8   | HTTP & REST        | GET/POST/PUT/PATCH/DELETE, stateless, CRUD     |
| 9   | Status koder, CORS | 2xx/4xx/5xx, 401 vs 403, CORS preflight        |
| 10  | Filter & Postman   | Query params, pm.test(), pm.environment.set()  |
| 11  | JWT               | Header.Payload.Signature, Bearer token, exp    |
| 12  | DevOps & Docker    | CI/CD, VM vs container, Dockerfile, docker run |
| 13  | Python & REST      | requests.get/post, json(), raise_for_status()  |

---

## Opgaver til selvstudium

1. Skriv Python-kode der henter alle "todos" fra `https://jsonplaceholder.typicode.com/todos` og udskriver kun dem der er `completed: true`.
2. Skriv Python-kode der opretter en ny "post" via POST-request til JSONPlaceholder API.
3. Hvad er forskellen på `response.json()` og `json.loads(response.text)`?
4. Hvad gør `response.raise_for_status()`? Hvornår er det nyttigt?
5. Gennemgå de 5 typiske eksamensspørgsmål ovenfor og skriv korte svar til hvert.
