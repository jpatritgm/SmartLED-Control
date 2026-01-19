# Smart LED-Control Dokumentation

Inhalt dieser Dokumentation:
1. [Idee und Projektstart](#idee-und-projektstart)
2. [Planung und Recherche](#planung-und-recherche)
3. [Erste Tests der HW](#erste-tests-der-hw)
4. [Einrichtung der Entwicklungsumgebung](#einrichtung-der-entwicklungsumgebung)
5. [Anforderungen](#anforderungen)
6. [Abgedeckte Kriterien](#abgedeckte-kriterien)
7. [Projektzeitplan](#projektzeitplan)
8. [Probleme und Lösungen](#probleme-und-lösungen)
9. [Ressourcen](#ressourcen)
10. [Recherche](#recherche)

# Idee und Projektstart
### Erste Ideen
- Licht steuern mit Sound/Tageszeit -> Home Assistant auf alten Laptop
- Display mit Informationen, wie Temperatur/Wetter -> touch Funktion
- +Licht steuern über Display (RGB?)
### Zusammenfasssung des Projekts
***Ziel dieses Projekts ist es, einen LED-Streifen mit einem Touch-Display zu steuern. Zusätzlich soll die Funktion implementiert werden, dass die Helligkeit des Lichts je nach derzeitiger Tageszeit(/Helligkeit) sich verändert - wenn es helllichter Tag ist, ist das Licht komplett aus, wenn es dunkler/später wird, so wird das Licht heller. Diese Funktion soll von Benutzer aktiviert/deaktiviert können werden. Damit das Display rund um die Uhr derzeitige Informationen liefern kann, wird Home Assistant über einen alten Laptop benutz. Als EK könnte man noch weitere Funktionen implementieren, wie z.B. auch andere Informationen, wie Wetter oder Temperatur am Display anzuzeigen.***
### Abzudeckende Kriterien
- Kommandozeile, (Linux,) Powershell, bash/sh
- Virtualisierung, Container
- Networking, Cloud
- Microcontroller, Datenerfassung, Session, Aktoren(?)
### Bewertungskriterien
**GKü** - *Die LED kann über das Display gesteuert werden (Helligkeit, An/Aus)*

**GKv** - *Die Funktion, dass die LED je nach Tageszeit ihre Helligkeit ändert, wurde implementiert und funktioniert korrekt.*

**EKü** - *Das Display kann weitere zusätzliche Informationen anzeigen, wie z.B. Datum/Uhrzeit, Wetter, Temperatur, usw.*

**EKv** - *Eine weitere nützliche Funktion wurde eingebaut (z.B. andere Sensoren, Funktionen)*

# Planung und Recherche
> Vor dem Arbeitsstart habe ich mir angeschaut, welche Komponenten ich für die Umsetzung brauche und welche Technologien am besten geeignet sind.
Ich habe mich entschieden, einen ESP32 oder Raspberry Pi zu verwenden, weil diese Geräte WLAN unterstützen und genug Leistung für die Displaysteuerung haben.  
Für die Daten (z. B. Wetter, Uhrzeit, Temperatur) möchte ich Home Assistant auf einem virtuellen Computer mit einem Linux Betriebssystem verwenden, um damit auch gleich den Bereich Kommandozeile abzudecken.  
Ich habe recherchiert, wie man Home Assistant installieren kann, und herausgefunden, dass man es gut in einem Docker-Container laufen lassen kann. Also habe ich mich entschieden, Home Assistant in Linux mit der Kommandozeile in einem Container zu installieren. Damit decke ich auch gleich das Thema Virtualisierung ab.
### Für den Hardware-Teil werden die folgenden Komponenten gebraucht
- Mikrocontroller (WLAN kompatibel)
- LED-Strip: 12 V, 30–60 LEDs pro Meter
- MOSFET (Metal-Oxide-Semiconductor Field-Effect Transistor)
- Touch-Display
- 2 Laptops (1 Arbeitslaptop, 1 „Server“ Laptop mit den Containern)
<img width="267" height="267" alt="image" src="https://github.com/user-attachments/assets/c8c705d3-fd17-4833-a04d-d20adbf261e3" />
<img width="266" height="266" alt="image" src="https://github.com/user-attachments/assets/ab28e94a-df5b-4222-b695-464f524eceea" />


# Erste Tests der HW
Als nächsten Schritt habe ich erste Tests mit meinen <ins>HW-Komponenten</ins> gemacht.
Ich habe ein kleines Testprogramm geschrieben, um die Funktionalität des Fotowiderstands und die Grundlegende Logik seiner Aufgabe zu testen.

-> Umso weniger Licht er bekommt, desto heller leuchtet die LED

Dazu habe ich folgendes Testprogramm verwendet:
```
const int ldrPin = 2;      // AO/Analog Pin
const int ledPin = 5;       // PWM Pin am ESP32
const int pwmChannel = 0;   // Kanal 0
const int freq = 5000;      // PWM Frequenz 5 kHz
const int resolution = 8;   // 8 Bit PWM

void setup() {
  ledcSetup(pwmChannel, freq, resolution);
  ledcAttachPin(ledPin, pwmChannel);
  Serial.begin(115200);
}

void loop() {
  int ldrValue = analogRead(ldrPin);       // LDR einlesen (0-4095)
  
  // Normalisieren auf 0-255
  int brightness = map(ldrValue, 0, 4095, 255, 0);  
  // Umkehrung: hell draußen -> LEDs dunkel, dunkel draußen -> LEDs hell

  // PWM ausgeben
  ledcWrite(pwmChannel, brightness);

  Serial.print("LDR: ");
  Serial.print(ldrValue);
  Serial.print(" -> LED Helligkeit: ");
  Serial.println(brightness);

  delay(200); // 200 ms Update
}
```

> Folgendes Bild zeigt auch noch eine mögliche zugehörige Schaltung


![image](https://github.com/user-attachments/assets/9f07c574-8327-4016-83fc-02aa4650a83b)

# Einrichtung der Entwicklungsumgebung
Als nächstes habe ich meine Entwicklungsumgebung vorbereitet, welche ich später verwenden werde.<br>
Ich habe auf einem alten Laptop, welcher als Server für Home Assistant dienen soll, Linux Ubuntu installiert und dort Docker installiert und eingerichtet. 
### Linux Ubuntu einrichten
Zuerst habe ich einen Boot-Stick mit dem Ubuntu 24.04.3 Iso-Image erstellt. Dazu habe ich das Programm Rufus verwendet. Als nächsten Schritt habe ich auf dem Laptop, der als Server dienen soll, aus dem Boot Menü (F12) vom Stick gebootet und Ubuntu installiert.

**Ubuntu Image:** https://ubuntu.com/download/desktop
<br>**Rufus:** https://rufus.ie/de/
### Tailscale
Danach habe ich auf diesem und meinem Schullaptop Tailscale installiert und getestet, ob eine stabile Verbindung möglich war.<br>
In Linux habe ich für die Installation von Tailscale den Befehl ``curl -fsSL https://tailscale.com/install.sh | sh`` verwendet. Um die Funktionalität davon zu prüfen, habe ich mit dem Befehl ``sudo tailscale up`` Tailscale in Ubuntu gestartet.<br>
Dann habe ich Tailscale übers Web auf dem anderen Arbeitslaptop (Windows) installiert und mich mit demselben Konto angemeldet, wie auch in Ubuntu. Dies war erfolgreich, das konnte man daran erkennen, dass die beiden Geräte auf der Website im selben Netzwerk erkennbar waren 
(-> heißt <ins>Tailnet</ins>).<br>

> Bild der Tailscale Web-Oberfläche

<img width="945" height="405" alt="image" src="https://github.com/user-attachments/assets/d7840175-abbb-4b45-b7ad-4a1fe35b6aa5" />
<br><br>

Sobald beide Geräte im Tailscale Netzwerk waren (in Linux zuerst mit ``sudo tailscale login`` in Tailscale einloggen), konnte ich mit ``ping XXX.XXX.XXX.XXX`` die Verbindung zu dem Linux-Laptop vom Windows Laptop aus testen und per SSH mit dem Befehl ``ssh benutzername@XXX.XXX.XXX.XXX`` (Tailscale ip in Linux mit ``tailscale ip`` herausfinden) auf das Linux Gerät, welches sich bei mir zu Hause befindet zugreifen und dort auch dann auch von der Schule aus arbeiten (oder auch von einem anderen Gerät  PC zuhause).

> Bild, wie von (Windows) PC auf Linux Laptop zugegriffen wird

<img width="945" height="387" alt="image" src="https://github.com/user-attachments/assets/8607d697-4f1f-4b08-9bd6-05dec9c03667" />
<br><br>

### Docker Container erstellen
Anschließend habe ich noch Home Assistant, ESPHome und Portainer in Docker Containern gestartet. Dazu habe ich eine YAML-Konfigurationsdatei erstellt. Um die Funktionalität von Home Assistant als Beispiel zu testen habe ich dann ``http://XXX.XXX.XXX.XXX:8123`` im Browser aufgerufen und mich dann in Home Assistant eingeloggt und bin auf folgende Seite gekommen. 

<img width="1047" height="414" alt="image" src="https://github.com/user-attachments/assets/246843d0-893e-4529-9681-895b3c2f783c" />
<br><br>

> YAML-Konfigurationsdatei für Docker-Container

```
version: '3.0'

services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce
    restart: always
    stdin_open: true
    tty: true
    ports:
      - "9000:9000/tcp"
    environment:
      - TZ=Europe/Vienna
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /opt/portainer:/data
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - /opt/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      TZ: Europe/Vienna
  esphome:
    container_name: esphome
    image: ghcr.io/esphome/esphome:stable
    volumes:
      - ./esphome_config:/config
      - /etc/localtime:/etc/localtime:ro
    network_mode: host 
    restart: always
```

### Home Assistant Konfigurationen
***Dashboard mit dem LCD-Display und LED-Streifen inkludiert***

<img width="945" height="388" alt="image" src="https://github.com/user-attachments/assets/1f28b026-25b4-4c07-b7a7-c70b70bbc265" />

<video src="assets/LEDControl.mp4">
</video>
[INSERT VIDEO]  

***LED-Streifen Entität***

> Das folgende Bild zeigt den LED-Streifen in Home Assistant als Entität realisiert

<img width="945" height="721" alt="image" src="https://github.com/user-attachments/assets/b5df5ea1-4ae4-4ccf-92c9-77bf5cb1e227" />
<br><br>

### Automationen

**Sonnenaufgang**
> Diese Automation steuert, dass 20 Minuten nachdem die Sonne am Horizont aufgeht, der LED-Streifen in Schritten von 5 Minuten/300 Sekunden dunkler wird, bis er schließlich komplett ausgeschalten ist. Diese Automation ist um 20 Minuten versetzt, da es oft noch nicht hell ist, auch wenn die Sonne schon am Horizont aufgegangen ist und es etwas Zeit braucht, bis das Licht uns erreicht.

<img width="945" height="500" alt="image" src="https://github.com/user-attachments/assets/87e619f4-30d9-49f3-8cfb-2968cd5f87ec" />
<br><br>

**Sonnenuntergang**
> Die 2. Automation bewirkt genau das Gegenteil der ersten. Sie automatisiert, dass der LED-Streifen ab 10 Minuten nach Untergang am Horizont heller wird. Dies erfolgt in Schritten von 10% alle 10 Minuten/600 Sekunden. Auch hier ist der Start der Automation um 10 Minuten versetzt, da es oft erst dunkel wird, schon nachdem die Sonne tatsächlich untergegangen ist.

<img width="945" height="500" alt="image" src="https://github.com/user-attachments/assets/b66ea38c-6207-4e76-9728-51ba23b14ecc" />


### ESPHome
ESPHome ist eine Firmware, welche ich ebenfalls als Container eingebaut habe (mehr siehe [ESPHome](https://esphome.io)).<br>

> ESPHome Page mit bereits hinzugefügten Geräten

<img width="945" height="277" alt="image" src="https://github.com/user-attachments/assets/98894389-c925-4f96-8ee1-9f12f6400635" />


> Konfigurationsdatei für LCD-Display

```
esphome:
  name: lcd-display
  friendly_name: LCD-Display

esp32:
  board: esp32dev
  framework:
    type: esp-idf

# Enable logging
logger:

# Enable Home Assistant API
api:
  encryption:
    key: "p951ySxZUJdnZcVduS/ZlFYbmcpFIU23b+KpzF7kJgY="

ota:
  - platform: esphome
    password: "PASSWORD1234"

wifi:
  ssid: "WIFI SSID"
  password: "WIFI PASSWORD"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Lcd-Display Fallback Hotspot"
    password: "PASSWORD1234"

captive_portal:

output:
  - platform: ledc
    pin: GPIO21
    id: backlight_pwm
  - platform: gpio
    pin: GPIO3
    id: gpio_3_out
    inverted: False
  - platform: ledc
    pin: GPIO1
    id: gpio_1_pwm
    inverted: False

light:
  - platform: monochromatic
    output: backlight_pwm
    name: Display Backlight
    id: backlight
    restore_mode: ALWAYS_ON
  - platform: binary
    output: gpio_3_out
    id: local_gpio3
    name: "GPIO 3 Out"
  - platform: monochromatic
    output: gpio_1_pwm
    id: local_gpio1
    name: "Test LED Dimmbar"
    gamma_correct: 2.8

spi:
  - id: tft
    type: single
    clk_pin: GPIO14
    mosi_pin: GPIO13
    miso_pin: GPIO12
  - id: touch
    clk_pin: GPIO25
    mosi_pin: GPIO32
    miso_pin: GPIO39

lvgl:
  buffer_size: 25%    
  touchscreens:
    - my_touch
  style_definitions:
    - id: header_footer
      bg_color: 0x2F8CD8
      bg_grad_color: 0x005782
      bg_grad_dir: VER
      bg_opa: COVER
      border_opa: TRANSP
      radius: 0
      pad_all: 0
      pad_row: 0
      pad_column: 0
      border_color: 0x0077b3
      text_color: 0xFFFFFF
      width: 100%
      height: 30
    
    - id: slider_style_main
      bg_color: 0xDDDDDD
      radius: 10
      border_width: 0
    - id: slider_style_indicator
      bg_color: 0x2F8CD8
      radius: 10
    - id: slider_style_knob
      bg_color: 0xFFFFFF
      border_width: 2
      border_color: 0x2F8CD8
      radius: 20
      pad_all: 5 # Macht den Knopf größer für Touch   
  pages:
    - id: main_page
      widgets:
        - obj:
            align: TOP_MID
            styles: header_footer
            widgets:
              - label:
                  text: "Home Assistant"
                  align: CENTER
                  text_align: CENTER
                  text_color: 0xFFFFFF
        - obj:
            align: bottom_mid
            styles: header_footer
            widgets:
              - label:
                  text: "Julian Patri 3AHIT"
                  align: CENTER
                  text_align: CENTER
                  text_color: 0xFFFFFF  
        - switch:
            id: light_switch
            align: CENTER
            y: -40
            on_click:
              light.toggle: local_gpio3

        - label:
            id: brightness_label
            text: "0 %"
            align: CENTER
            y: 0 # Genau in der Mitte zwischen Switch und Slider
            text_color: 0x000000 # Schwarz für bessere Lesbarkeit

        - slider:
            id: brightness_slider
            align: CENTER
            y: 40   # Nach unten verschoben
            width: 200
            height: 20
            min_value: 0
            max_value: 100
            value: 75
            
            animated: true
            anim_time: 1000ms
            #indicator: percentage
            on_value:
              then:
                - lvgl.label.update:
                    id: brightness_label
                    # String zusammenbauen und am Ende in C-String umwandeln
                    text: !lambda 'return ("Helligkeitswert: " + std::to_string((int)x) + " %").c_str();'
                - if:
                    condition:
                      # Wenn Wert 0 ist -> Ausschalten
                      lambda: 'return x == 0;'
                    then:
                      - light.turn_off: local_gpio1
                    else:
                      # Sonst -> Einschalten mit Helligkeit
                      - light.turn_on:
                          id: local_gpio1
                          # 'x' ist 0-100, brightness erwartet 0.0 bis 1.0 (float)
                          brightness: !lambda "return x / 100.0;"
            
touchscreen:
  platform: xpt2046
  id: my_touch
  spi_id: touch
  cs_pin: GPIO33  
  interrupt_pin: GPIO36
  update_interval: 50ms
  threshold: 400
  transform:
    swap_xy: true
    mirror_x: false
    mirror_y: false
  calibration:
    x_min: 280
    x_max: 3860
    y_min: 340
    y_max: 3860

display:
  - platform: ili9xxx
    spi_id: tft
    model: ILI9341
    dc_pin: GPIO2
    cs_pin: GPIO15
    rotation: 90
    invert_colors: False


```

# Anforderungen
  
### Funktionale Anforderungen
- Steuerung des LED-Strips über das Touch-Display
  - Ein/aus
  - Regelung der Helligkeit
- Automatische Helligkeitsanpassung
  - LED bei Sonnenschein aus
  - LED wird abends automatisch heller
  - LED kann (in Home Assistant) aktiviert/deaktiviert werden
- (EK) Anzeige von weiteren Informationen am Display über Home Assistant
  - Datum, Uhrzeit
  - Wetterinformationen
  - Temperatur
- Verbindung zu Home Assistant für Datenbereitstellung 

### Nicht-funktionale Anforderungen 
- Gute Performance: z.B. Reaktionszeit von <1 Sekunde von Eingaben über das Display
- Sicherheit: Stabile Verbindung zwischen Display und Home Assistant z.B. mithilfe von Netzwerkprotokollen
- Benutzerfreundlichkeit: eine benutzerfreundliche GUI auf dem Display
- Dauerbetrieb des Displays ohne Absturz/selbstständiges Auschalten


# Abgedeckte Kriterien
### Soweit wurden die folgenden Kriterien abgedeckt
***Kommandozeile (Linux, bash, sh)***

> Der Bereich Kommandozeile wurde mit der Einrichtung von Linux Ubuntu und Verwaltung von Home Assistant, ESPHome (und Portainer) auf dem 2. Laptop und dem Testen von Netzverbindungen (ping, curl, ssh) abgedeckt.

***Virtualisierung***

> Das Kriterium Virtualisierung wird durch den Betrieb von Home Assistant und ESP Home in Docker Containern auf dem Server-Laptop erfüllt.

***Networking***
> Networking ist in diesem Projekt dank der Kommunikation zwischen dem ESP32, dem Touch-Display und Home Assistant über WLAN ebenfalls enthalten. Über Home Assistant wird auf Daten, wie Wetter- und Temperaturdaten aus dem Internet zugegriffen. Außerdem wurden IP-Adressen, Ports und API-Schnittstellen konfiguriert.

***Mikrocontroller***
> Auch der Bereich Mikrocontroller wird in meinem Projekt erfüllt. Der ESP32C3 steuert den LED-Strip über GPIO an und der ESPS3 (erhalten im Display) setzt die GUI für das Display um. Außerdem wurden noch die Logik für automatische Helligkeitsanpassung und Speicherung von Benutzereinstellungen (Automatik an/aus) umgesetzt.

# Projektzeitplan
### Woche 1-4: Ideensammlung, Grundrecherche
*In dieser Zeit habe ich erste Ideen für das Projekt gesammelt, mir die Meinungen anderer angehört und erste Grundrecherche gemacht, was ich für dieses Projekt brauchen werde und ob es so gut umsetzbar ist, wie ich es mir vorgestellt hatte.*
### Woche 4-8: Hardwarebereitstellung, Beginn der Word-Dokumentation
*Hier habe ich mich damit beschäftigt, was die beste HW-Option für das Projekt wäre und nach reichlicher Recherche dann die nötige Hardware, also primär den LED-Strip und das MOSFET-Modul bestellt. Außerdem habe ich eine einfache Dokumentation in Word begonnen, um die Schritte möglichst zeitnah zu dokumentieren und wichtige Informationen festzuhalten.*
### Woche 8-10: Einrichtung Linux, Docker, Home Assistant
*In dieser Zeitspanne habe ich Linux Ubuntu auf meinem alten Laptop zuhause eingerichtet und Docker, sowie Home Assistant in einem Container installiert.*
### Woche 10-14: Datenanbindung an Home Assistant – Idee ESP-Home
*Etwas später bin ich dann auf die Integration ESPHome gestoßen und fand, dass sie gut zu meinem Projekt passen und hilfreich sein würde. Aus diesem Grund habe ich sie dann auch noch eingebunden und die Datenanbindung in Home Assistant vollendet. Da ich ESPHome davor noch nicht kannte, musste ich mich etwas länger damit beschäftigen, bevor ich loslegen und es benutzen konnte.*
### Woche 14-15: Entwicklung und Tests der Grundfunktionen (LED steuern mit LDR usw.)
*Erst hier habe ich erste Grundfunktionen geschrieben und Tests mit einem LDR/Photowiderstand durchgeführt, um die LEDs passend anzusteuern.*
### Woche 15-16: Intensive Auseinandersetzung mit Display (einige Probleme siehe [Probleme und Lösungen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#probleme-und-lösungen))
*In dieser Zeit habe ich mich dann mit dem Display auseinandergesetzt und habe schon relativ früh bemerkt, dass es nicht sonderlich viel Information und Dokumentationen zu meinem Display gibt. Dies hat einige Probleme mit sich gebracht, welche mehrere Tage andauerten. Eines dieser Probleme war, dass die Touch-Funktion auf dem Display nicht funktioniert hat und ich für die YAML-Datei nicht den korrekten Touch-Driver im Internet finden konnte. Leider konnte ich das Problem am Ende nicht lösen und musste auf ein anderes Display umsteigen, welches weniger Probleme bereitete, welche ich jedoch lösen konnte.*
### Woche 16-17: Letzte Schritte, in Richtung Projektende gehen, Dokumentation vollenden und überarbeiten
*Dies waren die letzten paar Wochen, in welchen in schon Richtung Projektende arbeiten und die Dokumentation vervollständigen musste. Jedoch gab es auch hier immer noch wenige Probleme, wie z.B. dass ich nicht den An/Aus Switch und Helligkeitsregler auf dem neuen Display korrekt anordnen konnte.*
### Woche 18: Übersichtlichere Dokumentation auf GitHub
*Hier habe ich die Idee bekommen, meine eher einfache Word-Dokumentation etwas professioneller zu gestalten und auf stattdessen auf GitHub umzusetzen.*
### Woche 19-20: Präsentation des Projekts
*In Woche 20 wird das Projekt präsentiert.*

# Probleme und Lösungen
### *Problem #1* - Das Display
Das mit Abstand größte Problem stellte das Display, welches ich erstmals für das Projekt verwenden wollte, dar ([ESP32-8048s070c](https://www.tinytronics.nl/en/development-boards/microcontroller-boards/with-wi-fi/jingcai-esp32-8048s070c-7-inch-tft-lcd-display-800*480-pixels-with-touchscreen-esp32-s3)). Es gab kaum Dokumentation oder Konfigurations-Informationen online zu finden, außerdem war keine Beschreibung oder Information zu Treibern vom Hersteller beigegeben. Stundenlanges Ausprobieren verschiedenster Treiber für den Touchscreen hat mich auch nicht viel weiter gebracht, also habe ich mich nach einigen Tagen dazu entschieden, auf ein anderes, bekannteres Display mit mehr Informationen zu der Konfiguration umzusteigen - das [ESP32-32e LCD-Display](https://www.lcdwiki.com/2.8inch_ESP32-32E_Display).

### WLAN-Probleme
Weiters hatte ich manchmal das Problem, das der ESP32 von dem Display und mein anderer ESP32C3 nicht in dem WLAN Netzwerk gefunden wurden und ich somit nicht ihre IP-Adresse herausfinden und sie direkt ansprechen konnte. Jedoch konnte ich dieses Problem relativ leicht lösen, indem ich ihnen direkt eine IP-Adresse mit der Code-Zeile ``use_adress: XXX.XXX.XXX.XXX`` zuwies.
# Ressourcen
### Hardware
- Touch-Display (mit eingebautem ESP32S3)
- 12V LED-Strip 
- MOSFET Modul für Leistungsregelung
- 2 Laptops (1x für Grundarbeiten, 1x als Server für Home Assistant)
### Software
- Home Assistant
- ESPHome
-	Docker
-	Linux Ubuntu
### Wissen
- Programmierung des Mikrocontrollers
-	API-Integrationen
-	Schreiben einer YAML-Konfigurationsdatei
-	Grundlagen für Virtualisierung (Docker)
-	Grundlagen in der Linux-Kommandozeile, sowie Wissen über grundlegende Befehle 

# Recherche

## Quellen
- Linux Ubuntu: https://ubuntu.com/download/desktop, https://docs.docker.com/desktop/setup/install/linux/ubuntu/
- Docker/Docker Compose: https://docs.docker.com/engine/install/ubuntu/, https://docs.docker.com/compose/
- Home Assistant: https://www.home-assistant.io/installation/linux
- Tailscale: https://tailscale.com, https://tailscale.com/kb/1031/install-linux
- ESPHome Konfigurationen: https://esphome.io/cookbook/
  - Labels - https://esphome.io/components/lvgl/widgets/#label
  - Switch - https://esphome.io/components/lvgl/widgets/#switch
  - Slider - https://esphome.io/components/lvgl/widgets/#slider
  - Animation/Time - https://esphome.io/guides/configuration-types/#time
  - Styling - https://esphome.io/components/lvgl/#lvgl-styling
- LCD-Display: https://www.lcdwiki.com/2.8inch_ESP32-32E_Display, https://www.youtube.com/watch?v=NvBblQnWhsQ
