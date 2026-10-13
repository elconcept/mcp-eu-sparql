# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) +
[Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

Bramki przed publikacja (2026-10-13). Bez zmian w kodzie serwera i w kontrakcie narzedzi - wydanie
tylko po to, zeby zaczely dzialac kroki w workflow (workflow nie jest czescia paczki).

### Added

- `release.yml`: krok "Run gates before publishing" (`npm test` = drift + offline, descriptor lock, bramka paczki) - do tej pory wydanie szlo `npm ci -> build -> publish` bez zadnego testu.
- `ci.yml`: `npm test` (drift + offline nie byly uruchamiane w CI) i bramka paczki.
- Bramka paczki `npm run test:package` (`npm pack --dry-run --json`; wymagane pliki wyprowadzone z `bin`/`main` + przechodnich `require`/`import` w `dist/`, pusta lista = BLOKADA; wzorzec soba-labs/krs-mcp #7) + kontrola pozytywna `npm run test:package-gate`. Mutant (usuniety `dist/queries.js`) -> BLOKADA, exit 1.

## [1.2.0] — 2026-07-08

Widen-round: pelne orzecznictwo CJEU przez istniejacy klient Cellar SPARQL + GDPRhub (decyzje krajowych DPA). **Backward-compatible**.

### Added

- `search_cjeu_by_ecli(ecli, lang?)` — lookup orzeczenia CJEU po ECLI (np. `ECLI:EU:C:2020:559` = Schrems II → CELEX `62018CJ0311`). ECLI w Cellar to literal `xsd:string` — matchowanie przez `STR()`.
- `search_cjeu`: nowe parametry `query` (keyword w tytule, case-insensitive — tytul CJEU zawiera strony, sygnature i slowa kluczowe) i `document_type` (JUDG / ORDER / OPIN_AG). Domyslnie obejmuje wszystkie trzy typy — w tym **opinie rzecznikow generalnych (OPIN_AG, 14 480 works), wczesniej niedostepne**.
- `search_gdprhub(query, limit?)` — pelnotekstowe wyszukiwanie w GDPRhub (gdprhub.eu, MediaWiki API, projekt noyb): decyzje krajowych DPA z calej UE + komentarze do artykulow RODO. Licencja tresci **CC BY-NC-SA 4.0** — flagowana w polu `license` kazdej citation i w INSTRUCTIONS.
- Pole `ecli` w `structuredContent.citations` (case-law) — takze w `search_by_celex`.
- Walidacja dat w `search_cjeu` (`invalid_date` zamiast cichego bledu SPARQL).
- Testy: `npm run test:offline` (fixtures z prawdziwych odpowiedzi live 2026-07-08, zero sieci) + `npm run smoke` (live, wszystkie 5 tooli). Query buildery i parsery wydzielone do `src/queries.ts` / `src/format.ts` (importowalne w testach).

### Fixed

- **Cichy no-op ORDER**: `search_cjeu` filtrowalo postanowienia po URI `.../ORDER_CJ`, ktory w Cellar ma **0 works** — poprawny URI to `.../ORDER` (8 362 works, zweryfikowane SPARQL COUNT 2026-07-08). Przed 1.2.0 `search_cjeu` zwracalo wylacznie wyroki.
- Wersja serwera w konstruktorze `Server` zsynchronizowana z package.json.

### Corpus totals (SPARQL COUNT, sektor-6 CELEX, 2026-07-08)

JUDG 34 261 / ORDER 8 362 / OPIN_AG 14 480 = **57 103 orzeczen i opinii CJEU**; 67 654 works z ECLI.

## [1.1.0] — 2026-05-25

Retrofit do kanonu MCP MateMatic (pattern z dograh-hq/dograh v1.31.0, BSD-2). **Backward-compatible**.

### Added

- `instructions` w konstruktorze Server (procedural orchestration: kolejnosc wywolan, CELEX kluczowy, wielojezycznosc 24 PL/ENG/FRA/DEU, stateless bez cache, iteracja po bledach).
- `ToolAnnotations` per tool (`readOnlyHint`, `openWorldHint=true` bo SPARQL endpoint live).
- Strukturalne `ErrorCode`: `missing_arg`, `invalid_date`, `upstream_error`, `empty_result`. Format `[code] tekst` + `structuredContent.error_code`.
- Walidacja formatu YYYY-MM-DD i kolejnosci dat (`invalid_date`) przed wyslaniem SPARQL.
- Drift test (`npm run drift`).

## [1.0.0] — 2026-05-20

Initial public release.

EU legislation + CJEU case law via Publications Office SPARQL (EUR-Lex / Cellar). 3 tools: search_by_celex / search_by_date_range / search_cjeu.

### Highlights

- Node 18+ stdio MCP server, single `dist/index.js` entry.
- LIVE smoke-tested on real data.
- `structuredContent.citations` consumed by [Patron](https://github.com/matematicsolutions/patron)
  and any other MCP-aware legal agent.
- MIT license, 500 ms request throttle, zero secrets required.

[1.0.0]: https://github.com/matematicsolutions/mcp-eu-sparql/releases/tag/v1.0.0
