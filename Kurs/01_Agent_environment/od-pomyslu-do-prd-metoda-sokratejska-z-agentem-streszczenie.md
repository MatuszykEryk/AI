### Wprowadzenie

Pierwszy ruch w 10xDevs to nie implementacja, lecz `/10x-shape` (wymusza precyzję) i `/10x-prd` (kontrakt na dysku). Dwa artefakty: `shape-notes.md` i `prd.md`. Łańcuch `/10x-shape → /10x-prd → wybór stacku → bootstrap` — pusty pierwszy kontrakt = szybsze dowożenie złych decyzji.

### Dlaczego Agent powinien pytać

Ad-hoc prośba o dokument → model uzupełni braki prawdopodobnymi założeniami; agent potem konsekwentnie implementuje błędne założenia. Kolejność: najpierw pytania, potem dokument.

### PRD jako kontrakt dla kolejnych kroków

Prompt zadaniowy: „zrób teraz tę rzecz". PRD: ramy projektu, w którym ta rzecz ma sens. Bez PRD prompty dryfują (logowanie, dashboard, statystyki — przypadkowy produkt). PRD stabilizuje: użytkownik i problem, zakres, non-goals, kryteria sukcesu, otwarte pytania. Konsumowany w M1L2 (stack), M1L3 (bootstrap).

### Dwa skille, jeden cel

**/10x-shape** — sesja sokratejska; pyta, drąży, nie wymyśla za ciebie. Wynik: `shape-notes.md`. **/10x-prd** — przepisuje notatki do PRD o ustalonej strukturze, wiernie, bez domyślania; braki → `## Open Questions`. Wynik: `prd.md`. Oba działają w trybie brownfield (wykrycie `package.json`, `Cargo.toml` itd.).

### Sesja /10x-shape w praktyce

Przykład **10xCards** (fiszki z AI) — typowe wejście wygląda sensownie, ale nie wystarcza do implementacji:

```text
## 10xCards - MVP
Główny problem: ręczne tworzenie fiszek jest czasochłonne, co zniechęca do spaced repetition.
MVP: generowanie fiszek przez AI (kopiuj-wklej), CRUD fiszek, konta użytkowników, gotowy algorytm powtórek.
Poza MVP: własny algorytm (Anki), import PDF/DOCX, współdzielenie zestawów, mobile.
Sukces: 75% fiszek z AI akceptowanych; 75% fiszek tworzonych z AI.
```

Brakuje odpowiedzi: kto jest użytkownikiem? co znaczy „powtarzanie"? który moment daje wartość? czy to reguła domenowa, czy CRUD z ładnym opisem? Ad-hoc „przygotuj PRD dla aplikacji do fiszek z AI" da profesjonalny tekst bez tych konkretów.

`/10x-shape` prowadzi przez sześć faz: **Vision & problem** („fiszki z AI" → problem konkretnej osoby), **Persona & access control** („użytkownik" → np. dorosły learner zamieniający przeczytany tekst w fiszki, którym ufa), **MVP discipline** (pierwszy przepływ >3 tygodni po godzinach → koszt na stole, świadome potwierdzenie), **FRs + user stories** i **Business logic + data** (wyzwanie sokratejskie per FR: „co musiałoby być prawdą, żeby ten FR był błędny?"; wykrywanie pustego CRUDa — wymuszenie reguły domenowej: generacja z tekstu, bramka akceptacji przed zapisem, algorytm powtórek), **closing soft-gate** — sześć pytań: access control, data model, business logic (jednozdaniowa reguła), project artifacts, MVP-in-three-weeks, non-goals. Twoja rola: odpowiadać konkretnie, także „nie wiem". Wynik: `shape-notes.md` — decyzje, nie transkrypt rozmowy.

### Generacja PRD: /10x-prd

```text
/10x-prd @context/foundation/shape-notes.md
```

Tworzy `context/foundation/prd.md`: wizja, persona, kryteria sukcesu, user stories, FR, reguła biznesowa, model danych, access control, non-goals, open questions. Bez tech stacku, testów, deploymentu — to kolejne skille.

### Ostrzeżenie przed PRD widmo

`/10x-prd` ostrzega, gdy `shape-notes.md` ma za mało konkretów (brak checkpointów, FR-NNN, Given/When/Then, reguły biznesowej) — propozycja powrotu do `/10x-shape`.

### Kontrola po wygenerowaniu PRD

Sprawdź: jeden konkretny użytkownik? pierwszy przepływ w 3 tygodniach po godzinach? logika biznesowa to reguła, nie CRUD? jawne non-goals? `## Open Questions` pusta lub naprawdę otwarta? Niepokój → edycja lub powrót do `/10x-shape`.

### Nie tylko na start projektu

Przy większych zmianach (nowy moduł, integracja) — miniwersja shape. Zanim agent edytuje pliki: jaki problem i po czym skończone.

### Brownfield: sesja na istniejącym projekcie

`/10x-shape` w katalogu z markerami (`package.json`, `tsconfig.json`, `Cargo.toml`, `go.mod`) → skill proponuje tryb brownfield i czeka na potwierdzenie (możesz przełączyć ręcznie). Te same sześć faz, inne pytania:

- **Vision & problem** — co jest dzisiaj, co boli i dlaczego teraz (nie „kto ma problem z niczego").
- **Persona & access control** — jak działa obecne auth i role (nie projektowanie logowania od zera).
- **MVP discipline** — najmniejsza zmiana udowadniająca poprawę + jej blast radius (delta do dowiezienia, nie MVP od pustego repo).
- **FRs & user stories** — wymagania jako nowe / zmodyfikowane / zachowane.
- **Business logic & data** — nowa reguła domenowa, modyfikacja istniejącej, czy zmiana infrastrukturalna.
- **Product framing** — czy zmienia się typ produktu, skala, ograniczenia (bramki tak/nie).

`shape-notes.md` dodatkowo: `## Current System` (co istnieje) i `## Constraints & Preserved Behavior` (co musi zostać). Brownfield PRD opisuje **deltę** (dzisiaj / zmiana / musi zostać), nie cały produkt od zera. Dalej: `/10x-stack-assess` i `/10x-health-check` zamiast selector + bootstrapper.

### Jak korzystać z 10x-cli?

```bash
npx @przeprogramowani/10x-cli@latest auth
npx @przeprogramowani/10x-cli@latest get m1l1
```

Skille lądują w `.claude/skills/`, `.cursor/skills/` lub `.github/skills/`. Paczka m1l1: `/10x-init`, `/10x-shape`, `/10x-prd`. Helper: `npx skills add przeprogramowani/10x-cli` (opcjonalnie `-g`).

**Ścieżka praktyczna (greenfield):** (1) `/10x-init` — scaffold folderu `context/`; (2) `/10x-shape` z mętnym pomysłem (wklejka lub `@plik`); (3) `/10x-prd` → sprawdź `prd.md` (użytkownik, przepływ, reguła biznesowa, non-goals, kryteria sukcesu). **Brownfield:** `/10x-shape` w roocie istniejącego projektu → potwierdź tryb brownfield → `/10x-prd`. Efekt: `context/foundation/shape-notes.md` + `context/foundation/prd.md`.

### Deep dive: jakie modele i narzędzia wybrać

> **Deep dive** — Subskrypcja ($20: Claude Pro / ChatGPT Plus; $100: Claude Max / ChatGPT Pro) — przewidywalny koszt przy intensywnej pracy z AI (intensywna sesja shape = dziesiątki tysięcy tokenów; pełny dzień z agentem = miliony). Prowadzący: Claude Max 5x + Opus 4.6 (nie 4.7 — więcej tokenów przy podobnej jakości analitycznej). Podział kursowy: **architekci** (Opus, Sonnet, Gemini Pro, GPT) — shape, planowanie, review; **implementatorzy** (DeepSeek V4 Flash, Qwen3 Coder, Qwen 3.6 Plus) — kod za ułamek ceny (benchmark implementacyjny ≠ zadania analityczne). **Module 1 = architekci.** Alternatywa: OpenRouter + OpenCode (pełna kontrola kosztów, darmowe tiery). Testuj 2–3 modele na własnym `/10x-shape` — to najlepszy test przed wyborem na stałe.
