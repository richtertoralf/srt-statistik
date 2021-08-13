# srt-statistik
Testumgebung 

IP-Kamera -> rtsp-Stream -> localServer (ffmpeg -> **srt-live-transmit**) -> SRT-Stream -> (cloudServer) ffmpeg -> rtmp-Server
```
# Install srt-live-transmit aus den Ubuntu Paketquellen 
sudo apt install srt-tools

# oder srt selbst kompilieren:
sudo apt-get install tclsh pkg-config cmake libssl-dev build-essential
cd ~
git clone https://github.com/Haivision/srt  
cd srt  
PATH="$HOME/bin:$PATH"  
./configure  
make  
sudo make install

# Install ffmpeg with srt (static build downloaden)
cd ~
wget https://johnvansickle.com/ffmpeg/builds/ffmpeg-git-amd64-static.tar.xz
tar xvf ffmpeg-git-amd64-static.tar.xz
rm ffmpeg-git-amd64-static.tar.xz
sudo mv ~/ffmpeg-git-*/* /usr/local/bin/
```

localServer Programm-Pipe 
`# JSON Prozessor für die Shell`  
`sudo apt install jq`   

```
# rtsp-Stream zu mpegts umwandeln und als srt-Stream verpacken
ffmpeg -i 'rtsp://192.168.95.55:554/1/h264major' -c copy -f mpegts 'srt://localhost:1995'
# srt-Stream mit srt-live-transmit auswerten und zum rtmp-Server in die Cloud weiterleiten
# die Statistikdaten im json-Format in Datei stats.log gespeichert
srt-live-transmit srt://:1995?mode=listener srt://217.160.70.147:1995 -s 1000 -pf json -statsout:stats.log
# letzten Datensatz aus der Dtei stats.log auslesen und der Variablen last_stat zuweisen
last_stat=$( tail -n 1 stats.log )
# Testen, ob der letzte Datensatz vollständig ist und ausgewertet werden kann. Testen,
# ob die Anzahl der Klammern '{' und '}' übereinstimmen -> json o.k.
if [ $(echo $last_stat | grep -o '{' | wc -w) == $(echo $last_stat | grep -o '}' | wc -w) ]; then echo "json o.k."; else echo "json error"; fi
# einzelne Werte auswählen und ausgeben
echo $last_stat | jq .link.rtt
echo $last_stat | jq .link.bandwidth
echo $last_stat | jq .recv.mbitRate
```
