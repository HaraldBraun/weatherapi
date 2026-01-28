# Weather API - LabVIEW Actor Framework

Dieses Repository enthält eine Implementierung der [Weather API Challenge](https://roadmap.sh/projects/weather-api-wrapper-service) von roadmap.sh. Das Ziel ist es, einen Wrapper-Service für Wetterdaten bereitzustellen, der Anfragen entgegennimmt, externe Wetter-APIs konsumiert und die Ergebnisse effizient verarbeitet.

Die Besonderheit dieses Projekts liegt in der Verwendung von **NI LabVIEW** und dem **Actor Framework**, um eine skalierbare, objektorientierte und modulare Architektur abzubilden.

## Architektur & Features

Das Projekt basiert auf dem **Actor Model**, was eine saubere Trennung von Verantwortlichkeiten (Separation of Concerns) ermöglicht:

- **Modularität:** Jeder Dienst (z. B. der Wetter-Client) läuft als eigenständiger Actor.
- **Fehlertoleranz:** Nutzung der integrierten Error-Handling-Mechanismen des Actor Frameworks zur Überwachung der Prozesse.
- **REST-Integration:** Anbindung an externe Wetter-Provider über HTTP-Requests.
- **JSON-Parsing:** Schnelle und zuverlässige Datenverarbeitung der API-Antworten.

## Voraussetzungen

Um dieses Projekt auszuführen oder weiterzuentwickeln, werden folgende Komponenten benötigt:

1. **LabVIEW:** Empfohlen Version 2020 oder neuer.
2. **JKI JSON Library:** Diese Bibliothek wird zwingend für das Parsen der Wetterdaten benötigt.
   - Die Installation erfolgt am einfachsten über den [JKI VI Package Manager (VIPM)](https://www.vipm.io/package/jki_lib_json/).
3. **Actor Framework:** Ist standardmäßig in LabVIEW enthalten.

## Projektstruktur

Das Repository ist nach Best-Practices für LabVIEW-Projekte organisiert:

- `Weather API.lvproj`: Die Hauptprojektdatei für LabVIEW.
- `Weather Actor.lvclass`: Enthält die Logik für den Abruf und die Verarbeitung der Wetterdaten.
- `Messages`: Beinhaltet die spezifischen Actor-Messages zur Kommunikation zwischen den Komponenten.
- `Documentation`: (Falls vorhanden) Enthält Diagramme zur Actor-Hierarchie.

## Installation & Verwendung

1. **Repository klonen:**
   ```bash
   git clone [https://github.com/HaraldBraun/weatherapi.git](https://github.com/HaraldBraun/weatherapi.git)
   ```

2. **Abhängigkeiten prüfen:** Stelle sicher, dass die JKI JSON Bibliothek über den VIPM installiert ist.

3. **Projekt öffnen:** Start die Datei Weather API.lvproj in LabVIEW.

4. **Splash Screen VI starten:** Suche im Projekt-Explorer nach dem Main-Entry-Point (meist Main.vi oder der Root-Actor) und starte die Ausführung.

## Konfiguration
Das Programm benötigt einen gültigen API-Schlüssel eines Wetter-Dienstanbieters (z. B. Visual Crossing oder OpenWeatherMap). Dieser kann in den Initialisierungsparametern des Weather-Actors hinterlegt werden.

## Roadmap.sh Challenge
Dieses Projekt deckt folgende Anforderungen der Challenge ab:

- [x] Abrufen von Wetterdaten über eine externe API.

- [x] Caching-Logik für API-Antworten (optional/in Arbeit).

- [x] Konsolen- oder GUI-basierte Ausgabe der Ergebnisse.

---

Entwickelt von [Harald Braun](https://github.com/HaraldBraun).
