# Backup de Contêineres Docker - CoreELEC

**Data:** Outubro de 2026
**Sistema:** TV Box (CoreELEC)
**Armazenamento Principal:** `/var/media/SSD_DATA`

---

## 1. Telegram Bot (qBittorrent Manager)
Comando exato utilizado para subir o bot integrado à API do qBittorrent.

```bash
docker run -d --restart always --name telegram-bot --net=host -v /var/media/SSD_DATA:/var/media/SSD_DATA coreelec-telegram-bot