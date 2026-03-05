# Uge 12 — DevOps & Docker

Denne uge introducerer DevOps-filosofien og containerteknologi med Docker. Vi ser på, hvad DevOps er, CI/CD-pipelines, virtualiseringsteknologier, og hvordan Docker gør det muligt at pakke og køre applikationer i isolerede, reproducerbare miljøer.

---

## 1. DevOps

### Hvad er DevOps?

DevOps er en **kultur og praksis** der bringer **Development** (Udvikling) og **Operations** (Drift) tættere sammen. Målet er at levere software hurtigere, mere pålideligt og med bedre kvalitet.

```
Traditionelt (silo-model):
+---------------+     "Det virker på min maskine!"     +---------------+
|  Developers   | ----------------------------------> |   Operations  |
|   (koder)     |                                     |  (deployer)   |
+---------------+                                     +---------------+
     Lav                                                    Lav
  kommunikation                                          kommunikation

DevOps (samlet model):
+-----------------------------------------------+
|        Dev  +  Ops  =  DevOps                 |
|   Plan → Code → Build → Test → Release →      |
|   Deploy → Operate → Monitor → (tilbage)       |
+-----------------------------------------------+
```

### DevOps Principper

1. **Continuous Integration (CI)** — integrér kode ofte, automatisk test ved hvert push
2. **Continuous Delivery (CD)** — automatisk levering til test/staging-miljø
3. **Continuous Deployment** — automatisk deployment til produktion
4. **Infrastructure as Code (IaC)** — infrastruktur beskrives i kode (Terraform, Ansible)
5. **Monitoring & Feedback** — overvåg systemer kontinuerligt
6. **Culture of collaboration** — kommunikation og fælles ansvar

### DevOps fordele

| Fordel                  | Beskrivelse                                               |
|-------------------------|-----------------------------------------------------------|
| Hurtigere releases      | Fra uger/måneder til timer/dage                           |
| Færre fejl i produktion | Automatiseret test opdager fejl tidligt                   |
| Hurtigere fejlretning   | Nemmere at finde fejl når changes er små                  |
| Bedre samarbejde        | Dev og Ops deler ansvar                                   |
| Skalerbarhed            | Automatiserede processer skalerer nemmere                 |

---

## 2. CI/CD Pipeline

### CI — Continuous Integration

```
Developer pusher kode til Git
    ↓
CI-server opdager ny kode (GitHub Actions, Jenkins, GitLab CI)
    ↓
Build: Kompilér kode
    ↓
Test: Kør unit tests + integrationstests
    ↓
Code Analysis: Tjek kodekvalitet (SonarQube)
    ↓
Artifact: Byg .jar / Docker image
    ↓
Notificér developer: Grøn (succes) eller Rød (fejl)
```

### CD — Continuous Delivery/Deployment

```
CI Pipeline lykkedes
    ↓
Deploy til Staging/Test-miljø automatisk
    ↓
Kør acceptance tests
    ↓
[Continuous Delivery]   → Manuel godkendelse til produktion
[Continuous Deployment] → Automatisk deploy til produktion
```

### GitHub Actions eksempel (CI Pipeline)

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout kode
      uses: actions/checkout@v3

    - name: Sæt Java 17 op
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Byg med Maven
      run: mvn clean package -DskipTests

    - name: Kør tests
      run: mvn test

    - name: Byg Docker image
      run: docker build -t myapp:${{ github.sha }} .

    - name: Push til Docker Hub
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push myapp:${{ github.sha }}
```

---

## 3. Virtualisering

### Hypervisor (traditionel virtualisering)

En **hypervisor** er software der kører oven på hardware og giver mulighed for at køre **virtuelle maskiner (VMs)**.

```
+-------------------+  +-------------------+
|   App A           |  |   App B           |
+-------------------+  +-------------------+
|   Gæste-OS (Linux)|  |   Gæste-OS (Win)  |
+-------------------+  +-------------------+
|        Hypervisor (VMware, VirtualBox, Hyper-V)        |
+----------------------------------------------------+
|              Host Hardware                         |
+----------------------------------------------------+
```

**Type 1 Hypervisor (Bare-metal):** Kører direkte på hardware (VMware ESXi, Hyper-V)
**Type 2 Hypervisor (Hosted):** Kører oven på et host-OS (VirtualBox, VMware Workstation)

### Virtuelle Maskiner (VM)

En VM er et **fuldstændigt emuleret computersystem** med sit eget OS.

**Fordele:**
- Stærk isolation (eget OS)
- Kan køre andre OS (Windows på Linux)
- Snapshots og gendannelse

**Ulemper:**
- Stor størrelse (GB)
- Langsom opstart (minutter)
- Høj ressourceforbrug (OS overhead)

---

## 4. Containerisering vs. Virtualisering

```
Virtualisering:                    Containerisering:
+----------+  +----------+         +------+  +------+  +------+
| App A    |  | App B    |         | App A|  | App B|  | App C|
+----------+  +----------+         +------+  +------+  +------+
| Gæste OS |  | Gæste OS |         | Libs |  | Libs |  | Libs |
+----------+  +----------+         +------+  +------+  +------+
|    Hypervisor            |        |      Container Runtime (Docker)     |
+--------------------------+        +----------------------------------+
|         Hardware         |        |     Host OS (Linux kernel)       |
+--------------------------+        +----------------------------------+
                                    |         Hardware                 |
                                    +----------------------------------+
```

### Sammenligning

| Aspekt          | VM                          | Container                      |
|-----------------|-----------------------------|--------------------------------|
| Størrelse       | GB (inkl. OS)               | MB (deler host OS kernel)      |
| Opstartstid     | Minutter                    | Sekunder                       |
| Isolation       | Fuld (eget OS)              | Process-niveau                 |
| Portabilitet    | Stor fil, langsommere       | Let og hurtig                  |
| Ressourcebrug   | Højt (OS overhead)          | Lavt                           |
| Brug            | Legacy apps, stærk isolation| Microservices, cloud-native    |

---

## 5. Docker — Grundlæggende

### Hvad er Docker?

Docker er den mest populære **containerplatform**. Den lader dig:
- Pakke en applikation og dens dependencies i et **image**
- Køre imagen som en **container** på enhver maskine med Docker installeret

### Centrale begreber

| Begreb         | Forklaring                                                        |
|----------------|-------------------------------------------------------------------|
| **Image**      | Skabelon/opskrift for en container. Immutable (kan ikke ændres).  |
| **Container**  | En kørende instans af et image. Kan startes/stoppes/slettes.      |
| **Dockerfile** | Tekstfil der beskriver, hvordan et image bygges                   |
| **Docker Hub** | Offentligt register for Docker images (hub.docker.com)            |
| **Registry**   | Lager for Docker images (Docker Hub, GitHub Container Registry)   |
| **Volume**     | Persistent lager der overlever container-sletning                 |
| **Network**    | Virtuelt netværk der forbinder containers                         |

### Docker arkitektur

```
+-------------------+          +-------------------+
|   Docker CLI      |          |   Docker Hub      |
| docker build      |          | (image registry)  |
| docker run        |          |                   |
| docker pull       |          +-------------------+
+-------------------+                  |
         |                             |
         v                             v
+-------------------------------------------+
|          Docker Daemon                    |
|                                           |
|  Images:           Containers:            |
|  [ubuntu:22.04]    [webapp-1 running]     |
|  [openjdk:17]      [webapp-2 stopped]     |
|  [myapp:1.0]       [mysql-db running]     |
+-------------------------------------------+
|              Host OS                      |
+-------------------------------------------+
```

---

## 6. Dockerfile

En `Dockerfile` er en tekstfil med instruktioner til at bygge et Docker image.

### Dockerfile instruktioner

| Instruktion | Formål                                           |
|-------------|--------------------------------------------------|
| `FROM`      | Basis-image at bygge på                          |
| `WORKDIR`   | Sæt arbejdsmappe i container                     |
| `COPY`      | Kopiér filer fra host ind i image                |
| `RUN`       | Kør kommando under build (installér packages)    |
| `EXPOSE`    | Dokumentér hvilken port appen lytter på          |
| `ENV`       | Sæt miljøvariabel                                |
| `CMD`       | Standardkommando når container startes           |
| `ENTRYPOINT`| Kommando der altid kører (kan ikke overrides)    |

### Spring Boot Dockerfile

```dockerfile
# Brug officiel Java 17 base image
FROM openjdk:17-jdk-slim

# Sæt arbejdsmappe i container
WORKDIR /app

# Kopiér den byggede JAR-fil
COPY target/myapp-0.0.1-SNAPSHOT.jar app.jar

# Dokumentér porten
EXPOSE 8080

# Start applikationen
CMD ["java", "-jar", "app.jar"]
```

### Multi-stage Dockerfile (bygger og kører)

```dockerfile
# Stage 1: Byg applikationen
FROM maven:3.9-openjdk-17 AS builder
WORKDIR /build
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Kør applikationen
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /build/target/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

Multi-stage fordel: Endelige image indeholder IKKE Maven eller kildekode — kun den kørende app.

---

## 7. Docker Kommandoer

### Bygge og køre

```bash
# Byg image fra Dockerfile i nuværende mappe
docker build -t myapp:1.0 .

# Byg med andet tag
docker build -t username/myapp:latest .

# Kør container fra image
docker run myapp:1.0

# Kør i baggrunden (detached mode)
docker run -d myapp:1.0

# Map port: host:container
docker run -d -p 8080:8080 myapp:1.0

# Giv container et navn
docker run -d -p 8080:8080 --name min-webapp myapp:1.0

# Kør med miljøvariabel
docker run -d -p 8080:8080 -e DB_URL=jdbc:mysql://... myapp:1.0
```

### Administrere containers

```bash
# List kørende containers
docker ps

# List alle containers (inkl. stoppede)
docker ps -a

# Stop container
docker stop min-webapp

# Start stoppet container
docker start min-webapp

# Slet container
docker rm min-webapp

# Slet kørende container med tvang
docker rm -f min-webapp

# Vis logs
docker logs min-webapp

# Vis logs i real-time
docker logs -f min-webapp

# Kør kommando i kørende container
docker exec -it min-webapp bash
```

### Administrere images

```bash
# List images
docker images

# Slet image
docker rmi myapp:1.0

# Hent image fra Docker Hub
docker pull nginx:latest

# Push image til Docker Hub
docker push username/myapp:1.0

# Søg på Docker Hub
docker search ubuntu
```

### docker run flag oversigt

| Flag            | Eksempel                    | Forklaring                              |
|-----------------|-----------------------------|-----------------------------------------|
| `-d`            | `docker run -d`             | Detached (kør i baggrunden)             |
| `-p`            | `-p 8080:8080`              | Port mapping (host:container)           |
| `--name`        | `--name webapp`             | Navngiv container                       |
| `-e`            | `-e DB=jdbc:mysql://...`    | Miljøvariabel                           |
| `-v`            | `-v /data:/app/data`        | Volume mount                            |
| `--rm`          | `--rm`                      | Slet container automatisk når den stopper |
| `-it`           | `-it ubuntu bash`           | Interaktiv terminal                     |
| `--network`     | `--network mynet`           | Tilslut til netværk                     |

---

## 8. Docker Compose

Docker Compose lader dig definere og køre **multi-container** applikationer med en enkelt YAML-fil.

```yaml
# docker-compose.yml
version: '3.8'

services:
  webapp:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:mysql://database:3306/mydb
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=hemmelighed
    depends_on:
      - database

  database:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=hemmelighed
      - MYSQL_DATABASE=mydb
    volumes:
      - db-data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  db-data:
```

```bash
# Start alle services
docker-compose up -d

# Stop alle services
docker-compose down

# Se logs
docker-compose logs -f

# Rebuild og start
docker-compose up -d --build
```

---

## 9. Opsummering

| Emne              | Nogleinformation                                               |
|-------------------|----------------------------------------------------------------|
| DevOps            | Kultur der samler Dev og Ops, hurtigere og bedre leverancer   |
| CI/CD             | Automatiseret build, test og deployment ved hvert code push    |
| VM                | Fuld OS emulering, GB størrelse, minutter at starte           |
| Container         | Deler host OS kernel, MB størrelse, sekunder at starte        |
| Docker Image      | Immutable skabelon, bygges fra Dockerfile                      |
| Docker Container  | Kørende instans af et image                                    |
| Dockerfile        | FROM, WORKDIR, COPY, RUN, EXPOSE, CMD                         |
| docker build      | `docker build -t navn:tag .`                                   |
| docker run        | `docker run -d -p 8080:8080 navn:tag`                         |
| Docker Compose    | Multi-container setup i YAML                                   |

---

## Opgaver til selvstudium

1. Hvad er forskellen på en VM og en container? Giv et eksempel på hvornår man ville bruge det ene frem for det andet.
2. Skriv en Dockerfile til en simpel Spring Boot applikation.
3. Hvad gør kommandoen `docker run -d -p 3000:8080 --name min-app myimage:latest`? Forklar hvert flag.
4. Hvad er forskellen på `CMD` og `ENTRYPOINT` i en Dockerfile?
5. Hvad er fordelen ved multi-stage builds i Docker?
6. Tegn en CI/CD pipeline for et Java Spring Boot projekt deployed med Docker.
