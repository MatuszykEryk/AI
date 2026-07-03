### Dlaczego nie zaczynamy od całego repo

Wchodzisz w legacy, którego nie znasz — stary moduł, repo po innym zespole, dokumentacja z czasów „tymczasowego" deploya.

Antywzorzec: „hej Agent, przeczytaj całe repo i wyjaśnij architekturę". Duże okno (~200K tokenów) nie to 200K na kod — w budżecie: instrukcje systemowe, reguły, narzędzia, historia, odpowiedź. Tokenizacja nierówna (1 token ≈ 4 znaki to intuicja); po testach, configu, generowanych plikach budżet znika szybciej.

Drugi problem: nawet gdy się zmieści — **dużo kontekstu ≠ zrozumienie**. Płaskie streszczenie: trochę folderów, trafne nazwy, domysły, fałszywe poczucie kontroli.

Zasada M4L1: agent lepiej z właściwym kontekstem we właściwym momencie. Prework 3.1: model nie ma magicznej pamięci projektu. Prework 3.3: strategia **`Select`** — wybierasz kontekst do następnego kroku, jak programista przy bugu.

Prośba o całe repo miesza trzy zadania: przeszukiwanie struktury, wybór obszarów architektonicznych, głębokie zrozumienie zachowania. Agent nazwie katalogi, ale nie wie, które zależności krytyczne, które entry pointy używane, gdzie cykle, gdzie zużyć uwagę człowieka.

Zamiast pełnego zrzutu: **szeroki skan klasycznymi CLI**, potem kosztowne czytanie. CLI nie „rozumieją architektury" — **redukują szum**. Zamiast setek plików: entry pointy, graf, cykle, symbole, sygnały z historii. Agent interpretuje mniejsze, lepsze dane. Różnica: „czytaj wszystko" vs „zinterpretuj dowody".

Narzędzia agent może wykonać w locie — nam chodzi o **celowe, powtarzalne** tool calle, nie losowe peeki.

### Wide Scan → Deep Focus

Proces pozyskiwania kontekstu z dużego repo — odpowiedź na limity okna i nieprzewidywalność LLM bez ram.

**Wide Scan** — szeroka, płytka analiza; wystarczy tańszy model. Sygnały: historia modułów, entry pointy, kierunek zależności, cykle, granice, aktywne przepływy zmian, miejsca ostrożności. Nie pełne zrozumienie — zawęża dalsze poszukiwania.

**Deep Focus** — głęboka analiza jednego obszaru mocniejszym agentem (M4L3): jeden feature/moduł, przepływ danych, ryzyka, modernizacja.

Ta lekcja = Wide Scan → **Mapa projektu**.

> **Mapa projektu** — operacyjna mapa terytorium legacy. Tanie sygnały z CLI + synteza agenta → krótki artefakt: moduły, entry pointy, zależności, cykle, wrażliwe obszary, niewiadome, miejsca wymagające dowodów przed zmianą. Nie spalać okna na losowe pliki; nie zmieniać legacy na ślepo. Format przykładowy — dostosuj do potrzeb.

Zapis: `context/map/repo-map.md` — format przyjazny agentowi: krótkie sekcje, listy, ślady z komend, jawne `unknowns`. Nie esej, nie zrzut czatu.

Artefakty robocze w `context/map/`:

```text
context/map/artifact-1-territory.md
context/map/artifact-2-structure.md
context/map/artifact-3-contributors.md
```

Finalna mapa = synteza w `repo-map.md`.

Dwa błędy:

1. **Losowy deep-read** — agent otwiera „ważnie wyglądający" plik, importy, sąsiednie moduły; po kilkunastu callach stary adapter nikt nie używa. Dużo ruchu, mało decyzji.
2. **Mapa bez decyzji** — graf bez wskazania istotnych obszarów, ryzyk, miejsc do dalszej weryfikacji = obrazek, nie narzędzie.

Dobra mapa kończy się decyzją: te obszary istotne; tu zmiana przejdzie przez warstwy; tu najpierw dowody, testy, historia PR, unknowns.

### Czego nie mówi drzewko katalogów

Eksplorator plików daje poczucie kontroli (`src/`, `services/`, `domain/`…) — dobry start, **zły model rzeczywistości**.

Foldery = statyczny snapshot „gdzie leży dziś". Nie pokazują:

- które miejsca zmieniają się co tydzień,
- pozostałości po starym podziale zespołu,
- moduły pękające razem przy jednej zmianie,
- gdzie naprawiają błędy,
- ścieżki martwe vs krytyczne dla biznesu,
- kto ma ukrytą wiedzę.

Search: `payments` znajdzie słowo, nie powie czy sterowane webhookami, jobami, flagami, migracjami. `auth` nie odróżni rdzenia od test helpera i komentarza w README.

Eksplorator: „co tu jest?". Mapa: „co ważne, powiązane, aktywne, ryzykowne?". Na tym etapie nie oceniasz jakości każdego modułu — przestajesz traktować drzewo katalogów jak prawdę; to **pierwsza hipoteza**.

### Efektywna Mapa projektu

Odpowiada na:

- **kluczowe moduły i rola** — większe części systemu,
- **częstotliwość zmian** — git: często dotykane vs stabilne,
- **obszary wrażliwe** — auth, płatności, dane, cache, integracje, migracje, runtime config,
- **coupling i blast radius** — importy, cykle, helpery, kontrakty, współzmiany,
- **kontrybutorzy** — kto pracował przy obszarze, ukryta wiedza zespołowa.

Nie musi być kompletna — wystarczająco dobra: główny układ, miejsca ostrożności, gdzie zmiana zaskoczy.

### Język naturalny i przewidywalne CLI

Agent łączy język naturalny z przewidywalnym CLI — nie „magicznie rozumie repo".

Pytania po ludzku: „pokaż entry pointy", „kto importuje auth", „frontend sięga do bazy?", „najczęściej zmieniane pliki w roku". Agent → seria komend: `rg`, `find`, `git log`, analiza importów, graf, AST.

CLI do momentu zwrotu wyniku działa **poza** oknem kontekstowym — przechodzi tysiące plików/historię/graf; do rozmowy trafia skondensowany wynik.

Potem rozmawiasz z wynikami: „czy to rdzeń?", „które wyniki to szum?", „zawęź do płatności", „porównaj z historią". Nie musisz znać idealnej komendy od razu.

Pętla Wide Scan: **pytanie → wybór narzędzia → komenda → wynik → rozmowa → kolejne pytanie**. Mocniejsze niż eksplorator; bezpieczniejsze niż całe repo.

### Trzy składowe Mapy projektu

Trzy pytania, trzy źródła sygnałów → jedna mapa.

**Terytorium** — gdzie naprawdę odbywa się praca; aktywne vs zamrożone; co zmienia się razem; częsta praca → ryzyko regresji.

**Struktura** — entry pointy, warstwy, kierunek importów, powiązania, kontrakty — niewidoczne z drzewa katalogów.

**Kontekst kontrybutorów** — kto przy edge case'ach, protokołach, migracjach, awaryjnych poprawkach.

Nie czekaj z interpretacją do końca — po każdej składowej zapisuj, co już wynika dla pracy w legacy.

Przykład lekcji: **Mattermost** (Go backend, React/TS frontend, miliony linii) — ćwiczenia na dowolnym dużym OSS (Mattermost, tldraw, React).

Na wczesnym etapie wystarczą **krótkie prompty ad hoc** — po kilku sesjach powtarzalne procedury wynieś do skilla.

### Mapa terytorialna — gdzie projekt żyje

Zanim otworzysz plik: które obszary aktywne, które zamrożone. **Historia Gita** odpowiada szybciej niż GUI klienta — ale GUI pokazuje commity/branche, nie przekrojową narrację „co i za sprawą kogo". Agent + git rozwiązuje to tanio.

**Prompt 1 — TOP 10 aktywności (12 mies.):**

```text
Korzystając z historii gita, w zakresie ostatnich 12 miesięcy, pokaż TOP 10 najczęściej modyfikowanych:

a) folderów lub modułów
b) plików

Odfiltruj szum: lockfile'y, snapshoty, generowane pliki, dotenvy, configi, etc.

Możesz zejść poziom niżej jeśli pierwsza seria wyników da ogólne rezultaty jak "src/frontend" i "src/backend" - chcemy poznać realne obszary aktywności hands-on.
```

Jeden ranking nie mówi: stałe centrum vs sezonowa kampania vs „gorący bo ciągle się psuje".

**Prompt 2 — trendy kwartalne:**

```text
Podziel te same dane na kwartały — chcę zobaczyć, jak zmieniał się nacisk pracy w projekcie przez ostatni rok.
```

**Prompt 3 — współzmiany folderów:**

```text
Jakie pary lub trójki katalogów najczęściej pojawiają się w tych samych commitach? Wyszukaj sprzężenia i krótko podsumuj wnioski dla top 3 z naszego rankingu.
```

**Prompt 4 — hub i świeżość:**

```text
Jeszcze dwie rzeczy przy okazji tych współzmian:

- Czy jest jakiś pojedynczy plik, który zmienia się razem z wieloma różnymi obszarami naraz? Myślę o czymś wspólnym dla całego repo — plik z tłumaczeniami, config, coś generowanego.
- I sprawdź, czy pliki, które wyszły jako mocno sprzężone, na pewno nadal są w repo.
```

Współzmiany = inny sygnał niż ranking — ukryte sąsiedztwa, cross-layer. Ograniczenia: historia w oknie czasowym; nie pokażą kontraktu, który _powinien_ być zsynchronizowany, a nie jest (→ `unknown`). Drugie pytanie łapie pliki już usunięte.

**Zapis sesji:**

```text
Zapisz podsumowanie tej sesji do `context/map/artifact-1-territory.md`
```

Root wyszukiwania może być wskazany folder, nie całe repo. Wide Scan Mattermost: ~45–50k tokenów zajęte — dużo miejsca na dalszą rozmowę.

### Mapa strukturalna — jak to jest zbudowane

Historia mówi „gdzie patrzeć". Następne: „jak zbudowane?"

**dependency-cruiser** (`depcruise`) — statyczny graf importów; outputy: DOT (Graphviz→SVG/PNG), Mermaid, JSON, Markdown, HTML, CSV. Lekcja: stack JS/TS; wzorzec uniwersalny dla innych języków.

Cztery sygnały (też dla pojedynczego modułu):

- **Terytorium modułu** — `--collapse` do folderów: węzły, kierunek, huby,
- **Fokus** — `--focus` / `--include-only`: wybrany obszar + sąsiedzi,
- **Metryki sprzężenia** — `--metrics`: `Ca` (afferent — ilu zależy), `Ce` (efferent — od ilu zależy), `instability = Ce/(Ca+Ce)`,
- **Cykle** — reguła `no-circular`: blast radius, kandydat Deep Focus.

Instalacja: `npm install --save-dev dependency-cruiser` (yarn/pnpm analogicznie). Agent może skonfigurować po `https://raw.githubusercontent.com/sverweij/dependency-cruiser/.../doc/cli.md`. Wymaga toolchainu (np. `npm install`, TypeScript).

**madge** — szybsze cykle i prosty graf w Node. **skott** — lepsze path aliasy TS.

Na tym etapie **Markdown z tabelami**, nie od razu Graphviz — krótka odpowiedź: splątanie, granice warstw, ryzyka testów. JSON do porównań między uruchomieniami; SVG po selekcji decyzyjnej.

Wejście: często `artifact-1-territory.md` w nowej sesji — aktywny plik to centrum, cienkie wejście, kontrakt między warstwami, korytarz zmian?

**Prompt 0 — odkrycie narzędzia:**

```text
Daj mi top 3 pomysły na eksplorację kodu legacy z biblioteką dependency-cruiser.

Chcę zrozumieć istotne i najbardziej wrażliwe na zmiany obszary, a także potencjalny dług technologiczny.

Jakiego rodzaju raporty mogę generować?
```

**Prompt 1 — cykle w aktywnych obszarach:**

```text
Użyj dependency-cruiser dla `webapp` i sprawdź cykle zależności w najaktywniejszych obszarach z `context/map/artifact-1-territory.md`: `channels/src/components/admin_console`, `channels/src/packages`, `channels/src/utils`, `channels/src/actions`, `platform/client/src`, `platform/types/src`.

Nie interesuje mnie pełna lista wszystkiego w repo. Chcę zobaczyć tylko te cykle, które dotykają obszarów aktywnych według mapy terytorium. Dla każdego cyklu napisz prostym językiem, dlaczego może utrudnić zmianę w repo legacy.

Format odpowiedzi:
- Nie generuj Graphviz/DOT na tym etapie.
- Zwróć wynik w Markdown.
- Zacznij od 3-5 najważniejszych obserwacji.
- Potem użyj tabeli z kolumnami:
  - Obszar
  - Co znalazłeś
  - Dowód z dependency-cruiser
  - Dlaczego to ważne przy zmianie
  - Związek z `artifact-1-territory.md`
  - Co sprawdzić dalej
```

**Prompt 2 — granice warstw:**

```text
Sprawdź, czy frontend respektuje granice warstw: `platform/types` jako fundament, `platform/client` poniżej `channels/src`, oraz brak niedozwolonych importów między tymi obszarami.

Zinterpretuj wyniki w kontekście aktywności z `context/map/artifact-1-territory.md`, szczególnie dla `admin_console`, `packages`, `client` i `types`. Chcę wiedzieć, czy często zmieniane miejsca korzystają z tych warstw w przewidywalny sposób, czy widać importy, które mogą zaskoczyć przy zmianie.

Format odpowiedzi:
- Nie generuj Graphviz/DOT na tym etapie.
- Zwróć wynik w Markdown.
- Zacznij od 3-5 najważniejszych obserwacji.
- Potem użyj tabeli z kolumnami:
  - Sprawdzana granica
  - Wynik
  - Dowód z dependency-cruiser
  - Dlaczego to ważne przy zmianie
  - Związek z `artifact-1-territory.md`
  - Co sprawdzić dalej
```

**Prompt 3 — ryzyka testów:**

```text
Użyj dependency-cruiser dla `webapp` i przeanalizuj ryzyka testowalności w najaktywniejszych obszarach z `context/map/artifact-1-territory.md`: `admin_console`, `packages`, `utils`, `actions`, `platform/client`, `platform/types`.

Sprawdź, które miejsca mogą być trudne do testowania w izolacji, bo ciągną za sobą dużo importów, akcje, klienta API, globalny stan, wspólne utilsy albo typy z platformy. Zwróć konkretną listę ryzyk: gdzie prawdopodobnie trzeba będzie dużo mockować, gdzie lepszy będzie test integracyjny, a gdzie zmiana może naturalnie kończyć się testem e2e.

Format odpowiedzi:
- Zwróć Markdown, bez Graphviz/DOT.
- Użyj sekcji:
  - `Podsumowanie`
  - `Lista ryzyk testowych`
  - `Najbardziej podejrzane moduły`
  - `Co sprawdzić dalej`
  - `Opcjonalny kolejny krok: graf`
```

Graphviz dopiero po selekcji — jeden cykl, jedna granica, jeden fragment ryzyka; `--focus`, `--include-only`, `--collapse`, metryki fan-in/out.

**Zapis:**

```text
Zapisz podsumowanie tej sesji do `context/map/artifact-2-structure.md`
```

#### Jak ujarzmić zbyt gęsty graf

Hairball setek krawędzi = bezużyteczny decyzyjnie. Dźwignie (agent tłumaczy pytanie po ludzku):

| Cel | Dźwignia | Co robi |
| --- | -------- | ------- |
| Widok z góry | `rankdir=TB` w DOT | hierarchia pionowa |
| Foldery zamiast plików | `--collapse "^src/[^/]+"` | węzły = katalogi |
| Kto zależy od X | `--reaches "X"` | upstream do X |
| Okolica X | `--focus "X" --focus-depth N` | downstream |
| Widok architektoniczny | `--output-type archi` | high-level reporter |
| Huby | `--metrics` + `--include-only` | filtr po stopniu |
| Bez node_modules | `--exclude 'node_modules'` | nie `doNotFollow` (wciąż rysuje liście) |

Szum: `--exclude 'node_modules|\.test\.|\.stories\.|__snapshots__|\.d\.ts$'`.

#### Alternatywy pod inne stacki

Wzorzec: **statyczny graf → DOT/SVG/JSON → synteza agenta**.

| Stack | Narzędzia |
| ----- | --------- |
| JS/TS | dependency-cruiser, madge, skott |
| Python | Tach, pydeps |
| Java | jdeps, Maven Dependency Plugin |
| Go | goda, `go mod graph` |
| C# | dotnet-deptree, NDepend |
| Swift | spmgraph, SwiftPM |
| Kotlin/Gradle | gradle-dependency-graph-generator |

Granice → `unknowns`: refleksja/`Class.forName`, złe aliasy TS, runtime DI/flagi/codegen — graf statyczny nie pokaże.

### Mapa kontrybutorów — kto wie co i o co go zapytać

Najczęściej pomijana warstwa — błąd. Kod przeczytasz; decyzje i edge case'y żyją w głowach.

`git blame` ≠ mapa kontrybutorów. Blame: kto zmienił linię. Mapa: **kto ma kontekst obszaru** i w jakim typie problemów się specjalizuje.

```text
Zapoznaj się z context/map/artifact-1-territory.md oraz context/map/artifact-2-structure.md, a następnie zidentyfikuj top 5 obszarów, które mogą wymagać potencjalnego kontaktu z kontrybutorami.
```

```text
Dla zidentyfikowanych obszarów pokaż kluczowych kontrybutorów z ostatnich 12 miesięcy. Odfiltruj boty i automatyzacje, a także commity agentów jak Claude, Codex czy Copilot, bez wyraźnego autorstwa człowieka.

Dla każdej osoby sklasyfikuj aktywności pogrupowane tematycznie, które mogą wskazywać kto zaoferuje support w wybranym obszarze tego projektu.
```

Wynik: nie „kto najwięcej commitów", lecz „kto nad jakim typem problemu" — do kogo zajrzeć przed zmianą.

```text
Zapisz wyniki do context/map/artifact-3-contributors.md
```

### Mapa projektu — finalna synteza

Po trzech składowych masz sygnały, nie mapę. Pełny graf = dekoracja. Mapa = **synteza dowodów** z gita, grafu i kontrybutorów — tylko kluczowe i wrażliwe obszary.

```text
Jesteś inżynierem wprowadzającym nowego developera do dużego repo legacy.

Twoim zadaniem jest utworzyć dokument onboardingowy `context/map/repo-map.md` z trzech już istniejących artefaktów — nie generuj danych od zera, nie powtarzaj ich tabel w całości.

Kontekst do mapy:
- `context/map/artifact-1-territory.md`
- `context/map/artifact-2-structure.md`
- `context/map/artifact-3-contributors.md`

Zasady:
1. Łącz trzy perspektywy w jeden spójny obraz: gdzie żyje system → jak jest powiązany → kogo zapytać.
2. Pokaż realne granice i te miejsca, gdzie struktura katalogów nie odpowiada realnej aktywności.
3. Dokument ma prowadzić od szerokiego obrazu do 5–8 „pierwszych plików do przeczytania”.
4. Zaznacz wprost ograniczenia: to mapa aktywności i struktury w oknie 1 roku.
5. Przy sprzężeniach dopisz, skąd je wiesz: z grafu importów, z historii gita, czy to obszar, którego narzędzie w ogóle nie objęło (np. inny język albo część stacku bez grafu). Jeśli jakaś warstwa nie ma grafu zależności, powiedz to wprost — to jest `unknown`, a nie „brak powiązań”.
6. Jeśli coś zmienia się razem, bo jest generowane albo mockowane, a nie dlatego, że ktoś edytuje to ręcznie — oznacz to. Zmiana „przez regenerację” to inny, tańszy rodzaj sprzężenia niż ręczna edycja i inaczej waży przy ocenie kosztu zmiany.

Struktura `repo-map.md`:
1. TL;DR (5–7 zdań) — czym jest repo, główne warstwy (Mermaid), gdzie skupia się praca, gdzie boli.
2. Teren — duża odpowiedzialność vs peryferia; moduły głębokie i płytkie; aktywność w czasie.
3. Realne powiązania — co naprawdę zmienia się razem (couplingi + warstwy + cykle);
4. Strefy ryzyka — 4–6 obszarów wysokiego ryzyka z jedną linijką „dlaczego”
5. Kogo zapytać — per strefa: 1–2 kandydatów dopasowanych tematycznie.
6. Pierwszy dzień — uporządkowana lista 5–8 plików/modułów wejściowych do przeczytania.
7. Ograniczenia — okno czasowe, metoda, czego mapa NIE mówi.

## Format
Markdown z Mermaid, zwięźle, tabele tylko gdy realnie pomagają.

Cel: nowy developer po 15 min czytania wie, gdzie rzeczy żyją, co jest niebezpieczne i od czego zacząć.

Zapisz do `context/map/repo-map.md`.
```

Ćwiczenie: `npx @przeprogramowani/10x-cli@latest get m4l2`; `/10x-init` jeśli brak `context/map/`. Artefakt ① raportu Architekta.

### Deep dive: Cztery rodziny wyszukiwania repo

> **Deep dive** — Pytanie „jak szukać?" ważniejsze niż „jakiego narzędzia".
>
> **Lexical / pattern** (`rg`, `grep`) — szybkie, tanie; wymaga słownictwa projektu; nie odróżni routera od komentarza.
>
> **Structural / symbol** (Probe, universal-ctags, SCIP, LSP) — „gdzie definiujemy symbol?", sygnatury, kontrakt warstwy; w lekcji pomocnicze; główny sygnał struktury z grafu zależności.
>
> **Semantic / RAG** (embeddingi) — gdy nie znasz nazwy; koszty: nieświeży indeks, złe chunkowanie; zawsze weryfikuj w kodzie.
>
> **Agentic** — iteracyjne zawężanie; właściwe w Deep Focus; w Wide Scan **kontrolowane**, inaczej losowy obszar i spalony budżet.
>
> Nie ranking — dobór do pytania. Endpointy → lexical/structural. Kto importuje płatności → graf. Definicja typu → symbol search. Przepływ feature przez warstwy → M4L3.

### Deep dive: Granice każdej składowej

> **Deep dive** — Żadna składowa nie jest wyrocznią.
>
> **Terytorium (git):** gdzie dotykano, nie dlaczego ani czy słusznie; duża aktywność = centrum produktu LUB rok napraw jednego buga; kwartały pomagają, wniosek = hipoteza; plik mógł zostać usunięty; współzmiany nie pokażą kontraktu, który _powinien_ być zsynchronizowany (model backend/front).
>
> **Struktura (graf):** importy w danym momencie, nie intencja; `shared/utils` ×8 = CE lub worek; cykl = problem lub artefakt buildu — ty rozstrzygasz.
>
> **Kontrybutorzy:** kto dotykał, nie formalny owner, nie czy nadal w firmie, nie czy PR-y słuszne — wejście do rozmowy i PR-ów.
>
> Workflow: sygnały z trzech źródeł → synteza z oznaczeniem źródła i `unknowns` → decyzja gdzie Deep Focus.

### Deep dive: Dwa znaczenia semantic search

> **Deep dive** — „Semantic code search" nie zawsze = embeddingi.
>
> **Probe:** „semantic" = kompletne funkcje/klasy z AST (tree-sitter), nie wektory — **structural**, nie RAG.
>
> | Rodzina | Mechanizm | Ryzyko |
> | ------- | --------- | ------ |
> | Structural | symbol, funkcja, struktura | wymaga wzorca/indeksu; weryfikuj |
> | Semantic/RAG | podobieństwo chunków | nieświeży indeks, probabilistyczny wynik |
>
> Mylenie rodzin → agent przypisuje złą pewność mapie.

### Deep dive: Jak klasyfikować moduły na mapie

> **Deep dive** — Lista modułów nie wystarczy — etykiety decyzyjne z dowodami:
>
> - **core / supporting / peripheral** — bliskość wartości produktu,
> - **deep / shallow** — logika vs cienki adapter/router,
> - **stable / volatile / seasonal** — profil zmian w czasie,
> - **load-bearing / contained** — zależni od kontraktu,
> - **high / medium / low sensitivity** — ostrożność przy zmianie.
>
> **Etykieta bez dowodu = domysł.** Core ≠ największy folder — dowód: entry point, endpoint, publiczny typ, częste importy, user flow.
>
> **Coupling:** incoming (blast radius kontraktu), outgoing (kruchość na sąsiadów), contract (API, eventy, schema), runtime (DI, flagi, webhooks, refleksja), co-change (sygnał mapy; pełna analiza M4L3).
>
> **Metryki grafu** (nie „ważność biznesowa", ale sygnały):
>
> | Sygnał | Praktycznie | W mapie |
> | ------ | ----------- | ------- |
> | Ca wysokie | wielu zależnych | kontrakt / element nośny |
> | Ce wysokie | wiele zależności | orkiestracja / kruchość |
> | niska instability + wysokie Ca | stabilny rdzeń | |
> | wysoka instability | cienki adapter | |
> | cycles | splątane granice | ostrożność |
>
> Zapisuj decyzję wspartą metryką, nie surową tabelę. Przykład bloku:
>
> ```text
> Module: server/public
> role: supporting / contract layer
> evidence: wysokie Ca; częste współzmiany z backendem i frontendem
> inference: zmiana typów może przejść przez warstwy
> caution: przed zmianą sprawdzić użycia w webapp i server/channels
> unknowns: graf nie pokazuje runtime ani zgodności API klientów
> ```
>
> **Statyczna analiza legacy — co liczy:** entry pointy (endpoint, CLI, job, webhook); kierunek zależności i odwrócone warstwy; kontrakty (DTO, schema, shared packages); centra grafu; cykle; adaptery i cienkie wejścia; luki: dynamic import, DI, refleksja, codegen, feature flagi.
>
> Prompty wymuszające evidence/inference/unknowns:
>
> ```text
> Na podstawie grafu zależności wypisz tylko moduły, które realnie wpływają na ryzyko zmiany. Dla każdego pokaż: evidence, inference, caution, unknowns.
> ```
>
> ```text
> Które moduły są prawdopodobnie core, ale dowód jest słaby? Wypisz, jaką komendą albo jakim dodatkowym źródłem można to zweryfikować.
> ```
>
> ```text
> Gdzie graf statyczny może kłamać przez runtime coupling: DI, dynamic import, feature flagi, konfigurację, webhooks albo codegen? Zapisz jako unknowns.
> ```
>
> Klasyfikacja po Wide Scan — bez refaktoru i hotspotów:
>
> ```text
> Na podstawie wyników Wide Scan sklasyfikuj tylko obszary istotne z perspektywy ryzyka zmiany.
> Dla każdego: role, depth, change profile, blast radius, sensitivity, contract/implementation layer jeśli widać.
> Sensitivity z dowodów: zależni, publiczny kontrakt, częste zmiany, co-change, cykle, runtime config, integracje, auth/dane/cache/migracje/build.
> Hipoteza → `unknown` / `needs verification`. Nie diagnozuj hotspotów, nie proponuj refaktoryzacji.
> ```
>
> Mattermost z pierwszych sygnałów: `server/channels` + `webapp/channels` → core; `server/public` → contract layer; `post_view.tsx` → shallow entry; `post_list_virtualized` → głębsze centrum. To legenda mapy, nie ocena jakości.
