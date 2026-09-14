# paperless-ngx selfhost notes

Heaviest thing in this fleet: web + postgres + redis. Give it ~2gb ram.
Runs on demand — up when scanning docs, down after. No autostart.

## up

```bash
mkdir -p selfhost/consume
cp selfhost/.env.example selfhost/.env
# set DB_PASSWORD, SECRET_KEY (python -c "import secrets; print(secrets.token_urlsafe(50))")
docker compose -f selfhost/docker-compose.yml --env-file selfhost/.env up -d
```

open http://localhost:8084. First user registered in the UI becomes admin...
actually no — create the superuser explicitly:

```bash
docker exec -it paperless manage.py createsuperuser
```

## ingest

drop pdfs into `selfhost/consume/`, they get picked up automatically.

## backup

```bash
docker exec paperless-db pg_dump -U paperless paperless > backups/paperless-$(date +%F).sql
docker run --rm -v paperless-media:/m -v $(pwd)/backups:/b alpine \
  tar czf /b/paperless-media-$(date +%F).tar.gz /m
```

media matters more than the db (the db can be rebuilt, scans can't).

## update

pull + up -d, then watch logs — paperless runs migrations on start.
