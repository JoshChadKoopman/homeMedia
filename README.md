# HomeMedia App 📺🎵🎥
**HomeMedia App** is a personal media server solution designed to streamline and automate the process of managing, downloading, and streaming media content.  
Built on a Docker Portainer cluster, it integrates powerful tools like Emby, Jackett, Sonarr, Radarr, and qBittorrent, delivering a seamless media management experience.

---

## Features 🌟
- **Emby**: Organize, stream, and manage your personal media library with a user-friendly interface.  
- **Jackett**: Acts as a bridge between torrent indexers and automation tools like Sonarr and Radarr.  
- **Sonarr**: Automates the downloading and organization of TV series.  
- **Radarr**: Handles automated downloading and organization of movies.  
- **qBittorrent**: Efficient, lightweight torrent client for downloading media.  
- **Portainer**: Simplifies Docker container management with an easy-to-use GUI.  

---

## Tech Stack 🛠️
- **Docker**: For containerizing applications.  
- **Portainer**: For managing Docker containers and services.  
- **Docker Compose**: To define and run multi-container applications.  

---

## Docker Compose Configuration 🐳
Below is the `docker-compose.yml` file used for setting up the HomeMedia server:  

```yaml
## Portainer Docker compose that I am using to create the homeMedia server
---
version: "3.0"
services:
  # =============== Jackett(NEW) ==================== #
  jackett:
    image: lscr.io/linuxserver/jackett:latest
    container_name: jackett
    environment: 
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
    ports:
      - 9117:9117
    expose:
      - "9117"
    volumes:
      - /Users/joshkoopman/Desktop/portainer/jackett:/config
      - /Users/joshkoopman/Downloads:/downloads
    restart: unless-stopped
    network_mode: mediaNet
  # ================================================= #
  # ============== Sonarr(NEW) ================== #
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
    ports:
      - 8989:8989
    volumes:
      - /Users/joshkoopman/Desktop/portainer/sonarr:/config
      - /Users/joshkoopman/Downloads:/downloads
    restart: unless-stopped
  # ================================================= #
  # ============== Radarr(NEW) ================== #
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
    ports:
      - 7878:7878
    volumes:
      - /Users/joshkoopman/Desktop/portainer/radarr:/config
      - /Users/joshkoopman/Downloads:/downloads
    restart: unless-stopped
    network_mode: bridge
  # ================================================= #
  # ============== qBittorrent(NEW) ================= #
  qBittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qBittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
      - WEBUI_PORT=8080
    ports:
      - 8080:8080
    expose:
      - "8080"
    volumes:
      - /Users/joshkoopman/Desktop/portainer/qbittorrent:/config
      - /Users/joshkoopman/Downloads:/downloads
    restart: unless-stopped
  # ================================================= #
  # ============== Emby(NEW) ================== #
  emby:
    image: emby/embyserver
    container_name: embyserver
    environment:
      - PUID=1000
      - PGID=100
      - PGIDLIST=100
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
    ports:
      - 8096:8096
    expose:
      - "8096"
    volumes:
      - /Users/joshkoopman/Desktop/portainer/emby:/config
      - /Users/joshkoopman/Downloads:/mnt/share1
      - /Users/joshkoopman/Downloads:/mnt/share2
    restart: unless-stopped
  # ================================================= #
  # ============== Kodi(NEW) ================== #
  kodi-headless:
    image: linuxserver/kodi-headless
    container_name: kodi-headless
    environment:
      - PUID=1001
      - PGID=1001
      - PGIDLIST=100
      - TZ=Africa/Johannesburg
      - AUTO_UPDATE=true
    ports:
      - 9777:9777
    expose:
      - "9777"
    volumes:
      - /Users/joshkoopman/Desktop/portainer/kodi:/config/.kodi
    restart: unless-stopped
  # ================================================= #
  
 ```

## Then we creating the link when the internet goes down for the new IP
```shell
sudo /usr/local/opt/ddclient/bin/ddclient -force -verbose -noquiet
```

## Define a mediaNet network to expose the services properly:
```yaml
version: "3.0"
services: 


networks:
  mediaNet:
    name: mediaNet
    driver: macvlan
    driver_opts:
      parent: en0
    ipam:
      config:
        - subnet: "255.255.255.0/24"
        - ip_range: "10.0.0.0/32"
        - gateway: "10.0.0.254"
```