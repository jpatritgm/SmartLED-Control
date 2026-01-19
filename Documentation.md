# Smart LED-Control Dokumentation

Inhalt dieser Dokumentation:
1. [Idee und Projektstart](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#idee-und-projektstart)
2. [Planung und Recherche](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#planung-und-recherche)
3. [Erste Tests der HW](#erste-tests-der-hw)
4. [Einrichtung der Entwicklungsumgebung](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#einrichtung-der-entwicklungsumgebung)
5. [Anforderungen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#anforderungen)
6. [Abgedeckte Kriterien](#abgedeckte-kriterien)
7. [Projektzeitplan](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#projektzeitplan)
8. [Probleme und Lösungen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#probleme-und-lösungen)
9. [Ressourcen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#ressourcen)
10. [Recherche](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#recherche)

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

<a name="my-custom-anchor-point"></a>
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
