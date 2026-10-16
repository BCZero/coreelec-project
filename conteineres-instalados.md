### Jellyfin

docker run -d \
  --name=jellyfin \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=America/Sao_Paulo \
  -p 8096:8096 \
  -v /storage/media/jellyfin:/config \
  -v /storage/media:/media \
  --restart unless-stopped \
  lscr.io/linuxserver/jellyfin:latest

  ## Consulte se está rodando
  docker logs -f jellyfin

  ## Reiniciar jellyfin
  docker restart jellyfin

### Qbittorrent

docker run -d \
  --name=qbittorrent \
  -e PUID=0 \
  -e PGID=0 \
  -e TZ=America/Sao_Paulo \
  -e WEBUI_PORT=8080 \
  -p 8080:8080 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v /storage/media/qbittorrent/config:/config \
  -v /storage/media/qbittorrent/downloads:/downloads \
  -v /storage/media/qbittorrent/animes:/animes \
  --restart unless-stopped \
  lscr.io/linuxserver/qbittorrent:14.3.9

  ## Testar conteineres operantes
  docker ps

