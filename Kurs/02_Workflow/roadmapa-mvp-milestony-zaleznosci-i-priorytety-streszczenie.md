### Wprowadzenie

Antywzorzec po module 1: skopiować `prd.md` i napisać „zbuduj MVP". Agent bez kierunku, priorytetów i mapy ryzyka pracuje niedeterministycznie (endpointy vs UI, migracje vs runtime) — tokeny spalone, powtarzalności brak. To nie błąd agenta, lecz za szeroki cel. **Agent pracuje tobą.** Moduł 2: rola Technical Project Managera — sekwencja milestone'ów zamiast „co teraz zakodować".

### Programista jako Technical Project Manager

TPM łączy cel produktowy z wykonaniem technicznym: kolejność pracy, ryzyko, blokery, capacity, ownership, synchronizacja. Agent przyspiesza kod, ale też pozorny postęp przy złej sekwencji. Kompetencje TPM wpływają na obsługę agentów — nie chodzi o spotkania i statusy. Projekt jako system naczyń połączonych; każde zadanie to wycinek. Cztery decyzje przed delegowaniem: **cel** (co po pierwszym milestone i dlaczego), **sekwencja** (kolejność, parking), **ryzyko** (gdzie się wywali, co redukuje), **capacity i ownership** (kto/co robi, synchronizacja, równoległość). Agent dostaje nie tylko „co zrobić", ale „to milestone 1 z pięciu, tu zależności".

### Łańcuch artefaktów

Zoom-levele tego samego projektu — pomieszanie poziomów to główna przyczyna chaosu, AI go wzmacnia:

- **`shape-notes.md`** — surowa sesja sokratejska, decyzje, otwarte pytania.
- **`prd.md`** — kontrakt produktowy: user stories, FR-y, success criteria, Non-Goals.
- **`tech-stack.md`** — hand-off techniczny: framework, bazy, hosting, auth, deploy.
- **`roadmap.md`** — sekwencja: foundations, slice'y e2e, zależności, blokery.
- **Task (change-id)** — jednostka operacyjna: status, owner, acceptance criteria.
- **`context/changes/<change-id>/plan.md`** — plan implementacji jednego slice'a.

PRD = co i dla kogo; tech-stack = czym; roadmapa = w jakiej kolejności; backlog = co teraz; plan = jak konkretnie.

### Terminologia i kody zadań

Prefixy w `/10x-roadmap`: **F-XX** (foundations) — techniczne i niefunkcjonalne, odblokowujące slice; **S-XX** (slice) — full-stackowe przepływy składające się na wartość dla użytkownika.

### Vertical-first jako domyślna strategia

**Vertical slice** — jeden przepływ przez UI, dane, logikę, integracje; kończy się czymś weryfikowalnym przez użytkownika. **Horizontal slicing** — warstwa po warstwie (baza → API → UI); naturalne dla zespołów funkcyjnych, słabe z agentem. Dlaczego vertical-first: **weryfikowalność** (klik, E2E, screenshot vs „warstwa istnieje"), **integracja** (konflikty interfejsów od razu), **pozorny postęp** (pełne katalogi bez wartości dla użytkownika). North star slice — najmniejszy działający przepływ udowadniający tezę produktu (10xCards: S-01 „First gated generation loop").

#### Co z fundamentami?

Fundamenty w sekcji `## Foundations`. Każde **F-NN** musi mieć `Unlocks: S-NN` — inaczej to horizontal drift na parking. Przykłady uzasadnione: `F-01 minimal-persistence-and-auth` → odblokowuje S-01; `F-02 openrouter-privacy-spike` → S-01; `F-03 srs-library-spike` → S-03. Źle: „kompletny model danych dla wszystkich encji" bez downstream slice. Żaden fundament bez konkretnego vertical milestone, który odblokowuje.

### Skill /10x-roadmap w praktyce

```bash
npx @przeprogramowani/10x-cli@latest get m2l1
```

`/10x-roadmap` czyta `prd.md`, audytuje repo, sekwencjonuje pionowe milestone'y z fundamentami. Nie wybiera frameworków (`tech-stack.md`), nie projektuje schematów (`/10x-plan`), nie pisze planu slice'a (`/10x-plan <change-id>`). Decyduje: **co najpierw, w jakiej kolejności, co odblokowuje co**.

```text
/10x-roadmap
```

Cztery kroki:

1. **Gotowość PRD** (skala 0–4): Vision, user stories Given/When/Then, must-have FR, Business Logic sensowna. Wynik <3 → rekomendacja domknięcia PRD; można kontynuować świadomie — dziury staną się `blocked slices`.
2. **Audyt kodu** — równoległe subagenty na 6 warstw (frontend, backend/API, data, auth, deploy, observability): present/partial/absent + plik-dowód.
3. **Minimalny wywiad** (max 3 pytania, punkt kontroli człowieka): `main_goal` (czas/feedback/jakość/dostępność/nauka), `north_star` (najmniejszy przepływ udowadniający tezę), `top_blocker` (decyzje/czas/dostępność/zewnętrzne/wiedza/motywacja/brak). Agent z rekomendacją i alternatywami — ty zatwierdzasz.
4. **Generuje `context/foundation/roadmap.md`** — foundations z absent/partial, kolejność, backlog handoff, parked.

Struktura wyjścia: `## Vision recap`, `## North star`, `## At a glance`, `## Streams` (opcjonalnie), `## Baseline`, `## Foundations`, `## Slices`, `## Backlog Handoff`, `## Open Roadmap Questions`, `## Parked`, `## Done`. Skill nie estymuje czasu (brak Day 1, story points — nieliniowość pracy z agentem). Format `SKILL.md` możesz dopasować do własnego workflow po kilku iteracjach.

**Zadanie:** w repo z gotowym `context/foundation/prd.md` uruchom `/10x-roadmap` i przejdź wywiad świadomie — sam wybierz `main_goal`, `north_star` i `top_blocker`, nie domyślne rekomendacje. Cel: ≥1 slice w statusie `ready`, jasny north star, brak osieroconych fundamentów. Wszystkie slice'y `blocked` → sygnał do domknięcia lub doprecyzowania PRD.

#### Anatomia jednego slice'a

Pola: **Outcome** (czasownik: „user can …"), **Change ID**, **PRD refs** (konkretne ID — self-review: każdy must-have w ≥1 slice), **Prerequisites**, **Parallel with**, **Blockers**, **Unknowns** (Owner, Block: yes → status `blocked`), **Risk** (jedno zdanie: dlaczego tu w sekwencji), **Status**. Outcome bez zdania = slice prawdopodobnie nie jest pionowy.

### Roadmapa 10xCards: jeden konkret

North star to nie S-01 (generacja), lecz **S-04** (pierwsza ocena w sesji powtórki) — milestone walidacyjny domykający tezę produktu; S-01/S-02 są prerekwizytami. F-01/F-02 małe fundamenty równoległe. S-04 `blocked` dopóki nie wybrano biblioteki SRS (ReviewState, skala ocen, polityka edycji). Acceptance criteria wyciągasz z Outcome, PRD refs, Risk, Unknowns — np. S-02: jawna akceptacja/odrzucenie, atomowy Save, brak pending/rejected w decku, testy `FlashcardDraft`. `main_goal: speed` + `top_blocker: time` → observability, import PDF, własny SRS w `Parked`.

### Backlog jako pamięć projektu

`roadmap.md` = poziom projektu; zewnętrzny backlog (Jira, Linear, GitHub Projects) = operacyjna pamięć + widoczność dla stakeholderów (status, ownership, dependencies, acceptance criteria, audit trail). Proaktywna komunikacja zmniejsza micromanagement.

### MCP i CLI: agent dostaje dostęp do systemu zadań

Kontrolowany dostęp agenta do issue trackerów: **Read** („pokaż otwarte z labelem `ready`"), **Create** (issue z outcome i AC), **Update** (status, komentarz z linkiem do PR). Dostęp przez **CLI** (np. GitHub CLI `gh`) lub **MCP** (np. Linear). Opcjonalnie: agent tworzy issues z `## Slices` — tytuł z `Outcome`, opis z `PRD refs`, `Prerequisites`, `Risk`, labelki **`slice`** / **`foundation`** / **`north-star`**, zależności między ticketami.

**Linear MCP** (`https://mcp.linear.app/mcp`, OAuth):

```text
claude mcp add --transport http linear-server https://mcp.linear.app/mcp
codex mcp add linear --url https://mcp.linear.app/mcp
```

Cursor: connector w marketplace. Agent jako opiekun backlogu: sprzątanie, aktualizacja kontekstu, triage, raporty — niski koszt pomyłki. Granice: token z ograniczonymi uprawnieniami, akcje destrukcyjne tylko w UI, audit log, ustalenia zespołowe (czyje statusy może zmieniać).

### Deep dive: Dlaczego rekomendacja zmieniła się od 10xDevs 2.0

> **Deep dive** — Wcześniej horizontal-first (człowiek ręcznie zszywał warstwy). W 2026 agenci (Claude Code, Codex) działają najlepiej na zawężonym zadaniu z weryfikowalnym wynikiem — vertical slice to praktyczne story splitting i walking skeleton z Agile. Fundamenty nadal istnieją w `## Foundations`, ale każdy musi wskazywać konkretny slice. Horizontal-first dopuszczalny przy legacy spike, compliance przed UI, performance-critical core — zawsze ograniczony i przypięty do `Unlocks: S-NN`. Pięć F-NN bez `ready` slice = sygnał problemu z PRD lub baseline.
