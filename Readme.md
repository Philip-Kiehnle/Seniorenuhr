# Seniorenuhr
Da meine liebe Oma so langsam etwas vergesslich wird und manchmal ihren geschätzten Tag bei der Tagespflege vergisst, musste eine Lösung her. Eine Seniorenuhr mit Wochentaganzeige sollte es sein. Gute, ausgediente Smartphones gibt es in dieser Welt genug, also warum nicht eins davon benutzen. Eine App zu entwickeln ist nicht nötig, da eine Webseite auch alle benötigten Funktionen bietet. Zusätzlich zu Uhrzeit und Tag, kann auch für jeden Wochentag ein Termin eingestellt werden.

Die Seite kann über [https://Philip-Kiehnle.github.io/Seniorenuhr](https://Philip-Kiehnle.github.io/Seniorenuhr) aufgerufen werden und wird dann lokal verwaltet.

# Tests mit Samsung Galaxy S5

Da kein Akku mehr vorhanden war, wird das Galaxy S5 mit einem "Akkuersatz" betrieben.
Genutzt wird ein Samsung 2A Netzteil. Ein USB-Kabel mit Schottky Diode in Serie wird direkt an die Akkupins gelötet. Ein zusätzlicher 150 µF Kondensator sorgt für Spannungsstabilisierung. Zwischen Pin 2 und Pin 3 (-) wurde ein 2,2 kOhm Widerstand gelötet, um eine passende Akkutemperatur zu simulieren.
Bei Anschluss über die USB Ladebuchse bleibt das Gerät auch mit 4850 µF Kapazität als Akkuersatz in einer Bootloop. 150 µF mit direkter Versorgung reichen auch wenn der Flugmodus verlassen wird. Getestet ohne Sim-Karte mit maximaler Helligkeit. Peak 9,4 W auf ELV Energy Master.

Bei der Messung des Stromverbrauchs wurde der Termin: "morgen Tagespflege 😊" angezeigt.
Gemessen mit ELV Energy Master.
| Modus                                    | Displayhelligkeit 100 % | Displayhelligkeit ~50 % | black page    | Disp Off |
| ---------------------------------------- | ----------------------- | ----------------------- | ------------- | -------- |
| Flugmodus + WLAN + weißer Hintergrund    | 1,6-1,7 W               | 1,0-1,3 W               | 0,5-0,7 W     |  0,05 W  |
| Flugmodus + WLAN + schwarzer Hintergrund | 0,7-0,8 W               | **0,6-0,8 W**           | **0,5-0,7 W** |  0,05 W  |
| Flugmodus + weißer Hintergrund           | 1,6-1,7 W               | 0,9-1,1 W               | 0,5-0,6 W*    |  0,0 W   |
| Flugmodus + schwarzer Hintergrund        | 0,7-0,8 W               | 0,6-0,7 W               | 0,5-0,6 W*    |  0,0 W   |

*ohne WLAN wird der Schriftzug "keine Internetverbindung" eingeblendet, auch im black page Modus während der Nacht.

# Lokaler Webserver statt github pages

## HTTP Server mit Python
```
python3 -m http.server

# Um die Wechselrichteranzeige aus der produktiven https Seite zu testen, müssen die dummy Daten mit dem Header "Access-Control-Allow-Origin: *" gesendet werden. Dazu:
python3 simple_cors_server.py
```
Kein wakelock support für http! Display geht nach timeout aus. Bei Galaxy S5 nur 10 Minuten Timeout konfigurierbar. Bis zu 24 Tage sollen per cmd konfigurierbar sein, aber das ist kein Lösung. Stattdessen wird https genutzt.

## HTTPS Server mit lighthttpd
```
sudo apt-get install lighttpd 

sudo nano /etc/lighttpd/lighttpd.conf
# Verzeichnis einstellen, z.B.
server.document-root       = "/var/www/html"

# Cert erzeugen
# Im Heimnetz lokale IP als FQDN, z.B. 192.168.2.117
cd /etc/lighttpd/certs
openssl req -new -x509 -keyout lighttpd.pem -out lighttpd.pem -days 365 -nodes
chmod 400 lighttpd.pem

# Cert angeben
sudo nano /etc/lighttpd/lighttpd.conf

$SERVER["socket"] == ":443" {
  ssl.engine = "enable" 
  ssl.pemfile = "/etc/lighttpd/certs/lighttpd.pem" 
}
```

# Wechselrichter Ansicht
Um den Wechselrichter im lokalen Netzwerk über http abzufragen, muss im Browser mixed-content erlaubt werden, falls die Website über https geladen wird.  
Dazu im Browser [chrome://flags/](chrome://flags/) die Inverter URL bei: "Insecure origins treated as secure" eintragen.