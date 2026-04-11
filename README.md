# Deploy Template

Vorlage für Auto-Deployment auf unseren Servern.

## Wie es funktioniert

1. Du pushst Code auf `main`
2. GitHub Action baut ein Docker Image und pushed es zu ghcr.io
3. Watchtower auf dem Server erkennt das neue Image und startet den Container neu

Du brauchst SSH nur **einmalig** um den Container das erste Mal zu starten. Danach ist alles automatisch — jeder `git push` deployed die neue Version innerhalb von ca. 30 Sekunden.

## Eigenes Projekt aufsetzen

### 1. Repo erstellen

Erstelle ein neues Repo mit einem `Dockerfile` und kopiere die `.github/workflows/deploy.yml` aus diesem Repo.

### 2. Erster Push

Push deinen Code auf `main`. Die GitHub Action baut das Image automatisch.

### 3. Container starten (einmalig)

SSH auf den Server:

```bash
ssh timo@h2980589.stratoserver.net
```

Container anlegen und starten:

```bash
mkdir -p ~/apps/MEIN-PROJEKT
cat > ~/apps/MEIN-PROJEKT/docker-compose.yml << 'EOF'
services:
  app:
    image: ghcr.io/DEIN-GITHUB-USER/DEIN-REPO:latest
    network_mode: bridge
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.MEIN-PROJEKT.rule=Host(`meine-domain.de`)"
      - "traefik.http.routers.MEIN-PROJEKT.entrypoints=websecure"
      - "traefik.http.routers.MEIN-PROJEKT.tls.certresolver=letsencrypt"
EOF
cd ~/apps/MEIN-PROJEKT && docker compose up -d
```

Ersetze:
- `DEIN-GITHUB-USER/DEIN-REPO` mit deinem Repo
- `MEIN-PROJEKT` mit einem kurzen Namen
- `meine-domain.de` mit deiner Domain (DNS muss auf den Server zeigen)

### 4. Fertig

Ab jetzt brauchst du kein SSH mehr. Jeder `git push` auf `main` deployed automatisch.

## Eigene Domain nutzen

Setze einen DNS A-Record deiner Domain auf die Server-IP. Traefik holt sich automatisch ein SSL-Zertifikat via Let's Encrypt.

## Datenbank hinzufügen

Füge einen Datenbank-Service in die `docker-compose.yml` ein:

```yaml
services:
  app:
    image: ghcr.io/DEIN-GITHUB-USER/DEIN-REPO:latest
    network_mode: bridge
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.MEIN-PROJEKT.rule=Host(`meine-domain.de`)"
      - "traefik.http.routers.MEIN-PROJEKT.entrypoints=websecure"
      - "traefik.http.routers.MEIN-PROJEKT.tls.certresolver=letsencrypt"
  db:
    image: postgres:17
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb

volumes:
  pgdata:
```
