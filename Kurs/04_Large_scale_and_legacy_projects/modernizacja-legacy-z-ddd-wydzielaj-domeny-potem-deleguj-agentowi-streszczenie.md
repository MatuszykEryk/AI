## Od „refaktoruj kod" do „odkryj domenę"

Refaktoryzacja (M4L4) odpowiada: _jak bezpiecznie zmienić kod?_ DDD odpowiada: _czy kod odpowiada biznesowi?_ Pomylenie pytań daje wiarygodny plan utrwalający bałagan nazw — agent projektuje wokół nazw, które same w sobie są pomyłką.

Lekcja nie jest kursem DDD — bierzemy tyle, ile potrzeba na etap post-MVP: **ubiquitous language**, **bounded contexts**, **niezmienniki**, **agregaty**, **Anti-Corruption Layer**. Klasyczne wejście w DDD było drogie (wywiad z ekspertem, warsztaty, mapowanie). LLM zbija koszt do „wystarczająco dobrego": AI łączy klasykę modelowania z semantyką modelu — wywiad domenowy i Event Storming solo, po godzinach, na cudzym repozytorium.

Cel lekcji: utrzymać rosnący projekt za tydzień, miesiąc, kwartał — przez **drugi duży cykl** post-MVP, w którym architekturę kształtuje domena, nie zgadywanie.

## Najpierw język: dwa sposoby na ubiquitous language

Zasada DDD: **granice językowe ujawniają granice kontekstów.** Tam, gdzie ten sam termin znaczy co innego, przebiega szew między obszarami.

Przykład `3xAccount` w fintechu:
1. Konto użytkownika — login, e-mail, profil, 2FA.
2. Rachunek bankowy — IBAN, saldo, waluta.
3. Konto księgowe — pozycja w planie kont.

W kodzie `AccountService`, `AccountStatus`, `AccountClosed` — nie wiadomo, co zamykasz. Rozwiązanie: `UserProfile`, `BankAccount`, `LedgerAccount`.

W 10xCards (przykład przez całą lekcję): PRD ma **„sesję generacji"** — byt z cyklem życia i finalizacją. W kodzie luźny identyfikator przy wierszach propozycji. Nazwy: `draft` / „propozycje" / `cards` / „kandydaci" — **jeden byt, cztery nazwy**. Precyzja języka ma znaczenie przy skalowaniu i koszcie tokenów.

### Tryb pierwszy: wywiad z ekspertem domenowym

Wyjątek od reguły „agent, nie chatbot" — tu zwykła rozmowa z największym modelem wystarczy.

**Ekspert domenowy** — rozumie reguły biznesu, procesy i wyjątki; nie musi programować (księgowa, lekarz, analityk SRS…).

Symulowany wywiad: procesy biznesowe, algorytmy (Leitner vs SM-2 vs FSRS — trade-offy), terminologia (`Review` vs `Test`, `Due Card` vs `Pending`). Wynik → plik kontekstowy jako referencja w researchach i planach. Unikasz problemu `3xAccount`.

**agent-forum** — dwa modele (programista + ekspert), opcjonalny `summarizer` → brief domenowy w `threads/` i `summary.md`:

```ts
new Forum({
  threadName: "spaced-repetition-interview",
  rounds: 3,
  agents: [
    { agentId: "beginner-developer", model: orModel("openai/gpt-4o-mini"), personality: BEGINNER_DEVELOPER() },
    { agentId: "spaced-repetition-expert", model: orModel("anthropic/claude-sonnet-4.6"), personality: SPACED_REPETITION_EXPERT() },
  ],
  summarizer: { agentId: "learning-insights", model: orModel("openai/gpt-4o"), personality: LEARNING_INSIGHTS },
});
```

Uruchomienie: sklonuj repo, `OPENROUTER_API_KEY` w `.env`, `npm start`. Pytania zadaje drugi agent — dostajesz gotowy zapis zamiast kopiowania z czatu.

### Tryb drugi: dokument kontekstowy kontra realny kod

Dla projektów z `prd`, `context/archive`, roadmapą — agent wyciąga język z dokumentów i **porównuje z kodem**. Produkt = MAPA domeny, nie kod. Prompt destylacji:

```text
Pracujesz jako specjalista Domain-Driven Design skupiony na destylacji domeny biznesowej z istniejących dokumentów źródłowych. Twoim produktem jest MAPA domeny, nie kod. Nie zakładaj z góry żadnych nazw bytów, agregatów, ścieżek ani numerów wymagań — masz je ODKRYĆ. Pracuj w trzech krokach: odkrycie → analiza → klasyfikacja.

KROK 0 — Odkryj kontekst projektu.
- Znajdź i przeczytaj dokumenty wizji/wymagań, jeśli istnieją: poszukaj prd.md, tech-stack.md, README (typowo w katalogu z dokumentami foundation/docs lub w korzeniu repo). Jeśli istnieje rozszerzona narracja/historia zmian — przeczytaj ją też jako materiał źródłowy.
- Jeśli brak dokumentów wymagań — oprzyj się na README + kodzie i wyraźnie to odnotuj jako ograniczenie.
- Ustal stack i strukturę repo: gdzie żyje logika biznesowa (warstwy: API/ serwis/domena/UI/persystencja), jakie są katalogi źródłowe.

KROK 1 — Zbuduj Ubiquitous Language.
- Wyciągnij pojęcia domenowe z dokumentów ORAZ z kodu (nazwy encji, bytów, stanów, operacji, reguł). NIE wymyślaj — cytuj źródło.
- Dla każdego pojęcia podaj: definicję, cytat źródłowy (plik:linia), oraz gdzie termin żyje w kodzie (plik:linia) LUB wyraźną adnotację "BRAK w kodzie".

KROK 2 — Sklasyfikuj subdomeny: Core / Supporting / Generic.
- Tabela: każde pojęcie/obszar przypisz do jednej kategorii i uzasadnij odwołaniem do celów produktu (success criteria / sekcja wizji / non-goals, jeśli istnieją). Rdzeń = to, co stanowi przewagę i sens produktu.

KROK 3 — Wskaż kandydatów na agregaty i ich niezmienniki.
- Dla każdego kandydata: jaka reguła biznesowa MUSI być zawsze prawdziwa (niezmiennik), z cytatem ze źródła, oraz status: czy kod ją egzekwuje, deklaruje, czy ignoruje.

KROK 4 — Zbuduj listę rozjazdów MODEL vs KOD.
- Tabela: dokument mówi X — kod robi Y — dowód (plik:linia). To najcenniejsza część: pokazuje gdzie wiedza domenowa istnieje, a kod jej nie odwzorowuje.

KROK 5 — Ranking refaktoru.
- Uszereguj kandydatów na agregaty wg wartości (jak rdzeniowy niezmiennik) i ryzyka (jak słabo jest dziś egzekwowany). Wskaż #1 do refaktoru i dlaczego.

OGRANICZENIA:
- Nie pisz kodu produkcyjnego. Cytuj wyłącznie ścieżki/linie, które realnie zweryfikowałeś.
- Zapisz dokument do: context/domain/01-domain-distillation.md (z frontmatter: title, created, type: domain-distillation).
- Na koniec zwróć podsumowanie 5–8 zdań: co zawiera artefakt i najważniejszy wniosek.

Zapisz rezultat do context/domain/01-domain-distillation.md
```

Pragmatyczne DDD: na starcie koszt błędnej nazwy niski — obszerny glosariusz przedwcześnie. DDD ma sens, gdy domena **boli** (post-MVP, `3xAccount` wszędzie).

## Niezmienniki, czyli reguły, których pilnujesz

**Niezmiennik** — reguła MUSI być prawdziwa niezależnie od tego, kto rusza system. To nie walidacja formularza — to sens produktu. W legacy reguła jest **wszędzie i nigdzie**: kawałki w UI, serwisie, kolejności handlerów; klient jako jedyny strażnik; błąd połykany zamiast zatrzymać operację.

10xCards: niezmiennik sesji generacji — **finalizacja dopiero, gdy każda propozycja rozstrzygnięta** (zaakceptowana lub odrzucona). Brak `GenerationSession` = brak strażnika. UI liczy karty, endpoint ufa żądaniu, w bazie luźny identyfikator.

**Agregat** — granica spójności; **jedyny** strażnik niezmiennika. Nielegalna operacja → nazwany błąd domenowy, nie cichy zapis.

Prompt planu refaktoru (produkt = PLAN, nie implementacja; agent **odkrywa** niezmiennik #1 — nie zakładaj z góry):

```text
Pracujesz jako specjalista Domain-Driven Design skupiony na identyfikacji i zabezpieczaniu domenowych niezmienników. Produkt to PLAN refaktoru, nie implementacja — nie modyfikuj kodu produkcyjnego. Nie zakładaj z góry, który niezmiennik jest rdzeniowy ani jak nazywają się byty — masz to ODKRYĆ i WYBRAĆ. Pracuj w krokach: odkrycie → identyfikacja → klasyfikacja → diagnoza → projekt.

KROK 0 — Odkryj kontekst.
- Przeczytaj dokumenty wymagań, jeśli istnieją (prd.md / tech-stack.md / README; szukaj w foundation/docs/root). Zwróć uwagę na sekcje "business logic", "success criteria", reguły i wymagania funkcjonalne.
- Ustal stack i warstwy, w których żyje logika biznesowa (API / serwis / domena / UI / persystencja).

KROK 1 — IDENTYFIKUJ niezmienniki biznesowe.
- Zbuduj listę reguł, które w tej domenie MUSZĄ być zawsze prawdziwe (np. "X powstaje tylko z Y", "operacja Z jest atomowa", "dane D nigdy nie są persystowane", "przejście stanu A→B wymaga warunku C"). Wyciągaj z dokumentów ORAZ z kodu. Cytuj źródło.

KROK 2 — KLASYFIKUJ i wybierz #1.
- Dla każdego niezmiennika oceń trzy osie:
  (a) jak rdzeniowy dla sensu produktu (odwołaj się do celów/wizji),
  (b) jak bardzo rozsmarowany po warstwach (w ilu plikach/warstwach żyje),
  (c) czy jest realnie EGZEKWOWANY, tylko deklarowany, czy naruszalny.
- Wybierz niezmiennik, który jest jednocześnie najbardziej rdzeniowy I najsłabiej egzekwowany. Uzasadnij wybór.

KROK 3 — DIAGNOZA wybranego niezmiennika.
- Pokaż dokładnie, gdzie dziś żyje reguła (cytaty plik:linia we wszystkich warstwach). Wskaż: które warstwy jej nie egzekwują, gdzie jest egzekwowana niespójnie, gdzie klient (UI) jest jedynym strażnikiem, gdzie błąd jest "połykany" zamiast zatrzymywać operację.

KROK 4 — PROJEKT agregatu-strażnika.
- Zaprojektuj agregat (root) będący JEDYNYM miejscem egzekwowania niezmiennika.
- Metody domenowe z preconditions; nielegalna operacja rzuca nazwany błąd domenowy (nie cicho aktualizuje stanu). Pokaż sygnatury + pseudokod.
- Repozytorium ładujące/zapisujące agregat zamiast rozsianych zapytań; jeśli niezmiennik wymaga atomowości — pokaż, jak całość idzie w JEDNEJ transakcji.
- Cienkie API/route: parse wejścia → metoda agregatu → mapowanie błędu domenowego na odpowiedź. Egzekucja przenosi się z klienta na serwer (jeśli dziś jest na kliencie).

KROK 5 — Before/after, plan, testy.
- Before/after dla każdego dzisiejszego miejsca reguły.
- Plan faz refaktoru. Jeśli projekt ma dyscyplinę test-first / istniejący runner — zaznacz, które fazy idą test-first i wypisz przypadki testowe dla niezmiennika (legalne i nielegalne przejścia/operacje).
- Lista nowych "load-bearing" nazw do zarejestrowania, jeśli projekt prowadzi rejestr kontraktów.

OGRANICZENIA:
- Fail-fast: nielegalna operacja zatrzymuje, nie loguje-i-jedzie dalej.
- Cytuj tylko zweryfikowane plik:linia.
- Zapisz dokument do: context/domain/02-invariant-aggregate-refactor.md (frontmatter: title, created, type: refactor-plan).
- Zwróć podsumowanie 5–8 zdań na koniec.

Zapisz rezultat do context/domain/02-invariant-aggregate-refactor.md
```

Efekt: reguła w kodzie, którego nie da się obejść; egzekucja na serwerze; wiedza w jednym pliku, nie w czterech.

## Anti-Corruption Layer, czyli przeciwko szkodnikom

**Przeciek** — typy biblioteki zewnętrznej rozłazą się po kodzie. Przykład `ts-fsrs`: `Card` ze `stability`, `difficulty`, `due` w DB, API, propsach Svelte; konwersja zduplikowana w trzech miejscach. Koszty: wymiana biblioteki = cały system; biblioteka serwerowa w bundlu klienta.

**ACL** — cienka warstwa na granicy: domenowy **value object** (jedyne miejsce wiedzy o kształcie zależności) + **wąski port** (interfejs domenowy) + **adapter** na bibliotekę. Reszta kodu zna tylko port.

Prompt ACL:

```text
Pracujesz jako specjalista Domain-Driven Design skupiony na identyfikacji przeciekających zależności i łamania granic warstw domeny. Produkt to PLAN refaktoru, nie implementacja — nie modyfikuj kodu produkcyjnego. Nie zakładaj z góry, która zależność przecieka ani jak nazywają się byty — masz to ODKRYĆ i WYBRAĆ. Kroki: odkrycie → identyfikacja → klasyfikacja → diagnoza → projekt.

KROK 0 — Odkryj kontekst.
- Przeczytaj dokumenty bazowe, jeśli istnieją (prd.md / tech-stack.md / README). Zwróć uwagę na deklaracje o wymienialności komponentów lub o tym, że jakiś byt jest celowo odseparowany "żeby dało się wymienić X".
- Ustal stack, listę zależności zewnętrznych (manifest pakietów) i warstwy kodu.

KROK 1 — IDENTYFIKUJ przeciekające zależności.
- Znajdź zależności zewnętrzne, które przeciekają przez granice warstw. Sygnały: ten sam pakiet importowany w wielu warstwach (API + UI + serwis), zduplikowana rekonstrukcja obiektów/typów biblioteki w kilku miejscach, typy biblioteki w sygnaturach domenowych lub w kontraktach wire (DTO/response), wołanie tego samego SDK po obu stronach granicy klient/serwer.
- Dla każdej: wylicz WSZYSTKIE pliki, które ją dziś "znają" (plik:linia).

KROK 2 — KLASYFIKUJ i wybierz #1.
- Oceń każdą oś: (a) liczba warstw/plików dotkniętych, (b) ryzyko/koszt wymiany biblioteki dziś, (c) czy dokumenty deklarują, że ma być wymienialna (rozjazd intencja-vs-kod jest mocnym sygnałem). Wybierz najgorszy przeciek. Uzasadnij.

KROK 3 — DIAGNOZA.
- Pokaż duplikację (cytaty plik:linia) i przecieki przez granice — zwłaszcza groźne (np. biblioteka serwerowa wciągana do bundla klienta). Jeśli dokument deklaruje wymienialność — zacytuj to (plik:linia) i pokaż, że kod jej nie dotrzymuje.

KROK 4 — PROJEKT ACL.
- Zaprojektuj domenowy value object/encję, która jest JEDYNYM miejscem wiedzy o kształcie zależności (mapowanie z/do persystencji, konwersja do/z typu biblioteki, operacje domenowe). Pokaż sygnatury + pseudokod.
- Zdefiniuj WĄSKI port (interfejs domenowy) i adapter implementujący go przez konkretną bibliotekę. Reszta kodu zna tylko port.

KROK 5 — Dowód izolacji + before/after.
- Udowodnij listą, że wymiana biblioteki dotyka tylko adaptera, nie tabel/API/UI.
- Before/after dla zduplikowanych miejsc; pokaż, że warstwa UI dostaje gotowe dane domenowe, nie surowy obiekt biblioteki.
- Jeśli istnieją otwarte pytania zależne od kontraktu tej biblioteki — rozstrzygnij je w oparciu o jej dokumentację i wskaż, gdzie zakodować decyzję (w ACL, nie w warstwie API).

KROK 6 — Weryfikacja i plan.
- Kryterium sukcesu: grep po nazwie pakietu zwraca wyłącznie pliki w katalogu ACL/ adaptera. Wypisz, które pliki dziś znają zależność, a które po refaktorze już nie.
- Plan faz zgodny z konwencją projektu.

OGRANICZENIA:
- Cytuj tylko zweryfikowane plik:linia. Nie pisz kodu produkcyjnego.
- Zapisz dokument do: context/domain/03-anti-corruption-layer.md (frontmatter: title, created, type: refactor-plan).
- Zwróć podsumowanie 5–8 zdań na koniec.

Zapisz rezultat do context/domain/03-anti-corruption-layer.md
```

Rozjazd intencja-vs-kod (dokument deklarował wymienialność) = najmocniejszy sygnał. Sukces = **sprawdzalne kryterium** grep, nie „czy ładniej".

## Interaktywne warsztaty DDD z event-storming-canvas

Event Storming (Brandolini): pomarańczowe zdarzenia, niebieskie komendy, żółci aktorzy, czerwone hotspoty na osi czasu. Klasyczny warsztat = sala, ściana karteczek, ekspert, moderator — dzień pracy kilku osób. Agent **nie zastępuje** eksperta, ale daje „wystarczająco dobry" start solo.

**event-storming-canvas** — `board.json` jako jedyne źródło prawdy; człowiek w przeglądarce, agent edytuje JSON; SSE na żywo. Gramatyka Brandoliniego + selektor faz ogranicza pasek narzędzi: `chaotic-exploration` → `timeline` → `hotspots` → `aggregates`.

```bash
node server.js
# http://localhost:4000
```

`CLAUDE.md` / `AGENTS.md` w repo = protokół moderatora (respektuj fazę, nie nadpisuj karteczek użytkownika, zamieniaj niejasności w hotspoty). Przykładowe prompty:

```text
Wyczyść tablicę i poprowadź warsztat Event Storming dla procesu asynchronicznej generacji fiszek w 10xCards. Zacznij od fazy chaotic-exploration i dorzuć 4–5 zdarzeń domenowych w czasie przeszłym.
```

```text
Przejdź do fazy hotspots i zaznacz na czerwono miejsca, w których proces może się wysypać — błąd modelu, częściowo zaakceptowana sesja, timeout generacji.
```

Agent czyta `board.json` przed każdą akcją — buduje na twoim stanie. Sesje bez pamięci (brak DB) — zachowaj `board.json` ręcznie. Czerwone hotspoty = kandydaci na niezmienniki i granice kontekstów.

## Wsad do dalszej refaktoryzacji

Cztery artefakty na dysku: `01-domain-distillation.md`, `02-invariant-aggregate-refactor.md`, `03-anti-corruption-layer.md`, `board.json`. To **wsad** (diagnozy, plany, pytania), nie produkt końcowy.

Skille z dokumentami DDD jako wejściem — te same co przy MVP, lepsze paliwo:

```text
/10x-research @context/domain/01-domain-distillation.md
  → zbadaj, jak głęboko subdomena Core jest dziś rozsmarowana po warstwach

/10x-plan @context/domain/02-invariant-aggregate-refactor.md
  → zamień plan refaktoru w konkretną zmianę z własnym change-id

/10x-roadmap
  → ułóż hotspoty z Event Stormingu w kolejność zmian wg wartości i ryzyka
```

`/10x-plan` nagradza gotowy research — `02-invariant-aggregate-refactor.md` to research z fazami, before/after i testami. DDD **zasila** workflow z modułów 1–3, nie go zastępuje.

Wpinki w drugi cykl:
- **`01`** — ranking refaktoru i rozjazdy `model-vs-kod` → `/10x-shape`, `/10x-roadmap`.
- **`02`**, **`03`** — gotowe plany → `/10x-plan` jako wsad pod `change-id`.
- **`board.json`** — hotspoty → potencjalne `change-id` na research.

Modernizacja legacy z AI = **powtarzalny cykl**: odkryj domenę → nazwij rozjazdy → zabezpiecz niezmienniki → ACL → te same skille co przy MVP.

### Ćwiczenia i raport architektoniczny (10xArchitect)

Trzy prompty → trzy pliki w `context/domain/`. Opcjonalnie: `event-storming-canvas` od `chaotic-exploration` po `hotspots`.

Ostatnia lekcja modułu 4 — prompt zbiera artefakty L2–L5 w `context/architect-report.md` (~2 strony). Podmień `{...}`; brak artefaktu → `BRAK artefaktu`, nie domysły:

```text
Zbuduj jeden sumaryczny raport architektoniczny z modułu 4 (ścieżka 10xArchitect).
Cel: zwięzły two-pager (~2 strony), czytelny dla człowieka, oparty wyłącznie na poniższych artefaktach. Nie wymyślaj faktów - jeśli czegoś brakuje, napisz wprost "BRAK artefaktu" i nie uzupełniaj luki domysłami.

Wejścia (artefakty z modułu 4):
- Mapa repozytorium (L2): {ścieżka-do-repo-map.md}
- Research wybranego ficzera (L3): {ścieżka-do-research.md}
- Plan refaktoryzacji (L4): {ścieżka-do-plan.md}
- Notatki o domenie / DDD (L5): {ścieżki-do-context/domain/*.md}

Uwaga: artefakty mogą pochodzić z RÓŻNYCH projektów. Dla każdego wejścia podaj, na jakim repozytorium powstało.

Struktura raportu:

1. Opisane projekty
   - Dla każdego repo użytego w module: nazwa, stack, skala (orientacyjnie), i przy którym artefakcie się pojawiło (L2/L3/L4/L5).

2. Mapa projektu (z L2)
   - 3-5 kluczowych wniosków z mapy: strefy ryzyka, lokalne centra, entry pointy, najważniejsze unknowns.

3. Analiza ficzera (z L3)
   - Który przepływ badałeś i dlaczego (link do strefy ryzyka z mapy).
   - Feature overview w 3-4 zdaniach: skąd input, gdzie zmienia się stan, co wraca.
   - Technical debt: 2-3 najważniejsze ryzyka (kruche sprzężenia, luki testowe, blast radius), z których co najmniej jedno potwierdzone ast-grepem.

4. Plan refaktoryzacji (z L4)
   - Co refaktoryzowane: wybrana opcja i jej docelowy kształt.
   - Czego świadomie NIE robimy.
   - Fazy planu w jednej linijce każda + jak weryfikowane (auto/ręcznie).

5. Domena wg DDD (z L5)
   - Ubiquitous language: 3-5 kluczowych pojęć + najważniejsze rozjazdy model-vs-kod.
   - Niezmiennik #1 i agregat, do którego należy.
   - Anti-Corruption Layer: która zależność przecieka i przez ile warstw.

6. Decyzje, które należą do mnie
   - 3-5 zdań: co AI podpowiedziało, a co rozstrzygnąłeś samodzielnie i dlaczego.

Zasady:
- Maksymalnie dwie strony. Tnij, nie streszczaj wszystkiego.
- Każde twierdzenie strukturalne (liczby, "tylko tutaj") oprzyj na artefakcie, nie na własnej pamięci o kodzie.
- Zapisz wynik jako context/architect-report.md
```

Przeczytaj jak recenzent: czy dwustronicowy dokument broni się sam, i czy sekcja „Decyzje, które należą do mnie" brzmi jak twoje decyzje.

### Deep dive: Kiedy sięgać po DDD (i czemu nie od pierwszego dnia)

> **Deep dive** — Wczesne DDD ma sens przy głębokiej znanej domenie lub ekspercie obok. Solo/nowy rynek: domena płytka, agregaty to zgadywanie — najpierw walidacja rynku. Kolejność pragmatyczna: **koduj, ucz się domeny, modeluj gdy boli.** Sygnały: `3xAccount`, byt z PRD bez kodu, cotygodniowe ratowanie DB skryptem, wymiana biblioteki = cały sprint. Moment post-MVP, drugi cykl.

### Deep dive: Pięć mitów o DDD

> **Deep dive** — (1) DDD ≠ ciężka architektura — strategia (język, konteksty) bez nowych klas. (2) Nie tylko mikroserwisy — granice w monolicie też. (3) Brak eksperta — LLM obniża koszt wejścia. (4) DDD ≠ Event Storming ani sam agregat — narzędzia, nie całość. (5) Bufet, nie zestaw obiadowy — weź ubiquitous language bez reszty. AI obniża ceremonię; dyscyplina była tania od zawsze.

### Deep dive: Kiedy ekspert domenowy zmyśla

> **Deep dive** — Halucynacje podstępne, bo pytasz o to, czego nie znasz. Model mocny przy ogólnych pojęciach (fiszki, SRS, logistyka); słaby przy precyzyjnych stałych, polach API, regulacjach. Wywiad = **generator hipotez**, nie prawda. Cross-check: dokumentacja źródłowa; dwa modele (`agent-forum`); konfrontacja z kodem i PRD (tryb 2). Ostatnie słowo — źródło, które da się otworzyć.
