### Wprowadzenie

Moduł trzeci zaczyna od pytania: dowozisz szybciej, ale skąd wiesz, że to nadal działa? Antywzorzec: agent pisze testy tam, gdzie najłatwiej (helpery, gettery, formatery) — coverage rośnie, CI zielone, a krytyczny flow użytkownika bez ochrony. Najpierw decyzja **co naprawdę musi być chronione**; znany cykl `research-plan-implement` wystarczy, jeśli wiesz co chronić. `/10x-test-plan` to warstwa wyżej — wirtualny inżynier jakości: **co** testować, **dlaczego**, **w jakiej kolejności**.

### Test plan w epoce AI

`/10x-test-plan` tworzy i pielęgnuje `context/foundation/test-plan.md` — strategia testowania + stan wdrożenia. Skill czyta plik przy każdym uruchomieniu i podpowiada kolejny krok. Greenfield bez kodu → skill zatrzyma się — wróć po `/10x-shape` i `/10x-prd`. Paczka lekcji:

```bash
npx @przeprogramowani/10x-cli@latest get m3l1
```

### Bramki przed produkcją

**Quality gates** — punkty kontrolne przed kolejnym etapem: lint, typecheck, testy jednostkowe, krytyczny flow, PR review, pre-commit, CI. Zmiana nie idzie dalej tylko dlatego, że agent skończył kod. Bramki opóźniają odruch „działa lokalnie, shipujemy". `/10x-test-plan` to orkiestracja nad cyklem z modułu 2 — nie silnik testów. Ponowne uruchomienie: agent patrzy na dysk (np. brak `research.md` → „czas na `/10x-research`"). Pliki = pamięć rollout'u; `/clear` lub nowy wątek OK.

### Najpierw ryzyka

Antywzorzec: `Write tests for this file.` — agent testuje widoczne ścieżki, nie ryzyka. `/10x-test-plan` startuje od ryzyk na podstawie: `context/foundation/prd.md`, `roadmap.md`, `context/archive/*/plan.md`, `tech-stack.md`, `AGENTS.md`/`CLAUDE.md`, konfiguracji testów. Historia GIT = sygnał churnu — skill proponuje zakres skanu do akceptacji (lockfile/snapshoty zatopią sygnał). Wywiad: co boisz się zepsuć, gdzie się sparzyłeś, co zmieniasz bez pewności; pytania dostosowane do stanu testów. Ostatnie pytanie: „na co NIE poszedłby budżet testowy?" → sekcja wykluczeń w planie. Mapa ryzyk: **wpływ** × **prawdopodobieństwo** (wysoki/średni/niski):

| Ocena | Wpływ | Prawdopodobieństwo |
| ----- | ----- | ------------------ |
| **Wysoki** | utrata dostępu, danych, pieniędzy; awaria publiczna | obszar zmienia się co tydzień lub już tu był błąd |
| **Średni** | gorsze działanie, obejście istnieje | kod ruszany od czasu do czasu |
| **Niski** | kosmetyka, łatwo cofnąć | kod stabilny, rzadko dotykany |

Priorytet: najpierw wysoki×wysoki, na końcu niski×niski. Skala zgrubna — zgoda co jest „górą" tabeli, nie precyzja do przecinka.

### Oś, o której łatwo zapomnieć: bezpieczeństwo

Ryzyka funkcjonalne wypływają z wywiadu; **nadużycia** — rzadko. Osobne pytanie: **co gdy ktoś użyje wbrew intencji?** Klasy: autoryzacja/dostęp (IDOR), wejście użytkownika (injection, walidacja serwerowa), sekrety/dane wrażliwe, nadużycie zasobów (rate-limit). Agent rzadko proponuje — happy path nie obejmuje atakującego; brak wiersza bezpieczeństwa na mapie przy auth/płatnościach = nikt nie zapytał.

### Sygnał, nie diagnoza

`/10x-test-plan` czyta repo dla **wskazówek**, nie diagnozy: „obszar często się zmieniał", „flow krytyczny w PRD" — nie „błąd w `session.ts:42`". Ryzyko opisane implementacyjnie → skill każe przeformułować na scenariusz awarii użytkownika. Konkret w kodzie → `/10x-research` w fazie rollout'u. Cykl: `/10x-research` → `/10x-plan` → `/10x-implement` → aktualizacja `test-plan.md`; research może skorygować plan (hot-spot vs rzeczywiste ryzyko w kodzie). Antywzorzec: udawanie wiedzy o kodzie na etapie planowania.

### Zielony test, który niczego nie chroni

**Problem wyroczni** (oracle problem): test musi znać oczekiwany wynik z wymagań/kontraktu, nie z implementacji. Agent domyślnie testuje kod kodem — `expect(fn()).toBe(0.15)` zabetonowuje błąd. Obrona: podaj progi/reguły biznesowe w prompcie; czytaj asercje („skąd ta wartość?"); zepsuj kod — test ma paść; przy „popraw aż zielone" pilnuj kierunku (test vs kod). `/10x-test-plan` opisuje zachowanie użytkownika, nie „funkcja X zwraca Y".

### Cookbook zamiast pamięci w głowie

Sekcja `## 6. Cookbook Patterns` startuje jako TBD — skill nie zgaduje lokalnych wzorców. Po fazie implementacji cookbook się wypełnia (lokalizacja, mocki, referencyjny test, komenda) — wzorzec dla kolejnych agentów. Bez cookbooka każda zmiana = zgadywanie kultury testów.

### Powroty do test-planu

`/10x-test-plan --status` — tabela faz i następna komenda. `/10x-test-plan --refresh` — rewizja planu po zmianach stacku/ryzyk. Po lekcji w repo: `context/foundation/test-plan.md` (mapa ryzyk, źródła, profil testów, fazy QA, typy testów, AI w QA, cookbook) — kontrakt wejściowy modułu 3.

### Deep dive: Testowanie oparte na ryzyku, bez AI

> **Deep dive** — Risk-based testing (ISTQB): impact × likelihood to klasyka QA, nie wynalazek LLM. Agent czyta PRD/roadmapę szybciej, ale potrzebuje twojej korekty biznesowej. Solo dev bez QA = mapa ryzyk jako segregacja.

### Deep dive: Sygnał kontra wiedza

> **Deep dive** — „obszar ryzykowny" (sygnał z PRD/git/wywiadu) ≠ „plik awarii" (wymaga researchu). Plan trzyma sygnały; `/10x-research` weryfikuje w kodzie. Churn w `src/lib/` ≠ dowód awarii w konkretnym pliku.
