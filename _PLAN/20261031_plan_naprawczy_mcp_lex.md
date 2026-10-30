# EU-SPARQL — plan usunięcia wyboru języka, niespójnych URL i duplikatów

> Audyt: 2026-10-31. Źródła: test wszystkich narzędzi przez portal lex.xlap.pl/mcp, testy bezpośrednich kontenerów przez Portainer/Synology, kontrola kodu repozytoriów elconcept. To **plan**, nie potwierdzenie wdrożenia naprawy. Oznaczenia: **POTWIERDZONE** = zaobserwowany wynik i/lub wskazana przyczyna w kodzie; **DO SPRAWDZENIA** = hipoteza. 

## Reguły wspólne
- Publiczny interfejs narzędzi, zapytania w języku naturalnym, metadane prezentacyjne, fragmenty i błędy: **tylko po polsku**. Bez widocznego selektora języka. Identyfikatory techniczne, oryginalne sygnatury i adresy źródłowe pozostają niezmienione. W okresie migracji stary parametr `lang` można akceptować dla zgodności, ignorować i stosować PL; potem usunąć z publicznego schematu.
- Wyniki prawne nie mogą być dopowiadane. Rozróżniać tekst urzędowy (ze wskazaniem języka i źródła) od komentarza/nieurzędowego przekładu; przy braku wiarygodnej polskiej wersji raportować brak, nie zastępować angielskim tekstem ani nie udawać polskiej wersji urzędowej.
- MCP `content` (dane merytoryczne) **i** `structuredContent` (metadane) zachować równolegle. Testować bezpośrednio oraz przez Cloudflare i portal `lex`; nie uznawać samej listy `citations` za sukces funkcjonalny.
- Diagnostyka i testy przed zmianą produkcyjną; PR → testy offline → smoke staging → porównanie surowych odpowiedzi → kontrolowany rollout → monitoring/rollback. Nigdy nie publikować haseł, tokenów, PII ani pełnych logów uwierzytelnienia.

## Materiał dowodowy
- **POTWIERDZONE:** wejściowe `lang='PL'` nie jest zgodne z enum `POL/ENG/FRA/DEU`, ale `src/index.ts` akceptuje dowolny string; `src/format.ts` buduje adres przez `lang === 'POL' ? 'PL' : 'EN'`; dlatego przy `PL` zwrócił `/legal-content/EN/TXT/`.
- **POTWIERDZONE:** `lang='POL'` zwrócił polski tytuł/URL; `formatRows` używa polskiego URL domyślnie, niezależnie od pola `lang`. Widoczna rozbieżność między `content` i `citations`.
- **POTWIERDZONE:** `search_by_date_range(document_type='REG')` zwrócił ten sam CELEX `32024R1781R(01)` w wariantach `CORRIGENDUM` i `REG`; `search_cjeu_by_ecli` zwrócił duplikaty tego samego orzeczenia; `search_gdprhub('Schrems')` dał zero cytowań.
- **W KODZIE:** `src/queries.ts` używa `OPTIONAL { ?work ... ?type }` obok filtrowania po wybranym URI, co może generować dodatkowe wartości typu dla tego samego work; to hipoteza przyczyny duplikatów do potwierdzenia fixture SPARQL.

## Plan
1. **P0 — bez selektora języka:** usunąć `lang` z publicznych definicji pięciu narzędzi; w backendzie stała `POL` dla Cellar, `PL` dla adresów EUR-Lex. Stary parametr, jeśli przesłany przez dotychczasowego klienta, bezpiecznie ignorować (PL) i oznaczyć jako przestarzały bez awarii.
2. **P0 — zunifikować formattery:** jeden `eurLexUrl(celex, 'PL')` we wszystkich odpowiedziach; polskie nagłówki/tytuły. Test ścisły: zarówno widoczna treść, jak i `structuredContent.citations[].url` to `/legal-content/PL/TXT/`.
3. **P1 — SPARQL cardinality:** ograniczyć typ dokumentu w `SELECT` do żądanego `?type`, zamiast rozłącznego `OPTIONAL`; rozważyć `SELECT DISTINCT`, grupowanie po `(CELEX,ECLI,work)`, deterministyczny wybór urzędowego tytułu. Przetestować corrigendum osobno jako typ, nie etykietę REG.
4. **P1 — GDPRhub:** zbadać odpowiedzi MediaWiki API (status, `searchinfo`, `search`), kodowanie fraz i restrykcje sieciowe; brak wyników nie oznacza automatycznie błędu. Odpowiedzi tylko PL: gdy źródło istnieje wyłącznie po angielsku, metadane/komunikat po polsku i wyraźny brak dostępnej polskiej treści, bez podszywania się pod urzędowe tłumaczenie.
5. **P2 — testy i kompatybilność:** fixture Schrems II (CELEX `62018CJ0311`, ECLI `ECLI:EU:C:2020:559`), RODO, range REG, GDPRhub; porównanie wprost i przez `lex`; drift tool schemas, dokumentacja i deprecjacja.

## Akceptacja
- Żadne narzędzie nie wymaga i nie proponuje wyboru języka; jednoznacznie polskie URL, tytuł i komunikaty; brak angielskich URL powstałych z `lang`.
- Zapytania filtrowane nie zwracają rekordów innych typów ani duplikatów CELEX/ECLI; ograniczenie liczby rezultatów jest respektowane.
- GDPRhub: prawidłowo rozróżniony brak wyników od błędu API; brak niedeklarowanego EN fallback.

## Pliki / dowody
`src/format.ts`, `src/queries.ts`, `src/index.ts`, `test/offline.mjs`, `test/smoke.mjs`.
