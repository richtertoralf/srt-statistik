# srt-statistik mit json und jq in der Bash
**erste Spielereien, um die in das SRT-Protkoll eingebauten Statistiken zu nutzen**  

meine Testumgebungen:  
- **IP-Kamera** -> rtsp-Stream -> "localServer (ffmpeg -> **srt-live-transmit**)" -> SRT-Stream -> (cloudServer) ffmpeg -> rtmp-Server  
- **TestStreamGenerator** in der Cloud (FFmpeg) -> **SRT** -> **srt-live-transmit** local auf dem Rechner -> **UDP** -> **OBS-Studio**  

## srt-live-transmit installieren
```
# srt-live-transmit aus den Ubuntu Paketquellen installieren
sudo apt install srt-tools
```
```
# oder srt selbst kompilieren:
sudo apt-get install tclsh pkg-config cmake libssl-dev build-essential
cd ~
git clone https://github.com/Haivision/srt  
cd srt  
PATH="$HOME/bin:$PATH"  
./configure  
make  
sudo make install
```
## FFmpeg mit der srt-Bibliothek installieren
Ich bin faul und hole mir einen fertigen "Build".  
```
# Install ffmpeg with srt (static build downloaden)
cd ~
wget https://johnvansickle.com/ffmpeg/builds/ffmpeg-git-amd64-static.tar.xz
tar xvf ffmpeg-git-amd64-static.tar.xz
rm ffmpeg-git-amd64-static.tar.xz
sudo mv ~/ffmpeg-git-*/* /usr/local/bin/
```
## json Parser für die Auswertung installieren  
`# JSON Prozessor für die Shell - getestet mit Ubuntu 20.04`  
`sudo apt install jq`   

# spielen mit json und jq in der Bash
```
# rtsp-Stream zu mpegts umwandeln, als srt-Stream verpacken und am Port 1995 ausspielen
ffmpeg -i 'rtsp://192.168.95.55:554/1/h264major' -c copy -f mpegts 'srt://localhost:1995'
# srt-Stream mit srt-live-transmit auswerten und zum rtmp-Server in die Cloud weiterleiten
# und die Statistikdaten im json-Format in die Datei stats.log speichern
srt-live-transmit srt://:1995?mode=listener srt://217.160.70.147:1995 -s 1000 -pf json -statsout:stats.log
# oder
# einen SRT-Stream abholen, z.B. vom *StreamGenerator aus der Cloud* https://github.com/richtertoralf/testStreamGenerator.git  
# und die Statistikdaten speichern:  
srt-live-transmit srt://23.88.52.184:9999?mode=caller udp://localhost:50099 -v -s 200 -pf json -statsout:stats.log  
#
# dann, den jeweils letzten Datensatz aus der Datei stats.log auslesen und der Variablen last_stat zuweisen 
last_stat=$( tail -n 1 stats.log )
# dann ein simpler Test, ob der letzte Datensatz vollständig ist und ausgewertet werden kann.
# Testen, ob die Anzahl der Klammern '{' und '}' übereinstimmen -> json o.k.
if [ $(echo $last_stat | grep -o '{' | wc -w) == $(echo $last_stat | grep -o '}' | wc -w) ]; then echo "json o.k."; else echo "json error"; fi

# wenn o.k. dann einzelne Werte auswählen und ausgeben, z.B.:
echo $last_stat | jq .link.rtt
echo $last_stat | jq .link.bandwidth
echo $last_stat | jq .recv.mbitRate

# und hier die kurze Variante, ohne erst etwas in eine Datei zu schreiben:  
srt-live-transmit srt://23.88.52.184:9999?mode=caller udp://localhost:50099 -q -s 200 -pf json | jq ' .link.rtt, .link.bandwidth, .recv.mbitRate'
```
