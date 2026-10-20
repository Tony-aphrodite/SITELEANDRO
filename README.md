# Mapa Lino: KMZ/KML Network Map Viewer

Mapa Lino is a Progressive Web App that shows electrical network maps (feeder lines and posts) from KMZ/KML files, organized by city. It has a public map viewer and an admin panel for managing city files. The front end uses plain HTML, CSS and JavaScript with Leaflet, and a small PHP back end stores data in files. The interface is in Brazilian Portuguese.

![Leaflet](https://img.shields.io/badge/Leaflet-1.9.4-199900?logo=leaflet&logoColor=white)
![JSZip](https://img.shields.io/badge/JSZip-3.10.1-blue)
![PHP](https://img.shields.io/badge/PHP-back%20end-777BB4?logo=php&logoColor=white)

## Features

- **City-based map viewer** (`index.html`): pick a city from the server list and its KMZ/KML file loads onto a Leaflet map. Side panels show and hide individual feeder lines and post groups, and counters report the number of markers, lines and polygons.
- **Search**: look up a numeric key code ("chave") in the loaded data, or search for a place through the Open-Meteo and geocode.maps.co geocoding APIs.
- **Map controls**: street (OpenStreetMap), satellite (Esri World Imagery) and terrain (OpenTopoMap) base layers, zoom buttons, live cursor coordinates and a light/dark theme toggle.
- **Locate me**: finds the user's position, reverse-geocodes it through `api/revgeo.php` (Nominatim), and loads the registered city with the matching name. Each post marker's popup has a link that opens directions in Google Maps.
- **Large-file performance**: KML is parsed in batches during idle time, lines are simplified by distance in meters with zoom-dependent levels of detail, and markers and labels appear only at higher zoom levels, with a limit on how many labels show at once.
- **Admin panel** (`admin.html`): create, edit and delete cities (name, prefix, KMZ/KML file), set a default city, edit the message banner on the home page, and upload a logo that the server turns into PWA icons with GD.
- **Chunked uploads**: large KMZ/KML files are uploaded in parallel chunks through `api/chunked_upload.php`, with retries and progress reporting.
- **Offline support**: a web app manifest, a service worker that caches the app shell and KMZ files (cache-first), and an offline fallback page.

## Tech stack

| Layer | Tools |
| --- | --- |
| Front end | HTML, CSS, vanilla JavaScript, Leaflet 1.9.4, JSZip 3.10.1 (both loaded from cdnjs) |
| Back end | PHP, file-based JSON storage, no database |
| PWA | `manifest.webmanifest`, `service-worker.js`, Cache Storage |
| External services | OpenStreetMap / Esri / OpenTopoMap tiles, Nominatim, Open-Meteo and geocode.maps.co geocoding |

## Getting started

### Prerequisites

- PHP with the `curl`, `fileinfo` and `gd` extensions
- A web server that runs PHP. The built-in PHP server is enough for local use.
- Write access for PHP to `uploads/` and `data/`

### Run locally

There is no build step and no package manager. From the repository root, run:

```bash
php -S localhost:8000
```

Then open:

- `http://localhost:8000/index.html` for the public viewer
- `http://localhost:8000/admin.html` for the admin panel

### Configuration

The app reads no environment variables. Its settings are hard-coded in the source files:

- **Admin credentials and tokens**: set your own admin password and message token before deploying. They are handled in `api/auth.php`, `api/message.php` and `admin.html`.
- **Performance thresholds**: zoom levels, batch size and the label limit are constants at the top of `assets/script.js`.

## Project structure

```
.
├── index.html              # public map viewer
├── admin.html              # admin panel (cities, message, logo)
├── offline.html            # offline fallback page
├── manifest.webmanifest    # PWA manifest
├── service-worker.js       # caching for the app shell and KMZ files
├── api/
│   ├── cities.php          # list/get/create/update/delete cities, set default
│   ├── chunked_upload.php  # chunked file upload
│   ├── message.php         # home-page message (GET public, POST with token)
│   ├── upload_logo.php     # logo upload and PWA icon generation
│   ├── revgeo.php          # reverse geocoding proxy (Nominatim)
│   └── auth.php            # session login / logout / change password
├── assets/
│   ├── script.js           # map, KML parsing, search, layers, admin logic
│   ├── styles.css
│   ├── chunked-uploader.js, ultra-fast-uploader.js, compression-worker.js
│   └── icons/, img/
├── data/message.json       # current home-page message
└── uploads/
    ├── logo.png
    └── cities/             # one folder per city + _index.json metadata
```

## Notes and known limitations

- The admin panel is meant for trusted users. Review its authentication before you expose it on the public internet.
- The upload scripts call `/api/chunked_upload.php` with an absolute path, so chunked uploads work only when the app is served from the web root.
- The "Baixar minha cidade" button looks for `lat`/`lng` on each city, but `api/cities.php` does not store coordinates. Use the "locate me" button, which matches cities by name, instead.
- `uploads/cities/` in the repository contains sample city files and an `_index.json` with file paths from the machines where they were uploaded.
