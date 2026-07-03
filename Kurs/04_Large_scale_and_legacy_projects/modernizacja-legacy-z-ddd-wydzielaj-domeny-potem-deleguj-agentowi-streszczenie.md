## Od „refaktoruj kod" do „odkryj domenę"

Refaktoryzacja (M4L4) odpowiada: _jak bezpiecznie zmienić kod?_ DDD odpowiada: _czy kod odpowiada biznesowi?_ Pomylenie pytań daje wiarygodny plan utrwalający bałagan nazw.

Lekcja nie jest kursem DDD — bierzemy tyle, ile potrzeba na etap post-MVP: **ubiquitous language**, **bounded contexts**, **niezmienniki**, **agregaty**, **Anti-Corruption Layer**. AI łączy klasykę modelowania z semantyką LLM — wywiad domenowy i Event Storming solo na poziomie „wystarczająco dobrym".

### Tryb pierwszy: wywiad z ekspertem domenowym

Zasada DDD: **granice językowe ujawniają granice kontekstów.** Przykład `3xAccount` (profil / rachunek bankowy / konto księgowe) → `UserProfile`, `BankAccount`, `LedgerAccount`.

W 10xCards: PRD ma „sesję generacji"; w kodzie luźny identyfikator; `draft` / „propozycje" / `cards` / „kandydaci" — jeden byt, cztery nazwy.

Do poznania języka: największy model z wiedzą o domenie. **Ekspert domenowy** — rozumie reguły biznesu, nie musi programować. Wyjątek od reguły „agent, nie chatbot" — tu zwykła rozmowa wystarczy.

Symulowany wywiad: pytania o procesy (SRS, algorytmy powtórek), terminologię (`Review` vs `Test`, `Due Card` vs `Pending`). Wynik → plik kontekstowy jako referencja.

**agent-forum** — dwa modele (programista + ekspert), `summarizer` → brief domenowy w `threads/` i `summary.md`:

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

### Tryb drugi: dokument kontekstowy kontra realny kod

Dla projektów z `prd`, `context/archive`, roadmapą — agent wyciąga język z dokumentów i **porównuje z kodem**. Prompt destylacji (produkt = MAPA domeny, nie kod):

```text
Pracujesz jako specjalista Domain-Driven Design skupiony na destylacji domeny biznesowej z istniejących dokumentów źródłowych. Twoim produktem jest MAPA domeny, nie kod. Nie zakładaj z góry żadnych nazw bytów, agregatów, ścieżek ani numerów wymagań — masz je ODKRYĆ.

KROK 0 — Odkryj kontekst projektu.
- Znajdź i przeczytaj prd.md, tech-stack.md, README (foundation/docs lub root). Jeśli brak — README + kod, odnotuj ograniczenie.
- Ustal stack i gdzie żyje logika biznesowa.

KROK 1 — Zbuduj Ubiquitous Language.
- Pojęcia z dokumentów ORAZ kodu. NIE wymyślaj — cytuj źródło.
- Dla każdego: definicja, cytat (plik:linia), kod (plik:linia) LUB "BRAK w kodzie".

KROK 2 — Sklasyfikuj subdomeny: Core / Supporting / Generic — uzasadnij celami produktu.

KROK 3 — Kandydaci na agregaty i niezmienniki — reguła MUSI być prawdziwa; status egzekwowania w kodzie.

KROK 4 — Tabela rozjazdów MODEL vs KOD (dokument mówi X — kod robi Y — dowód).

KROK 5 — Ranking refaktoru; wskaż #1.

OGRANICZENIA: nie pisz kodu produkcyjnego; cytuj tylko zweryfikowane plik:linia.
Zapisz do: context/domain/01-domain-distillation.md
```

Na starcie projektu koszt błędnej nazwy niski — obszerny glosariusz przedwcześnie. DDD ma sens, gdy domena **boli** (post-MVP).

## Niezmienniki, czyli reguły, których pilnujesz

**Niezmiennik** — reguła, która MUSI być prawdziwa (np. sesja finalizuje się dopiero, gdy każda propozycja rozstrzygnięta). W legacy reguła jest **wszędzie i nigdzie** — kawałki w UI, serwisie, kolejności handlerów; klient jako jedyny strażnik.

**Agregat** — jedyny strażnik niezmiennika; nielegalna operacja → nazwany błąd domenowy, nie cichy zapis.

Prompt planu refaktoru (produkt = PLAN, nie implementacja):

```text
Pracujesz jako specjalista Domain-Driven Design skupiony na identyfikacji i zabezpieczaniu domenowych niezmienników. Produkt to PLAN refaktoru, nie implementacja.

KROK 1 — IDENTYFIKUJ niezmienniki (dokumenty + kod, cytaty).
KROK 2 — KLASYFIKUJ i wybierz #1 (rdzeniowość, rozsmarowanie, egzekwowanie).
KROK 3 — DIAGNOZA (plik:linia we wszystkich warstwach).
KROK 4 — PROJEKT agregatu-strażnika (preconditions, repozytorium, transakcja).
KROK 5 — Before/after, plan faz, przypadki testowe.

Zapisz do: context/domain/02-invariant-aggregate-refactor.md
```

Agent **odkrywa** niezmiennik #1 — nie zakładaj z góry. Efekt: reguła w kodzie, którego nie da się obejść; egzekucja na serwerze.

## Anti-Corruption Layer, czyli przeciwko szkodnikom

**Przeciek** — typy biblioteki zewnętrznej (`ts-fsrs` `Card`) w DB, API, UI. Koszt: wymiana biblioteki = cały system; biblioteka serwerowa w bundlu klienta.

**ACL** — cienka warstwa: domenowy value object + **wąski port** (interfejs domenowy) + **adapter** na bibliotekę. Reszta kodu zna tylko port.

Prompt ACL:

```text
Pracujesz jako specjalista DDD skupiony na przeciekających zależnościach. Produkt to PLAN refaktoru.

KROK 1 — IDENTYFIKUJ przecieki (ten sam pakiet w API+UI+serwisie, typy w DTO, duplikacja rekonstrukcji).
KROK 2 — KLASYFIKUJ i wybierz #1 (warstwy dotknięte, koszt wymiany, rozjazd intencja-vs-kod z dokumentów).
KROK 3 — DIAGNOZA.
KROK 4 — PROJEKT ACL (value object, port, adapter).
KROK 5 — Dowód izolacji + before/after.
KROK 6 — Kryterium sukcesu: grep po nazwie pakietu zwraca wyłącznie pliki ACL/adaptera.

Zapisz do: context/domain/03-anti-corruption-layer.md
```

Sprawdzalne kryterium sukcesu — nie „czy ładniej".

## Interaktywne warsztaty DDD z event-storming-canvas

Event Storming (Brandolini): zdarzenia, komendy, aktorzy, hotspoty na osi czasu. Klasyczny warsztat = wysoki koszt wejścia; agent jako moderator „wystarczająco dobry" przed zaproszeniem ludzi.

**event-storming-canvas** — `board.json` jako jedyne źródło prawdy; człowiek w przeglądarce, agent edytuje JSON; SSE na żywo. Fazy: `chaotic-exploration` → `timeline` → `hotspots` → `aggregates` — pasek narzędzi ograniczony do roli fazy.

```bash
node server.js
# http://localhost:4000
```

`CLAUDE.md` / `AGENTS.md` w repo = protokół moderatora. Przykładowe prompty:

```text
Wyczyść tablicę i poprowadź warsztat Event Storming dla procesu asynchronicznej generacji fiszek w 10xCards. Zacznij od fazy chaotic-exploration i dorzuć 4–5 zdarzeń domenowych w czasie przeszłym.
```

```text
Przejdź do fazy hotspots i zaznacz na czerwono miejsca, w których proces może się wysypać.
```

Sesje bez pamięci — zachowaj `board.json` ręcznie.

## Wsad do dalszej refaktoryzacji

Cztery artefakty: `01-domain-distillation.md`, `02-invariant-aggregate-refactor.md`, `03-anti-corruption-layer.md`, `board.json`. To **wsad**, nie produkt końcowy — **drugi duży cykl** post-MVP: architektura z domeny, nie zgadywania.

Skille z dokumentami DDD jako wejściem:

```text
/10x-research @context/domain/01-domain-distillation.md
/10x-plan @context/domain/02-invariant-aggregate-refactor.md
/10x-roadmap
```

`/10x-plan` nagradza gotowy research — im więcej pracy DDD na wejściu, tym węższy plan. DDD **zasila** workflow z modułów 1–3, nie go zastępuje. Modernizacja legacy z AI = powtarzalny cykl: odkryj domenę → nazwij rozjazdy → zabezpiecz niezmienniki → ACL → te same skille co przy MVP.

Raport architektoniczny (10xArchitect) — prompt zbiera artefakty L2–L5 w `context/architect-report.md` (~2 strony). Struktura: opisane projekty; mapa (3–5 wniosków); analiza ficzera (overview + debt z ast-grep); plan refaktoryzacji (co, czego NIE, fazy); domena DDD (język, niezmiennik #1, ACL); **„Decyzje, które należą do mnie"** (3–5 zdań: co AI podpowiedziało, co rozstrzygnąłeś sam). Nie wymyślaj — `BRAK artefaktu` jeśli brakuje. Przeczytaj jak recenzent, nie autor.

### Deep dive: Kiedy sięgać po DDD (i czemu nie od pierwszego dnia)

> **Deep dive** — Wczesne DDD ma sens przy głębokiej znanej domenie lub ekspercie obok. Solo/nowy rynek: domena płytka, agregaty to zgadywanie — najpierw walidacja rynku. Kolejność pragmatyczna: **koduj, ucz się domeny, modeluj gdy boli.** Sygnały: `3xAccount`, byt z PRD bez kodu, cotygodniowe ratowanie DB skryptem, wymiana biblioteki = cały sprint. Moment post-MVP, drugi cykl.

### Deep dive: Pięć mitów o DDD

> **Deep dive** — (1) DDD ≠ ciężka architektura — strategia (język, konteksty) bez nowych klas. (2) Nie tylko mikroserwisy — granice w monolicie też. (3) Brak eksperta — LLM obniża koszt wejścia. (4) DDD ≠ Event Storming ani sam agregat — narzędzia, nie całość. (5) Bufet, nie zestaw obiadowy — weź ubiquitous language bez reszty. AI obniża ceremonię; dyscyplina była tania od zawsze.

### Deep dive: Kiedy ekspert domenowy zmyśla

> **Deep dive** — Halucynacje podstępne, bo pytasz o to, czego nie znasz. Model mocny przy ogólnych pojęciach (fiszki, SRS, logistyka); słaby przy precyzyjnych stałych, polach API, regulacjach. Wywiad = **generator hipotez**, nie prawda. Cross-check: dokumentacja źródłowa; dwa modele (`agent-forum`); konfrontacja z kodem i PRD (tryb 2). Ostatnie słowo — źródło, które da się otworzyć.
