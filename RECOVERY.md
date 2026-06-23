# 🛟 Disaster Recovery Plan — VPS Infra

> **VPS:** Tencent Cloud 2vCPU/2GB/40GB, Ubuntu 24.04
> **IP:** 43.129.35.12
> **Domain:** *.luqmanulh.my.id (Hostinger)
> **GitHub:** github.com/luqmanulh/vps-infra
> **Backup cron:** /home/lnh/backup.sh (setiap hari jam 03:00)

---

## 📋 Checklist Pra-VPS

Sebelum VPS baru:

- [ ] Backup terbaru ada? Cek `/home/lnh/backups/` 
      Atau restore dari GitHub + database backup
- [ ] Domain DNS records ready? (luqmanulh.my.id via Hostinger)

---

## 🔴 FULL RECOVERY — VPS Baru

### Step 1: Provision VPS Baru

```bash
# Tencent Cloud → Create instance → Ubuntu 24.04
# SSH in:
ssh root@<new-ip>
```

### Step 2: Install Docker + Compose

```bash
apt update && apt install -y docker.io docker-compose-v2
systemctl enable --now docker
```

### Step 3: Clone Config dari GitHub

```bash
git clone https://github.com/luqmanulh/vps-infra.git
cd vps-infra
cp .env.example .env
vim .env  # Edit dengan values asli (email, dll)
```

### Step 4: Setup Traefik Let's Encrypt

```bash
# Create acme.json
touch traefik/acme.json
chmod 600 traefik/acme.json

# Create docker network
docker network create traefik-net
```

### Step 5: Start All Services

```bash
docker compose up -d
```

### Step 6: Setup Forgejo Runner

```bash
# Register runner via Forgejo UI → Settings → Actions → Add Runner
# Copy token, then:
docker compose restart actions-runner
```

### Step 7: Restore Database (jika ada backup)

```bash
# Stop services first
docker compose down forgejo uptime-kuma grafana

# Restore Forgejo
docker cp backups/forgejo-<date>.sqlite $(docker create --name temp-forgejo forgejo:15):/data/gitea/gitea.db
docker rm temp-forgejo

# Restart services
docker compose up -d
```

### Step 8: Restore Grafana Datasource & Prometheus Config

```bash
# Already in git (monitoring/prometheus/prometheus.yml)
# Grafana provisioning already in git (monitoring/grafana/provisioning/)

# Just restart
docker compose restart prometheus grafana
```

---

## 🟡 PARTIAL RECOVERY — Service Crash

### Service Down (tapi VPS hidup)

```bash
# Cek status
docker ps -a | grep <service-name>

# Lihat log
docker logs <service-name> --tail 50

# Restart
docker compose restart <service-name>

# Atau rebuild (kalau image corrupted)
docker compose up -d --force-recreate <service-name>
```

### Docker Daemon Mati

```bash
systemctl status docker
systemctl restart docker

# Kalau nggak bisa:
journalctl -u docker --tail 50  # Cek error
```

### Disk Penuh (40GB)

```bash
# Cek
df -h && du -sh /var/lib/docker/

# Cleanup
docker system prune -a --volumes -f
find /home/lnh/backups -name "*.tar.gz" -mtime +14 -delete  # Old backups

# Or expand disk in Tencent Cloud console
```

---

## 📞 CONTACTS & LINKS

| Resource | Detail |
|---|---|
| VPS Console | cloud.tencent.com |
| DNS (Hostinger) | hpanel.hostinger.com |
| Forgejo | git.luqmanulh.my.id |
| Uptime Kuma | monitor.luqmanulh.my.id |
| Grafana | grafana.luqmanulh.my.id |
| GitHub Config | github.com/luqmanulh/vps-infra |

---

## 📝 NOTES

- **Backup files** ada di `/home/lnh/backups/` — TAPI ini di VPS!
- **PENTING:** Secara berkala, download backup dari VPS ke laptop:
  ```bash
  # Dari laptop
  rsync -avz -e 'ssh -p 2222' lnh@43.129.35.12:~/backups/ ~/vps-backups/
  ```
- **Uptime Kuma** akan kirim notifikasi Telegram kalau ada service down
- **Firewall rules** (UFW): port 22, 2222, 80, 443 only
