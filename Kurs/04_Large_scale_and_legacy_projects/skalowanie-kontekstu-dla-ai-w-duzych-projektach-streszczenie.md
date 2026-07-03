### Ekonomia kontekstu przy dużej skali

Moduł 4 pracuje z projektami dużymi i legacy. Pierwszy odruch — dopisywanie do jednego `AGENTS.md` — działa do czasu; drugi — pliki instrukcji w każdym folderze „na zapas" — kosztuje utrzymanie bez zwrotu.

Pytanie lekcji: co agent powinien wiedzieć **właśnie teraz**? Okno kontekstowe to skończony **budżet uwagi** (Maximum Effective Context Window): oficjalne miliony tokenów zawyżają okno efektywne — najmocniejsze modele ~250 tys. tokenów jakości, tańsze tracą już ~100 tys. Degradacja zależy od modelu i zadania.

Mechanizm z M1L4 przeniesiony na poziom architektury plików: im więcej kontekstu „na stałe", tym mniej uwagi na bieżące zadanie. Ta sama zasada wróci przy kodzie — **progresywne ujawnianie skali** (nie czytasz całego repo naraz).

Cztery tryby awarii monolitu (OpenAI harness engineering): **wypycha zadanie**, **rozcieńcza wskazówki** (gdy wszystko ważne, nic nie jest), **gnije** (reguły dezaktualizują się szybciej niż przegląd), **utrudnia weryfikację**. Anthropic: przepełniony `CLAUDE.md` → model ignoruje realne instrukcje przez szum. Badania (Codex, małe PR-y): nadmiarowy kontekst podnosi koszt i pogarsza wynik — traktuj kierunkowo.

**Higiena przed przebudową:** wytnij reguły niepotrzebne, zostaw te chroniące przed powtarzającymi się błędami agenta. Potem przenieś treść tam, gdzie agent sięgnie **just-in-time**.

### Architektura: lean root + context/

Trzy rodzaje kontekstu w pracy z agentami:

- **konwencje** → `AGENTS.md` / `CLAUDE.md` + `rules/` dopasowane do technologii,
- **referencje** (PRD, plany, research, decyzje) → `context/`,
- **procedury** (powtarzalne kroki) → skille, prompty.

Mapy kodu, analizy feature'ów i plany refaktoryzacji z modułu 4 trafiają do `context/`.

Root `AGENTS.md` = **spis treści, nie encyklopedia** — konwencje globalne, najważniejsze komendy, wskaźniki do reszty. `context/` = system-of-record. Agent sięga _just-in-time_: sam, gdy zadanie wymaga, lub po twojej referencji do pliku. Ty odpowiadasz za kontekst do bieżącego zadania; automatyczne skille i doczytywanie tylko częściowo wyręczają.

### Od czego zacząć?

Minimum na start dowolnego projektu: jeden `AGENTS.md` / `CLAUDE.md` + `context/`.

10xCards (MVP): `CLAUDE.md` ~82 linie — sekcje `Project`, `Commands`, `Architecture`, `Conventions`, `Reference`; zero PRD/planów w roocie.

`context/`:

- `context/foundation/` — `prd.md`, `roadmap.md`, `tech-stack.md`, `test-plan.md`,
- `context/changes/<id>/` — pojedyncze zmiany z planami i researchem,
- `context/archive/` — ukończone zmiany.

Procedury w folderach skilli i promptów.

Mechanika (moduł 1): `/10x-init` → szkielet `context/`; `/10x-agents-md` → lean root **odsyłający** do `context/`; `/10x-rule-review` → czy root nie spuchł i czy reguły zasługują na miejsce.

**Pierwszy, często jedyny etap:** root + scentralizowany `context/`. Per-modułowe `AGENTS.md` i `context/` to kolejne etapy — tylko gdy projekt wymaga, nie na zapas.

### Drabina dojrzałości i sygnały eskalacji

Drabina = wchodzisz **na żądanie**, nie odhaczasz kroków po kolei.

1. **root `AGENTS.md` + `context/`** — punkt startowy,
2. **`AGENTS.md` per moduł + indeks w roocie** — moduł złożony ze specyficznymi wzorcami (kluczowe dla agenta w tym obszarze, nieistotne dla innych) lub utrzymywany przez dedykowany zespół w monorepo,
3. **własny `context/` w module** — dedykowany PRD, roadmapa, intensywna praca zespołu.

**Sygnały eskalacji** (prawdziwa reguła decyzyjna — nie liczba folderów, nie „aplikacja duża"):

- główny `AGENTS.md` puchnie i jest nieczytelny (Anthropic: cel <~200 linii; kurs: podział powyżej ~300),
- agent **wielokrotnie gubi kontekst** modułu mimo poprawek w roocie,
- przy każdej pracy w module przekazujesz coraz więcej referencji lub brakuje dedykowanego PRD/roadmapy,
- moduł ma **własny deploy lub właściciela** — realna granica odpowiedzialności.

Dokładaj strukturę po porażkach agenta w konkretnych zadaniach lub przy realnej linii odpowiedzialności między zespołami.

### Kalibracja: twoje MVP vs większe repo

Projekty MVP (10xDevs): kilka modułów, ograniczona złożoność — **zostań na pierwszym szczeblu jak najdłużej**. Per-folderowe pliki = płacenie za strukturę bez zwrotu.

Wzorce ze skali (oglądamy **układ**, nie zmierzoną skuteczność agenta):

- **cloudflare/workers-sdk** — zwięzły root, jawny indeks potomnych `AGENTS.md`, potomne tylko lokalny kontekst + odesłanie do roota,
- **open-mercato** — wariant ciężki: 400+ linii root, router instrukcji, wiele `AGENTS.md` per moduł,
- **openai/codex** — 286-liniowy root, jedno potomne `AGENTS.md` mimo dziesiątek modułów; ironia: harness engineering chwali ~100-liniowy root, a publiczne repo ma 286.

**Liczy strategia, nie magiczna liczba plików.** Nie kopiuj workers-sdk do MVP. Scentralizowany kontekst, dopóki nie ma sygnałów z drabiny.

Ścieżka modułu 4 (10xArchitect) — narastająca analiza, każda lekcja = element raportu:

- L2 `context/map/repo-map.md` — mapa repozytorium,
- L3 `context/changes/{id}/research.md` — Feature overview + Technical debt,
- L4 `context/changes/{id}/plan.md` — plan refaktoryzacji,
- L5 `context/domain/` — notatki DDD.

Cztery artefakty → raport architektoniczny. Można różne repo per lekcja, byle komplet istniał i był obronny. **AI = analityk; decyzje architektoniczne = ty.**

### Jak wydzielić kontekst dla złożonych modułów

Reguła **simple by default** — MVP zostaje na szczeblu 1.

**Własny `AGENTS.md` per moduł** (szczebel 2): konwencje specyficzne nie mieszczą się w roocie, który i tak jest duży (~300+ linii, wyczucie). W module plik z konwencjami modułu; w roocie indeks potomnych.

Wzorzec cloudflare/workers-sdk — root kończy spisem dzieci:

```text
# AGENTS.md (root) — ~245 linii, konwencje całego monorepo
...
## Packages with their own AGENTS.md for deeper context
- packages/wrangler/AGENTS.md  — CLI architecture, command structure, test patterns
- packages/miniflare/AGENTS.md — Worker simulation
```

Potomny otwiera linią dziedziczenia, **nie kopiuje** monorepo:

```text
# packages/wrangler/AGENTS.md — ~31–91 linii
Wrangler-specific context only. See root AGENTS.md for monorepo conventions.

## Gotchas
- Entry point is `src/cli.ts`, NOT `src/index.ts`
- No `console.*` — use the `logger`
- No global `fetch` — use undici
```

Root: „jak pracujemy w całym repo i gdzie szczegóły"; potomny: „tylko Wrangler, resztę dziedziczę".

**Własny `context/` w module** (szczebel 3): dedykowany PRD/roadmapa — (1) dedykowany zespół w monorepo, (2) złożoność i tempo pracy przekraczające główny `context/`. Inicjalizacja: `/10x-init` w module; w `AGENTS.md` modułu — informacja o lokalnym `context/` bez powielania zawartości.

Reguła: **jeden `context/` w roocie zawsze; per-modułowy dopiero gdy brak realnie przeszkadza.** Nie `context/` w każdym module „na wszelki wypadek". **Dziel według granicy własności, złożoności i realnych potrzeb — nie liczby folderów.**

### Żeby reguły modułu naprawdę zadziałały

Wydzielenie kontekstu to połowa sukcesu. Dwa miejsca utraty korzyści:

**1. Miejsce startu sesji** — katalog roboczy decyduje, które pliki instrukcji ładują się automatycznie.

Claude Code z roota: na start tylko root `CLAUDE.md`; plik modułu **leniwie** przy pracy w module — agent może pominąć reguły modułu bez jawnej referencji. Start z wnętrza modułu: plik modułu + wszystkie nadrzędne, przewidywalnie. (Różnice między narzędziami — Deep Dive.)

Wniosek: reguły modułu krytyczne → odpal agenta z modułu **albo** przekaż jawny `@plik` z roota.

**2. Utrzymanie** — własne pliki modułu wymagają pielęgnacji, inaczej za pół roku szkodzą nieaktualnymi instrukcjami.

- Nie wkładaj szybko zmieniających się faktów do pliku reguł — unikaj per-modułowych plików na wczesnym etapie życia modułu,
- Okresowy przegląd (miesiąc/kwartał): `/10x-rule-review` + krytyczna analiza instrukcji.

Rozbudowuj **jeden moduł naraz**, w odpowiedzi na sygnał — nie „w pracy wypada mieć strukturę".

Ten sam budżet uwagi decyduje o nawigacji po kodzie w dużych projektach — punktowo wyciągasz informacje (mapa, moduł), potem refaktoryzacja i modernizacja nawet przy setkach tysięcy linii.

### Deep dive: Jak narzędzia naprawdę ładują pliki

> **Deep dive** — Model mentalny: **rozszerzanie**, nie podmiana. Plik bliższy zadaniu _dokłada się_ do roota. „Najbliższy wygrywa" — błąd.
>
> **Claude Code:** od korzenia FS w dół do cwd, skleja napotkane pliki (bliższy na końcu = dopowiada). Podkatalogi **leniwie** przy pierwszym dotknięciu. Importy `@ścieżka` rozwijają się przy starcie — **nie oszczędzają tokenów**. Test: sesja w podkatalogu monorepo vs z roota.
>
> **Codex:** wykrywa korzeń gita w górę; ładuje root→cwd **raz** przy starcie, skleja w jeden dokument; **brak** leniwego doczytywania. Haczyk: wspólny limit `project_doc_max_bytes` (~32–64 KiB, wersja zależna) — po przekroczeniu **po cichu obcina**; limit wspólny dla wszystkich sklejonych plików.
>
> **Cursor:** cztery typy reguł — always, inteligentne po opisie, globy na pliki, ręczne; wiązanie z _tematem_, nie tylko katalogiem.
>
> **Copilot:** **scala** reguły — `applyTo` + ogólna działają jednocześnie (addytywność).
>
> Wspólna nazwa pliku ≠ wspólne zachowanie. Spec `agents.md` 1.1 (propozycja): lokalny `AGENTS.md` _rozszerza_ nadrzędne. **Claude Code jako jedyne nie czyta `AGENTS.md`** — używa `CLAUDE.md`.

### Deep dive: Multi-repo: wyzwania

> **Deep dive** — Zagnieżdżanie kontekstu działa **wewnątrz jednego repo**. Agent w repo A domyślnie nie widzi repo B. Monorepo zamyka złożoność w drzewie plików; **multirepo/polyrepo** przerzuca ją na dystrybucję i scalanie kontekstu — własna infrastruktura (pakiet, CLI, MCP), ten sam rodzaj tarcia co współdzielenie kodu między serwisami.
>
> Odruch „meta-kontekstu" agregującego wszystkie serwisy szybko **dezaktualizuje się**.
>
> **Warstwa 1 — uniwersalny kontekst:** współdzielone skille, prompty, konwencje firmowe, polityki bezpieczeństwa — każdy agent w każdym repo.
>
> **Warstwa 2 — dystrybucja:** jedno źródło prawdy (`company/ai-toolkit`) + mechanizm synchronizacji — bez kopiuj-wklej rozrzuconych po repo kopii. Mechanizmy (rosnąca złożoność): pakiet npm (`@przeprogramowani/ai-toolkit`), CLI (`10x-cli` + API), serwer MCP. Każde repo i tak potrzebuje własnych zacommitowanych plików kontekstowych.
>
> **Warstwa 3 — widoczność cross-repo w runtime:** uniwersalne reguły nie pozwalają agentowi w A zrozumieć B. Potrzebna aktualna specyfikacja: OpenAPI/Swagger, GraphQL SDL, Protobuf, AsyncAPI; pakiety specyfikacji testów UI/API; TypeDoc w CI/CD — taniej i pewniej niż artefakty LLM. **Repomix** = doraźny snapshot (wysoki koszt tokenowy), nie infrastruktura kontraktowa.
>
> Trzy warstwy układają się jedna na drugiej — żadna nie zastępuje pozostałych. Unikaj przedwczesnego rozbijania monolitów na mikroserwisy. Praktyczna budowa Shared AI Registry — **M5L3**.

### Deep dive: Co mówią badania o AGENTS.md

> **Deep dive** — Dwa badania, oba Codex + małe PR-y (kierunkowo):
>
> [2601.20404]: dobry zwięzły `AGENTS.md` → **~28% niższy** medianowy czas zadania, **~17% mniej** tokenów — argument o _efektywności_, nie poprawności dla każdego agenta.
>
> [2602.11988]: nadmiarowy zbędny kontekst → skuteczność spadała **>20%**, koszt rósł; winowajca **redundancja i niepotrzebne wymagania**, nie sama długość pliku.
>
> Wniosek operacyjny: plik instrukcji opisuje **minimum konieczne**.
