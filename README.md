# WolfISMS

Ein Informationssicherheitsmanagementsystem (ISMS) als Docker-Container.

## Installation

```bash
git clone <repo-url>
cd WolfISMS
cp .envdefault .env
```

`.env` bearbeiten und Werte anpassen.

```bash
docker compose up -d
```

## Voraussetzungen

- Docker
- Docker Compose
- NPM als Docker (siehe `NPM.md`)

## Zugriff nur über den Reverse-Proxy

Der Container spricht ausschließlich HTTP auf Port 8000. HTTPS terminiert
der Nginx Proxy Manager; ein anderer Reverse-Proxy (Traefik, Apache, IIS …)
geht ebenso, solange er TLS terminiert und `X-Forwarded-For`/`-Proto`
setzt. HTTPS ist Pflicht: die Session-Cookies sind `Secure`, über reines
HTTP schlägt die Anmeldung fehl. `WEBPATH` muss deshalb mit `https://`
beginnen.

Der Host-Port der App hängt fest an `127.0.0.1` und dient nur der Diagnose
auf dem Docker-Host. Stehen zwei Proxys hintereinander (z. B. Cloudflare vor
NPM), `ISMS_PROXY_HOPS=2` in der `.env` setzen.


## Externe Datenbank

Standard ist eine MariaDB als Container im Stack (`DB_MODUS=intern`). Für
eine vorhandene MariaDB/MySQL in der `.env`:

```
DB_MODUS=extern
DB_HOST=db.firma.local
DB_PORT=3306
DB_NAME=isms
DB_USER=isms
DB_PASSWORD=...
DB_SSL_MODE=verify-full                       # optional, TLS
DB_SSL_CA=/var/www/isms/db-certs/db-ca.pem    # Datei liegt in ./db-certs
```

Dann startet kein DB-Container. Datenbank und Benutzer müssen auf dem Server
angelegt sein (Beispiel-SQL in `.envdefault`). Benötigt Docker Compose ≥ 2.20.
`DB_TYPE` ist für weitere Datenbanktypen vorgesehen; derzeit werden `mariadb`
und `mysql` unterstützt.
