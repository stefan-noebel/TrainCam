# TrainCam

## Hardware

Achtung: Das System ist kompakt und wird im Betrieb sehr heiß. Die Kamerarückseite erreicht >60°C, also Verbrennungsgefahr.

* [Seeed Studio XIAO ESP32 S3 Sense](https://wiki.seeedstudio.com/xiao_esp32s3_getting_started/), 8 MB PSRAM, 8 MB Flash
* OV5640 5 MP-Kameramodul, 65° Sichtwinkel; Fokuspunkt auf Unendlich fixiert, Kleber am Einstellring lösen für näheren Fokuspunkt
* microSD-Karte SanDisk Ultra 32 GB
* Lithium Polymer-Akku 3,7 V, 250 mAh mit JST-Anschluss; Ladeanzeige blinkt rot beim Laden, erlischt falls voll

## Software

Fork des Projekts [s60sc/ESP32-CAM_MJPEG2SD](https://github.com/s60sc/ESP32-CAM_MJPEG2SD)

### Ersteinrichtung

1. WLAN einrichten:

Netzwerk der Kamera: ESP-CAM_MJPEG_[serial] beitreten -> [http://192.168.4.1](http://192.168.4.1) öffnen -> WLAN-Daten eintragen -> (Eingabe sorgfältig prüfen, sonst Gerät nicht mehr erreichbar) -> Connect

2.  http://ESP-CAM-MJPEG-[serial].fritz.box/ öffnen:

Website zur Steuerung der Kamera, Start der Aufzeichnung, Kopieren der Videos, etc. Falls nicht per lokalem hostname erreichbar, auf der Routerkonfigurationsseite nach zugewiesener IP-Adresse suchen und alternativ nutzen


3. NTP Zeit-Server setzen:

* Menü oben -> Edit Config -> Network -> NTP Server address: 0.de.pool.ntp.org -> Save
* Menü links -> Schrauenschlüssel-Symbol -> Select region: Europe -> Select Location: Berlin -> Save Settings

4. Motion Detection ausschalten:

Menü links -> Augen-Symbol -> Motion Detect auf Aus schalten -> Save Settings

5. Kameraeinstellungen optimieren:

* Menü links -> Fotoapparat-Symbol -> Resolution: HD(1280x720) -> FPS: 30 -> Quality: 4 -> Save Settings (--> entspricht effektiv ca. 24 FPS im Live Stream)
* Menü links -> Fernseher-Symbol -> Clock MHz: 24 -> Gain Ceiling: 300 -> Save Settings

Bisher die besten Parameter innerhalb der Vortests, um eine gute Bildqualität und ausreichend hohe Bildrate für ruckelfreie Kamerafahrten zu erhalten. Um einer Bildrate >25 FPS zu erreichen, muss die Auflösung auf VGA(640x480) reduziert werden. Die Helligkeit und Belichtung unbedingt per Parameter Gain Ceiling und evtl. Exposure Level, Brightness und Contrast an die Beleuchtungssituation auf der Anlage anpassen. Der Sensor OV5640 kann nicht mit aktuellen Smartphose oder gar Digitalkameras mithalten. Wichtig ist die ausreichend Bildrate und kein Rauschen oder andere Artefakte, wie bunte Linien oder Pixel.

6. Videostream separat öffnen oder speichern:

http://esp-cam-mjpeg-[serial].fritz.box/sustain?video=1 im Browser oder VLC media player öffnen

7. Dateien per WEBDAV kopieren (langsam: <200 kB/s):

webdav://esp-cam-mjpeg-[serial].fritz.box/webdav/ in Adressleiste des Windows Explorers eingeben

8. Dateien von microSD-Karte kopieren (per microSD -> SD-Card-Adapter):

SD-Card nur im stromlosen Zustand entnehmen oder einsetzen

9. Videodateien in gängige und archivierbare Formate umwandeln:

In den erzeugten .avi-Dateien liegen die Video- und Audiostreams als Rohdaten vor:
    
    Video: mjpeg (Baseline) (MJPG / 0x47504A4D), yuvj422p(pc, bt470bg/unknown/unknown), 1280x720, 17295 kb/s, 23 fps, 23 tbr, 23 tbn, 23 tbc
    Audio: Audio: pcm_s16le ([1][0][0][0] / 0x0001), 16000 Hz, mono, s16, 256 kb/s 
    
Diese können noch deutlich komprimiert werden. Dafür fehlt dem verwendeten ESP32S3 die nötige Rechenleistung. Am einfachsten ist es per Kommandozeile mit dem Programm ffmpeg
    
    for i in *.avi; do ffmpeg -i "$i" -filter:v "scale=-1:720" -vcodec libx264 -profile:v baseline -level 3.0 -pix_fmt yuv420p -acodec libfdk_aac "${i%.avi}_libx264_720p.mp4"; done
    
Das Ausgabeformat ist gut archivierbar und WhatsApp-tauglich. Mit dem VLC media player können die Datein ebenfalls geöffnet und konvertiert werden: Medien -> Konvertieren/Speichern -> Hinzufügen -> .avi-Datei(en) auswählen -> Konvertieren/Speichern -> Einstellungen: Konvertieren: Video - H.264 + MP3 (MP4) -> Zieldatei: Durchsuchen -> Dateiname auf .mp4 ändern -> Speichern -> Start

# offene Baustellen

* Die Kamera OV5640 bietet einen echten mechanischen Autofokus. Dieser muss extern über einen momentan nicht verbundenen Pin angesteuert werden. Erfordert noch etwas zusätzliche Bastel- und Lötarbeit, ist aber machbar.
https://www.youtube.com/watch?v=922BWy3OOoQ

* Der Ladezustand des Akkus wird aktuell nicht online angezeigt, da der Batterieeingang nicht standardmäßig mit einem Analog-Digital-Wandler-Pin (ADC) verbunden ist. Falls gewünscht, könnte die Batterie extern mit den Pins verbunden werden: Plus von Batterie an freien GPIO. Masse ist schon die gleiche Bezugsmasse auf der Platine (GND) und muss nicht nochmals verbunden werden.

* Updates: Die Software kann grundsätzlich unter Beibehaltung aller Einstellungen aktualisiert werden. Am einfachsten geht das in der Arduino IDE mit dem Code im Anhang. In aktualisierten Versionen des ESP32-CAM_MJPEG2SD-Projekt müssten jeweils alle markierten Anpassungen wieder übernommen werden. Das on-the-flight Updaten in der Weboberfläche per OTA habe ich noch nicht ausprobiert.