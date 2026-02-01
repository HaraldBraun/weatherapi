# README – Open-Meteo Weather Query with Geocoding & LabVIEW AF

This document describes:
- How Open-Meteo works   
- Why some zip codes are not supported   
- How to reliably find locations   
- What the data structures look like   
- How to implement a suggestion list (Auto-Suggest)   
- The workflow for Geocoding → Forecast → Display   
- The JSON clusters (LabVIEW-compatible)   

## 1. Open-Meteo – Overview
Open-Meteo offers:
- A free weather API without an API key   
- A free geocoding API to find locations and coordinates   

The Forecast API operates exclusively based on coordinates (lat/lon).
[api.openmeteo.com]   

The geocoding API serves as a preprocessing step:
It takes location names or zip codes and returns matching locations.
[digitalvalley.de]   

## 2. Problem: Why doesn't zip code 97334 work?
The API accepts zip codes, but only if they are included in the GeoNames database (Open-Meteo is fully based on GeoNames).
[digitalvalley.de]   

- French overseas regions (e.g., Cayenne) have zip code blocks 97xxx → these are recognized.
- German zip codes are NOT fully included in the dataset.
- Zip code "97334" therefore matches Cayenne, GF, and not Nordheim am Main.

➡ Therefore, using zip codes is unreliable.  
➡ The only reliable method is searching by location name + country filter.

## 3. Solution: Location Name + countryCode
To make the search reliable and filterable, use:
```
name=<LocationName>
countryCode=<ISO-CountryCode>
language=en
count=20
```   

Example for "Nordheim" only in Germany:
```Plain Text
https://geocoding-api.open-meteo.com/v1/search?name=Nordheim&countryCode=DE&language=en&count=20  
```   

The API returns multiple hits (Nordheim am Main, Nordheim in BW, Hesse, Thuringia, etc.), each with coordinates.

## 4. Procedure: Dynamic Suggestion List (Auto-Suggest)
To allow the user to select the correct location, a dynamic hit list is built:
**Workflow:**
1. User types into a text field (e.g., "Nord").
2. UI sends a message to the Geocoding Actor:
   ```
   GetSuggestions("Nord", "DE")
   ```   
3. Geocoding Actor calls the API:
   ```Plain Text
   https://geocoding-api.open-meteo.com/v1/search?name=Nord&countryCode=DE&count=10&language=en  
   ```   
4. API returns a list of potential locations.
5. UI Actor populates the Suggestion Listbox / Dropdown.
6. User clicks on an entry.
7. UI extracts latitude + longitude.
8. UI sends query to Weather Actor → Forecast API.

## 5. Forecast Query with Coordinates
The Open-Meteo Forecast API uses coordinates exclusively.
[api.openmeteo.com]   

Example for Nordheim am Main:
```Plain Text
https://api.open-meteo.com/v1/forecast?latitude=49.8587&longitude=10.18545&current=temperature_2m,wind_speed_10m&hourly=temperature_2m,wind_speed_10m,relative_humidity_2m  
```   

You receive:
- Current values (current)   
- Hourly values (hourly)   
- Date/time arrays   
- Timezone   

## 6. Data Model (Cluster) for Geocoding Response
The geocoding API returns:
```JSON
{  "results": [...],  
   "generationtime_ms": 0.193}  
```   

**Cluster "GeocodingResponse"**
```Plain Text
GeocodingResponse Cluster  
├─ results[] : Array of GeocodingResult  
└─ generationtime_ms : DBL
```   

### Cluster "GeocodingResult"
Based on the API structure:   
[digitalvalley.de]   

```Plain Text
GeocodingResult Cluster  
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
```   

**Note:**
- Some fields may be missing for certain locations → this is normal according to the API   .
- LabVIEW ignores missing JSON fields → no parsing error   .

## 7. Minimal Cluster (Optional)
If you don't need all meta-info:
```Plain Text
GeocodingResult Small Cluster  
├─ name         : String  
├─ latitude     : DBL  
├─ longitude    : DBL  
├─ country_code : String  
├─ admin1       : String  
├─ admin2       : String
```   

## 8. Actor Framework Architecture
Recommended structure:
```
Root Actor  
  ├── UI Actor  
  ├── Geocoding Actor  
  └── Weather Actor  
```   

**UI Actor**
- Text input → sends GetSuggestions   
- Listbox shows results   
- Click → sends QueryWeather(lat, lon)   

**Geocoding Actor**
- Executes Geocoding API queries   
- Filters results as needed   
- Sends Suggestion-Set back to UI   

**Weather Actor**
- Receives coordinates   
- Calls Forecast API   
- Sends weather data back to UI   

## 9. Fallback Solution for Zip Codes (Optional)
Since Open-Meteo does not reliably support zip codes in Germany, the reliable way is:
Zip code → Coordinates via OpenStreetMap Nominatim   
E.g.:
`https://nominatim.openstreetmap.org/search?postalcode=97334&countrycodes=de&format=json`   

→ Guaranteed to provide coordinates for Nordheim am Main   .
Then:
`https://api.open-meteo.com/v1/forecast?latitude=<lat>&longitude=<lon> ...`   

## 10. Best Practices
- Always search using Location Name + countryCode   .
- Use `count=20` for a better hit list   .
- Do not "block" the UI → perform geocoding in its own Actor   .
- Preview entries should include: `Nordheim a.Main (Bavaria, Lower Franconia)`   .
- Store coordinates as a separate hidden data structure   .
- Call the forecast only after explicit user selection   .

## 11. Summary
Open-Meteo provides precise weather data, but geocoding data depends on GeoNames, so some zip codes do not work   .
The correct approach is:
1. Location name search with:
   - `name=`
   - `countryCode=DE`
   - `language=en`   
2. Display selection list   .
3. Apply coordinates   .
4. Call Forecast API   .

All relevant structures and workflows are defined above   .