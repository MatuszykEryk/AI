### Wprowadzenie

Po M1L1 masz `prd.md` — kontrakt _co_ budujesz. Ad-hoc prompt „wybierz stack" da odpowiedź w sesji, ale bez trwałego artefaktu i reguł powtarzalności. Skille awansują z hierarchii instrukcji (prework 3.2) do pełnego mechanizmu. Na końcu lekcji: `/10x-tech-stack-selector` → `tech-stack.md` (M1L3); brownfield → `/10x-stack-assess`.

### Czym jest Agent Skill

Folder z wymaganym `SKILL.md`: **SKILL.md** (~500 linii; więcej → osobne pliki), **references/** (dokumentacja, rejestry — ładowane na żądanie), **scripts/** (kod wykonywalny przez bash; wynik, nie kod, w kontekście), **assets/** (szablony, ikony).

Progresywne ujawnianie w trzech etapach: (1) **Metadane** (~100 tokenów) — nazwa i opis, widoczne od startu; (2) **Instrukcje** (do ~5000 tokenów) — `SKILL.md` po aktywacji; (3) **Zasoby** (bez limitu) — `references/`, `scripts/`, `assets/` w trakcie pracy. 20 skilli ≈ 2000 tokenów metadanych, nie ~100 000 pełnych instrukcji.

### Skill vs jednorazowy prompt

Prompt to jednorazowa instrukcja w sesji. Skill daje: (1) plik na dysku — wersjonowalny; (2) progresywne ładowanie vs prompt w całości od razu; (3) kontrakt wejścia/wyjścia w procedurze (`prd.md` → `tech-stack.md`); (4) uruchomienie po nazwie (`/10x-tech-stack-selector`); (5) powtarzalność między sesjami (kontrakt stały, wynik modelu może się różnić).

Reguła: **powtarzalny proces → skill; jednorazowa eksploracja/edycja → prompt**. Heurystyka: trzeci podobny prompt → czas na skilla.

| Sytuacja | Skill czy prompt? |
| --- | --- |
| Popraw literówkę w README | Prompt |
| Wyjaśnij funkcję | Prompt |
| Wybierz stack do PRD | Skill (`prd.md` → `tech-stack.md`) |
| Plan implementacji logowania | Skill (`/10x-plan`) |
| Podsumuj sesję | Prompt |

### Skąd brać skille

Rejestry: **anthropics/skills** (oficjalne, w tym **skill-creator**), **skills.sh** (publiczny rejestr), skille producentów (**supabase/agent-skills**, **vercel-labs/agent-skills**). **Audyt przed instalacją** — skille mogą mieć kod w `scripts/`; traktuj jak instalację oprogramowania. Sprawdź `SKILL.md` (cel, `allowed-tools`), `scripts/`, autora i źródło.

### Dwa typy skilli

- **Procesowy** (`/10x-tech-stack-selector`) — jawne wywołanie, czyta plik, produkuje plik; ogniwo łańcucha.
- **Doradczy** (`react-best-practices`) — automatyczna aktywacja wg `description`; wpływa na decyzje, nie produkuje artefaktu.

### Aktywacja: jawna vs automatyczna

Automatyczna aktywacja zakłada, że model rozpozna własną lukę — model nie wie, czego nie wie. W praktyce: skille procesowe wywołuj jawnie; doradcze — automatyczna aktywacja jako pomoc, nie gwarancja; przy ważnym wyniku nazwij skill wprost lub wpisz w reguły projektowe (M1L4).

### PRD → tech-stack.md w praktyce

Ad-hoc: „wybierz stack do @prd.md" → odpowiedź w sesji, bez pliku, bez pytań o preferencje, bez sprawdzonego startera — `/10x-bootstrapper` nie ruszy.

Skill:

```text
/10x-tech-stack-selector @context/foundation/prd.md
```

Wynik `context/foundation/tech-stack.md`:

```yaml
starter_id: 10x-astro-starter
bootstrapper_confidence: first-class
path_taken: standard
has_auth: true
has_ai: true
has_payments: false
has_realtime: false
has_background_jobs: false
```

Plus sekcja „Why this stack". Kontrakt dla `/10x-bootstrapper`: `starter_id`, `bootstrapper_confidence: first-class`.

W `SKILL.md`: `description`, `allowed-tools`, `references/starter-registry.yaml`, `references/handoff-schema.md`, `references/agent-friendly-criteria.md` — ładowane na żądanie, jawny kontrakt między ogniwami.

### Ocena istniejącego stacku

Brownfield → `/10x-stack-assess`: wykrywa komponenty, ocenia przez cztery bramki (te same co selector, ale diagnostycznie): **Typed**, **Convention-based**, **Popular in training data**, **Well-documented**. Wynik: `context/foundation/stack-assessment.md`. Werdykty: **ready**, **ready-with-compensation** (większość brownfield), **significant-friction**. Plan kompensacji → wpisy do `CLAUDE.md`/`AGENTS.md` (M1L4). Obie ścieżki zbiegają się z równoważnym kontekstem.

### Skille jako budulec 10xWorkflow

Łańcuch modułu 2 (ten sam format: `SKILL.md` + `references/` + opcjonalnie `scripts/`): `/10x-research` (pytanie → `research.md`), `/10x-frame` (problem → `frame.md`), `/10x-plan` (problem + research + frame → `plan.md` + `plan-brief.md`), `/10x-implement` (`plan.md` → kod + commity; nigdy `git add -A`), `/10x-impl-review` (`plan.md` + kod → `impl-review.md`). Każdy czyta plik z poprzedniego kroku. Definicja AI-native software engineering: elastyczny zestaw skilli, nie pojedyncze prompty.

### Deep dive: tworzenie własnych skilli

> **Deep dive** — Ścieżka 1: rozmowa z agentem — opisz wejście, wyjście, kroki, czego nie robić; 3–4 iteracje. Szybka, dla skilli pod siebie. Ścieżka 2: **skill-creator** z evalami — automatyczna weryfikacja wersji; tradeoff: koszt, evaly mogą faworyzować happy path. Wybór: pod siebie → ścieżka 1; współdzielony/łańcuch → ścieżka 2; naturalna progresja 1 → 2.

### Deep dive: problemy z automatyczną aktywacją

> **Deep dive** — Eval Vercela (Next.js 16): baseline 53%, skill domyślny 53% (w 56% agent w ogóle nie sięgnął), skill z jawną aktywacją 79%, AGENTS.md z indeksowaną dokumentacją 100%. Wniosek: skille procesowe wywoływane manualnie (`/10x-tech-stack-selector`, `/10x-plan`, `/10x-implement`) — zgodne z praktyką kursu; automatyczna aktywacja doradcza — bonus, nie gwarancja.
