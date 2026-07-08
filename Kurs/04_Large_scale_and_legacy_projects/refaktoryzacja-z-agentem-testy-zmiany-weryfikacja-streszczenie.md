Po M4L3 masz raport Deep Focus: ② Feature overview i ③ Technical debt. Hotspoty z ③ to **problemy, nie zadania** — refaktor zaczyna się, gdy nadasz problemowi docelowy kształt, ugruntujesz go w historii i zaplanujesz odwracalną ścieżkę.

W tej lekcji dokładasz element **④ Refactor opportunities**: ranking opcji ugruntowany w kodzie i historii, potem decyzję i plan chroniony testami.

## Antywzorzec: refaktor bez kształtu

Antywzorzec: „Zrefaktoruj ten moduł" — imponujący diff bez **celu** (agent wybrał „zwykle pasujący" kształt), **historii** (świadome decyzje vs przypadkowe dziwactwa) i **drogi odwrotu** (40 plików naraz). Drugi biegun: modernizacja wszystkiego według katalogu wzorców — abstrakcje bez wartości.

Wniosek: agentowi zleca się **przedstawienie opcji**; refaktor zaczyna się od **twojej decyzji**. Potrzebne trzy perspektywy: docelowy kształt, historia, odwracalna ścieżka.

## Docelowe kształty modułu

### Spektrum archetypów

Katalog Fowlera (nie ranking „lepszy/gorszy"):

- **Transaction Script** — procedury na żądanie od początku do końca,
- **Table Module** — logika dla tabeli/widoku,
- **Domain Model** + **Service Layer** — dane i zachowanie razem, cienka orkiestracja.

Spektrum zamienia „brzydki kod" w tezę do obrony/obalenia. Mattermost: `SaveMultiple` 180 linii inline vs `app/post.go` 3957 linii / 60 metod → teza: _przerośnięty Transaction Script potrzebujący modelu domeny_. Mały moduł może zostać Transaction Scriptem.

### Historia jako test intencji

**Złożoność istotna** (natury problemu) vs **przypadkowa** (sposób zapisu) — z kodu często nie widać która. Sprawdź **ADR-y**; bez nich — archeologia gita (`git log -L`, blame, uzasadnienia w commitach i PR-ach).

Przykład C2 (pozycyjne tablice kolumn): commit `27d536b212` (2020) — świadoma optymalizacja wsadowego INSERT pod import; zero błędów kolejności w 6 latach → **świadome ograniczenie**, nie przypadkowa złożoność. Werdykt się odwraca.

### Guard, nie przebudowa

Reguła legacy: **guard, nie przebudowa** — świadomemu ograniczeniu dokładasz tanią deterministyczną osłonę (test zgodności tablic), nie zmieniasz kształtu wydajnościowej decyzji. Right-sizing:

- mały moduł → zostaje Transaction Script,
- świadome ograniczenie → guard,
- przypadkowa złożoność rosnąca w koszt → przebudowa.

Pominiesz filtr — wzorce wszędzie, wartość nigdzie.

## Odwracalna droga do celu

**Strangler Fig** — nowy kod na brzegach, stary wygaszany; mniejsze ryzyko niż big-bang rewrite.

**Branch by Abstraction** — abstrakcja nad starym kodem, trunk wydawalny przez całą zmianę.

Mechanika dużej zmiany: spróbuj docelowej zmiany → zobacz co pada → **cofnij** → rozwiązuj od liści do korzenia (metoda Mikado — Deep Dive).

**Test charakteryzujący** (Feathers) — przybija _obecne_ zachowanie przed edycją chronionego kodu; utrwala stan zastany świadomie (ostrzeżenie wyroczni z M3L1). Reguła: **dodaj test, zanim dotkniesz.**

## Element ④ (Refactor opportunities): eksploracja i ranking opcji

Cykl: `/10x-new` → `/10x-research`. Zasada spinająca lekcję: **eksploracja kończy się raportem, nie decyzją.**

Intencja w `change.md`:

```text
/10x-new refactor-opportunities

Intencja: mamy analizę modułu, która dokumentuje dług techniczny
i ryzyka strukturalne: context/changes/post-flow-analysis/research.md. Ta zmiana odpowiada na pytanie, które tamta analiza celowo zostawiła otwarte:
KTÓRE z tych problemów warto naprawić, w jakim docelowym kształcie i w jakiej kolejności. Eksplorujemy każdy zapisany problem w kodzie i historii, a potem porządkujemy jako refactor opportunities.
Zmiana przebiega etapami: eksploracja → decyzja i plan → implementacja. Na etapie eksploracji nie dzieje się żaden refaktor i nie zapada żadna decyzja.
Wynik eksploracji: research.md tej zmiany, zakończony rankingiem opcji z trade-offami. Najpierw przeczytam raport; decyzja, co realizujemy, zapada na etapie planowania, a refaktor rusza dopiero według przyjętego planu.
```

Nie dopisuj do `post-flow-analysis` — osobna zmiana z własną intencją; wspólna pamięć to `context/`, każda zmiana to wpis z własną intencją.

Kontrakt eksploracji:

```text
/10x-research refactor-opportunities Przeczytaj analizę: context/changes/post-flow-analysis/research.md - zapis długu technicznego i ryzyk strukturalnych tego repozytorium. Traktuj jej ustalenia jako zebrane dowody: nie wyprowadzaj ich na nowo, buduj na nich. Jeśli odwołuje się do innych artefaktów (mapa repo, wcześniejszy research), przeczytaj je również jako priory.

Wypisz każdy problem, który raport odnotowuje, niezależnie od etykiety (dług, ryzyko, hotspot, znalezisko).
Sklasyfikuj każdy: KANDYDAT to problem, którego naprawa zmieniłaby strukturę kodu; wszystko inne (np. brakujący test, luka w dokumentacji) nie jest kandydatem - zachowaj to jako wejście do oceny wykonalności i kosztu. Wypisz listę i klasyfikację kandydatów na początku wyniku, żebym mógł ją zaudytować.

Następnie zbadaj każdego kandydata trzema sub-agentami; wszystkie pracują w trybie eksploracji, bez wprowadzania zmian:

1. Obecny kształt - potwierdź w kodzie, jaki kształt kandydat ma dziś: gdzie żyje logika, jak mieszają się odpowiedzialności, jakie abstrakcje lub powiązania już istnieją. Cytuj plik:linia. Oznacz każde twierdzenie jako evidence / inference / unknown.

2. Historia i intencjonalność - ustal, DLACZEGO kod ma taki kształt:
ADR-y i dokumenty projektowe, jeśli istnieją; w przeciwnym razie archeologia gita (git log -L, blame, uzasadnienia w commitach i PR-ach). Werdykt per kandydat: świadome ograniczenie (decyzja nośna) vs przypadkowa złożoność - albo uczciwie oznacz jako unknown, jeżeli ciężko to określić.

3. Wykonalność migracji - czego wymagałaby inkrementalna, odwracalna ścieżka (istniejąca abstrakcja vs nowa abstrakcja), co wynika z danych o blast radius z raportu, jakie osłony i testy już istnieją wokół (sprawdź konfigurację CI) i jaki byłby pierwszy krok-prerekwizyt.

Twarde granice:
- Żadnych zmian w kodzie. Żadnego refaktoru. Dowody przed interpretacją.
- Nie projektuj docelowej architektury poza nazwaniem adekwatnego docelowego kształtu per kandydat. Jeśli prawdziwa naprawa kandydata to przeprojektowanie pojęć biznesowych, a nie struktury kodu - powiedz to i zatrzymaj się: to przedmiot do innej, późniejszej analizy.
- Gdzie brakuje danych, napisz unknown - nie wypełniaj luk wiarygodnymi domysłami.

Synteza (po raportach wszystkich trzech subagentów): zapisz research.md w folderze tej zmiany.
Per kandydat: obecny kształt (z dowodami), werdykt intencjonalności, notatki o wykonalności. Zamknij sekcją "Refactor opportunities" z 2-3 najmocniejszymi kandydatami w rankingu - dla każdego: obecny → docelowy kształt, czemu zasługuje na to miejsce (koszt długu vs koszt zmiany), blast radius, szkic inkrementalnej ścieżki, pierwszy krok-prerekwizyt. Wypisz też kandydatów rozważonych i odrzuconych, z krótkim podsumowaniem dlaczego. Oceniaj na podstawie dowodów. NIE proś mnie o wybór, potwierdzenie ani zgodę - zakończ zapisaniem gotowego raportu. Ranking to propozycja dla osobnej sesji planowania, która odbędzie się po mojej lekturze.
```

Trzy sub-agenty = trzy perspektywy lekcji: kształt, historia z intencjonalnością, wykonalność z odwracalnością. Granica pojęć biznesowych pilnuje terenu M4L5 (DDD).

Wynik Mattermost (case study):
- 7 problemów P1–P7; research znalazł więcej niż etykiety w ③.
- **C2** (tablice) odrzucone jako refaktor — świadoma decyzja 2020, zero błędów → guard, nie przebudowa. „Zoptymalizuj przepływ" z issue okazało się mylące — przepływ już zoptymalizowany.
- **C4** (ręczne warstwy store) na szczycie — `PostStore` 54 metody; searchlayer nadpisuje `Save`, nie `SaveMultiple`; zapisy bulk omijają indeksowanie. Ranking odwrócił faworyta z ③ bez perspektywy historii.
- Raport ③ dostał korekty (np. tablice: dwie, nie trzy; dryf kontraktu Post trójstronny). Deep Focus to prior do ulepszania, nie prawda objawiona.

## Przeczytaj, zanim zdecydujesz

Ranking ≠ decyzja. Kontrakt zabrania agentowi kończyć „czy zatwierdzasz?" — pytanie pod preszą ≠ decyzja, którą obronisz (prework [1.3]: wygeneruj, potem **zrozum**).

Przeczytaj `research.md` na spokojnie:
- audyt listy kandydatów vs problemy z ③,
- werdykty intencjonalności z dowodem (commit/ADR/PR), nie przeczuciem,
- ranking — co byś przesunął i dlaczego.

Weryfikacja ast-grep przed planowaniem (jak M4L3) — twierdzenia strukturalne za rankingiem:

```text
Zweryfikuj raport context/changes/refactor-opportunities/research.md.

Wypisz z niego twierdzenia STRUKTURALNE, na których stoi ranking (liczby metod, "nadpisuje X, ale nie Y", liczność call-site'ów, pary lustrzanych typów).

Dla każdego zbuduj wzorzec ast-grep, wywołaj go i podaj wynik jako: twierdzenie -> potwierdzone / doprecyzowane / obalone, z plikami i liniami. Każde zero z ast-grep potwierdź klasycznym grepem.

Po weryfikacji zaktualizuj analizowany raport:
- błędne liczby i numery linii popraw w miejscu, w formacie "150 (raport: 145)" — tak, żeby ślad korekty został w tekście;
- dodaj sekcję "## Weryfikacja twierdzeń (ast-grep)" z tabelą: twierdzenie → werdykt → dowód (plik:linia) → metoda (wzorzec/reguła);
- zaktualizuj frontmatter: last_updated, dopisz tag "verified" i commit weryfikacji;
- sekcji "Refactor opportunities (ranked)" oraz werdyktów intencjonalności NIE zmieniaj.

Jeśli wynik podważa pozycję kandydata, opisz to wyłącznie w sekcji weryfikacji, z adnotacją "do decyzji na etapie planowania".
```

Weryfikacja może nie wywrócić rankingu, ale doszlifować liczby i wzmocnić dowody (np. potwierdzenie „Save bez SaveMultiple" + wszystkie `SaveMultiple` to bulk import). Masz prawo przesunąć C1 wyżej mimo rankingu — to rozmowa planowania z odpowiedzialnością.

## Bramka decyzyjna w /10x-plan

```text
/10x-plan refactor-opportunities
```

Planer czyta `research.md` automatycznie — budżet pytań idzie w decyzje, nie diagnostykę. Pierwsze pytanie: **którą opcję realizujemy?** — jawna decyzja, nie cicha akceptacja rankingu.

Przykład wyboru: C4 + test-guard z C2 (tani zysk niezależnie od rankingu). Planer może domknąć otwarte pytanie z rankingu (np. żywy bug `SaveMultiple` — dowód: import bez auto-reindex; zaimportowane posty nieprzeszukiwalne do ręcznego reindex).

Ewolucja projektu testu tablic (wątek przez wszystkie etapy):
1. Test okrężny (zapisz→odczytaj) **bezwartościowy** — odczyt po nazwach wybacza, zapis pozycyjny nie.
2. Porównanie typów per indeks **kruche** — JSON/wskaźniki; zamiana sąsiednich kolumn tego samego typu przechodzi.
3. Finalnie: **wartości-wartowniki** — charakterystyczna wartość per pole, asercja tożsamości per nazwa kolumny.

`plan.md` guard-first, cztery fazy:

- **Dodaj test, zanim dotkniesz** — charakteryzacja `Save→indexPost` przed edycją niepokrytego kodu; test najpierw, edycja potem.
- **Mechanizm na zielono, egzekwowanie osobno** — walidacja kompletności domyślnie wyłączona; włączana per warstwa po przeglądzie ~93 nienadpisanych metod z obowiązkowym uzasadnieniem wyjątku.
- jawne **„czego NIE robimy"** — bez C1, C3, migracji na nazwy; wąski zakres = kontrola blast radius.
- fazy = osobne odwracalne commity (od liści do korzenia w wersji operacyjnej).

Strangler/Branch czekają na kandydatów strukturalnych (C3); tu guard, nie migracja — ale research oceniał wykonalność przez te perspektywy.

## Realizacja planu: wpinamy się w znany cykl

```text
/10x-plan-review
/10x-implement refactor-opportunities phase 1
```

Lekcja kończy się na planie. ast-grep w tej lekcji = weryfikator rankingu; rewrite przy mechanicznych fazach (jak M4L3). Bezpieczeństwo = zaprojektowana własność planu (kryteria weryfikacji auto/ręczne per faza, odwracalność per commit).

Element ④ dołącza do raportu (① mapa, ② overview, ③ debt, ④ ranking+decyzja). Luka domenowa (zduplikowana walidacja zaplanowanych postów — pojęcia biznesowe, nie struktura) → M4L5 DDD, element ⑤.

### Ćwiczenia (m4l4)

Paczka: `npx @przeprogramowani/10x-cli@latest get m4l4`. Wejście: `context/changes/{change-id}/research.md` z M4L3 (②+③).

**Krok 0** — `/10x-new refactor-opportunities` z intencją (jak wyżej; podmień `{change-id}`).

**Krok 1** — kontrakt research bez skracania granic. Po zakończeniu nie odpowiadaj agentowi — eksploracja kończy się raportem.

**Krok 2** — poza sesją: audyt kandydatów, werdykty z dowodem, ranking; weryfikacja strukturalna promptem `m4l4-3` z paczki (jak sekcja ast-grep wyżej). `unknown` to też wynik — decyzja w planie, nie w połowie implementacji.

**Krok 3** — `/10x-plan refactor-opportunities`: pierwsze pytanie = wybór opcji; przetestuj ⭐ kontrpytaniem; wąski wycinek + jawne „czego NIE robimy".

**Krok 4** — sprawdź plan: charakteryzacja przed dotknięciem; fazy odwracalne od najtańszej; kryteria auto/ręczne; mechanizm na zielono, egzekwowanie osobno. Potem `/10x-plan-review` w nowej sesji.

Wynik: sekcja „Refactor opportunities" w `research.md`, `plan.md`, `/10x-implement refactor-opportunities phase 1` w schowku.

### Deep dive: Od jednego kroku do kampanii

> **Deep dive** — `/10x-test-plan` gdy backlog ma kilka kandydatur i tygodnie pracy — orkiestrator faz rolloutu; rozpoznaje istniejący plan jako fazę 1 zamiast duplikować. Runner-upy (C1, pokrycie błędów, guardy na kolejne sub-store'y) → kolejne fazy z własnym `change-id`. Jeden wąski wycinek jak w lekcji — wystarczy sam `plan.md`.

### Deep dive: Skąd pochodzą techniki?

> **Deep dive** — Archetypy: Fowler POEA (2002). Złożoność istotna/przypadkowa: Brooks „No Silver Bullet" (1986). Strangler Fig: Fowler 2004. Branch by Abstraction: Hammant 2007. Mikado: Ellnestam/Brolund 2014. Testy charakteryzujące i szwy: Feathers (2004). ADR: Nygard 2011.

### Deep dive: Szersza perspektywa na refaktoryzację

> **Deep dive** — Kwadrant długu technicznego (Fowler/Cunningham): świadomy/nieświadomy × rozważny/lekkomyślny — spłacaj najwyższe „odsetki". Joel Spolsky „Things You Should Never Do" (2000): rewrite od zera traci wiedzę z poprawek; tańsze generowanie kodu przez AI nie obniża tego kosztu.
