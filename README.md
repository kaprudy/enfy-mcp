# enfy.nl energievergelijker: MCP-server

Een gratis, openbare [MCP-server](https://modelcontextprotocol.io) waarmee een AI-assistent (Claude, ChatGPT, Cursor, VS Code en andere MCP-clients) **stroom- en gascontracten van alle Nederlandse energieleveranciers** voor je kan doorrekenen. De bedragen komen uit dezelfde rekenkern als de vergelijker op [enfy.nl](https://enfy.nl): met energiebelasting, netbeheerkosten, teruglevering en btw, per maand doorgerekend.

- **Adres:** `https://enfy.nl/mcp`
- **Transport:** Streamable HTTP (`POST /mcp`), versie 1.6.4 (wijzigingen: [onderaan](#wijzigingen))
- **Inloggen of API-sleutel:** niet nodig
- **Alle tools zijn alleen-lezen.** Ze sluiten niets af, versturen niets en wijzigen niets.

Deze repository bevat alleen documentatie, de registervermelding (`server.json`) en het icoon; geen code. De server zelf draait op enfy.nl.

## Wat de server doet

- De goedkoopste contracten voor jouw verbruik berekenen. Vast en variabel staan op volgorde van kosten in het eerste jaar; dynamisch staat apart, als schatting met een bandbreedte.
- Je huidige contract doorrekenen met de tarieven van je jaarnota en laten zien wat overstappen oplevert, met een indicatie van de opzegvergoeding.
- Dynamische stroom- en gasprijzen van vandaag en morgen, netbeheerkosten, energiebelasting, prijswijzigingen en prijshistorie.
- Uitrekenen of een thuisbatterij loont, met de regels vanaf 2027 (geen salderen).

Elk antwoord linkt naar een pagina op enfy.nl met dezelfde cijfers, zodat je de berekening kunt nalezen. Gaf je iets niet op (verbruik, postcode, aansluitwaarde), dan zegt het antwoord wat we aannamen (`defaults_used`), zodat de assistent gericht kan doorvragen.

## Tools

| Tool | Wat het doet |
|---|---|
| `vergelijk_energie` | Goedkoopste stroom- en gascontracten voor een huishouden: maandbedrag en kosten eerste jaar, vast en variabel op kosten, dynamisch apart als schatting. Invoer zoals op de jaarnota (kWh normaal en dal, m³ gas, teruglevering, aansluitwaarde) en de netbeheerder of alleen de 4 cijfers van de postcode. Zonder teruglevering maar met het vermogen van je zonnepanelen (Wp) schat de rekenkern opwek, eigen gebruik en teruglevering. |
| `eigen_contract_vergelijken` | Rekent je huidige contract door met de tarieven van je jaarnota en zet het naast de goedkoopste aanbiedingen: besparing per jaar en hoeveel aanbiedingen goedkoper zijn. Met de einddatum van je vaste contract ook een indicatie van de opzegvergoeding en vanaf wanneer overstappen loont. |
| `dynamisch_of_vast` | Goedkoopste vast, variabel en dynamisch contract naast elkaar, plus de kosten van dynamisch in een paar marktjaren (zoals 2022 en 2024) en de afgelopen 30 dagen. |
| `netbeheerder` | Netbeheerder voor stroom en gas bij de 4 cijfers van een postcode, met de netbeheerkosten van dit en volgend jaar. |
| `stroomprijs_vandaag` | Dynamische stroomprijs (day-ahead) per uur en kwartier voor een dag, kaal en all-in, eventueel met de inkoopvergoeding van een leverancier. |
| `gasprijs_vandaag` | Dynamische gasprijs per gasdag (06:00 tot 06:00), kaal en all-in. |
| `leverancier_tarieven` | De tarieven per product van één leverancier, zoals op diens pagina op enfy.nl, met de kosten voor een gemiddeld huishouden. |
| `prijswijzigingen` | Gecontroleerde tariefwijzigingen van leveranciers, nieuwste ingangsdatum eerst, met het effect per maand voor een gemiddeld huishouden. |
| `prijshistorie` | Stroom, gas, energiebelasting en day-ahead-prijzen per jaar en over de laatste 13 maanden, met bronnen. |
| `energiebelasting` | Schijven van de energiebelasting en de belastingvermindering voor een jaar, excl. en incl. btw. |
| `thuisbatterij_berekenen` | Of een thuisbatterij loont: per uur doorgerekend over de afgelopen 12 maanden, met advies, besparing per jaar, terugverdientijd, beste grootte en de verliezen bij laden en ontladen. |
| `methode_en_dekking` | Hoe we vergelijken (volgorde alleen op kosten, wat wel en niet meetelt, privacy) en hoe volledig en actueel de gegevens zijn: leveranciers en producten in de vergelijking, laatste tariefcontrole, marktprijzen en de jaren van energiebelasting en netbeheer. Optioneel voor één leverancier. |

Daarnaast:
- **Prompt** `jaarnota_vergelijken`: vraagt de cijfers van je jaarnota één voor één uit.
- **Resource** `energievergelijker://methode`: hoe de vergelijking werkt (volgorde, invoer, aannames, privacy). Hetzelfde, met dekking en actualiteit, als tool `methode_en_dekking`.

## Koppelen

Gebruik overal hetzelfde adres: `https://enfy.nl/mcp`. Authenticatie: geen. Of je eigen connectors kunt toevoegen, hangt bij ChatGPT en Claude af van je abonnement.

**Claude (claude.ai en de desktop-app)**
1. Ga naar Instellingen → Connectors → *Aangepaste connector toevoegen*.
2. Geef een naam (bijvoorbeeld "enfy.nl") en vul de URL in.
3. Zet de connector aan in een gesprek via het menu met tools.

**Claude Code**
```bash
claude mcp add --transport http energievergelijker https://enfy.nl/mcp
```

**ChatGPT**
1. Ga naar Instellingen → Apps en connectors → Geavanceerd en zet de ontwikkelaarsmodus aan.
2. Kies *Maken*, geef de connector een naam en vul de URL in. Authenticatie: geen.
3. Kies de connector in een nieuw gesprek.

**Cursor** (`~/.cursor/mcp.json` of `.cursor/mcp.json` in je project)
```json
{
  "mcpServers": {
    "energievergelijker": { "url": "https://enfy.nl/mcp" }
  }
}
```

**VS Code** (`.vscode/mcp.json`, of *MCP: Add Server* → HTTP)
```json
{
  "servers": {
    "energievergelijker": { "type": "http", "url": "https://enfy.nl/mcp" }
  }
}
```

**Andere MCP-clients:** voeg een server toe met transport `http` (Streamable HTTP) en de URL hierboven.

## Voorbeeldvragen

- "Vergelijk mijn energiecontract. Mijn jaarnota: 2.100 kWh normaal, 1.400 kWh dal, 1.150 m³ gas, aansluiting 3x25, postcode 3511."
- "Hier zijn het verbruik en de tarieven van mijn jaarnota. Wat kost mijn huidige contract per jaar en hoeveel bespaar ik als ik overstap?"
- "Mijn vaste contract loopt nog tot oktober 2028. Kost overstappen nu een opzegvergoeding, en loont het toch?"
- "Ik heb zonnepanelen en lever 2.800 kWh per jaar terug. Is een dynamisch contract dan goedkoper dan een vast contract?"
- "Wat kost stroom morgen per uur, en wanneer is het het goedkoopst?"
- "Welke netbeheerder hoort bij postcode 7707 en wat kost die per jaar?"
- "Wordt Eneco per 1 januari duurder?"
- "Ik heb 12.000 Wp zonnepanelen en een dynamisch contract. Loont een thuisbatterij van 10 kWh?"
- "Hoe bepalen jullie de volgorde, en hoe actueel zijn de tarieven?"
- In het Engels: "What is the cheapest energy contract for 2,800 kWh and 1,100 m3 of gas in postcode 3511?"

## Privacy

- **Geen persoonsgegevens nodig.** Alleen de 4 cijfers van de postcode (daaruit leiden we de netbeheerder af). Een volledige postcode, huisnummer of toevoeging wordt geweigerd. Naam, adres, klantnummer of rekeningnummer zijn nooit nodig.
- **Geen account, geen sleutel, geen cookies.** We bewaren geen IP-adressen en nooit het gesprek met de assistent. Van elke tool-aanroep bewaren we 90 dagen de tool, de meegestuurde getallen en keuzes (postcode alleen 4 cijfers, geen vrije tekst) en een samenvatting van ons antwoord, om onze antwoorden te controleren.
- Voor de limiet tellen we verzoeken onder een versleutelde code die elke dag verandert en na een minuut vervalt. Per dag tellen we alleen het aantal verzoeken per tool, per soort client en per uitkomst.
- Links in antwoorden hebben `utm_source=mcp`, `utm_medium=ai` en de tool als `utm_campaign`. Zo zien we hoeveel mensen via een assistent op de site komen; de pagina's tonen dezelfde cijfers.
- Volledige privacyverklaring: <https://enfy.nl/privacy#api>

## Bronnen en bronvermelding

- **Tarieven:** de openbare tariefpagina's en -documenten van de leveranciers zelf. Elke tariefversie gaat door een validatiestap voordat hij op de site en in de server komt.
- **Dynamische prijzen:** day-ahead-prijzen van ENTSO-E, gecontroleerd tegen een tweede bron; gasprijzen per gasdag.
- **Energiebelasting:** de tarieven van de Belastingdienst.
- **Netbeheer:** de tarieven van de netbeheerders; postcode naar netbeheerder uit de open data van Liander, Enexis, Stedin, Westland Infra, Coteq en RENDO.
- **Thuisbatterij:** weergegevens van het KNMI en de beursprijzen van de afgelopen 12 maanden.

Vrij te gebruiken met bronvermelding: noem "enfy.nl" als bron en link naar de pagina uit het antwoord. Geaggregeerde cijfers mag je hergebruiken onder [CC BY 4.0](https://enfy.nl/open-data).

## Limieten

- Standaard 60 MCP-verzoeken per minuut per client. Daarboven volgt status 429 met `Retry-After`. De actuele limieten staan op <https://enfy.nl/api#limieten>.
- Antwoorden zijn compact: de tekst bevat de samenvatting met links, de gegevens staan één keer in `structuredContent` (met een `outputSchema` per tool). Gedeelde aannames staan één keer bovenaan, per aanbieding alleen wat afwijkt.
- De lijsten (`tools/list`, prompts, resources) mogen clients een uur cachen (`ttlMs`, `cacheScope: public`); ze veranderen alleen bij een nieuwe versie.

## Voorwaarden

- Gratis te gebruiken. Een berekening is geen aanbod van een leverancier: wie wil overstappen, controleert de voorwaarden bij de leverancier.
- **Volgorde alleen op kosten** in het eerste jaar. Een betaalde positie staat nooit in die volgorde, maar apart met het label "Betaalde positie". Hoe de site geld verdient: <https://enfy.nl/hoe-wij-vergelijken#verdienmodel>.
- Antwoorden linken alleen naar pagina's op enfy.nl, nooit naar een leverancier of een affiliate-link. Alleen als bronvermelding staan er soms links naar openbare bronnen (ACM, CBS, wetten.overheid.nl, Belastingdienst). De eigen bron van een leverancier (bijvoorbeeld over groene stroom) staat op onze leverancierspagina; het antwoord linkt daarheen.
- Fout gezien in een uitkomst? Meld het via <https://enfy.nl/fout-melden>.

## Links

- Uitleg voor consumenten: <https://enfy.nl/energie-vergelijken-met-ai>
- API en MCP voor ontwikkelaars: <https://enfy.nl/api> (OpenAPI: <https://enfy.nl/openapi.json>)
- Methode: <https://enfy.nl/hoe-wij-vergelijken>
- Contact: info@enfy.nl
- Repository (alleen documentatie): <https://github.com/kaprudy/enfy-mcp>

---

## English

**enfy.nl energievergelijker** is a free, public MCP server for the Dutch energy market. It lets an AI assistant compare electricity and gas contracts from all Dutch suppliers for a household's own usage, with energy tax, grid costs, feed-in and VAT included. Tool output is in Dutch (questions in English work too; the tool descriptions carry English keywords) and every answer links to a page on enfy.nl with the same figures. Inputs that were not given are listed in `defaults_used` (e.g. average usage or grid operator), so the assistant can ask a follow-up question.

- **Endpoint:** `https://enfy.nl/mcp` (Streamable HTTP, version 1.6.4). No account or API key. Changelog: see [Wijzigingen](#wijzigingen) below.
- **Tools (all read-only):** `vergelijk_energie` (cheapest contracts), `eigen_contract_vergelijken` (your current contract vs. the market), `dynamisch_of_vast` (dynamic vs. fixed), `netbeheerder` (grid operator and grid costs from a 4-digit postcode), `stroomprijs_vandaag` and `gasprijs_vandaag` (dynamic prices), `leverancier_tarieven` (one supplier's tariffs), `prijswijzigingen` (price changes), `prijshistorie` (price history), `energiebelasting` (energy tax), `thuisbatterij_berekenen` (does a home battery pay off) and `methode_en_dekking` (how we rank, coverage of suppliers and how current the data is, new in 1.6.0).
- **Connect:** Claude (Settings → Connectors → Add custom connector), ChatGPT (developer mode → create connector, no authentication), Claude Code (`claude mcp add --transport http energievergelijker https://enfy.nl/mcp`), Cursor and VS Code (HTTP server with the URL above, see the JSON snippets in the Dutch section).
- **Privacy:** no personal data. Only the first 4 digits of a postcode are accepted; house numbers are refused. No cookies, no IP addresses stored. Daily counts per tool, client family and outcome, and a minimised log of tool calls (numbers and choices only, no free text, no IP) kept for 90 days to check our answers.
- **Ranking:** by first-year cost only. Any paid position is shown separately and labelled. Answers link only to enfy.nl (plus public sources such as the ACM or CBS as citations), never to suppliers or affiliate links.
- **Sources and attribution:** suppliers' own published tariffs (validated), ENTSO-E day-ahead prices, Dutch tax authority, grid operators' tariffs and open data, KNMI weather. Free to use with attribution; aggregated figures under CC BY 4.0.
- **Limits:** 60 MCP requests per minute per client by default (HTTP 429 with `Retry-After` above that).
- **More:** <https://enfy.nl/api> · <https://enfy.nl/privacy#api>

---

## Wijzigingen

### 1.6.4 (18-11-2026)
- **Titel `enfy.nl energievergelijker (Nederland)`** (was `enfy …`), gelijk aan de servernaam en de bronvermelding.
- **Publieke repository** met deze documentatie, `server.json` en het icoon: <https://github.com/kaprudy/enfy-mcp> (geen code; de server draait op enfy.nl). `server.json` noemt hem als `repository`.

### 1.6.3 (18-11-2026)
- **Servernaam `enfy.nl`**: `serverInfo.name` is nu het domein, gelijk aan de bronvermelding (was `enfy`). De registernaam `nl.enfy/energievergelijker` en de URL `https://enfy.nl/mcp` blijven gelijk; bestaande koppelingen werken door.

### 1.6.2 (18-11-2026)
- **Bronvermelding noemt het domein**: `citation.text`, `citation.name`, `attribution` en `meta.source` zeggen "enfy.nl" in plaats van "enfy" ("Bron: enfy.nl (18 november 2026), https://enfy.nl/…"), ook in de REST-API.
- **Links weer met `utm_medium=ai`**: elke link naar enfy.nl in een antwoord heeft `utm_source=mcp&utm_medium=ai&utm_campaign=<tool>` (in 1.6.0 en 1.6.1 zonder `utm_medium`).

### 1.6.1 (18-11-2026)
- **`dynamisch_of_vast`: scenario's met de gasprijs van dat jaar.** Het 2022-scenario rekende gas met de terugvalprijs (€ 0,35/m³) omdat er voor 2021–2022 te weinig dubbel gecontroleerde gasprijzen waren; de rekenkern gebruikt nu alleen gecontroleerde prijzen als dat er genoeg zijn, anders alle prijzen van die periode. Het scenario "zo duur als de afgelopen 30 dagen" schaalt ook de gasprijs. Per scenario `gas_market_avg`, in `market` het gasniveau, en `scenario_gas_follows_market` (false = gasprijs van het contract is vast of variabel, alleen stroom beweegt). Een scenario dat op een terugvalprijs zou rekenen, valt weg.
- **`dynamisch_of_vast`: de kop vergelijkt ook met de goedkoopste vaste (elke looptijd) of variabele aanbieding**, naast vast 1 jaar; nieuw veld `dynamic_minus_best_fixed_or_variable_year1_cents`.
- **Filtertekst**: `eigen_contract_vergelijken` en `vergelijk_energie` noemen de gefilterde set ("alle 48 vaste contracten van 1 jaar", "variabele contracten met groene stroom uit Nederland") in plaats van altijd "vaste en variabele aanbiedingen"; `eigen_contract_vergelijken` geeft ook `filters` terug.
- **Instructies**: roep alleen tools aan die in je toollijst staan; met een oudere lijst zonder `methode_en_dekking` de resource `methode`.

### 1.6.0 (18-11-2026)
- **Nieuwe tool `methode_en_dekking`**: hoe we vergelijken (volgorde alleen op kosten, welkomstbonus, wat wel en niet meetelt, privacy, geen betaalde volgorde) en hoe volledig en actueel de gegevens zijn: leveranciers en producten in de vergelijking (en welke niet, met reden), de laatste tariefcontrole per leverancier met de oudste en nieuwste, tot welke dag we stroom- en gasprijzen hebben en welke jaren energiebelasting en netbeheertarieven (definitief, voorlopig of voorstel). Met `leverancier` voor één leverancier.
- **`defaults_used`** in `vergelijk_energie`, `eigen_contract_vergelijken`, `dynamisch_of_vast` en `thuisbatterij_berekenen`: welke invoer we aannamen (`field`, `value`, `label`, `ask`), plus één regel "Aangenomen … Preciezer met …" in de tekst. Nieuwe invoer `ingang` (ingangsdatum).
- **Slankere antwoorden** (`vergelijk_energie` op echte data: tekst −27%, `structuredContent` −23%): geen link meer per aanbieding in de tekst (de slotregel linkt naar alle aanbiedingen), per aanbieding `anchor` in plaats van de hele `url` (link = `result_url` + `#` + `anchor`), gedeelde aannames één keer in `offer_assumptions` en `dynamic_assumptions` met per aanbieding alleen `extra_assumptions`, het lege veld `product.green` weg, en de links zonder `utm_medium=ai`.
- **Alleen links naar enfy.nl** (en openbare bronnen): `green_origin.*.source_url` (de site van de leverancier) is vervangen door `green_origin.evidence_url`, het deel van onze leverancierspagina met die bron en de controledatum. De REST-API is ongewijzigd.
- **Neutrale slotregel** in plaats van "Volgende stap … stap over": "Alle aanbiedingen en de berekening per aanbieding: [onze vergelijking](…)".
- **serverInfo** met titel, beschrijving, website en het enfy-icoon (PNG 512 en SVG). De lijsten zijn een uur cachebaar voor iedereen (`ttlMs: 3600000`, `cacheScope: public`).
- **Engels**: Engelse trefwoorden in de toolbeschrijvingen en de instructies (Nederlands eerst, toolnamen ongewijzigd).

### 1.5.0 (18-11-2026)
- Gepubliceerd in het officiële MCP Registry als `nl.enfy/energievergelijker` (domeinverificatie via enfy.nl, icoon).
- Sinds 1.4.0 (additief): herkomst van groene stroom per aanbieding (`green_origin`, filter `groen_nl`), leveranciersnamen linken naar hun pagina op enfy.nl, advies over de soort thuisbatterij (`installatie=advies`, `recommendation`), `contract_advice` in `dynamisch_of_vast`, een geminimaliseerd logboek van tool-aanroepen (90 dagen, zonder IP) en rekenende tools in de rekenlimiet.
