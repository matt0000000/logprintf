+++
title = "docker media server setup"
date = "2024-09-03"
description = "docker-compose.yml for my docker media server setup"
tags = [
  "debian",
  "docker"
]
+++

Below is my docker-compose.yml I use for my media server needs. Stack includes Sonarr, Radarr and Sabnzbd for download management. For streaming media it includes Emby. We tried both Plex and Jellyfin before Emby. Emby works best with our Samsung Smart TV so we stayed with Emby.

```bash

services:
  sabnzbd:
    image: linuxserver/sabnzbd
    container_name: sabnzbd
    environment:
      - PUID=1000 # Replace with your user ID
      - PGID=1000 # Replace with your group ID
    volumes:
      - ./config/sabnzbd:/config # Replace with the path to your config directory
      - ./downloads:/downloads
      - ./usenet:/usenet
      - /mnt/merged:/mnt/merged # Replace with the path to your media files
    ports:
      - 8080:8080
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000 # Replace with your user ID
      - PGID=1000 # Replace with your group ID
    volumes:
      - ./config/radarr:/config # Replace with the path to your config directory+
      - ./downloads:/downloads
      - ./movies:/movies
      - /mnt/merged:/mnt/merged # Replace with the path to your media files
    ports:
      - 7878:7878
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000 # Replace with your user ID
      - PGID=1000 # Replace with your group ID
    volumes:
      - ./config/sonarr:/config # Replace with the path to your config directory
      - ./downloads:/downloads
      - ./tv:/tv
      - /mnt/merged:/mnt/merged # Replace with the path to your media files
    ports:
      - 8989:8989
    restart: unless-stopped

  emby:
    image: emby/embyserver:latest
    container_name: emby
    environment:
      - UID=1000 # Replace with your user ID
      - GID=1000 # Replace with your group ID
    volumes:
      - ./config:/config # Replace with the path to your Emby config directory
      - /mnt/merged:/mnt/media # Replace with the path to your media files
    ports:
      - 8097:8096
      - 8920:8920 # Optional for HTTPS
    restart: unless-stopped

networks:
  default:
    driver: bridge

```

