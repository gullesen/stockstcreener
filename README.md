# StockScreener Pro

En personlig aksjescreener for å prioritere selskaper som fortjener videre analyse. Løsningen henter dagsdata fra Yahoo Finance, beregner tekniske indikatorer og publiserer resultatene i en mobilvennlig nettapp på GitHub Pages.

**App:** https://gullesen.github.io/stockscreener/

> Screeneren er et analyse- og prioriteringsverktøy, ikke et automatisk handelssystem. Composite Score og tekniske signaler skal ikke tolkes som selvstendige kjøpssignaler.

## Arkitektur

```text
companies.json
      ↓
screener.py
      ↓
aksjer.json i GitHub Gist
      ↓
index.html
      ↓
GitHub Pages
```

Backend kjører på en Linux-basert Intel NUC. Frontend ligger i dette repoet og består hovedsakelig av HTML, CSS og JavaScript i `index.html`.

## Funksjonalitet

Appen viser blant annet:

- ticker og selskapsnavn
- sektor, land, børs og valuta
- siste kurs og kursendring
- RSI, MACD, SMA og EMA
- volum mot gjennomsnitt
- ATR og risiko
- støtte- og motstandsnivåer
- relativ styrke mot OSEBX
- trend, setup og Composite Score
- historiske OHLCV-data og grafer
- søk, sortering og filtrering
- Mine- og Radar-lister lagret i `localStorage`

## Datakilder

Markedsdata hentes med Python-biblioteket `yfinance`.

`OSEBX.OL` brukes som referanseindeks for relativ styrke.

Selskapsmetadata vedlikeholdes separat i `companies.json`, med felter som:

```json
{
  "name": "DNB Bank ASA",
  "sector": "Finans",
  "country": "Norge",
  "exchange": "Oslo Børs",
  "currency": "NOK",
  "type": "equity",
  "status": "active"
}
```

Backend legger metadataen inn i `aksjer.json`, slik at frontend bare trenger én datakilde.

## JSON-grensesnitt

Frontend forventer blant annet disse feltnavnene:

```text
Aksje
Navn
Sektor
Land
Børs
Valuta
```

Feltnavnene er en del av grensesnittet mellom backend og frontend og bør ikke endres uten at begge deler oppdateres samtidig.

Eksempel:

```json
{
  "Aksje": "AKBM.OL",
  "Navn": "Aker BioMarine ASA",
  "Sektor": "Konsumvarer",
  "Land": "Norge",
  "Børs": "Oslo Børs",
  "Valuta": "NOK"
}
```

## Forsiktig bruk av Yahoo Finance

Backend bruker lokal kurscache for å redusere antall forespørsler:

- nye tickere henter omtrent 15 måneder historikk
- eksisterende tickere henter normalt bare de siste 35 kalenderdagene
- data slås sammen etter dato
- cachen beholdes som et rullerende vindu på omtrent 15 måneder
- små batcher, `threads=False` og pauser brukes
- `Ticker.info` brukes ikke som daglig metadatakilde
- produksjonsfilen overskrives ikke dersom datagrunnlaget er for svakt

## Automatisk oppdatering

Produksjonsjobben kjører normalt én gang per ukedag klokken 23:10 norsk tid, etter at europeiske og amerikanske markeder er stengt.

```cron
CRON_TZ=Europe/Oslo
10 23 * * 1-5 flock -n /tmp/stockscreener.lock /home/gullesen/aksjer/run_screener.sh >> /home/gullesen/aksjer/cron_log.txt 2>&1
```

GitHub-tokenet ligger i en lokal miljøfil på serveren og skal aldri lagres i kildekoden eller dette repoet.

## Teknologi

- Python og `yfinance`
- HTML5, CSS og JavaScript
- Chart.js
- GitHub Gist
- GitHub Pages
- Linux, cron og lokal CSV-cache

## Lokal frontend-utvikling

Klon repoet:

```bash
git clone https://github.com/gullesen/stockscreener.git
cd stockscreener
```

Opprett en arbeidsgren:

```bash
git pull
git switch -c navn-på-gren
```

Etter endringer:

```bash
git status
git diff -- index.html
git add index.html
git commit -m "Beskrivelse av endringen"
git push -u origin navn-på-gren
```

Opprett deretter en pull request til `main`.

## Begrensninger

Tekniske indikatorer beskriver historiske pris- og volumdata. De sier ikke alene om et selskap er godt, billig eller egnet som investering.

Videre analyse bør minst inkludere:

- verdsettelse
- regnskap og kontantstrøm
- gjeld og finansiering
- ledelse og kapitalallokering
- konkurransefortrinn
- likviditet
- nyheter og selskapsmeldinger
- sektor- og makroøkonomisk risiko

## Ansvarsfraskrivelse

Prosjektet er laget for personlig analyse og læring. Det er ikke investeringsrådgivning, og data kan være forsinket, feil eller mangelfull.
