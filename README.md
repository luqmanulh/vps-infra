# VPS Infrastructure

Backup konfigurasi infrastruktur VPS untuk devops-lab.

## Stack

- **Traefik v3.7** - Reverse proxy dengan Let's Encrypt
- **Forgejo v15** - Git server (GitHub-compatible)
- **Actions Runner v12** - CI/CD runner
- **Uptime Kuma v2** - Monitoring & alerting

## Struktur Folder

```
vps-infra/
├── docker-compose.yml    # Main compose file
├── .env                  # Environment variables (JANGAN COMMIT!)
├── traefik/
│   ├── traefik.yml       # Traefik config
│   └── acme.json         # Let's Encrypt certificates (auto-generated)
├── forgejo/              # Forgejo config (excluded from git)
├── actions-runner/       # CI runner config
└── uptime-kuma/          # Monitoring config
```

## Setup Ulang

1. **Clone repo ini:**
   ```bash
   git clone git@github.com:luqmanulh/vps-infra.git
   cd vps-infra
   ```

2. **Copy .env.example ke .env dan edit:**
   ```bash
   cp .env.example .env
   vim .env  # Edit dengan values yang sesuai
   ```

3. **Start semua services:**
   ```bash
   docker compose up -d
   ```

4. **Verify:**
   ```bash
   docker compose ps
   ```

## Security Notes

- SSH hardening: deploy user restricted dengan command wrapper
- Traefik auto-renew Let's Encrypt certificates
- Services isolated di Docker network terpisah
- `.env` file JANGAN di-commit ke git!

## Backup & Restore

### Backup Config
```bash
tar -czf vps-infra-backup.tar.gz \
  --exclude='forgejo/data' \
  --exclude='forgejo/log' \
  --exclude='actions-runner/data' \
  --exclude='uptime-kuma/data' \
  vps-infra/
```

### Restore
```bash
tar -xzf vps-infra-backup.tar.gz
cd vps-infra
docker compose up -d
```

## Maintenance

### Update Services
```bash
docker compose pull
docker compose up -d --force-recreate
```

### View Logs
```bash
docker compose logs -f traefik
docker compose logs -f forgejo
docker compose logs -f actions-runner
```

### Restart Service
```bash
docker compose restart traefik
```

## Changelog

### 2026-06-11
- **Refactor Traefik config**: Migrate Let's Encrypt email dari hardcoded di `traefik.yml` ke env variable `LETSENCRYPT_EMAIL` di `docker-compose.yml`
- **Centralize sensitive config**: Semua sensitive data sekarang di `.env` file, tidak ada lagi hardcoded values di config files
- **Security improvement**: Email tidak lagi exposed di version control

### 2026-06-08
- Initial backup setup
- Configure SSH deployment dengan deploy user
- Setup CI/CD pipeline dengan Forgejo Actions
