# Termenplukker

Een webtool om rond een thema termen te verzamelen uit het [Termennetwerk](https://termennetwerk.netwerkdigitaalerfgoed.nl/) van het Netwerk Digitaal Erfgoed. Je kiest zelf welke termen erbij horen en downloadt het resultaat als `skos:Collection` in JSON-LD. Gemaakt voor de themapagina's van [CollectieNederland.nl](https://www.collectienederland.nl/), bijvoorbeeld *Kleding en Streekdracht*.

## Wat het doet

1. **Thema → zoektermen.** "Kleding en Streekdracht" wordt `kleding` + `streekdracht`. Zoektermen kun je aanvullen of verwijderen.
2. **Zoeken** in de gekozen bronnen via de GraphQL-API van het Termennetwerk. Standaard zijn dat de CHT en de AAT; de andere bronnen kun je aanzetten.
3. **Rangschikken.** Treffers worden *kernterm* (het label is precies de zoekterm), *samenstelling* of *minder zeker*. Die laatste groep staat standaard uit.
4. **Uitklappen** via `skos:narrower`. Kerntermen gaan automatisch één niveau dieper (instelbaar 0–2). Per term kan het verder: één niveau of de hele tak.
5. **Ontdubbelen.** Elke URI komt maar één keer voor, en het opgegeven maximum geldt voor het totaal.
6. **Koppelen.** Hetzelfde begrip in twee bronnen blijft twee keer staan, gekoppeld met `skos:exactMatch`. De tool doet voorstellen op basis van labels en van bestaande `exactMatch`-relaties in de bron. Elke koppeling is aan of uit te zetten.
7. **Exporteren** als JSON-LD met collectie-URI `https://collectienederland.nl/<opdrachtnaam>`, met per lid het voorkeurslabel, de alternatieve labels, de bron en de gekozen koppelingen.

Zie [public/help.html](public/help.html) voor de gebruikershandleiding.

## Opbouw

Een statische site zonder build-stap en zonder afhankelijkheden:

```
public/
  index.html    de tool (HTML, CSS en JS in één bestand)
  help.html     gebruikershandleiding
  privacy.html  privacyverklaring
  page.css      stijl voor help en privacy
  404.html      foutpagina
  _headers      beveiligingsheaders (CSP e.d.)
wrangler.jsonc  Cloudflare-configuratie (statische assets)
.github/ISSUE_TEMPLATE/  formulieren voor fouten, wensen en termmeldingen
```

De browser praat rechtstreeks met `https://termennetwerk-api.netwerkdigitaalerfgoed.nl/graphql` (CORS staat open). Er is geen eigen backend.

Gebruikte queries:

- `sources { name uri alternateName }` voor de lijst met bronnen;
- `terms(sources: [...], query: "...", limit: 100)` voor het zoeken;
- `lookup(uris: [...])` voor het ophalen van smallere termen, per 40.

## Lokaal draaien

Elke statische webserver werkt, bijvoorbeeld:

```bash
python -m http.server 8080 --directory public
```

Open daarna http://localhost:8080.

## Publiceren

Live op **https://termenplukker.jolietjakeblues64.workers.dev**. Het draait als Cloudflare Worker die alleen statische bestanden serveert (zie `wrangler.jsonc`), zonder code aan de serverkant.

Elke push of merge naar `main` zet de site automatisch live via [.github/workflows/deploy.yml](.github/workflows/deploy.yml). Daarvoor zijn twee repository-secrets nodig:

| Secret | Waarde |
|---|---|
| `CLOUDFLARE_API_TOKEN` | API-token met het sjabloon *Edit Cloudflare Workers* |
| `CLOUDFLARE_ACCOUNT_ID` | het Cloudflare-account-ID |

Met de hand kan het ook:

```bash
npx wrangler deploy
```

`public/_headers` zet de beveiligingsheaders. De Content-Security-Policy staat alleen verbindingen met de Termennetwerk-API toe.

## Privacy

Geen cookies, geen analytics, geen externe lettertypen of scripts en geen opslag. Zoektermen gaan alleen naar de API van het Termennetwerk. Zie [public/privacy.html](public/privacy.html).

## Wie

Een persoonlijk project van Joop Vanderheiden voor CollectieNederland.nl. Geen officiële dienst.

## Beperkingen

- Het Termennetwerk zoekt op labels, niet op betekenis. Daarom moet een mens altijd nog door de lijst heen.
- Voorgestelde `exactMatch`-koppelingen zijn gebaseerd op gelijke labels. Controleer ze.
- Alternatieve labels krijgen geen taalcode, omdat de API die niet meegeeft.

## Licentie

[MIT](LICENSE)
