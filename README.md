# Simulatore di Telecamere RTSP con Docker

Questo progetto permette di simulare **tre telecamere RTSP** utilizzando file video locali, senza bisogno di hardware reale.
È pensato per studenti che devono lavorare con flussi RTSP (ad esempio per progetti di riconoscimento targhe) e che possono usare **Docker Desktop** ma non installare software aggiuntivo.

Il sistema utilizza:

- **MediaMTX** come server RTSP/HLS/WebRTC
- **FFmpeg** per trasmettere i video come se fossero telecamere reali
- **Docker Compose** per avviare tutto con un solo comando

## Versionamento automatico

La versione corrente del progetto è riportata nel file `VERSION`.

Le immagini Docker usano volutamente il tag `latest`:

- `bluenviron/mediamtx:latest`
- `jrottenberg/ffmpeg:latest`

Una GitHub Action controlla automaticamente i digest delle due immagini. Quando rileva che una delle immagini `latest` è cambiata:

1. incrementa la **minor version** del progetto (`1.0.0` → `1.1.0`);
2. aggiorna `VERSION`;
3. salva i nuovi digest;
4. crea un tag Git `v1.1.0`;
5. crea una GitHub Release con le note generate automaticamente.

Il primo controllo registra solamente lo stato iniziale delle immagini e non genera una release. I controlli successivi generano una nuova minor version solo quando almeno una delle immagini è cambiata.

Il controllo viene eseguito automaticamente ogni giorno e può essere avviato anche manualmente dalla scheda **Actions** di GitHub.

---

# Struttura del progetto

La cartella deve essere organizzata così:

```
simulazione-telecamere/
│
├── docker-compose.yml
├── mediamtx.yml
└── video/
    ├── cam1.mp4
    ├── cam2.mp4
    └── cam3.mp4
```

### Cartella `video/`
Contiene i file video che verranno trasmessi come telecamere.

- `cam1.mp4` → Telecamera 1
- `cam2.mp4` → Telecamera 2
- `cam3.mp4` → Telecamera 3

Puoi sostituire questi file con qualsiasi video (MP4 consigliato).

---

## Avvio del sistema

Assicurati di essere nella cartella del progetto, poi esegui:

```bash
docker compose up
```

Docker scaricherà le immagini necessarie e avvierà:

- il server RTSP (MediaMTX)
- tre istanze FFmpeg che trasmettono i video come telecamere

# URL delle telecamere
RTSP (per OpenCV, VLC, ecc.)

```
rtsp://localhost:8554/camera1
rtsp://localhost:8554/camera2
rtsp://localhost:8554/camera3
```

# Interfaccia Web (HLS/WebRTC)

Le immagini Docker utilizzano `latest`, quindi al riavvio Docker può scaricare una versione più recente di MediaMTX o FFmpeg. Le release Git del progetto permettono di tenere traccia di questi aggiornamenti.

# Come funziona
## MediaMTX
- È un server RTSP/HLS/WebRTC leggero e stabile.
- Accetta flussi in ingresso e li rende disponibili ai client.

## FFmpeg
Ogni telecamera è un container FFmpeg che:

- legge un file video dalla cartella `video/`
- lo riproduce in loop infinito
- lo invia a MediaMTX tramite RTSP

# Modificare i video
Per cambiare i video delle telecamere:

- Sostituisci i file nella cartella `video/`
- Mantieni gli stessi nomi (`cam1.mp4`, `cam2.mp4`, `cam3.mp4`)

Riavvia Docker:

```bash
docker compose down
docker compose up
```

## Aggiungere altre telecamere

Puoi duplicare uno dei servizi `cameraX` nel `docker-compose.yaml` e cambiare:

- il nome del servizio
- il file video
- il nome dello stream RTSP

Esempio:

```yaml
camera4:
  image: jrottenberg/ffmpeg:latest
  depends_on:
    - mediamtx
  command: >
    -re -stream_loop -1 -i /video/cam4.mp4
    -c:v libx264 -preset veryfast -tune zerolatency
    -f rtsp rtsp://mediamtx:8554/camera4
  volumes:
    - ./video:/video:ro
  restart: unless-stopped
```

# Arrestare il sistema

```bash
docker compose down
```

# Requisiti

- Docker CLI o Docker Desktop (Windows+WSL, macOS, Linux)
- Nessuna installazione aggiuntiva
- Qualsiasi file video MP4
