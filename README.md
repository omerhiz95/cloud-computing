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
