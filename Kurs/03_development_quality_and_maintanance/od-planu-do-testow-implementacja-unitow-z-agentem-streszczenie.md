### Wprowadzenie

Antywzorzec: prosisz o testy, dostajesz zielony coverage — ale testowanie to obszar, w którym AI łatwo oszukać. Na benchmarkach TestEval pokrycie ~92% linii; na realnym kodzie spada do ~45% (Huang i in., ACM TOSEM 2026). Najtrudniejsze to nie składnia, lecz **problem wyroczni** — decyzja, jaki test realnie chroni projekt. Przykład `getNextInterval(prevInterval, grade)` zwraca `prevInterval * grade`; dla `grade=0` powinien reset do 1 dnia, nie zero — agent asercjonuje błąd z implementacji. Wnioski: **duży coverage ≠ ochrona**; **naiwne promptowanie za mało** — potrzeba analizy ryzyk, kontekstu i optymalnej liczby testów. Fundament: `test-plan.md` z m3-l1.

### Vibe Testing, czyli jak nie wprowadzać testów

Antywzorzec: `Write tests for the auth module.` — testy odzwierciedlają implementację, nie zachowanie użytkownika. Przykład lustra: `expect(result).toEqual(await authenticate(input))`. Trzy klasy problemów:

| Antywzorzec | Jak wygląda | Co powinno się wydarzyć |
| ----------- | ----------- | ----------------------- |
| Mirror implementacji | wewnętrzne wywołanie, prywatny szczegół, ta sama logika | obserwowalne zachowanie, wynik scenariusza biznesowego |
| Happy paths | łatwe dane poprawne | co najmniej jeden edge case z ryzyka |
| Brak przypadków brzegowych | brak null/pustych/błędów | przypadek realnie psujący UX |

Wymagania procesu: **konkretne wejście ze scenariuszem ryzyka** + **osobna weryfikacja asercji**.

### Jak Test Plan rozbraja te pułapki

`test-plan.md` + `/10x-new` → `/10x-research` → `/10x-plan` → `/10x-implement`: (1) ryzyko zamiast pliku — mapa mówi co nie może się zepsuć; (2) wyrocznia z researchu — oczekiwane zachowanie, nie implementacja; (3) **cost × signal** — najtańszy test z realnym sygnałem, nie pogoni za coverage; (4) handoff w `/10x-plan` — każdy etap: zachowanie, regresja, kontekst z `research.md`, edge case, unikany antywzorzec.

### Case study: od ryzyka do testów integracyjnych

10xCards — ryzyko #2: zapis sesji uszkadza talię (tylko `accepted` do talii). Zero testów na ścieżce zapisu. **Research zmienił cel:** promocja `pending/rejected` — dobrze zabezpieczona (filtr `.eq("state","accepted")`, CHECK); **częściowy commit** — realne ryzyko: `saveSession()` = 3 niezależne operacje Supabase bez transakcji; przy błędzie log `orphan_*` i `ok: true` bez rollbacku. Nie testuj rollbacku, którego nie ma — przypnij obecny kontrakt. Strategia: integracja na lokalnym Supabase (reguły DB mock by skłamał) + hermetyczne stuby klienta (awarie w sekwencji). Fazy planu: 2.1 środowisko + promocja → 2.2 reguły DB → 2.3 stuby + endpoint → 2.4 cookbook. Dobre praktyki: ryzyko nie plik, wyrocznia z researchu, uczciwy kontrakt, cost×signal, asercja behawioralna (seed mixed draftów → tylko accepted w `flashcards`), celowe psucie kodu weryfikuje asercję.

### Opcjonalny skill: `/10x-tdd`

```bash
npx @przeprogramowani/10x-cli@latest get m3l2
```

`/10x-tdd` — ten sam `plan.md` i `## Progress`, kolejność **RED → GREEN → REFACTOR**. Zatrzymuje model na asercji przed implementacją. Antywzorzec: „zaimplementuj, potem testy". Prompt TDD: „Najpierw test, że zapis promuje wyłącznie drafty `accepted`… potem implementacja fazy XYZ." Reguła: jeśli umiesz nazwać pierwszy czerwony test jednym zdaniem → `/10x-tdd`; jeśli nie → `/10x-implement` lub research. Dobre warunki: obserwowalny wynik (401, reset interwału przy grade 0). Unikaj: scaffolding, CI/config, UI kosmetyka, spike. Wymaga istniejącego środowiska testowego. Mieszanie: `/10x-implement` phase 1 (bootstrap) + `/10x-tdd` phase 2–3.

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

> **Deep dive** — Celowo psuj kod (mutant); killed = test wykrył, survived = ślepa asercja. `mutation score = (killed + timeout) / (killed + timeout + survived + no coverage) × 100%`. Coverage = martwe strefy; mutation score = słabe asercje — przy testach AI ważniejszy drugi sygnał.

### Deep dive: Stryker krok po kroku

> **Deep dive** — JS/TS: `npx stryker init`, Vitest config (`mutate` tylko produkcyjne pliki, `thresholds.break: null` na start). Dopisz w `AGENTS.md` instrukcję selektywnego Strykera. Uruchomienie: `npx stryker run`. Survived mutant = zadanie na lepszą asercję. Zawężanie: `--mutate "src/file.ts:1-40"`, `--incremental`. Pętla: test ryzyka → coverage → Stryker na module → żywy mutant → popraw test → ponowny run.

### Deep dive: Stryker a 10x-Workflow

> **Deep dive** — Stryker nie zastępuje skilli; dodatkowa pętla QA. Prompt: „Uruchom Strykera na testach ukończonej fazy" + „Jak oceniasz jakość testów względem wyników Strykera?"

### Deep dive: Co Stryker znalazł w 10xCards

> **Deep dive** — 50 zielonych testów, ~38% mutacji przeszłoby niezauważone. Obserwacje: (1) 6 niemal identycznych testów sentinel = 0 mutantów — nadmiar forma; (2) survived = słaba asercja (pytaj „jaka decyzja powinna spaść?"); (3) nie każdy survived = nowy test (prompt systemowy = mirror). Agent zostawia kod pokryty, nieobroniony.

### Deep dive: Jak Stryker zarządza mutantami

> **Deep dive** — Dry run mapuje test→linie; per mutant tylko testy dotykające linii. Koszt = liczba mutantów → `concurrency` i `--mutate`. Pułapka: plik bez testów → „No tests were executed".

### Deep dive: Mutation score to nie cel

> **Deep dive** — Gonienie 100% = mirror implementacji. Trzy kategorie żywych mutantów: realna luka → asercja; równoważny → ignoruj; realna zmiana niewarta przypięcia → świadomie ignoruj. Selektywna bramka: krytyczna domena, ryzyko z test-plan, często zmieniany moduł, przed ważnym mergem.
