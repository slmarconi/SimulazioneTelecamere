# Simulatore di Telecamere RTSP con Docker

Questo progetto permette di simulare **tre telecamere RTSP** utilizzando file video locali, senza bisogno di hardware reale.  
È pensato per studenti che devono lavorare con flussi RTSP (ad esempio per progetti di riconoscimento targhe) e che possono usare **Docker Desktop** ma non installare software aggiuntivo.

Il sistema utilizza:

- **MediaMTX** (ex rtsp-simple-server) come server RTSP/HLS/WebRTC
- **FFmpeg** per trasmettere i video come se fossero telecamere reali
- **Docker Compose** per avviare tutto con un solo comando

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
Apri nel browser:

```
rtsp://localhost:8554/camera1
rtsp://localhost:8554/camera2
rtsp://localhost:8554/camera3
```

Da qui puoi:

- vedere i flussi in HLS
- vedere i flussi in WebRTC (bassa latenza)
- verificare che le telecamere siano attive

# Come funziona
## MediaMTX
- È un server RTSP/HLS/WebRTC leggero e molto stabile.
- Accetta flussi in ingresso e li rende disponibili ai client.

## FFmpeg
Ogni telecamera è un container FFmpeg che:

- legge un file video dalla cartella video/
- lo riproduce in loop infinito
- lo invia a MediaMTX tramite RTSP

# Modificare i video
Per cambiare i video delle telecamere:

- Sostituisci i file nella cartella video/
- Mantieni gli stessi nomi (cam1.mp4, cam2.mp4, cam3.mp4)
Riavvia Docker:

```
docker compose down
docker compose up
```

## Aggiungere altre telecamere
Puoi duplicare uno dei servizi cameraX nel docker-compose.yml e cambiare:
- il nome del servizio
- il file video
- il nome dello stream RTSP

Esempio:

```
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
- Docker cli o Docker Desktop (Windows+WSL, macOS, Linux)
- Nessuna installazione aggiuntiva
- Qualsiasi file video MP4