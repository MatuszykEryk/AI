## Antywzorzec: refaktor bez kształtu

Antywzorzec: „Zrefaktoruj ten moduł" — imponujący diff bez **celu** (agent wybrał „zwykle pasujący" kształt), **historii** (świadome decyzje vs przypadkowe dziwactwa) i **drogi odwrotu** (40 plików naraz). Drugi biegun: modernizacja wszystkiego według katalogu wzorców — abstrakcje bez wartości.

Wniosek: agentowi zleca się **przedstawienie opcji**; refaktor zaczyna się od **twojej decyzji**. Potrzebne trzy perspektywy: docelowy kształt, historia, odwracalna ścieżka.

## Docelowe kształty modułu

### Spektrum archetypów

Katalog Fowlera (nie ranking „lepszy/gorszy"):

- **Transaction Script** — procedury na żądanie,
- **Table Module** — logika dla tabeli/widoku,
- **Domain Model** + **Service Layer** — dane i zachowanie razem, cienka orkiestracja.

Spektrum zamienia „brzydki kod" w tezę do obrony/obalenia. Mattermost: `SaveMultiple` 180 linii inline vs `app/post.go` 3957 linii / 60 metod → teza: _przerośnięty Transaction Script potrzebujący modelu domeny_. Mały moduł może zostać Transaction Scriptem.

### Historia jako test intencji

**Złożoność istotna** (natury problemu) vs **przypadkowa** (sposób zapisu) — z kodu często nie widać która. Sprawdź **ADR-y**; bez nich — archeologia gita (`git log -L`, blame, PR-y).

Przykład C2 (pozycyjne tablice kolumn): commit `27d536b212` (2020) — świadoma optymalizacja wsadowego INSERT pod import; zero błędów kolejności w 6 latach → **świadome ograniczenie**, nie przypadkowa złożoność.

### Guard, nie przebudowa

Reguła legacy: **guard, nie przebudowa** — świadomemu ograniczeniu dokładasz tanią deterministyczną osłonę (test zgodności tablic), nie zmieniasz kształtu. Right-sizing:

- mały moduł → zostaje Transaction Script,
- świadome ograniczenie → guard,
- przypadkowa złożoność rosnąca w koszt → przebudowa.

## Odwracalna droga do celu

**Strangler Fig** — nowy kod na brzegach, stary wygaszany; mniejsze ryzyko niż big-bang.

**Branch by Abstraction** — abstrakcja nad starym kodem, trunk wydawalny przez całą zmianę.

Mechanika dużej zmiany: spróbuj docelowej zmiany → zobacz co pada → **cofnij** → rozwiązuj od liści do korzenia (metoda Mikado — Deep Dive).

**Test charakteryzujący** (Feathers) — przybija _obecne_ zachowanie przed edycją chronionego kodu; utrwala stan zastany świadomie (ostrzeżenie wyroczni z M3L1). Reguła: **dodaj test, zanim dotkniesz.**

## Element ④ (Refactor opportunities): eksploracja i ranking opcji

Cykl: `/10x-new` → `/10x-research`. Zasada: **eksploracja kończy się raportem, nie decyzją.**

Intencja w `change.md`:

```text
/10x-new refactor-opportunities

Intencja: analiza context/changes/post-flow-analysis/research.md dokumentuje dług i ryzyka.
Ta zmiana odpowiada: KTÓRE problemy naprawić, w jakim kształcie i kolejności.
Etap: eksploracja → decyzja i plan → implementacja. Na eksploracji: żaden refaktor, żadna decyzja.
Wynik: research.md z rankingiem opcji. Decyzja na planowaniu.
```

Nie dopisuj do `post-flow-analysis` — osobna zmiana z własną intencją; wspólna pamięć to `context/`.

Kontrakt `/10x-research` (pełny prompt z lekcji):

```text
/10x-research refactor-opportunities Przeczytaj analizę: context/changes/post-flow-analysis/research.md - zapis długu technicznego i ryzyk strukturalnych tego repozytorium. Traktuj jej ustalenia jako zebrane dowody: nie wyprowadzaj ich na nowo, buduj na nich.

Wypisz każdy problem, który raport odnotowuje, niezależnie od etykiety (dług, ryzyko, hotspot, znalezisko).
Sklasyfikuj każdy: KANDYDAT to problem, którego naprawa zmieniłaby strukturę kodu; wszystko inne (np. brakujący test, luka w dokumentacji) nie jest kandydatem - zachowaj to jako wejście do oceny wykonalności i kosztu.

Następnie zbadaj każdego kandydata trzema sub-agentami; wszystkie pracują w trybie eksploracji, bez wprowadzania zmian:

1. Obecny kształt - potwierdź w kodzie, jaki kształt kandydat ma dziś. Cytuj plik:linia. Oznacz evidence / inference / unknown.
2. Historia i intencjonalność - ADR-y; w przeciwnym razie git log -L, blame, PR-y. Werdykt: świadome ograniczenie vs przypadkowa złożoność vs unknown.
3. Wykonalność migracji - inkrementalna, odwracalna ścieżka; blast radius z raportu; osłony i CI; pierwszy krok-prerekwizyt.

Twarde granice:
- Żadnych zmian w kodzie. Żadnego refaktoru.
- Nie projektuj docelowej architektury poza nazwaniem adekwatnego docelowego kształtu per kandydat. Przeprojektowanie pojęć biznesowych → inna, późniejsza analiza (M4L5).
- Gdzie brakuje danych, napisz unknown.

Synteza: research.md z sekcją "Refactor opportunities" — 2-3 najmocniejsi w rankingu + odrzuceni z powodami. NIE proś o wybór ani zgodę — zakończ gotowym raportem.
```

Trzy sub-agenty = trzy perspektywy lekcji: kształt, historia, wykonalność/odwracalność.

Wynik Mattermost: 7 problemów P1–P7; C2 (tablice) odrzucone jako refaktor → guard; C4 (ręczne warstwy store) na szczycie — `Save` nadpisane, `SaveMultiple` nie, zapisy bulk omijają indeksowanie. Ranking odwrócił faworyta z raportu ③ bez historii.

## Przeczytaj, zanim zdecydujesz

Ranking ≠ decyzja. Przeczytaj `research.md`: audyt kandydatów, werdykty intencjonalności z dowodem (commit/ADR/PR), zgoda z kolejnością.

Weryfikacja ast-grep przed planowaniem (jak M4L3):

```text
Zweryfikuj raport context/changes/refactor-opportunities/research.md.
Wypisz twierdzenia STRUKTURALNE za rankingiem.
Dla każdego: ast-grep → potwierdzone/doprecyzowane/obalone; zero potwierdź grepem.
Zaktualizuj raport; dodaj "## Weryfikacja twierdzeń (ast-grep)".
Sekcji "Refactor opportunities (ranked)" i werdyktów intencjonalności NIE zmieniaj.
Podważenie pozycji → tylko w sekcji weryfikacji, "do decyzji na etapie planowania".
```

Masz prawo przesunąć C1 wyżej mimo rankingu — to rozmowa planowania z odpowiedzialnością.

## Bramka decyzyjna w /10x-plan

```text
/10x-plan refactor-opportunities
```

Planer czyta `research.md` automatycznie. Pierwsze pytanie: **którą opcję realizujemy?** — jawna decyzja, nie cicha akceptacja rankingu. Przykład wyboru: C4 + test-guard z C2. Planer może domknąć otwarte pytanie z rankingu (np. żywy bug `SaveMultiple` — dowód: import bez auto-reindex).

Ewolucja projektu testu tablic: test okrężny bezwartościowy → porównanie typów kruche → **wartości-wartowniki** per kolumna.

`plan.md` guard-first, cztery fazy:

- **Dodaj test, zanim dotkniesz** — charakteryzacja przed edycją niepokrytego kodu,
- **Mechanizm na zielono, egzekwowanie osobno** — walidacja kompletności domyślnie wyłączona, włączana per warstwa po przeglądzie wyjątków,
- jawne **„czego NIE robimy"** — wąski zakres, kontrola blast radius,
- fazy = osobne odwracalne commity.

Strangler/Branch czekają na kandydatów strukturalnych (C3); tu guard, nie migracja.

## Realizacja planu: wpinamy się w znany cykl

```text
/10x-plan-review
/10x-implement refactor-opportunities phase 1
```

Lekcja kończy się na planie. ast-grep w tej lekcji = weryfikator rankingu; rewrite przy mechanicznych fazach (jak M4L3). Bezpieczeństwo = zaprojektowana własność planu (kryteria weryfikacji auto/ręczne per faza).

Element ④ dołącza do raportu (① mapa, ② overview, ③ debt, ④ ranking+decyzja). Luka domenowa (zduplikowana walidacja zaplanowanych postów) → M4L5 DDD.

### Deep dive: Od jednego kroku do kampanii

> **Deep dive** — `/10x-test-plan` gdy backlog ma kilka kandydatur, tygodnie pracy — orkiestrator faz rolloutu; rozpoznaje istniejący plan jako fazę 1 zamiast duplikować. Jeden wąski wycinek jak w lekcji — wystarczy sam `plan.md`.

### Deep dive: Skąd pochodzą techniki?

> **Deep dive** — Archetypy: Fowler POEA (2002). Złożoność istotna/przypadkowa: Brooks „No Silver Bullet" (1986). Strangler Fig: Fowler 2004. Branch by Abstraction: Hammant 2007. Mikado: Ellnestam/Brolund 2014. Testy charakteryzujące i szwy: Feathers (2004). ADR: Nygard 2011.

### Deep dive: Szersza perspektywa na refaktoryzację

> **Deep dive** — Kwadrant długu technicznego (Fowler/Cunningham): świadomy/nieświadomy × rozważny/lekkomyślny — spłacaj najwyższe „odsetki". Joel Spolsky „Things You Should Never Do" (2000): rewrite od zera traci wiedzę z poprawek; AI nie obniża tego kosztu.
