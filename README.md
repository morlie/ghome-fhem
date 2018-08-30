```
sudo apt install certbot
sudo certbot
```



# Google Home/Assistant FHEM Connector

ghome-fhem verbindet FHEM mit Google Assistant und erlaubt dadurch die Nutzung der FHEM Geräte in Verbindung mit jedem Google Assistant fähigem Gerät. Dies ist ein Fork des ursprünglich von yanniks bereitgestellten Repositories. Ein großes Danke für seine Entwicklung!

## Vorbereitende Arbeiten
1. Fehlende Pakete installieren. Je nach Distribution können Pakete fehlen. 
```
sudo apt-get -qq install  git

#pwgen - optional zum generieren von Passwörtern
sudo apt-get -qq install pwgen 

sudo apt-get -qq install curl

#NPM installieren -- Achtung, sudo curl bis sudo -E bash - ist eine Zeile

sudo curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

sudo apt-get -qq install nodejs
```

## Optional, jetzt schon Passwörter und Benutzernamen mit pwgen erstellen

| Feldame in der Anleitung | pwgen Befehl | Beispiel |
|---|---|---|
|<change_me___oauthClientId>|pwgen -N 1 -s 8   |  wgbkHUA2 |
|<change_me___oauthClientSecret>|pwgen -N 1 -s 42   |G9T0TKc0qdrzWwYHurecO0IZYUf93qB80nJPZ4XAcx   |
|<change_me___oauthUser>|pwgen -N 1 -s 8   | wDgsn36x|
|<change_me___password>|pwgen -N 1 -s 42   | WqoHFS0FzNzeEfPPwvC5gRAnZCt5vvM8LVEF3aL4LQ  |
|<change_me___authtoken>| pwgen -N 1 -s 42  | agUACUoaCFQt2qFLcKzY2J0FDAOyIcjsGcOckpVBEo  |


## Domain registrieren z.B. bei ddnss.de (gratis)
1. Account bei ddnss.de anlegen
2. Autoupdate der IP einrichten
```
	sudo crontab -e
	39 * * * * /usr/bin/wget -q -O - "https://ddnss.de/upd.php?key=CHANGEME&host=CHANGEME.ddnss.de"
```

2. Alternativ kann den Update der IP auch der Router übernehmen wenn er diese Funktion bietet (z. B. Fritzbox). Anleitungen bieten idR. die Anbieter des dyndns-Dienstes.

## Zertifikat erstellen

Letsencrypt installieren

 1. Source zu apt hinzufügen

```
sudo echo "deb http://ftp.debian.org/debian stretch-backports main" >> /etc/apt/sources.list.d/letsencrypt.list
sudo apt-get -qq update
```

 2. Eigentliche Installation
 
```
sudo apt-get -qq install certbot
```

letsencrypt Zertifikat für diesen Host erstellen (unbedingt notwendig, ohne gültiges Zertifikat geht nichts!)
1. Port 80 auf RPi weiterleiten. Das muss am Router (z. B. Fritzbox) gemacht werden. Hierzu den externen Port 80 zum internen Port 80 auf den Raspberry weiterleiten. < Internet > -->Port 80--> < Router > -->Port 80--> < Raspberry >
2. certbot ausführen und Fragen beantworten. Der Parameter "--standalone" startet einen eigenen, temporären Webserver. Wenn nginx oder apache auf dem Server läuft diesen entweder beenden, oder certbot für den entsprechenden Webserver aufrufen. Anleitungen hierzu auf der Herstellerseite https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx.html
```
sudo certbot certonly --standalone 
```
3. Port 80 Weiterleitung entfernen
4. Bei jedem Zertifikatsrenew (certbot renew) muss Port 80 wieder weitergeleitet werden (alle 3 Monate)

## ghome-fhem installieren
1. GitHub repo lokal auschecken

```
cd $HOME
git clone https://github.com/dominikkarall/ghome-fhem
```

Jetzt sollte es Ordner /home/pi/ghome-fhem mit dem Inhalt des Git-Projektes geben. Wenn das Setup nicht mit dem User pi sondern mit einem anderen gemacht werden ist <pi> durch den enstsprechenden Namen zu ersetzen.


2. Im Ordner folgendes Kommando ausführen:
```
cd $HOME/ghome-fhem
npm install
```
3. config.json kopieren
```
cd $HOME
mkdir .ghome
cp ghome-fhem/config-sample.json .ghome/config.json
```

4. config.json anpassen

Im Beispiel alle <change_me__xxxx> durch generierte Zeichenfolge ersetzen. So stellt ihr sicher, dass der Zugang für unbefugte Personen zumindest erschwert wird.

```
{
    "ghome": {
        "port": 3000,
        "name": "Google Home",
        "keyFile": "./key.pem",
        "certFile": "./cert.pem",
        "nat-pmp": "",
        "nat-upnp": false,
        "oauthClientId": "<change_me___oauthClientId>",
                "oauthClientSecret": "<change_me___oauthClientSecret>",
                "oauthUsers": {
                        "<change_me___oauthUser>": {
                                "password": "<change_me___password>",
                                "authtoken": "<change_me___authtoken>"
                        }
                }
    },
    
    "connections": [
        {
            "name": "FHEM",
            "server": "127.0.0.1",
            "port": "8083",
            "webname": "fhem",
            "filter": "room=GoogleHome"
        }
    ]
}
```

Mit den Beispielwerten von oben würde die Datei so aussehen ... 
```
{
    "ghome": {
        "port": 3000,
        "name": "Google Home",
        "keyFile": "./key.pem",
        "certFile": "./cert.pem",
        "nat-pmp": "",
        "nat-upnp": false,
        "oauthClientId": "wgbkHUA2",
                "oauthClientSecret": "G9T0TKc0qdrzWwYHurecO0IZYUf93qB80nJPZ4XAcx",
                "oauthUsers": {
                        "wDgsn36x": {
                                "password": "WqoHFS0FzNzeEfPPwvC5gRAnZCt5vvM8LVEF3aL4LQ",
                                "authtoken": "agUACUoaCFQt2qFLcKzY2J0FDAOyIcjsGcOckpVBEo"
                        }
                }
    },
    
    "connections": [
        {
            "name": "FHEM",
            "server": "127.0.0.1",
            "port": "8083",
            "webname": "fhem",
            "filter": "room=GoogleHome"
        }
    ]
}
```


4. letsencrypt Zertifikat kopieren
```
sudo cp /etc/letsencrypt/DOMAIN/privkey.pem $HOME/ghome/ghome-fhem/key.pem
sudo cp /etc/letsencrypt/DOMAIN/fullchain.pem $HOME/ghome/ghome-fhem/cert.pem
```

5. Port 443 (extern) auf 3000 (intern, auf das Gerät wo ghome läuft) weiterleiten. Auch das muss wieder am Router gemacht werden. Hier ist zu beachten, dass sich externer und interner Port unterscheidet.

6. bin/ghome starten
Wenn die Schritte bis hierher korrekt ausgeführt wurde startet ghome mit etlichen Meldungen. 


<<<< TODO, Log posten  <<<<<



7. SystemD Dienst anlegen

Die Datei kann auch mit einem Editor angelegt werden und dann WinSCP oder anderem SSH-Client auf den Server gelegt werden. Hierbei ist zu beachten, dass die Formatiertung (End of Line) Unix-konform ist. Sonst läuft es möglicherweise nicht. Empfohlen wird nano oder vi Editor.


```
sudo nano /lib/systemd/system/ghome.service
```
Nun öffnet sich der Editor nano. 
- Inhalt des Scriptes unten mit < STRG > + < V > kopieren
- mit der RECHTEN Maustaste in der Console klicken, das überträge den kopierten Text
- mit < STRG > + < X > nano beenden. Die Frage ob gespeichert werden soll mit J/Y (je nach Sprache) beantworten.

Inhalt für das Script.
```
[Unit]
Description=Google Assistant FHEM Connector
After=network-online.target

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/ghome/ghome-fhem
ExecStart=/home/pi/ghome/ghome-fhem/bin/ghome
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=ghome.service
```

Service aktivierte damit ghome bei einem Systemstart mitgestartet wird

```
sudo systemctl enable ghome.service
```

Testen mit start (starten), stop (stoppen), status (Status wird angezeigt. Wenn der Diest läuft sollte ein grüner Punkt vor dem Prozess sein)

```
sudo service ghome start
sudo service ghome stop
sudo service status
```


## Google Action Projekt erstellen

Screenshots der einzelnen Schritte im "doc"-Ordner. (Google_Actions.docx)

1. https://console.actions.google.com/ Add/import project auswählen
2. Projektname FHEM-Connector
3. Home Control auswählen
4. Smart home auswählen
5. Overview - Quick Setup
   - Name your Smart Home action: FHEM Connector
   - Add account linking
     - Account creation: No, I only want to allow account creation on my website
     - Linking type: OAuth, Authorization code
     - Client information: ClientID (oauthClientId) und ClientSecret (oauthClientSecret) aus der config.json verwenden
     - Client information: Authorization URL (https://CHANGEME.ddnss.de/oauth), Token URL (https://CHANGEME.ddnss.de/token)
     - Testing instructions: "Schalte das Licht ein" eintragen
6. Overview - Build your Action
   - Add Action - Add your first Action
     - Create smart home action: URL https://CHANGEME.ddnss.de
   - Test Actions in the simulator
     - Testing rechts oben aktivieren, wenn nicht automatisch passiert
     
     
## Google action und lokalen Server bekannt machen

Den Inhalt der action.json kopieren und   <change_me__domainname>  durch die Domain die registriert wurde ersetzen (im Beispiel ddnss.de). Unter welcher der Dienst bei euch erreichbar ist.

action.json kopieren
```
cd $HOME
mkdir .ghome
cp ghome-fhem/config-action.json .ghome/action.json
```

Inhalt von .ghome/action.json anpassen (domainname)
```
{
  "actions": [
          {
              "name": "actions.devices",
              "deviceControl": {
              },
              "fulfillment": {
                "conversationName": "automation"
              }
    }
  ],
  "conversations": {
    "automation": {
      "name": "automation",
      "url": "<change_me__domainname>"
    }
  },
  "locale": "de"
}
```
8. gactions downloaden und ausführen

Download von hier: https://developers.google.com/actions/tools/gactions-cli

In diesem Codeblock wird die ARM Version (Raspberry) mit wget heruntergeladen.
```
cd $HOME/.ghome
wget -c https://dl.google.com/gactions/updates/bin/linux/arm/gactions
chmod +x gactions
./gactions update --action_package action.json --project <change_me__google_project_ID>
```
## Google Home App einrichten
In der Google Home-App auf einem Smartphone oder Tablet lässt sich nun im Smart Home-Bereich ein neuer Gerätetyp hinzufügen. In der Liste aller Typen taucht jetzt auch euer eigener auf, er beginnt mit [test].
   
Eventuell müsst ihr euer Konto mehrmals verknüpfen, bei mir hat es nicht immer beim ersten mal geklappt.

## Mögliche Kommandos
* “ok google, schalte <gerät> ein”
* “ok google, schalte das Licht im Raum <raum> aus”
* “ok google, stell die Temperatur in <raum> auf <wert> Grad”
* “ok google, dimme das Licht in Raum <raum> auf <anzahl> Prozent”
* “ok google, wie warm ist es in <raum>?“
* “ok google, ist das Licht in <raum> an?“
