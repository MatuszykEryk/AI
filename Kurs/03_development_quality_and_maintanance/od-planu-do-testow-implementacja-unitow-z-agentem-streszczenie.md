### Wprowadzenie

Antywzorzec: prosisz o testy, dostajesz zielony coverage — ale testowanie to obszar, w którym AI łatwo oszukać. Skuteczność agentów spada na funkcjach nietypowych względem danych treningowych. Na benchmarkach TestEval pokrycie ~92% linii i ~82% gałęzi; na realnym kodzie spada do ~45% linii i ~30% gałęzi (Huang i in., ACM TOSEM 2026) — wynik z benchmarku nie przenosi się automatycznie na twój projekt. Najtrudniejsze to nie składnia, lecz **problem wyroczni** — decyzja, jaki test realnie chroni projekt. Przykład `getNextInterval(prevInterval, grade)` zwraca `prevInterval * grade`; dla `grade=0` powinien reset do 1 dnia, nie zero (zero = karta nigdy nie wraca do nauki) — agent asercjonuje błąd z implementacji. Model odwzorował to, co kod robi, nie to, co powinien. Wnioski: **duży coverage ≠ ochrona** (linia wykonana ≠ sensowna asercja); **naiwne promptowanie za mało** — potrzeba analizy ryzyk, kontekstu i optymalnej liczby testów. Fundament z m3-l1: `context/foundation/test-plan.md` — mapa ryzyk, fazowy rollout i cykl `/10x-new` → `/10x-research` → `/10x-plan` → `/10x-implement`.

### Vibe Testing, czyli jak nie wprowadzać testów

Antywzorzec: `Write tests for the auth module.` — testy odzwierciedlają implementację, nie zachowanie użytkownika. Przykład lustra: `expect(result).toEqual(await authenticate(input))`. Trzy klasy problemów:

| Antywzorzec | Jak wygląda | Co powinno się wydarzyć |
| ----------- | ----------- | ----------------------- |
| Mirror implementacji | wewnętrzne wywołanie, prywatny szczegół, ta sama logika | obserwowalne zachowanie, wynik scenariusza biznesowego |
| Happy paths | łatwe dane poprawne | co najmniej jeden edge case z ryzyka |
| Brak przypadków brzegowych | brak null/pustych/błędów | przypadek realnie psujący UX |

Wymagania procesu: **konkretne wejście ze scenariuszem ryzyka** + **osobna weryfikacja asercji**.

### Jak Test Plan rozbraja te pułapki

`context/foundation/test-plan.md` + cykl `/10x-new` → `/10x-research` → `/10x-plan` → `/10x-implement` rozbraja pułapki przed pierwszą asercją:

- **Ryzyko zamiast pliku** — mapa mówi, jaki scenariusz biznesowy nie może się zepsuć (wpływ, prawdopodobieństwo, źródło), nie „pokryj ten plik".
- **Wyrocznia z researchu** — przy ryzyku: jakie zachowanie udowodni ochronę i jakie wygodne założenie agent ma zakwestionować (np. „udane logowanie nie znaczy od razu, że użytkownik widzi właściwą treść strony").
- **Cost × signal** — dla każdego ryzyka najtańszy test z realnym sygnałem; bez skoków do e2e „bo bezpieczniej" i bez pogoni za procentem coverage.
- **Handoff w `/10x-plan`** — każdy etap testowy: jakie zachowanie asercjonuje, jaką regresję łapie, skąd kontekst z `research.md`, co najmniej jeden edge case, jakiego antywzorca unika.

### Case study: od ryzyka do testów integracyjnych

10xCards — zapis (`save`) promuje zaakceptowane drafty do talii. Ta sama pętla co w zadaniu: `test-plan.md` → `/10x-research` → `/10x-plan` → `/10x-implement` → działające testy (tu głównie integracyjne).

### Punkt startowy: jedno ryzyko, zero testów na ścieżce zapisu

**Ryzyko #2** — zapis sesji uszkadza talię: `pending`/`rejected` trafiają do talii z `accepted` albo częściowy zapis zostawia niespójny stan. Kontrakt: tylko zaakceptowane karty do talii. Ścieżka (`saveSession()` w `src/lib/save-session.ts`, endpoint zapisu, sąsiednie endpointy) miała **zero testów**; hermetyczne testy z wcześniejszej fazy nie dotykały prawdziwej bazy.

### Co odkrył research

Zanim padł prompt o test, ruszył `/10x-research`. **Research zmienił cel zadania** — jedno ryzyko ma dwie twarze:

- **Promocja `pending`/`rejected`** — dobrze zabezpieczona: filtr `.eq("state","accepted")`, CHECK w bazie. To inwariant do obrony, nie ukryty bug.
- **Częściowy commit przy błędzie** — realne i nietestowane: `saveSession()` wykonuje po kolei trzy operacje Supabase (`flashcards` upsert → `review_states` upsert → `flashcard_drafts` delete) bez transakcji. Przy błędzie log `orphan_review_state` lub `orphan_drafts` i mimo to `ok: true` — **brak rollbacku**.

Antywzorzec: pisać test na rollback, którego nie ma. Decyzja o transakcji (np. Postgres RPC) to otwarte ryzyko produktowe. Zadanie fazy: udowodnić reguły już zabezpieczone i przypiąć świadomie tolerowany kontrakt częściowych awarii.

Strategia testów (najtańszy sygnał na kawałek ryzyka):

- **Integracja na lokalnym Supabase** — reguły, których mock by skłamał: tylko `accepted` w `flashcards`, jeden `review_states` na kartę, po zapisie znikają wszystkie drafty sesji, idempotencja zapisu. Wymaga infrastruktury: klient service-role, świeże konto i sesja na test. Service-role omija RLS — tu akceptowalne, bo `saveSession` filtruje po `accountId`.
- **Hermetyczne stuby klienta** — awarie w środku sekwencji, których prawdziwej bazy nie da się łatwo wymusić. Stub zwraca błąd na wybranej operacji; asercja: „padł `review_states` → log `orphan_review_state` i `ok: true`".

Research wyprodukował mapę „najtańszy test na kawałek ryzyka" jeszcze przed pierwszym promptem o test.

### Od researchu do planu

`/10x-plan` rozbił pracę na cztery fazy według zależności:

| Faza | Co dostarcza | Dlaczego tu |
| ---- | ------------ | ----------- |
| 2.1 Środowisko integracyjne + reguła promocji | pomocniki lokalnego Supabase + test „tylko accepted promuje” | bez tego nie ma reszty testów integracyjnych |
| 2.2 Pozostałe reguły DB | parowanie `review_states`, czyszczenie draftów, idempotencja | korzysta ze środowiska z 2.1 |
| 2.3 Hermetyczne stuby + endpoint | gałęzie częściowych zapisów, statusy HTTP | bez bazy, w domyślnym `npm test` |
| 2.4 Cookbook + korekta bramki | wzorce w `test-plan.md`, bramka integracyjna ad hoc | transfer wiedzy do kolejnych faz |

Faza 2.1 ma od razu dawać dowód, nie samo rusztowanie. Po setupie środowiska testowego zaktualizuj `AGENTS.md`/`CLAUDE.md` — jak testujemy kod w tym projekcie.

### Gdzie to się zgadza z dobrymi zasadami testowania

- **Ryzyko, nie plik** — start od Ryzyka #2, nie „pokryj `save-session.ts`"; research rozdzielił zabezpieczone od niepokrytego.
- **Wyrocznia z researchu** — nieatomowość i `ok: true` przy częściowej awarii nie wynikają z kształtu funkcji; ocena: pełna promocja albo policzalny stan częściowy, nigdy cicha korupcja talii.
- **Uczciwy kontrakt** — test pilnuje tego, co kod robi dziś; RPC jako otwarte ryzyko produktowe.
- **Cost × signal** — integracja tylko gdzie mock kłamie (constraint, kaskady, realny SQL); service-role zamiast pełnego RLS (auth osobna faza); bramka integracyjna ad hoc, nie na każdy commit.
- **Asercja behawioralna** — seed mieszanych draftów (`accepted` + `pending` + `rejected`) → tylko accepted w `flashcards`, po zapisie znika każdy draft sesji.
- **Celowe psucie kodu** — odwróć gałąź `saveSession` lub zawęź delete; jeśli test zostaje zielony, asercja niczego nie chroni (uproszczona mutacja ręczna).

Żaden krok nie brzmiał „wygeneruj testy do pliku" — każdy: ten kawałek ryzyka, ta hipoteza, ten najtańszy test.

### Opcjonalny skill: `/10x-tdd`

```bash
npx @przeprogramowani/10x-cli@latest get m3l2
```

Opcjonalny skill — nie osobny workflow, nie obowiązkowa bramka. Czyta ten sam `context/changes/<change-id>/plan.md` i `## Progress` co `/10x-implement`; zmienia tylko kolejność: **RED → GREEN → REFACTOR** (najpierw test na czerwono, potem minimalny kod, refactor bez zmiany zachowania). Nie wymaga `test-plan.md` — możesz stosować przy wdrażaniu testów na bieżąco. Zatrzymuje model na asercji, zanim zacznie implementację.

Antywzorzec:

```text
Zaimplementuj promocję zaakceptowanych draftów do talii wg fazy XYZ, a potem pokryj to testami jednostkowymi.
```

Zamiast tego (przykład TDD):

```text
Najpierw wprowadź test, który udowadnia, że zapis promuje wyłącznie drafty w stanie `accepted`,
a `pending`/`rejected` nigdy nie trafiają do talii. Potem przejdź do implementacji założeń fazy XYZ.
```

Czerwony test na początku wymusza pytania o źródło prawdy: co ma się wydarzyć, jaki wynik widzimy z zewnątrz, co ma się zepsuć przy naruszeniu ochrony, czy asercja opisuje zachowanie czy implementację. Bez odpowiedzi — nie masz materiału na dobry test.

Reguła: jeśli umiesz nazwać pierwszy czerwony test jednym zdaniem → rozważ `/10x-tdd`; jeśli nie → `/10x-implement` lub wróć do researchu.

Dobre warunki startowe (obserwowalny wynik, bez wskazywania mocków ani prywatnych metod):

- promuje wyłącznie drafty `accepted`, `pending`/`rejected` nigdy nie trafiają do talii,
- zwraca `ok:true` i loguje `orphan_review_state`, gdy upsert `review_states` padnie w trakcie zapisu,
- zwraca 401, gdy użytkownik nie ma dostępu do kursu,
- resetuje interwał powtórki do 1 dnia, gdy ocena wynosi 0.

`/10x-tdd` szczególnie przy: walidatorach, parserach, transformacjach; logice biznesowej z jasnym wejściem/wyjściem; kontraktach API (status, body, auth, błędy); reducerach i maszynach stanów; bugfixach (najpierw odtwórz błąd); regression guardach z `test-plan.md`.

Nie forsuj przy: scaffoldingu katalogów, konfiguracji runnerów/CI/deploya, dokumentacji, kosmetyce UI bez automatycznej asercji, cienkim okablowaniu (test = mirror), spike'u bez kontraktu.

Warunek: środowisko testowe musi już istnieć — skill nie instaluje runnera ani CI od zera; bez testów najpierw bootstrap (`/10x-implement`).

Mieszanie w jednym planie:

```text
/10x-implement <change-id> phase 1   # środowisko testowe
/10x-tdd <change-id> phase 2         # kontrakt wrappera
/10x-tdd <change-id> phase 3         # kontrakt API
/10x-implement <change-id> phase 4   # cookbook i synchronizacja planu
```

Na czerwonym teście zatrzymaj się i oceń: czy łapie ryzyko, czy nie odwzorowuje implementacji, czy ma edge case'y, czy padnie po celowym zepsuciu kodu.

### A co, jeśli koniecznie chcesz testować wg plików

Wychodź od ryzyka; jeśli musisz od pliku — surowy prompt z bezpiecznikami:

```text
Napisz testy jednostkowe dla @PLIK bazując na @TECH_STACK oraz wymaganiach z @PRD.
```

Fragment w tym samym prompcie:

```text
Przejdź przez trzy etapy:

1) Ustal oczekiwane zachowanie ze źródeł, a nie z samej implementacji: przeczytaj TECH_STACK
oraz PRD i na tej podstawie zdecyduj, które zachowania tego pliku są istotne dla
użytkownika i dla biznesu. Wypisz je przed wdrożeniem testów. Nie zakładaj, że to, co
kod aktualnie robi, jest tym, co robić powinien - jeśli z PRD lub tech-stacku nie
wynika jednoznacznie poprawne zachowanie (problem wyroczni), ZATRZYMAJ się i
zadaj mi pytania, zamiast zgadywać i kopiować wynik z implementacji.

2) Dla każdego istotnego zachowania napisz test behawioralny: asercja ma
sprawdzać obserwowalny wynik scenariusza, a nie wewnętrzne wywołania, prywatne
szczegóły czy wynik policzony tą samą logiką co testowany kod. Dołóż przynajmniej
przypadki brzegowe wynikające z ryzyka (np. `null`, puste dane, błąd zależności,
nieprawidłowe wejście). Pilnuj optymalnej liczby testów - bez kilku niemal
identycznych kopii sprawdzających to samo; każdy test ma łapać inną regresję.

3) Napisz podsumowanie zmian - opisz w formie tabeli, jaką regresję łapie każdy test i jaką
zmianę w kodzie by wykrył. Jeśli dla jakiegoś fragmentu nie umiesz tego nazwać,
oznacz go jako niepewny i dopytaj mnie o dodatkowy kontekst lub wymagania.
```

Bez środowiska testowego → najpierw `/10x-research`.

### Deep dive: Dlaczego coverage nie wystarczy

> **Deep dive** — Coverage: czy linia się wykonała; mutation testing: czy test zauważyłby zmianę. Agent pokrywa kod słabymi asercjami — coverage rośnie, ochrony brak.

### Deep dive: Mutation testing jako quality gate

> **Deep dive** — Celowo psuj kod (mutant: zamiana operatora, odwrócenie warunku). **Killed** = test wykrył wadę; **survived** = test przeszedł mimo zmiany (ślepa asercja); **no coverage** = żaden test nie dotknął linii. `mutation score = (killed + timeout) / (killed + timeout + survived + no coverage) × 100%`. Mutanty wykluczone (`compile error`, `ignored`) nie wchodzą do równania. Coverage = martwe strefy; mutation score = słabe asercje — przy testach AI ważniejszy drugi sygnał.

### Deep dive: Stryker krok po kroku

> **Deep dive** — JS/TS: `npx stryker init`. Vitest: `testRunner: vitest`, `mutate` tylko pliki produkcyjne (nigdy `*.test.ts`), `thresholds.break: null` dopóki to selektywna bramka, nie twardy gate CI. Dopisz w `AGENTS.md`/`CLAUDE.md`, że Stryker jest selektywny — `--mutate "path/file.ts:start-end"`, bez gonienia 100%, survived mutant = decyzja czy to realny bug użytkownika. `npx stryker run` → raport HTML → szukaj **Survived**. Zawężanie: `--mutate "src/srs.ts:1-40"`, `--incremental`. Pętla: test ryzyka → coverage → Stryker na module → żywy mutant → popraw test → ponowny run.

### Deep dive: Stryker a 10x-Workflow

> **Deep dive** — Stryker nie zastępuje skilli; dodatkowa pętla QA. Prompt: „Uruchom Strykera na testach ukończonej fazy" + „Jak oceniasz jakość testów względem wyników Strykera?"

### Deep dive: Co Stryker znalazł w 10xCards

> **Deep dive** — Po Fazie 1 i 2: 50 zielonych testów, ale **~38% celowych mutacji przeszłoby niezauważone** — kod pokryty, nieobroniony. Przykładowe mutation score: `save.ts` endpoint 87,5%; `save-session.ts` 69,5%; `generate.ts` 66,2%; `openrouter.ts` 53,6%. Trzy obserwacje: (1) 6 niemal identycznych testów sentinel w `generate.test.ts` zabiło 0 mutantów — nadmiar formy, wystarczy jeden sparametryzowany test; (2) survived = słaba asercja — pytaj „jaka decyzja w logice powinna spaść, a nie spadła?"; (3) nie każdy survived = nowy test — mutacje promptu systemowego to mirror implementacji, świadomie zostawiasz.

### Deep dive: Jak Stryker zarządza mutantami

> **Deep dive** — Dry run mapuje test→linie; per mutant tylko testy dotykające linii. Koszt = liczba mutantów → `concurrency` i `--mutate`. Pułapka: plik bez testów → „No tests were executed".

### Deep dive: Mutation score to nie cel

> **Deep dive** — Gonienie 100% = mirror implementacji. Trzy kategorie żywych mutantów: realna luka → asercja; równoważny → ignoruj; realna zmiana niewarta przypięcia → świadomie ignoruj. Selektywna bramka: krytyczna domena, ryzyko z test-plan, często zmieniany moduł, przed ważnym mergem.
