# Predviđanje indeksa kvalitete zraka (AQI)

Web aplikacija koja na temelju koncentracija onečišćujućih tvari (CO, NO₂, SO₂, O₃, PM2.5, PM10), grada i datuma predviđa **indeks kvalitete zraka (AQI)**. Predviđanje radi regresijski model objavljen kao web servis na **Azure Machine Learning**, a između preglednika i Azurea nalazi se mali **FastAPI** poslužitelj koji čuva pristupni ključ.

Projekt iz kolegija RUAP, FERIT Osijek.
Autor: Anto Kožarević

## Kako radi

```
preglednik (HTML/JS)  ──POST /predict──►  FastAPI poslužitelj  ──►  Azure ML endpoint (regresijski model)
        ▲                                        │                          │
        └─────────── AQI + kategorija ◄──────────┴────── "Scored Labels" ◄──┘
```

1. Korisnik unosi datum, grad (Brasilia, Cairo, Dubai, London, New York, Sydney) i vrijednosti onečišćivača.
2. Poslužitelj prosljeđuje podatke Azure ML servisu. Ključ i adresa servisa čitaju se iz `.env` i nikad ne dolaze do preglednika.
3. Odgovor modela pretvara se u AQI vrijednost, a klijent ju razvrstava u kategoriju (Dobro, Umjereno, Nezdravo za osjetljive skupine, Nezdravo, Vrlo nezdravo, Opasno).

## Struktura

| Mapa | Sadržaj |
|------|---------|
| `aqi-web/client/` | Web sučelje (HTML, CSS, JS) |
| `aqi-web/server/` | FastAPI poslužitelj (`/predict`, `/health`) |

## Pokretanje

Potreban je vlastiti Azure ML endpoint s modelom za AQI.

1. Poslužitelj:

```bash
cd aqi-web/server
pip install -r requirements.txt
cp .env.example .env            # upišite AZURE_URL i AZURE_KEY
uvicorn server:app --reload --port 8000
```

2. Sučelje (u novom terminalu):

```bash
cd aqi-web/client
python -m http.server 5500
```

Otvorite http://localhost:5500, kliknite "Popuni primjer" pa "Izračunaj". Ispravnost konfiguracije poslužitelja provjerava `http://127.0.0.1:8000/health`.

## API

`POST /predict`

```json
{ "Date": "2025-01-01 12:00:00+00:00", "City": "London", "CO": 250, "NO2": 40, "SO2": 12, "O3": 60, "PM25": 18, "PM10": 30 }
```

Odgovor: `{ "aqi": 43.69, "city": "London", "date": "2025-01-01 12:00:00+00:00" }`

## Napomena o konfiguraciji

Datoteka `.env` s Azure ključem nije dio repozitorija (`.gitignore`). Predložak je u `aqi-web/server/.env.example`.
