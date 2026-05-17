---
title: Authoritative sources for live data (WebFetch index)
topic: meta
last_verified: 2026-05-17
---

# Authoritative Sources

The skill **must WebFetch from these URLs** for any volatile fact: prices, fares, opening hours, schedules, current festival dates, ticket availability, live service disruptions. Never quote those numbers from cached research files.

Pick the source whose domain matches the question. If multiple sources cover a topic, prefer the official operator/organization over a third-party aggregator.

## Public Transport — Barcelona Metro Area

| What to fetch | URL |
|---|---|
| Metro/bus/tram fares, T-Casual, T-Usual, T-Familiar prices | https://www.tmb.cat/en/fares-metro-bus-barcelona |
| Hola Barcelona Travel Card (2/3/4/5-day) prices | https://www.holabarcelona.com/ |
| Metro operating hours, line maps, disruptions | https://www.tmb.cat/en/barcelona-transport/metro |
| Bus routes, Nitbus night service | https://www.tmb.cat/en/barcelona-transport/bus |
| FGC (Catalunya–Sarrià, Tibidabo) fares & schedules | https://www.fgc.cat/ |
| Tram (Trambaix, Trambesòs) | https://www.tram.cat/ |
| Fare zone integration (ATM) | https://www.atm.cat/en |

## Trains — Regional & Long-Distance

| What to fetch | URL |
|---|---|
| Rodalies de Catalunya (R1–R17) schedules, fares | https://rodalies.gencat.cat/en/horaris-i-tarifes/ |
| RENFE long-distance, AVE, Avant — schedules & prices | https://www.renfe.com/es/en |
| Live train status, station info | https://www.adif.es/ |

## Airport — El Prat / Josep Tarradellas (BCN)

| What to fetch | URL |
|---|---|
| Aerobús A1/A2 prices and schedule | https://www.aerobusbarcelona.es/en/ |
| Airport facilities, transfers overview, T1/T2 shuttle | https://www.aena.es/en/josep-tarradellas-barcelona-el-prat.html |
| Airport metro L9 Sud fare (special airport ticket) | https://www.tmb.cat/en/fares-metro-bus-barcelona |
| Official taxi fixed rates to/from BCN | https://www.amb.cat/en/web/mobilitat/taxi |

## Major Sights — Tickets, Hours, Availability

| What to fetch | URL |
|---|---|
| Sagrada Família — tickets, hours, tower availability | https://sagradafamilia.org/en/ |
| Park Güell — Monumental Zone tickets, hours | https://parkguell.barcelona/en |
| Casa Batlló | https://www.casabatllo.es/en/ |
| La Pedrera / Casa Milà | https://www.lapedrera.com/en |
| Palau de la Música Catalana | https://www.palaumusica.cat/en |
| Hospital de Sant Pau (Recinte Modernista) | https://www.santpaubarcelona.org/en |
| Picasso Museum (incl. free hours) | https://www.museupicasso.bcn.cat/en |
| MNAC — Museu Nacional d'Art de Catalunya | https://www.museunacional.cat/en |
| Catedral de Barcelona | https://catedralbcn.org/ |
| Santa Maria del Mar | https://www.santamariadelmarbarcelona.org/ |
| Spotify Camp Nou — match tickets, tour status | https://www.fcbarcelona.com/en/tickets |
| Magic Fountain of Montjuïc — schedule | https://www.barcelona.cat/en/what-to-do-in-bcn/magic-fountain |
| Tibidabo amusement park hours | https://www.tibidabo.cat/en |

## Day Trips

| What to fetch | URL |
|---|---|
| Montserrat — funicular/cable car, monastery, Trans Montserrat / Tot Montserrat passes | https://www.montserratvisita.com/en/ |
| Girona tourism (events, opening hours) | https://www.girona.cat/turisme/eng/ |
| Sitges tourism | https://www.visitsitges.com/en/ |
| Tarragona tourism (Roman sites) | https://www.tarragonaturisme.cat/en |
| Figueres — Dalí Theatre-Museum tickets | https://www.salvador-dali.org/en/museums/dali-theatre-museum-in-figueres/ |
| Costa Brava bus (Moventis Sarfa, ALSA) | https://compras.moventis.es/ and https://www.alsa.com/en/web/bus/home |

## Events, Festivals, City Calendar

| What to fetch | URL |
|---|---|
| Official Barcelona events calendar | https://www.barcelona.cat/en/what-to-do-in-bcn/all-activities |
| La Mercè programme | https://www.barcelona.cat/lamerce/en |
| Generalitat de Catalunya tourism | https://act.gencat.cat/?lang=en |
| Visit Barcelona Turisme | https://www.barcelonaturisme.com/wv3/en/ |

## Weather & Conditions

| What to fetch | URL |
|---|---|
| Weather forecast (Spain met service) | https://www.aemet.es/en/eltiempo/prediccion/municipios/barcelona-id08019 |
| Catalan meteorological service | https://www.meteo.cat/ |
| Beach water quality, flag status (in season) | https://www.barcelona.cat/en/what-to-do-in-bcn/beaches |

## Health, Safety, Embassies

| What to fetch | URL |
|---|---|
| Emergency: dial **112** (no fetch needed) | — |
| Pharmacy on-duty finder (24h rotation) | https://www.farmaceuticonline.com/en/farmacies-de-guardia/ |
| Tourist police info | https://mossos.gencat.cat/en/ |

## Money

| What to fetch | URL |
|---|---|
| VAT refund (DIVA) procedure | https://sede.agenciatributaria.gob.es/Sede/en_gb/aduanas/diva.html |

---

## Fetching Rules

1. **Always WebFetch the volatile claim**, even if a research file appears to have a number. Research files are stale by design.
2. **Cite with timestamp**: `[tmb.cat, fetched YYYY-MM-DD]`.
3. **One fetch per claim**, not one per response — bundle related questions into a single fetch where the source page covers them.
4. **If a fetch fails**, surface the URL to the user and refuse to give a cached number.
5. **Prefer English versions** of official sites (most have `/en/` paths) so quoted text is usable.
