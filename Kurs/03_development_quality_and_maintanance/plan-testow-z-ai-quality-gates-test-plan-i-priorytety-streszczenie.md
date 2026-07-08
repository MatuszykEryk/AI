### Wprowadzenie

Moduł trzeci zaczyna od pytania: dowozisz szybciej, ale skąd wiesz, że to nadal działa? Antywzorzec: agent pisze testy tam, gdzie najłatwiej (helpery, gettery, formatery) — coverage rośnie, CI zielone, a krytyczny flow użytkownika bez ochrony. Najpierw decyzja **co naprawdę musi być chronione**; znany cykl `research-plan-implement` wystarczy, jeśli wiesz co chronić. `/10x-test-plan` to warstwa wyżej — wirtualny inżynier jakości: **co** testować, **dlaczego**, **w jakiej kolejności**.

### Test plan w epoce AI

`/10x-test-plan` tworzy i pielęgnuje `context/foundation/test-plan.md` — strategię testowania połączoną z bieżącym stanem wdrożenia. Skill czyta ten plik przy każdym uruchomieniu i podpowiada kolejny krok rolloutu QA. Greenfield bez kodu → skill zatrzyma się i odeśle do wcześniejszych etapów (`/10x-shape`, `/10x-prd`), bo plan testów potrzebuje realnego kontekstu z repo. Paczka lekcji:

```bash
npx @przeprogramowani/10x-cli@latest get m3l1
```

### Bramki przed produkcją

**Quality gates** to punkty kontrolne przed kolejnym etapem: lint, typecheck, testy jednostkowe, test krytycznego flow, PR review, pre-commit, CI. Zmiana nie idzie dalej tylko dlatego, że agent skończył kod i wygląda to „na gotowe". Bramki celowo opóźniają odruch „działa lokalnie, shipujemy". `/10x-test-plan` nie zastępuje testów — orkiestruje cykl z modułu 2. Ponowne uruchomienie działa stanowo: agent patrzy na dysk (np. brak `research.md` → „czas na `/10x-research`"). Pliki = pamięć rolloutu; możesz bezpiecznie czyścić sesję (`/clear`) i wracać później.

### Najpierw ryzyka

Antywzorzec: `Write tests for this file.` — agent testuje widoczne ścieżki, nie ryzyka biznesowe. `/10x-test-plan` startuje od mapy ryzyk na podstawie: `context/foundation/prd.md`, `roadmap.md`, `context/archive/*/plan.md`, `tech-stack.md`, `AGENTS.md`/`CLAUDE.md` i istniejącej konfiguracji testów. Historia GIT to sygnał churnu, nie wyrok: skill proponuje zakres skanu do akceptacji (lockfile/snapshoty zatopią sygnał). Wywiad zbiera obawy projektowe: co boisz się zepsuć, gdzie już się sparzyłeś, co ruszasz bez pewności; pytania dopasowują się do stanu projektu. Ostatnie pytanie „na co NIE powinien iść budżet testowy?" trafia do sekcji wykluczeń w planie i chroni przed przyszłym „testowaniem wszystkiego".

| Ocena | Wpływ | Prawdopodobieństwo |
| ----- | ----- | ------------------ |
| **Wysoki** | utrata dostępu, danych, pieniędzy; awaria publiczna | obszar zmienia się co tydzień lub już tu był błąd |
| **Średni** | gorsze działanie, obejście istnieje | kod ruszany od czasu do czasu |
| **Niski** | kosmetyka, łatwo cofnąć | kod stabilny, rzadko dotykany |

Priorytet: najpierw wysoki×wysoki, na końcu niski×niski. Skala jest celowo zgrubna — celem jest wspólna decyzja „to jest wysoko / to jest nisko", a nie pseudo-precyzyjna matematyka.

### Oś, o której łatwo zapomnieć: bezpieczeństwo

Ryzyka funkcjonalne zwykle wypływają z wywiadu, ale scenariusze **nadużyć** często nie. Dlatego dodaj osobne pytanie: **co się stanie, gdy ktoś użyje systemu wbrew intencji?** Klasy bazowe: autoryzacja i dostęp (IDOR), wejście użytkownika (injection + walidacja po stronie serwera), sekrety i dane wrażliwe (wycieki), nadużycie zasobów (rate-limit, kosztowne operacje w pętli). Agent rzadko zaproponuje to sam — happy path nie obejmuje atakującego. Brak choć jednego wiersza security przy auth/płatnościach to zwykle sygnał, że nikt o to nie zapytał.

### Sygnał, nie diagnoza

To najważniejsza granica skilla: `/10x-test-plan` zbiera **sygnały**, nie stawia diagnozy kodu. Ma mówić „ten obszar często się zmieniał", „to krytyczny flow z PRD", a nie „błąd jest w `session.ts:42`". Gdy wpiszesz ryzyko w formie implementacji, skill każe przeformułować je na scenariusz awarii użytkownika. Dopiero `/10x-research` schodzi do kodu i sprawdza, gdzie ryzyko naprawdę żyje. Praktyczny cykl: `/10x-research` → `/10x-plan` → `/10x-implement` → aktualizacja `test-plan.md`. Research może skorygować wcześniejsze założenia planu (np. hot-spot z churnu okaże się mniej ważny niż realna granica API). Antywzorzec: udawanie wiedzy o kodzie na etapie planowania strategicznego.

### Zielony test, który niczego nie chroni

**Problem wyroczni** (oracle problem): test potrzebuje niezależnego źródła prawdy (wymaganie, kontrakt, reguła biznesowa), a nie tego, co aktualnie zwraca implementacja. Agent domyślnie „testuje kod kodem" — `expect(fn()).toBe(0.15)` potrafi zabetonować błąd i dać zielony wynik. Obrona: podaj progi/reguły biznesowe wprost w prompcie; czytaj asercje pytaniem „skąd ta wartość?"; celowo zepsuj kod i sprawdź, czy test pada; przy poleceniu „napraw, aż zielone" pilnuj kierunku (kod ma dochodzić do wymagań, nie test do kodu). `/10x-test-plan` zmniejsza to ryzyko, bo definiuje ochronę na poziomie zachowania użytkownika, a nie wartości zwracanej przez pojedynczą funkcję.

### Cookbook zamiast pamięci w głowie

Sekcja `## 6. Cookbook Patterns` startuje jako TBD — skill nie powinien zgadywać lokalnych wzorców, zanim zobaczy je w praktyce. Po każdej fazie implementacji cookbook się uzupełnia (lokalizacja testów, polityka mockowania, referencyjny test, komenda uruchomienia). Dzięki temu kolejny agent (albo ty za miesiąc) nie zgaduje kultury testów w projekcie.

### Powroty do test-planu

`/10x-test-plan --status` zwraca krótką tabelę faz i następną komendę. `/10x-test-plan --refresh` uruchamia rewizję planu po zmianach stacku, narzędzi albo ryzyk. Po tej lekcji w repo powinien istnieć `context/foundation/test-plan.md` (mapa ryzyk, źródła, profil testów, fazy QA, typy testów, rola AI, cookbook) — kontrakt wejściowy dla kolejnych lekcji modułu 3.

### Deep dive: Testowanie oparte na ryzyku, bez AI

> **Deep dive** — Risk-based testing (ISTQB): impact × likelihood to klasyka QA, nie wynalazek LLM. Agent czyta PRD/roadmapę szybciej, ale potrzebuje twojej korekty biznesowej. Solo dev bez QA = mapa ryzyk jako segregacja.

### Deep dive: Sygnał kontra wiedza

> **Deep dive** — „obszar ryzykowny" (sygnał z PRD/git/wywiadu) ≠ „plik awarii" (wymaga researchu). Plan trzyma sygnały; `/10x-research` weryfikuje w kodzie. Churn w `src/lib/` ≠ dowód awarii w konkretnym pliku.
