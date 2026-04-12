# 🐉 The Green Dragon — Homelab Docs

Personal reference for my self-hosted homelab running on Unraid. If something breaks and I'm staring blankly at a terminal, this is where I come first.

> 📖 **Formatted guides:** [homelab-docs on GitHub Pages](https://hobbitsgit.github.io/homelab/)

---

## 🖥️ Hardware

| Component | Details |
|---|---|
| **Case** | Jonsbo N6 |
| **Motherboard** | MSI PRO B650M-A WiFi ProSeries |
| **CPU** | AMD Ryzen 5 8500G |
| **RAM** | 32GB G.Skill Flare X5 DDR5 |
| **Storage** | 3x Seagate IronWolf Pro 28TB HDD |
| **Cache / Appdata** | Samsung 990 Evo Plus 1TB NVMe SSD |
| **Backup** | 256GB SATA SSD (Unassigned Device) |
| **PSU** | Corsair RM1000x |
| **OS** | Unraid |

### Storage Layout
| Drive | Role | Size |
|---|---|---|
| IronWolf Pro #1 | Parity | 28TB |
| IronWolf Pro #2 | Array | 28TB |
| IronWolf Pro #3 | Array | 28TB |
| Samsung 990 Evo Plus | Cache + Appdata | 1TB |
| 256GB SATA SSD | Local Appdata Backup | 256GB |

**Usable storage:** ~56TB (single parity)

---

## 🐳 Services

| Container | Purpose | Port | Domain |
|---|---|---|---|
| Gluetun | VPN gateway for download clients | — | — |
| qBittorrent | Torrent client (behind Gluetun) | 8080 | qbittorrent.shiretech.dev |
| SABnzbd | Usenet client (behind Gluetun) | 8085 | sabnzbd.shiretech.dev |
| Prowlarr | Indexer manager | 9696 | prowlarr.shiretech.dev |
| Radarr | Automated movie management | 7878 | radarr.shiretech.dev |
| Sonarr | Automated TV & anime management | 8989 | sonarr.shiretech.dev |
| Bazarr | Subtitle management | 6767 | bazarr.shiretech.dev |
| Profilarr | Quality profile management | 6868 | profilarr.shiretech.dev |
| FlareSolverr | Cloudflare bypass for Prowlarr | 8191 | — |
| Jellyfin | Media server | 8096 | jellyfin.shiretech.dev |
| Jellyseerr | Media requests | 5055 | jellyseerr.shiretech.dev |
| Immich | Photo backup & management | 8086 | immich.shiretech.dev |
| Homarr | Homepage dashboard | 10004 | homarr.shiretech.dev |
| Pi-hole | Network-wide DNS filtering & ad blocking | 8155 | pihole.shiretech.dev |
| Nginx Proxy Manager | Reverse proxy + SSL certificates | 81 (admin) | — |
| Tailscale | Secure remote access (plugin) | — | — |
| Dozzle | Docker log viewer | 8888 | dozzle.shiretech.dev |
| Uptime Kuma | Service uptime monitoring | 3001 | uptime.shiretech.dev |
| Rangarr | Automated missing media searches | — | — |
| PostgreSQL | Database for Immich | 5433 | — |

📄 [Full setup guide →](https://hobbitsgit.github.io/homelab/docs/rebuild-guide.html)

---

## 🗺️ Planned / In Progress

- [ ] Home Assistant
- [ ] Kopia + Backblaze B2 offsite backups
- [ ] Resolve Immich machine learning issues (smart search / face detection)
- [ ] n8n AI IT Hobbit integration — once a GPU is installed
- [ ] Docker Compose file of whole stack for easy re-deployment

---

## 📁 Repo Structure

```
homelab/
├── index.html                    # Docs index (GitHub Pages home)
├── README.md                     # You are here
├── docs/                         # Documentation
│   └── ...
├── configs/                      # Config files (no secrets)
│   └── ...
└── scripts/                      # Utility scripts
    └── ...
```

---

## ⚠️ Notes

- Secrets, API keys, and passwords are **never** committed here — all configs use placeholder values
- When in doubt, check the GitHub Pages site for the full formatted guides
- Unraid appdata is backed up to the **256GB SATA SSD** mounted as `/mnt/disks/Backup`

---

*Built and maintained by Mr. Hobbit — mostly held together by second breakfast and an unwavering belief that if you just restart the container, it'll sort itself out.*
