### Wprowadzenie
Po M5L4 agent działa w zespole ze wspólnymi artefaktami. M2L5 skalował równoległą pracę przy biurku; tu kolejny krok: uruchomić zadanie, odejść od laptopa, sprawdzić status z telefonu, wrócić po zaplanowanym przebiegu.
Antywzorzec: mieszanie „agent w chmurze", „remote coding", „background agent", „loop" w jeden worek — to różne sytuacje operacyjne.
Kontrola nie znika — zmienia miejsce: konfigurujesz środowisko przed startem, monitorujesz wybiórczo, robisz review po zakończeniu. Niezależnie od trybu kończysz przy review wykonanej pracy.

### Kiedy kontrolujesz pracę agenta?
Zacznij od pytania: **kiedy człowiek ma kontrolować pracę?** Trzy tryby różnią moment kontroli, miejsce wykonania i główne ryzyko:

| Tryb | Gdzie działa agent | Co kontrolujesz | Kiedy pasuje | Główne ryzyko |
| --- | --- | --- | --- | --- |
| 1: zdalna kontrola lokalnego agenta | Twoja maszyna albo własny serwer | Sesję z innego miejsca | Krótka poprawka, diagnoza, kontynuacja z telefonu | Środowisko i bezpieczeństwo po twojej stronie |
| 2: sandbox w chmurze | Zarządzany sandbox | Setup, sieć, sekrety, zakres zadania | Zadanie po zamknięciu laptopa | Źle przygotowane środowisko albo za szerokie uprawnienia |
| 3: pętle i routines | Chmura albo harmonogram | Trigger, kryteria sukcesu, warunki stopu | Regularny przegląd, raport, self-check | Zielony przebieg bez realnego sukcesu, koszt, zapętlenie |

Tabela ważniejsza niż nazwy produktów — z trybu łatwiej dobrać narzędzie.

### Tryb 1: zdalna kontrola lokalnego agenta
Obliczenia nie idą do chmury — zmienia się tylko interfejs kontroli. Klasycznie: SSH/mosh + `tmux` (sesja przeżywa rozłączenie); bez nowego dostawcy w łańcuchu zaufania, ale serwer, sekrety i sieć utrzymujesz sam.
Wybór miejsca: codzienna maszyna vs VPS — VPS bezpieczniejszy (agent nie na prywatnych plikach i kluczach), kosztem przygotowania środowiska.
**Claude Code Remote Control:** `claude remote-control` lub `/remote-control` w sesji → przejęcie z `claude.ai/code` lub aplikacji mobilnej. Agent lokalnie, telefon/przeglądarka to okno na sesję; ruch wychodzący HTTPS, bez otwierania portów. Research preview; plany Pro/Max/Team/Enterprise (nie API key); Claude Code ≥2.1.51; na Team/Enterprise wymaga włączenia przez admina.
**Happy** — gdy Remote Control nie pasuje (klucz API, open source, Codex): lokalny Claude Code/Codex, sterowanie z telefonu przez zaszyfrowany relay. Rozwiązuje NAT bez port forwardingu, ale to trzecia strona w ścieżce sterowania — sprawdź szyfrowanie, repo, parowanie urządzeń i politykę bezpieczeństwa zespołu.
Tryb gdy chcesz mobilności bez oddawania wykonania do zarządzanego sandboxa.

### Co przygotowujesz przed startem?
Skrypt setupu = kontrakt uruchomieniowy przed pracą Agenta: zależności systemowe, CLI, runtime, menedżery pakietów. Brakujące `gh`, migracje, menedżer pakietów — zapisz w setupie, nie licz że Agent „jakoś ogarnie".

### Kiedy sandbox ma dostęp do internetu?
Sieć to kluczowy przełącznik: allowlist vs internet tylko w setupie. Jeśli pakiety w setupie, a Agent pracuje offline — prompt „zainstaluj pakiet w trakcie" kończy się frustracją; to źle ustawiona granica, nie błąd Agenta.

### Jak przenosisz MCP i konfigurację narzędzi?
Sandbox widzi tylko konfigurację dostępną przy starcie. MCP w lokalnym profilu użytkownika — chmura nie zobaczy. Wzorzec: `.mcp.json` w repozytorium (artefakt ze Shared AI Registry M5L4). Test dojrzałości: czy agent w chmurze ma ten sam kontekst narzędziowy co lokalny?

### Co trafia do cache'u?
Cache przyspiesza powtarzalny setup; zmiana setupu, zmiennych, sekretów lub sieci go unieważnia. Powtarzalny, krótki, jawny setup — cache pomaga; zbiór ręcznych napraw — cache ukrywa problem.

### Jak ograniczasz sekrety?
Minimalny zakres, sensowny czas życia; bez sekretów produkcyjnych. W pracy asynchronicznej skrót „token z szerokim dostępem bo testy" boli mocniej — agent działa długo bez nadzoru.

### Jakie narzędzia zakładasz jako dostępne?
Nie mieszaj list między dostawcami (Claude Code Web, Codex Cloud — różne obrazy, sieć, MCP, cache). Zapisz jawnie: czego wymaga zadanie, co w setupie, co przez MCP, do jakich hostów może łączyć, jakich sekretów nie dostać. Asynchroniczna praca zaczyna się przed uruchomieniem Agenta.

### Dlaczego ograniczenia pozwalają odejść od biurka
Izolacja filesystemu i sieci ogranicza promień rażenia — bez niej agent może czytać pliki spoza projektu, wysyłać dane na zewnątrz, używać nadmiarowych dostępów. W pracy bez stałego nadzoru granice muszą być ustawione wcześniej. **Izolacja nie jest hamulcem — jest warunkiem autonomii.**
Reguła: im mniej patrzysz w czasie rzeczywistym, tym bardziej środowisko musi wymuszać zakres. Repo zadania + allowlista hostów + krótkie sekrety + jasny setup = dłuższa autonomia. Lokalna maszyna z pełnym dostępem + zamknięty laptop = ryzyko operacyjne, nie delegacja.
Case study: aktualizacja test planu modułu płatności po zmianie limitów subskrypcji — bez sekretów produkcyjnych, Stripe, decyzji produktowych. Zakres: diff i dokumentacja modułu, dopisz scenariusze, nie zmieniaj kodu aplikacji.
Przed startem: cel, granice, setup, sieć, MCP, sekrety, warunek stopu, kryteria review — część w konfiguracji sandboxa, reszta w poleceniu:

```text
/goal Gotowe, gdy test plan modułu płatności zawiera scenariusze limitów subskrypcji.
Zakres: edytuj tylko docs/test-plans/payments.md; kod z src/modules/payments/** czytaj, nie zmieniaj.
Nie zmieniaj kodu produkcyjnego bez osobnej zgody i nie merguj.
Setup: `npm ci` w fazie przygotowania; zależności pobierają się w setupie, nie w trakcie pracy.
Sieć: wyłączona w fazie pracy, pakiety pobrane wcześniej w setupie.
MCP: tylko konfiguracja widoczna dla sandboxa (np. .mcp.json z repo), bez serwerów z lokalnego profilu.
Sekrety: brak; żadnych tokenów produkcyjnych ani z prawem zapisu.
Zatrzymaj się po aktualizacji test planu i krótkim raporcie z luk (limit ~20 tur).
Review (zielony przebieg to NIE sukces): diff dotyczy tylko test planu, raport wymienia założenia i niepewności, Agent nie wyszedł poza zakres ani nie poprosił o szersze sekrety.
```

Telefon = panel kontroli, nie czytanie diffu: czy setup przeszedł, czy blokada sieci/MCP/sekretów zatrzymała zadanie, czy agent rozszerza zakres, czy doprecyzować/zatrzymać/zostawić do review. Pełna ocena przy komputerze: diff, logi, testy, kryteria z przed startu.

### Tryb 3: pętle i rutyny
Kontrola cykliczna, nie „po jednej sesji". Rutyna, zaplanowany przebieg, pętla `/loop` — zadanie powtarzalne według harmonogramu lub zdarzenia (PR co godzinę, dokumentacja po release, issue z etykietą `ai-candidate`, self-check modułu).
Kontrakt przebiegu: **cel**, **warunek stopu**, **limit zakresu** (pliki, repo, etykiety), **limit kosztu** (iteracje, czas, budżet), **kryteria sukcesu**, **kryteria porażki** (zielony status techniczny ≠ sukces zadania).
Antywzorzec w pętlach: nudne ryzyko — dwie godziny pobierania z zablokowanego hosta, duplikaty raportów, poprawianie objawu, branch bez bezpiecznego merge. Pętla automatyzuje start, nie zwalnia z review.

### Czym właściwie delegujesz wykonanie
Trzy tryby mówią _kiedy_ odzyskujesz kontrolę; tu _co_ oddajesz — zwykle plan z `/10x-plan`, nie luźny prompt. `/10x-implement` = interaktywnie przy biurku. Asynchronicznie: `npx @przeprogramowani/10x-cli@latest get m5l5` → **`/10x-goal-implement`** pod `/goal` (interaktywnie lub headless `claude -p`).
Polityka zamiast człowieka w pętli: implementacja fazy → bramki jakości (kryteria z planu, deliberate-break check, pełen zestaw testów) → commit na zielono (Conventional Commits) → kolejna faza. Przy niejednoznaczności lub rozjazdzie z planem: STOP zamiast zgadywania. Po przebiegu: raport i ręczne kroki `#### Manual`. Plan = runbook, `/goal` = warunek stopu, skill = polityka bez ciągłej obecności.

### Kontrola nie zniknęła
Tryb 1: prowadzisz agenta z innego urządzenia. Tryb 2: kontrolujesz setup, sieć, MCP, sekrety i zakres przed startem, wynik w review. Tryb 3: trigger, warunek stopu, budżet, kryteria sukcesu. Sekwencja: wybierz tryb → skonfiguruj środowisko → deleguj ograniczone zadanie → wróć do review.

### Deep dive: Szczegóły sandboxów w chmurze
> **Deep dive** — Trwały model: skrypt setupu, polityka sieci, MCP, cache, minimalne sekrety, jawne założenia o obrazie bazowym. Claude Code Web i Codex Cloud — ten sam mechanizm, różne domyślne polityki sieci (allowlist vs internet w setupie vs faza pracy). Przed wdrożeniem w zespole sprawdź aktualną dokumentację dostawcy.

### Deep dive: SSH, tmux, Remote Control i Happy
> **Deep dive** — SSH+`tmux`: sensowne przy własnych maszynach dev, VPN, bez nowego dostawcy; brak automatycznej izolacji — bezpieczeństwo z twojej konfiguracji. Remote Control: wykonanie lokalnie, sterowanie przez API Anthropic. Happy: lokalnie + relay, wymaga decyzji zaufania. Pytanie decyzyjne: problemem jest interfejs kontroli (lokalny agent + zdalna kontrola) czy miejsce wykonania (sandbox/VPS)?

### Deep dive: Cloudflare Agents i Workflows
> **Deep dive** — Nie czwarty sandbox do tabeli, lecz platforma na własne trwałe środowisko asynchroniczne: tożsamość agenta, stan, sesje, harmonogramy, Workflows (ponawianie, oczekiwanie, zatwierdzenia). Ścieżka przy własnym systemie agentowym dla zespołu; budowanie środowiska uruchomieniowego to osobny projekt.

### Deep dive: Ralph, `/goal`, `/loop` i routines
> **Deep dive** — Ralph: pętla ze stanem na dysku, świeży kontekst, iteracja do kryteriów sukcesu. Routines i `/loop` (nazwa zależna od narzędzia) — zdalna/zaplanowana pętla wymaga warunku stopu i review; bez tego: niekontrolowany koszt i zapętlenie.
