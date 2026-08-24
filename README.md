# homelab

Docker Compose definitions for my self-hosted services.

## Ports

| Service | Port |
|-|-|
| qBittorrent | 8080 |
| Jellyfin | 8096 |
| Radarr | 7878 |
| Sonarr | 8989 |
| Prowlarr | 9696 |
| Bazarr | 6767 |
| Syncthing | 8384 |


## Host setup

Config directories must exist before first start.

Jellyfin's official image runs as root, so it only needs the directories:

```bash
sudo mkdir -p /opt/containers/jellyfin/{config,cache}
```

The LinuxServer.io containers run as `1000:1000` and will fail to write unless ownership matches:

```bash
sudo mkdir -p /opt/containers/{qbittorrent,prowlarr,sonarr,radarr,bazarr,syncthing}/config
sudo chown -R 1000:1000 /opt/containers/{qbittorrent,prowlarr,sonarr,radarr,bazarr,syncthing}
```

The library directories also need to be writable by `1000:1000`:

```bash
sudo mkdir -p /mnt/storage/media/{downloads,tv,movies}
sudo chown -R 1000:1000 /mnt/storage/media
```

## Application setup

One-time wiring that can't live in compose. Everything is host-networked, so services address each other over `localhost`.

1. **Prowlarr** (`:9696`) - set authentication, add indexers.
2. **Sonarr** (`:8989`) - set authentication; add a root folder under `/media` (e.g. `/media/shows`); add qBittorrent as a download client at host `localhost`, port `8080`.
3. **Radarr** (`:7878`) - same, with a movies root folder (e.g. `/media/movies`).
4. **Prowlarr -> Settings -> Apps** - add Sonarr (`http://localhost:8989`) and Radarr (`http://localhost:7878`) using each app's API key from Settings → General. Indexers then sync automatically.
5. **qBittorrent** (`:8080`) - confirm the default save path is under `/media`.
6. **Bazarr** (`:6767`) - set authentication; under Settings -> Sonarr and Settings -> Radarr, point at `localhost:8989` / `localhost:7878` with each app's API key. Under Settings -> Languages, define a languages profile and set it as the default for series and movies. Under Settings -> Providers, add at least one subtitle provider (OpenSubtitles.com needs a free account).
7. **Jellyfin** (`:8096`) - add libraries pointing at `/media/shows` and `/media/movies`.
8. **Syncthing** (`:8384`) - set a GUI username/password; add `/media` as a synced folder and configure remote devices as needed.
