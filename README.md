Omer Hiz
# Storingsmelder

Een klein meldingensysteem voor storingen. Je meldt een storing via een
formulier, je ziet de openstaande meldingen in een lijst, en je sluit ze af als
ze opgelost zijn.

Deze applicatie is het startpunt voor **Cloud computing (B-UCLL-MGN13A)**. Je
werkt er het hele semester mee. De applicatie zelf verandert bijna niet, wat
verandert is hoe en waar ze draait.

---
## Opdracht 1 Storingsmelder in containers=
### Wat ik gedaan heb?
**Week 1**
- Docker Desktop, Git en Visual Studio Code geïnstalleerd.
- Mijn eerste containers gestart met `docker run hello-world` en `docker run -d nginx`.
- Containers weergegeven met `docker ps` en hun logs bekeken met `docker logs`.
- Nginx geopend in de browser via poort 8000 op mijn laptop, met een poortkoppeling (`-p`).
- Mijn eigen GitHub-repo `cloud-computing` aangemaakt, gecloned en mijn eerste commit gepusht.

**Week 2**
- Mijn eigen `Dockerfile` geschreven en er een image van gebouwd.
- Een container gestart en Storingsmelder geopend op http://localhost:8000.
- De bouwtijd gemeten met twee volgordes van de regels. Als `requirements.txt` apart en vóór de rest van de code wordt gekopieerd, blijft de `pip install`-laag in de cache en is een nieuwe build veel sneller.
- De grootte van drie base images vergeleken (`python:3.14`, `-slim` en `-alpine`). Ik heb `python:3.14-slim` gekozen omdat [mijn reden, veel kleiner dan de volledige variant en bouwt zonder problemen].
- Een `.dockerignore` gemaakt zodat `.git`, de database en andere onnodige bestanden niet in het image komen.

**Week 3**
- Een volume gebruikt zodat mijn meldingen een herstart en het verwijderen van de container overleven.
- Een `compose.yaml` gemaakt met mijn applicatie en een PostgreSQL-database (`postgres:18`), met een eigen volume (`db-data`) voor de database.
- De applicatie verbindt met de database via de servicenaam `db` in plaats van `localhost`, omdat `localhost` in een container naar de container zelf verwijst.
- Een healthcheck (`pg_isready`) op de database toegevoegd en `depends_on` met `condition: service_healthy`, zodat `web` pas start als de database klaar is. Daarnaast `restart: unless-stopped` als vangnet.
- Mijn wachtwoord in een `.env`-bestand gezet dat in `.gitignore` staat, met een `.env.example` als voorbeeld.
- Mijn image met versienummer 1.0 gepusht naar `ghcr.io/omerhiz95/storingsmelder:1.0` en het package publiek gemaakt.

### Starten vanaf nul

```
git clone https://github.com/omerhiz95/cloud-computing.git
cd cloud-computing
copy .env.example .env
```

Open `.env` en vul de waardes in (`POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`). Start daarna alles:

```
docker compose up -d --build
```

De applicatie draait op http://localhost:8000. Controleer met `docker compose ps` dat de database `healthy` is.

### Stoppen

- `docker compose down` stopt alles. De meldingen blijven bewaard in de volumes.
- `docker compose down -v` stopt alles en verwijdert ook de volumes, dus ook de meldingen.

### Bestanden in deze repo

```
app.py              de applicatie
requirements.txt    de pakketten die ze nodig heeft
templates/          de HTML van de webpagina
Dockerfile          bouwt het image van de applicatie
compose.yaml        start de applicatie en PostgreSQL samen
.env.example        de namen van de instellingen, zonder waardes
.dockerignore       wat niet mee in het image gaat
```

---
### Gebruikte commando's

**Git**

| Commando | Wat het doet |
|---|---|
| `git clone <url>` | Haalt een repo van GitHub naar mijn laptop |
| `git status` | Toont welke bestanden gewijzigd of nieuw zijn (en of `.env` niet per ongeluk meegaat) |
| `git add .` | Zet alle wijzigingen klaar voor de volgende commit |
| `git commit -m "bericht"` | Slaat de klaargezette wijzigingen lokaal op als een commit |
| `git push` | Stuurt mijn commits naar GitHub |
| `git pull` | Haalt nieuwe commits van GitHub naar mijn laptop |

**Docker**

| Commando | Wat het doet |
|---|---|
| `docker build -t storingsmelder .` | Bouwt een image uit de `Dockerfile` in de huidige map |
| `docker run -d -p 8000:8000 --name storingsmelder storingsmelder` | Start een container op de achtergrond en koppelt poort 8000 van mijn laptop aan de container |
| `docker run -v storingsmelder-data:/data ...` | Hangt een volume aan `/data`, zodat de data de container overleeft |
| `docker ps` / `docker ps -a` | Toont draaiende containers / alle containers, ook de gestopte |
| `docker logs <naam>` | Toont wat een container naar de console schreef |
| `docker restart <naam>` | Herstart een container |
| `docker rm -f <naam>` | Verwijdert een container (ook als hij nog draait) |
| `docker images` | Toont de lokale images en hun grootte |
| `docker history <image>` | Toont de lagen van een image en hun grootte |
| `docker volume ls` | Toont de volumes |

**Docker Compose**

| Commando | Wat het doet |
|---|---|
| `docker compose up -d --build` | Bouwt de image en start alle services op de achtergrond |
| `docker compose ps` | Toont de status per service (bij `db` hoort `healthy` te staan) |
| `docker compose logs <service>` | Toont de logs van één service |
| `docker compose config` | Toont het compose-bestand met alle waardes uit `.env` ingevuld |
| `docker compose down` | Stopt en verwijdert de containers en het netwerk, de volumes (en dus de data) blijven |
| `docker compose down -v` | Doet hetzelfde maar verwijdert ook de volumes, dus ook de data |

**Image naar de registry (ghcr.io)**

| Commando | Wat het doet |
|---|---|
| `docker login ghcr.io -u omerhiz95 --password-stdin` | Logt in bij de GitHub-registry met een token (nooit in de repo) |
| `docker tag storingsmelder ghcr.io/omerhiz95/storingsmelder:1.0` | Geeft het image de volledige registrynaam met versienummer |
| `docker push ghcr.io/omerhiz95/storingsmelder:1.0` | Uploadt het image naar de registry |
| `docker pull ghcr.io/omerhiz95/storingsmelder:1.0` | Haalt het image van de registry naar mijn laptop |

## Wat er in deze repo zit

```
app.py              de applicatie
requirements.txt    de pakketten die ze nodig heeft
templates/          de HTML van de webpagina
```

## Wat er bewust niet in zit

Geen Dockerfile en geen compose-bestand. Die schrijf je zelf, dat is precies
wat je in dit vak leert.

---

## Wat de applicatie nodig heeft

| | |
|---|---|
| Python | 3.11 of hoger |
| Pakketten | staan in `requirements.txt` |
| Poort | 8000, tenzij je `PORT` anders instelt |

## Instellingen

Alles wordt geregeld met environment variables. Er staat niets vast in de code,
en er is geen configuratiebestand dat je moet aanpassen.

| Variabele | Standaard | Wat het doet |
|---|---|---|
| `DATABASE_URL` | leeg | Leeg betekent opslaan in een SQLite-bestand. Vul je hier een PostgreSQL-adres in, dan gebruikt de applicatie die database |
| `SQLITE_PAD` | `/data/storingen.db` | Waar het SQLite-bestand komt te staan |
| `APP_TITEL` | `Storingsmelder` | De titel bovenaan de pagina |
| `PORT` | `8000` | De poort waarop de applicatie luistert |

### Twee standen

De applicatie draait met of zonder aparte database.

**Zonder `DATABASE_URL`** schrijft ze naar een SQLite-bestand. Dat werkt meteen
en je hebt niets anders nodig. Let op waar dat bestand terechtkomt, want daar
kom je later op terug.

**Met `DATABASE_URL`** praat ze met PostgreSQL. Bijvoorbeeld:

```
DATABASE_URL=postgresql://storing:geheim@db:5432/storingen
```

Je hoeft de tabel niet zelf aan te maken. De applicatie doet dat bij het
opstarten als ze nog niet bestaat.

---

## Controleren of ze draait

Naast de gewone pagina is er een adres dat kort antwoordt of alles in orde is:

```
GET /health
```

Draait alles, dan krijg je `{"status": "ok", ...}` met code 200. Kan de
applicatie haar database niet bereiken, dan krijg je code 503. Dat adres komt
later in het semester nog van pas.

---

## Wat je er dit semester mee doet

| Blok | Wat je doet |
|---|---|
| Week 1 tot 4 | Je zet deze applicatie in een container en laat ze samen met een database draaien |
| Week 5 tot 9 | Je zet diezelfde applicatie op een Kubernetes-cluster en maakt ze bereikbaar in je browser |
| Week 10 tot 11 | Je laat een pipeline het image bouwen en klaarzetten |

Je begint dus nooit opnieuw. Elke opdracht bouwt verder op de vorige.

---

## Deze repo is van jou

Je hebt hem gemaakt vanaf het sjabloon, dus dit is jouw kopie. Werk erin, commit
elke les, en push. Indienen doe je met de link naar een commit. Hoe dat precies
gaat, staat in de opdrachtbundel op Toledo.
