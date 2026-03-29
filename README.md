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
| **OS** | Unraid |

### Storage Layout
| Drive | Role | Size |
|---|---|---|
| IronWolf Pro #1 | Parity | 28TB |
| IronWolf Pro #2 | Array | 28TB |
| IronWolf Pro #3 | Array | 28TB |
| Samsung 990 Evo Plus | Cache + Appdata | 1TB |

**Usable storage:** ~56TB (single parity)

---

## 🐳 Services

### 📺 Media Stack
| Container | Purpose | Port |
|---|---|---|
| Gluetun | VPN gateway for download clients | — |
| qBittorrent | Torrent client (behind Gluetun) | 8080 |
| SABnzbd | Usenet client (behind Gluetun) | 8081 |
| Prowlarr | Indexer manager | 9696 |
| Radarr | Automated movie management | 7878 |
| Sonarr | Automated TV management | 8989 |
| Jellyfin | Media server | 8096 |
| Jellyseerr | Media requests | 5055 |

📄 [Full setup guide →](docs/media_stack_setup.html)

---

### 🌐 Network
| Container | Purpose | Port |
|---|---|---|
| Pi-hole | Network-wide ad blocking | 80 |
| WireGuard | VPN for remote access | 51820 |

---

### 🏠 Home Automation
| Container | Purpose | Port |
|---|---|---|
| Home Assistant | Home automation hub | 8123 |

---

### 🔁 Reverse Proxy
| Container | Purpose | Port |
|---|---|---|
| Nginx Proxy Manager | SSL + subdomain routing | 81 (admin) |

---

### 🤖 AI / Automation
| Container | Purpose | Port |
|---|---|---|
| n8n | Workflow automation / AI IT team | 5678 |

The plan here is to wire up a few n8n agents that act as an AI-powered IT team — automated monitoring, alerts, self-healing tasks, and general homelab management without me having to babysit everything.

---

## 📁 Repo Structure

```
homelab-docs/
├── index.html                    # Docs index (GitHub Pages home)
├── README.md                     # You are here
├── unraid-media-stack-guide.html # Media stack setup guide
├── configs/                      # Config files (no secrets)
│   └── ...
└── scripts/                      # Utility scripts
    └── ...
```

---

## 🗺️ Planned / In Progress

- [ ] Reverse proxy config and subdomain map
- [ ] Pi-hole + WireGuard setup guide
- [ ] Home Assistant configuration notes
- [ ] n8n AI IT agent workflows
- [ ] Full Docker Compose file for the whole stack
- [ ] Backup strategy documentation

---

## ⚠️ Notes

- Secrets, API keys, and passwords are **never** committed here — all configs use placeholder values
- When in doubt, check the GitHub Pages site for the full formatted guides
- Unraid appdata is backed up to `[YOUR BACKUP LOCATION]`

---

*Built and maintained by [YOUR_NAME] — mostly held together by caffeine and stubbornness.*
