Auf dem Docker-Host einmalig:

```bash
docker network create proxy
```

Verreichnis npm anlegen und darin docker-compose.yml anlegen:

```bash
services:
  npm:
    image: jc21/nginx-proxy-manager:2
    restart: unless-stopped
    ports:
      - "80:80"     # HTTP + ACME-Challenge
      - "443:443"   # HTTPS
      - "81:81"     # Admin-Oberflaeche
    volumes:
      - npm_data:/data
      - npm_letsencrypt:/etc/letsencrypt
    networks:
      - proxy

networks:
  proxy:
    external: true
    name: proxy

volumes:
  npm_data:
  npm_letsencrypt:
```

```bash
docker compose up -d
```


http://dockerserver:81

Proxy Host in NPM
Hosts → Proxy Hosts → Add Proxy Host:

```bash
Feld	Wert
Domain Names	isms.deine-domain.tld
Scheme	http
Forward Hostname	wolfisms-app (dein PROXY_ALIAS)
Forward Port	8000
Block Common Exploits	an
Websockets Support	an
```

HTTPS ist Pflicht, also Zertifikat für den Host anlegen.

Im Reiter SSL: „Request a new SSL Certificate", Force SSL und HTTP/2 an. Let's Encrypt braucht dafür, dass deine Domain öffentlich auf die Host-IP zeigt und Port 80 von außen erreichbar ist — sonst musst du auf eine DNS-Challenge ausweichen.

## Warum nur über NPM

Der WolfISMS-Container hat kein eigenes HTTPS. NPM terminiert TLS und reicht
die Anfragen per HTTP im Netz `proxy` an die App weiter.

- Die Session-Cookies sind `Secure`. Ohne „Force SSL“ oder bei einem Aufruf
  per IP schlägt die Anmeldung fehl.
- `WEBPATH` in der `.env` muss genau die Adresse aus dem Browser sein, mit
  `https://`. Daraus bilden sich die SSO-Redirect-URIs und die Links in Mails.
- Die App übernimmt `X-Forwarded-For`/`-Proto`/`-Host`/`-Port` von genau so
  vielen Proxys, wie `ISMS_PROXY_HOPS` sagt (Vorgabe 1 = NPM). Steht ein
  weiterer Proxy davor (z. B. Cloudflare), `ISMS_PROXY_HOPS=2` setzen.
- Deshalb darf die App nicht an NPM vorbei erreichbar sein: über einen offenen
  Port könnte jeder Client diese Header selbst schicken und seine IP
  (Rate-Limit) oder HTTPS vortäuschen. Die `docker-compose.yml` veröffentlicht
  Port 8000 nur auf `127.0.0.1` – für `curl` auf dem Host oder einen
  SSH-Tunnel zur Diagnose.
- Zwischen NPM und App läuft HTTP unverschlüsselt, aber nur innerhalb des
  Docker-Hosts im Netz `proxy`.
