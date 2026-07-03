### Problem: agent nie zna konwencji projektu

Po bootstrapie agent potrafi budować kod, ale bez instrukcji projektowych każda nowa sesja powtarza te same rozjazdy: inna obsługa błędów, brak walidacji API, niespójne nazewnictwo. Poprawiasz ręcznie, promptujesz ponownie — stajesz się korektorem agenta zamiast prowadzącym projekt. Przyczyna to nie słaby model ani zły prompt, lecz brak wiedzy o lokalnych konwencjach — które reguły wynikają z frameworka, a które są waszą umową.

### Co agent widzi i czego nie widzi na starcie

Każda sesja startuje z wstępnie wypełnioną pamięcią roboczą (oknem kontekstowym): instrukcje systemowe, reguły projektu, pamięć między sesjami, komendy i bieżący prompt. Nawet w pustym repo agent nie startuje od zera — ale brakuje mu kontekstu specyficznego dla projektu.

Składowe okna kontekstowego mają różnych właścicieli: **System prompt** (producent — bezpieczeństwo, narzędzia, kompresja; nie edytujesz), **Auto-memory** (agent buduje w sesji; wymaga regularnego przeglądu), **CLAUDE.md / AGENTS.md** (stałe reguły zespołu — konwencje, ograniczenia, reguły niewywnioskowalne z kodu), **Skille i komendy** (`/init`, `/commit`, `/review`, własne procedury), **Prompt zadaniowy / Messages** (wiadomości bieżącego zadania). Twój prompt to wierzchołek góry lodowej — reszta warstw decyduje, jak agent go zinterpretuje, co zrobi najpierw i które pliki wczyta.

Bez instrukcji projektowych prompt staje się jednocześnie poleceniem, pamięcią projektu i awaryjnym hamulcem. Błąd interpretacji może uruchomić kaskadę: zły plik, zły wzorzec, zła komenda, poprawka do własnej błędnej poprawki. Instrukcje przesuwają kontrolę przed start sesji — agent dostaje ramy ograniczające zgadywanie.

### Od /init do pierwszego szkicu

Wdrożenie reguł zacznij od wbudowanych komend — dają `safe defaults`. W Claude Code: `/init` analizuje repo i generuje startowy `CLAUDE.md` (komendy build/test/format, struktura, konwencje); jeśli plik istnieje — proponuje ulepszenia zamiast nadpisywać. Codex przy tej samej komendzie tworzy `AGENTS.md`. Eksperymentalny skill `/10x-agents-md` to alternatywa lub ścieżka awaryjna bez własnego generatora — przygotowuje szkic `AGENTS.md`, ale wymaga ręcznego przeglądu lokalnych konwencji. GitHub Copilot nie ma `/init` w CLI; główna konwencja to `.github/copilot-instructions.md` (wspiera też `AGENTS.md` — zdecyduj, który plik jest źródłem prawdy).

#### Eksperymentalny `/init` w Claude Code

Flaga `CLAUDE_CODE_NEW_INIT` rozszerza `/init` o krótki onboarding: pytanie o zespołowy `CLAUDE.md`, prywatny `CLAUDE.local.md` lub oba; propozycje skilli i hooków. Skille — powtarzalne, niekoniecznie automatyczne workflowy (weryfikacja, raport, deploy, release checklist). Hooki — mechaniczne kontrole, których agent nie pomija (formatter po edycji, lint/test, komenda na końcu tury). Hook = deterministyczny sygnał z narzędzia; skill = procedura wywoływana na żądanie. Agent najpierw bada repo, dopytuje o luki niewyczytane z kodu, potem proponuje artefakty.

```bash
CLAUDE_CODE_NEW_INIT=true claude
```

```text
/init
```

Funkcja eksperymentalna — traktuj jako skrót konfiguracyjny, nie stabilny standard. Efekt oceniaj tym samym filtrem co zwykły szkic: czy instrukcja, skill lub hook usuwa realny, powtarzalny błąd agenta.

### Jak oczyścić szkic

Antywzorzec: puchnięcie pliku — architektura, stack, konwencje frameworka, komendy z README, „pisz czytelny kod". Obszerne reguły zajmują okno kontekstowego (gorsza interpretacja kolejnych wiadomości) i wprowadzają redundancję względem README lub wiedzy treningowej.

Granica praktyczna: **ok. 200 linii** na plik `CLAUDE.md` — większy plik często oznacza płacenie kontekstem za rzeczy niepotrzebne w każdej sesji. Cursor: dobra reguła jest **skupiona, wykonywalna i ograniczona zakresem** — mówi, co zrobić inaczej w konkretnym miejscu, nie opisuje całego projektu.

Filtr czyszczenia: reguła dla całego repo → główny `AGENTS.md` lub `CLAUDE.md`; tylko część kodu → zagnieżdżony `AGENTS.md`, lokalny `CLAUDE.md` lub `.cursor/rules` z globem; powtarza README, style guide, dokumentację frameworka lub linter → usuń, zostaw referencję; brzmi jak intencja („pisz czytelny kod") → zamień na sprawdzalne zachowanie (np. `unikaj typu "any"`) lub usuń; nie wynika z powtarzalnego błędu agenta → nie dodawaj „na zapas". Zamiast kopiować przykładowy komponent — wskaż plik wzorcowy: `@src/features/users/user.service.ts`, `@docs/api-errors.md`. Zacznij prosto, rozbijaj duże reguły, referencjonuj pliki zamiast kopiować treść; aktualizuj dopiero przy powtarzalnym błędzie.

### Jak oceniać jakość reguł i instrukcji dla Agenta?

**Test inkluzji:** Czy agent mógłby to wiedzieć bez tego pliku? Czy publiczne dane treningowe (książki, artykuły, repozytoria w twoim stacku) mogły go przygotować? Jeśli tak — zwykle nie dopisuj.

Do `AGENTS.md` / `CLAUDE.md` należą: nieoczywiste konwencje (format błędów), nietypowe nazewnictwo, zasady importów niewywnioskowalnych z kodu, pułapki projektu, „wstydliwe" obejścia z historii kodu. Nie należą: dokumentacja mainstreamowego frameworka, komendy z README (wystarczy `@README.md`), popularne zasady już w `tsconfig.json`, ogólne intencje („pisz czysty kod"). Plik to onboarding agenta znającego TypeScript, ale nie waszych lokalnych konwencji. Narzędzia wymienione w `AGENTS.md` są używane 1,6–2,5× częściej — plik realnie kieruje zachowaniem.

Po szkicu (`/init`, `/10x-agents-md` lub ręcznie) uruchom przegląd:

```text
/10x-rule-review AGENTS.md
/10x-rule-review .cursor/rules/api.mdc
/10x-rule-review src/api/AGENTS.md
```

Skill ocenia plik w pięciu wymiarach (długość, wklejony kod/konfiguracja, precyzja, redundancja, kolejność) — werdykty `OK`, `WARN`, `FAIL` z poprawkami; domyślnie nie edytuje pliku. Potem uruchom świeżą sesję z reprezentatywnym zadaniem — dopiero wtedy widać, czy reguła zmienia zachowanie.

### Pięć wzorców do przetestowania

Kalibracja reguł: sprawdź, gdzie agent łamie konwencje. Wybierz jeden wzorzec: (1) format błędów `{ error: { code, message, context } }`, (2) nazewnictwo `feature.handler.ts`, (3) absolute imports `@/`, (4) struktura modułu `index.ts`, `types.ts`, `__tests__/`, (5) daty przez UTC i `formatDate()`.

Test A/B: (1) implementacja bez reguły, 3–5× ze świeżej sesji i tego samego stanu repo; (2) zanotuj, gdzie złamał konwencję, czas, komendy, tokeny; (3) dopisz minimalną regułę w 1–3 zdaniach; (4) ta sama zmiana w świeżej sesji — porównaj konwencję, czas, liczbę plików i iteracji. Agent trafia w konwencję bez reguły → wpis niepotrzebny; systematycznie wybiera zły wzorzec → dobra instrukcja niestandardowa.

### Hierarchia instrukcji — AGENTS.md oraz CLAUDE.md

Claude Code wczytuje `CLAUDE.md` z kilku miejsc (kolejność ma znaczenie): katalog użytkownika (`~/.claude/CLAUDE.md`), korzeń repo, podkatalogi bieżącej pracy — im głębiej, tym reguły bardziej szczegółowe i nadpisujące wyższe. Codex i Copilot analogicznie dla `AGENTS.md` — najbliższy plik w drzewie wygrywa. Claude Code importuje `@AGENTS.md` jedną linią w `CLAUDE.md` — naturalny punkt ujednolicenia przy wielu narzędziach.

Mechanizm **U-shaped attention**: modele poświęcają więcej uwagi treściom na początku i końcu kontekstu niż w środku. Reguły zakopane głęboko w długim pliku lub wczytywane jako czwarty/piąty dokument mają mniejsze szanse na spójne przestrzeganie. Wniosek: najważniejsze reguły na górze pliku; reguły obszarowe (`src/api/CLAUDE.md`) — krótsze, wczytywane selektywnie, blisko początku swojej sekcji.

### Lekcje z incydentów agenta

Dobre reguły biorą się z incydentów, nie z teorii. Do `context/foundation/` (obok `prd.md`, `tech-stack.md`) dodaj `context/foundation/lessons.md` — rejestr powtarzalnych lekcji dopisywany tylko na końcu pliku; wpływają na frame, research, planowanie, implementację, review. Nie kronika każdego błędu — tylko reguły, które będą wracać.

```text
/10x-lesson
/10x-lesson feature flags should always have a kill date
```

Skill zadaje cztery pytania i dopisuje lekcję: **Context** (gdzie obowiązuje), **Problem** (co psuje się bez reguły), **Rule** (1–2 zdania rozkazujące), **Applies to** (`frame`, `research`, `plan`, `plan-review`, `implement`, `impl-review` lub `all`). Plik rośnie przez dopisywanie — nie sortuj, nie deduplikuj, nie przerabiaj wcześniejszych wpisów. Incydent jednorazowy → notatka lokalna lub poprawka promptu; klasa błędów zmieniająca decyzje w kolejnych zadaniach → `/10x-lesson`. Referencja w głównych instrukcjach: `Zobacz: context/foundation/lessons.md`.

### Ustawienia zespołowe

Społeczność kieruje się ku `AGENTS.md`; Claude Code importuje go przez `CLAUDE.md`:

```markdown
This file provides guidance to AI Agents when working with code in this repository:
@AGENTS.md
```

Układ wielonarzędziowy: `AGENTS.md` jako wspólne źródło prawdy, `CLAUDE.md` jako cienka warstwa z importem, `.github/copilot-instructions.md` tylko gdy Copilot potrzebuje doprecyzowań. Symlink: `ln -s AGENTS.md CLAUDE.md`. Jedna reguła nie powinna żyć w trzech miejscach — duplikaty rozjeżdżają się jak duplikaty kodu.

### Deep dive: badania nad AGENTS.md

> **Deep dive** — Dwa badania: na Codexie dobrze użyty `AGENTS.md` skrócił medianowy czas małych PR-ów (~100 linii) o ~28% i tokeny o ~16%; na Claude Code, Codexie i Qwen Code redundantne pliki (LLM-generowane lub kopia dokumentacji) podniosły koszty o ~20–23% i pogorszyły skuteczność. Szkodzi redundancja z istniejącymi dokumentami, nie sam fakt posiadania pliku.

### Deep dive: systemowe prompty popularnych narzędzi

> **Deep dive** — Publiczne system prompty pokazują układ: tożsamość i bezpieczeństwo, operacje systemowe, kontekst projektu, reguły zadań, dokumentacja narzędzi. Claude Code: hierarchia bezpieczeństwa ponad kodem; zasada **blast radius** (szacuj odwracalność — lokalne edycje OK, destrukcyjne operacje wymagają zgody); ostrzeżenie, że auto-memory to roszczenie z przeszłości, nie fakt („memory says X" ≠ „X exists now"); narzędzia ładowane leniwie (`Deferred Tools` + `ToolSearch`). Cursor: stan terminala jako pliki w `terminals/` — pasywna injekcja zamiast jawnego API. **AGENTS.md zawsze po warstwie producenta, przed promptem zadaniowym** — producent definiuje co agent może, `AGENTS.md` jak pracujesz w projekcie.

### Deep dive: ślady po sesjach

> **Deep dive** — Auto-memory trafia do okna kontekstowego przyszłych zadań. Claude Code: `/memory`, wyłączenie `autoMemoryEnabled` lub `CLAUDE_CODE_DISABLE_AUTO_MEMORY`; plik `~/.claude/projects/<nazwa-katalogu>/memory/MEMORY.md` (ścieżka repo z `/` → `-`); na starcie ładuje pierwsze 200 linii lub 25 KB. Po sesji sprawdź `MEMORY.md` — usuń błędne decyzje, dopisz brakujące ważne. Reset: usuń katalog `memory/`. Codex: pamięć domyślnie wyłączona; włączenie `[features] memories = true` w `~/.codex/config.toml`; kontrola w wątku przez `/memories`; pliki w `~/.codex/memories/`. Pamięć lokalna pomaga, ale nie jest źródłem prawdy — zasady zespołowe trzymaj w `AGENTS.md` lub dokumentacji w repo.
