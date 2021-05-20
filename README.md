# srt-statistik
Testumgebung 

IP-Kamera -> rtsp-Stream -> localServer (ffmpeg -> **srt-live-transmit**) -> SRT-Stream -> (cloudServer) ffmpeg -> rtmp-Server
```
# Install srt-live-transmit
sudo apt install srt-tools
# Install ffmpeg with srt
cd ~
wget https://johnvansickle.com/ffmpeg/builds/ffmpeg-git-amd64-static.tar.xz
tar xvf ffmpeg-git-amd64-static.tar.xz
rm ffmpeg-git-amd64-static.tar.xz
sudo mv ~/ffmpeg-git-*/* /usr/local/bin/
```

localServer Programm-Pipe 
```
# rtsp-Stream zu mpegts umwandeln und als srt-Stream verpacken
ffmpeg -i 'rtsp://192.168.95.55:554/1/h264major' -c copy -f mpegts 'srt://localhost:1995'
# srt-Stream mit srt-live-transmit auswerten und zum rtmp-Server in die Cloud weiterleiten
# die Statistikdaten im json-Format in Datei stats.log gespeichert
srt-live-transmit srt://:1995?mode=listener srt://217.160.70.147:1995 -s 1000 -pf json -statsout:stats.log
# letzten Datensatz aus der Dtei stats.log auslesen und der Variablen last_stat zuweisen
last_stat=$( tail -n 1 stats.log )
# testen ob die Anzahl der Klammern '{' und '}' übereinstimmen -> json o.k.
if [ $(echo $last_stat | grep -o '{' | wc -w) == $(echo $last_stat | grep -o '}' | wc -w) ]; then echo "json o.k."; else echo "json error"; fi
```
