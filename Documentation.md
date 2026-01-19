# Smart LED-Control Dokumentation

Inhalt dieser Dokumentation:
1. [Idee und Projektstart](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#idee-und-projektstart)
2. [Planung und Recherche](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#planung-und-recherche)
3. [Erste Tests der HW](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#erste-tests-der-hw)
4. [Einrichtung der Entwicklungsumgebung](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#einrichtung-der-entwicklungsumgebung)
5. [Anforderungen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#anforderungen)
6. [Abgedeckte Kriterien](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#abgedeckte-kriterien)
7. [Projektzeitplan](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#projektzeitplan)
8. [Ressourcen](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#ressourcen)
9. [Recherche](https://github.com/jpatritgm/SmartLED-Control/edit/main/Documentation.md#recherche)

# Idee und Projektstart
### Erste Ideen
- Licht steuern mit Sound/Tageszeit -> Home Assistant auf alten Laptop
- Display mit Informationen, wie Temperatur/Wetter -> touch Funktion
- +Licht steuern über Display (RGB?)
### Zusammenfasssung des Projekts
> Ziel dieses Projekts ist es, einen LED-Streifen mit einem Touch-Display zu steuern. Zusätzlich soll die Funktion implementiert werden, dass die Helligkeit des Lichts je nach derzeitiger Tageszeit(/Helligkeit) sich verändert - wenn es helllichter Tag ist, ist das Licht komplett aus, wenn es dunkler/später wird, so wird das Licht heller. Diese Funktion soll von Benutzer aktiviert/deaktiviert können werden. Damit das Display rund um die Uhr derzeitige Informationen liefern kann, wird Home Assistant über einen alten Laptop benutz. Als EK könnte man noch weitere Funktionen implementieren, wie z.B. auch andere Informationen, wie Wetter oder Temperatur am Display anzuzeigen.
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

# Erste Tests der HW

# Einrichtung der Entwicklungsumgebung

# Anforderungen

# Abgedeckte Kriterien

# Projektzeitplan

# Ressourcen

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
