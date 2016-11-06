# Hevi Setup


## Hardware

### Einkaufsliste

- Raspberry Pi  ([Amazon](https://www.amazon.de/Raspberry-Pi-3-Model-B/dp/B01CEFWQFA))
    - USB Netzteil
    - Netzwerk Kabel
- USB to RS232 Adapter (z.B.: [Amazon](https://www.amazon.de/Ugreen-Seriell-Adapter-Konverter-USB-Seriell/dp/B00QUZY4R4))
- Nullmodemkabel (z.B.: [Amazon](https://www.amazon.de/Cables-81417-Nullmodemkabel-Buchse-schwarz/dp/B002DEM02C))

### Alternative: RS232 Adapter

Es ist nicht zwingend notwendig einen USB to RS232 Adapter zu kaufen. Der Raspberry Pi verfuegt ueber einen TxD und einen RxD Pin, die man anstatt des Adapters benutzen kann. Einziges Manko: Der Raspberry Pi benutzt intern 3.3V und nicht +/-12V die fuer RS232 benoetigt werden. Mit einem [MAX3232CPE](http://www.reichelt.de/MAX-3232-CPE/3/index.html?ACTION=3&LA=446&ARTICLE=9398&artnr=MAX+3232+CPE&SEARCH=max3232cpe) und ein paar 0.1µF Kondensatoren kann man einen Adapter basteln. Ich werde hier nicht ins Detail gehen. Wer mehr darueber wissen will kann sich [hier](http://elinux.org/RPi_Serial_Connection) und [hier](http://codeandlife.com/2012/07/01/raspberry-pi-serial-console-with-max3232cpe/) informieren. 

### Alternative: Nullmodemkabel

Mit ein bisschen geschick und einem Loetkolben kann man selbst ein Nullmodemkabel basteln. 
Der Schaltplan sieht wir folgt aus:

![Nullmodemkabel](https://raw.githubusercontent.com/schlan/assets/master/froelingio/doku/nullmodemkabel.png)


## Installation

### 1. Raspberry Pi - Grundinstallation

Als Betriebssytem fuer den Raspberry Pi empfehle ich Rasbian. Es wird keine Desktop Oberflaeche benoetigt. Ich werde hier nicht darauf eingehen wie man den Raspberry einrichtet, eine genaue Beschreibung findet ihr [hier](https://www.raspberrypi.org/help/) und [hier](https://www.raspbian.org/FrontPage).

Im weiterem Verlauf der Anleitung gehe ich davon aus, dass der Raspberry mit dem Internet verbunden ist und man sich per SSH einloggen kann. 

#### 2. Verkabelung

Die Verkabelung sieht wie folgt aus:

```
                                                            Kessel
                                                            +----+
                                                            |    |
Internet  +--------------+ USB to RS232   Nullmodemkabel    |    |
----------| Raspberry Pi |--------------|-------------------|    |
          +--------------+                              COM1|    |
                                                            +----+ 
```

Der Raspberry Pi wird per USB to RS232 und Nullmodemkabel an die Service Schnittstelle des Kessels angeschlossen. 
Die Service Schnittstelle (COM1) findet man unter der Abdeckung, auf der Oberseite des Kessels. Siehe Fotos:

<a href="https://github.com/schlan/assets/raw/master/froelingio/doku/abdeckung.jpg"><img src="https://github.com/schlan/assets/raw/master/froelingio/doku/abdeckung.small.jpg" width="48%"></a>&nbsp;&nbsp;<a href="https://github.com/schlan/assets/raw/master/froelingio/doku/schnitstelle.jpg"><img src="https://github.com/schlan/assets/raw/master/froelingio/doku/schnitstelle.small.jpg" width="48%"></a>

### 3. Installation des Hevi Clients

#### Am Raspberry Pi einloggen
Per SSH am Raspberry Pi anmelden. Sollte man die Installation von einem Windows PC ausfuerhen kann man [PuTTY](http://www.putty.org/) benutzen um sich einzuloggen, unter Linux oder macOS per Terminal:

```
$ ssh root@<ip-address>
```

Alle weiteren Schritte sind fuer alle Platformen gueltig.

#### Download des Hevi Clients

Erstelle ein Verzeichnis namens `hevi` im  Homeverzeichnis von `root`

```
[pi@pi ~]$ mkdir ~/hevi
[pi@pi ~]$ cd hevi
```

Download der neuesten Clients Version:

```
[pi@pi hevi]$ wget https://github.com/schlan/hevi_client/releases/download/v0.2.0/hevi-0.2.0-arm.tar.gz
```

(Ich versuche den Link stets zu atkualisieren. Zur Sicherheit findet ihr [hier](https://github.com/schlan/hevi_client/releases) die Liste aller verfuegbaren Versionen.)

Entpacken:

```
[pi@pi hevi]$ tar -xvf hevi-0.2.0-arm.tar.gz
[pi@pi hevi]$ ln -s hevi_0.2.0_arm/hevi hevi
```

#### Konfiguration des Clients

Zum Erstellen der Konfiguration benutze ich den Editor `nano`. Eine Uebersicht der Kommandos gibts es [hier](http://www.codexpedia.com/text-editor/nano-text-editor-command-cheatsheet/).

Erstelle und oeffne eine Konfigurationsdatei:

```
[pi@pi hevi]$ nano hevi.config
```

Kopiere und fuege das folgende Snippet in den Editor:

```
[Main]
port = /dev/ttyUSB0
device_token = <device_token>
```

  - Mit `port` ist die Serielle Schnittstelle gemeint. Benutzt man einen USB to RS232 Adapter sollte `/dev/ttyUSB0` zutreffen.
    Um das zu testen fuehre folgendes Befehl aus:

    ```
    [pi@pi hevi]$ ls /dev/ttyUSB*
    ```

    Die Ausgabe des Kommandos liefert alle verfuegbaren seriellen Schnittstellen.

  - Das `device_token` bekommt man auf [froeling.io](https://froeling.io), unter `Settings` --> `Device Token`

    ![](https://github.com/schlan/assets/raw/master/froelingio/doku/device_token.png)

Speicher und schliesse die Datei mit `CTRL + X`.


#### Test die Verbindung zum Kessel

Der nachfolgende Befehl testet die Verbindung zum Kessel.

```
[pi@pi hevi]$ ./hevi --config hevi.config --test
```

Ist alles richtig eingerichtet sollte die Ausgabe so aussehen:

```
[    INFO] Hello froeling.io!
```

#### Heizkreise auslesen

Dieser Schritt ist optional und nur noetig wenn mindestens ein Heizkreis vorhanden ist. 

Zum auslesen der Heizkreise folgenden Befehl ausfuehren:

```
[pi@pi hevi]$ ./hevi --config hevi.config --genconfig
```

!!*Dieser Befehl kann ein paar Minuten Zeit in anspruch nehmen*

Wenn ich den Befehl auf meinem Raspberry ausfuehre sieht die Ausgabe ungefaehr so aus:

```
Please append the following to your existing configuration:


[hevi_hc|Heizkreis 04]
...

[hevi_hc|Heizkreis 02]
...

[hevi_hc|Heizkreis 03]
...

```

Je nach Konfiguration, kann die Ausgabe unterschiedlich aussehen.


Alles nach 

```
Please append the following to your existing configuration:
```

wird mit `CTRL+C` kopiert.

Dann die Konfigurationsdatei mit `nano` oeffnen und die Heizkreis-Konfiguration am Ende der Datei einfuegen.

#### Cron Job einrichten

Zuletzt muss noch ein Cron Job eingerichtet werden der regelmaessig Daten vom Kessel ausliest und zu froeling.io schickt.

Die Konfigrationsdatei von Cron wird mit folgenden Befehl geoffnet:

```
[pi@pi hevi]$ EDITOR=nano crontab -e
```

Copy&Paste die nachfolgende Zeile in den offenen Editor.

```
*/5 * * * * ~/hevi/hevi --config ~/hevi/hevi.config --submit
```

Danach schliesse und speicher die Datei mit `CTRL + X`.


#### Fin!
Nach ein paar Minuten sollten die ersten Daten auf froeling.io zu sehen sein. Unter `Settings` --> `Dashboard` kann das Dashboard nach belieben konfiguriert werden.

