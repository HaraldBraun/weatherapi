# README – Open‑Meteo Wetterabfrage mit Geocoding & LabVIEW AF
Dieses Dokument beschreibt:

Wie Open‑Meteo funktioniert
Warum manche PLZs nicht unterstützt werden
Wie man zuverlässig Orte findet
Wie die Datenstrukturen aussehen
Wie man eine Vorschlagsliste (Auto‑Suggest) implementiert
Den Ablauf für Geocoding → Forecast → Anzeige
Die JSON‑Cluster (LabVIEW‑tauglich)


## 1. Open‑Meteo – Überblick
Open‑Meteo bietet:

eine kostenlose Wetter‑API ohne API‑Key
eine kostenlose Geocoding‑API, um Orte und Koordinaten zu finden

Die Forecast‑API arbeitet ausschließlich koordinatenbasiert (lat/lon).
 [api.openmeteo.com]
Zur Vorverarbeitung dient die Geocoding‑API:
Sie nimmt Ortsnamen oder Postleitzahlen und liefert dazu passende Orte zurück.
 [digitalvalley.de]

## 2. Problem: Warum funktioniert PLZ 97334 nicht?
Die API akzeptiert zwar PLZ, aber nur, wenn diese in der GeoNames‑Datenbank enthalten sind (Open‑Meteo basiert vollständig auf GeoNames).
 [digitalvalley.de]

Französische Übersee‑Regionen (z. B. Cayenne) haben PLZ‑Blöcke 97xxx → diese werden erkannt
Deutsche PLZ sind NICHT vollständig im Datensatz enthalten
PLZ „97334“ matched daher Cayenne, GF und nicht Nordheim am Main

➡ Daher ist die Verwendung von PLZ unzuverlässig.
➡ Die einzig zuverlässige Methode ist die Ortsnamen‑Suche + Country‑Filter.

## 3. Lösung: Ortsname + countryCode
Um die Suche zuverlässig und filterbar zu machen, nutzt man:
'''
name=<Ortsname>
countryCode=<ISO-Ländercode>
language=de
count=20
'''

Beispiel für „Nordheim“ nur in Deutschland:
'''Plain Text
https://geocoding-api.open-meteo.com/v1/search?name=Nordheim&countryCode=DE&language=de&count=20Weitere Zeilen anzeigen
'''

Die API liefert mehrere Treffer (Nordheim am Main, Nordheim in BW, Hessen, Thüringen usw.), jeweils mit Koordinaten.

## 4. Vorgehen: Dynamische Vorschlagsliste (Auto‑Suggest)
Damit der Benutzer den richtigen Ort auswählen kann, wird eine dynamische Trefferliste aufgebaut:
Ablauf:

User tippt in ein Textfeld (z. B. „Nord“)
UI schickt an Geocoding‑Actor:
'''
GetSuggestions("Nord", "DE")
'''

Geocoding‑Actor ruft API auf:
'''Plain Text
https://geocoding-api.open-meteo.com/v1/search?name=Nord&countryCode=DE&count=10&language=deWeitere Zeilen anzeigen
'''

API liefert Liste von möglichen Orten
UI‑Actor füllt die Suggestion‑Listbox / Dropdown
User klickt auf einen Eintrag
UI extrahiert latitude + longitude
UI sendet Abfrage an Weather‑Actor → Forecast‑API


## 5. Forecast‑Abfrage mit Koordinaten
Open‑Meteo Forecast API nutzt ausschließlich Koordinaten.
 [api.openmeteo.com]
Beispiel für Nordheim am Main:

'''Plain Text
https://api.open-meteo.com/v1/forecast?latitude=49.8587&longitude=10.18545&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,wind_speed_10m,relative_humidity_2mWeitere Zeilen anzeigen
'''
Du erhältst:

* aktuelle Werte (current)
* stündliche Werte (hourly)
* Datum/Zeit‑Arrays
* Zeitzone


## 6. Datenmodell (Cluster) für Geocoding‑Antwort
Die Geocoding‑API liefert:
'''JSON
{  "results": [...],  
   "generationtime_ms": 0.193}  
'''

Cluster „GeocodingResponse“
'''Plain Text
GeocodingResponse Cluster  
├─ results[] : Array of GeocodingResult  
└─ generationtime_ms : DBL

'''
### Cluster „GeocodingResult“
Basierend auf der API‑Struktur:
 [digitalvalley.de]
Plain TextGeocodingResult Cluster  
├─ id               : I32  
├─ name             : String  
├─ latitude         : DBL  
├─ longitude        : DBL  
├─ elevation        : DBL  
├─ feature_code     : String  
├─ country_code     : String  
├─ timezone         : String  
├─ population       : I32  
├─ postcodes[]      : Array of Strings  
├─ admin1           : String  
├─ admin2           : String  
├─ admin3           : String  
├─ admin4           : String  
├─ admin1_id        : I32  
├─ admin2_id        : I32  
├─ admin3_id        : I32  
├─ admin4_id        : I32

Hinweis:

Manche Felder fehlen bei bestimmten Orten → das ist laut API normal.
LabVIEW ignoriert fehlende JSON‑Felder → kein Parsing‑Fehler.


## 7. Minimal‑Cluster (optional)
Falls du nicht alle Meta‑Infos brauchst:
Plain TextGeocodingResult Small Cluster  
├─ name         : String  
├─ latitude     : DBL  
├─ longitude    : DBL  
├─ country_code : String  
├─ admin1       : String  
├─ admin2       : String

## 8. Actor‑Framework Architektur
Empfohlene Struktur:
Root Actor  
  ├── UI Actor  
  ├── Geocoding Actor  
  └── Weather Actor  

UI Actor

Textinput → sendet GetSuggestions
Listbox zeigt Ergebnisse
Klick → sendet QueryWeather(lat, lon)

Geocoding Actor

führt Geocoding‑API‑Abfragen aus
filtert Ergebnisse nach Bedarf
sendet Suggestion‑Set zurück an UI

Weather Actor

erhält Koordinaten
ruft Forecast‑API auf
sendet Wetterdaten zurück an UI


## 9. Fallback‑Lösung für PLZ (optional)
Da Open‑Meteo PLZ in DE nicht zuverlässig unterstützt, ist der zuverlässige Weg:
PLZ → Koordinaten über OpenStreetMap Nominatim
z. B.:
https://nominatim.openstreetmap.org/search?postalcode=97334&countrycodes=de&format=json

→ liefert garantiert Koordinaten von Nordheim am Main
Dann:
https://api.open-meteo.com/v1/forecast?latitude=<lat>&longitude=<lon> ...


## 10. Best Practices

Immer nur Ortsname + countryCode suchen
count=20 für bessere Trefferliste
UI nicht „blockieren“ → Geocoding in eigenem Actor
Vorschau‑Eintrag sollte enthalten:
Nordheim a.Main (Bayern, Unterfranken)


Koordinaten als separate Hidden‑Datenstruktur speichern
Forecast erst nach expliziter User‑Auswahl aufrufen


## 11. Zusammenfassung
Open‑Meteo liefert präzise Wetterdaten, aber die Geocoding‑Daten sind abhängig von GeoNames, daher funktionieren manche Postleitzahlen nicht.
Der richtige Ansatz ist:

Ortsnamensuche mit

name=
countryCode=DE
language=de


Auswahlliste anzeigen
Koordinaten übernehmen
Forecast‑API aufrufen

Alle relevanten Strukturen und Abläufe sind oben definiert.
