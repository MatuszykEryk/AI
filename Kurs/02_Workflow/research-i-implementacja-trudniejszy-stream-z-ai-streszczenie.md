### Wprowadzenie

Po CRUD (generacja, zapis, edycja) pojawia się `S-04 srs-review-session` — wygląda jak endpoint, nie jest. `/10x-plan srs-review-session` bez przygotowania → pytania o ratingi, politykę powtórek, `due date`, bibliotekę. Workflow nie jest popsuty — logika biznesowa robi się unikalna. Trudniejsze slice'y: **szersze przygotowanie przed planem** + toolkit external researchu. Ten sam workflow `research → plan → implement`, przygotowanie rośnie z ryzykiem.

### Po CRUD mamy karty, ale brakuje powtórek

Checkpoint: użytkownik ma `Flashcard` w decku. Bez sesji powtórek to notatnik. `S-04`: `ReviewState` → które karty dziś → ocena (Again/Hard/Good/Easy lub 1–5) → aktualizacja stanu → kolejny termin. Prostota to pułapka.

### SRS to decyzja kontraktowa

Antywzorzec: traktować S-04 jak każdy slice — change-id, `/10x-plan`, fazy. Pierwsza decyzja: kształt `ReviewState`. SM-2 (Anki): `easeFactor`, `interval`, `repetitions`, 6-stopniowa skala. FSRS: `Difficulty/Stability/Retrievability`, 4 stopnie, edycja karty wpływa na stan inaczej. Każda decyzja propaguje się do modelu danych, UI, API, testów, edge case'ów. Blokuje brak osadzenia w decyzjach domenowych — plan bez researchu → agent wybierze sam, połowie implementacji schemat nie pasuje do API biblioteki. Ten sam wzorzec: subskrypcje SaaS, trasy kurierów, matching marketplace, rekomendacje edukacyjne.

### External research dla agentów

Antywzorzec: „wkleję do ChatGPT, wezmę pierwszą odpowiedź". Chatbot z pamięci treningowej → halucynacje, brak `groundingu`. Agent w harnessie: `WebFetch` (konkretny URL), `WebSearch` (szukanie źródeł) — nierówność między narzędziami (Cursor/Exa, Codex/Bing, Claude Code/Brave wg obserwacji). MCP i dostawcy AI-native uzupełniają luki.

#### Agentic search

Warstwa pod agenta: naturalne zapytania, wyniki w markdown z metadanymi, API pod łańcuch wywołań. Gracze: Brave, Firecrawl, Exa, Tavily, Perplexity, Parallel, SerpAPI. Benchmark AIMultiple (maj 2026, 100 pytań dev): czwórka liderów statystycznie nie do odróżnienia; kategoria **Technical Documentation** — Exa najwyższa jakość. Cursor `@web` używa Exy. Jeśli masz Cursora lub innego lidera — nie migruj odruchowo; testuj na swoim zadaniu.

#### Exa.ai jako AI-native search

MCP, free tier 1000 zapytań. Setup przez konto Exa → prompt na custom command (np. `.claude/commands/exa-init.md`). Użycie: przeszukanie algorytmów SRS i kandydatów bibliotek TypeScript.

#### Context7 jako live library docs

Po wyborze biblioteki (np. `ts-fsrs`) — aktualne API zamiast halucynacji z pamięci modelu. MCP/CLI/skille; free 1000 wywołań. Instalacja poza agentem, nowa sesja, OAuth.

```text
1. resolve-library-id: "ts-fsrs" → /open-spaced-repetition/ts-fsrs
2. query-docs: "/open-spaced-repetition/ts-fsrs" → markdown z docsami
```

Naturalne zapytanie: `Pobierz dokumentację <library_name>, która pozwoli zaimplementować S-04 z roadmapy. Użyj Context7 mcp`. Efekt: plan z `createEmptyCard` i `fsrs().next(card, rating)` zamiast wymyślonych funkcji. Exa + Context7 = content pod LLM (bez paywalla, popupów).

### Internal research na projektowym kodzie

`/10x-research` — równoległe subagenty (`Explore` + `general-purpose`), synteza z `file:line`. **Scope check** przed startem (`AskUserQuestion`). Output: `context/changes/<change-id>/research.md` (Summary, Detailed Findings, Code References, Architecture Insights, Open Questions). Wczytuje `lessons.md` jako znany kontekst.

```text
/10x-research <change-id> <query>
```

Czego **nie** robi: nie wybiera biblioteki SRS (zaproponuje własny harmonogram — pułapka), nie modyfikuje kodu (tylko `research.md`), nie wychodzi poza zakres, nie zgaduje bez referencji plik:linia. `research.md` zasila `/10x-plan` (`@research.md`), `/10x-plan-review`, `/10x-implement`.

### Plan oparty o zebrany research

```text
/10x-plan srs-review-session Build S-04 implementation plan based on @research.md and @ts-fsrs.md
```

Plan zorientowany na rozwiązanie: zamiast „jaki rating scale?" → „`Rating.Again|Hard|Good|Easy` z `ts-fsrs`, cztery przyciski UI". `plan-brief.md` = kontrakt decyzji (dlaczego ta biblioteka, cztery przyciski, edit resetuje stan). Wsad: Exa (uzasadnienie wyboru), Context7 (API), `/10x-research` (konwencje 10xCards).

### Plan review i implementacja do udanego domknięcia

`/10x-plan-review` ważniejszy przy kontraktowych decyzjach (rating scale, `ReviewState`, edit-vs-reset). Potem `/10x-implement` fazowo jak w M2L2. Trudne problemy → mniejsze podzadania. Kolejność: kontekst → research (wewnętrzny + zewnętrzny) → plan → implementacja.

### Context drift, czyli kiedy agent kaskaduje błędy

**Context drift** — kaskada drobnych błędnych decyzji → spójny kod pod zły problem. Źródła: **brak przygotowania** (zgadywanie zamiast `/10x-research`, Exa, Context7) i **przeciąganie rozmowy** (context rot — jakość spada ze wzrostem tokenów, „lost in the middle"; Chroma „Context Rot", Anthropic context engineering). Objawy: powtarzanie poprawek, przekręcone nazwy, mieszanie faz/slice'ów. Odpowiedź workflow: research w źródłach, plan w pliku, fazy z commitami, `## Progress` dla świeżej sesji. Sygnał driftu → nowa sesja + `plan.md` + Progress, nie „popraw jeszcze raz".

### SRS domknięty, ale co, gdy plan się nie układa?

Trzy objawy tego samego: **błędny framing** — problem nazwany źle, research produkuje szczegóły na złe pytanie. Obserwacja ≠ przyczyna ≠ fix sklejone w jedno zgłoszenie.

### Zadania praktyczne (opcjonalne)

Research i framing tylko przy szczególnej złożoności slice'a — nie komplikuj planowania bez potrzeby. Ćwiczenie external: Exa MCP + Context7 MCP, świeża sesja, 2–3 kandydatów przez Exę, API zwycięzcy przez Context7 — cel: jedno zdanie „wybieramy X, bo …" z wycinkiem aktualnej dokumentacji. Ćwiczenie pełna pętla: `/10x-research <change-id> <query>` → `research.md` → `/10x-plan` z `@research.md` i docsami — porównaj `plan-brief.md` z planem „z głowy"; otwarte pytania jako jawne `Open Questions`, nie zgadywanie.

### `/10x-frame` jako koło ratunkowe

Gdy: brak ekspertyzy, niepewność kierunku, poprzednie próby bez efektu. Przykład Space Explorers: „kolizje winne" — naprawiano złą warstwę. `/10x-frame` wymusza **Frame Brief**: obserwacja, zakładana przyczyna, proponowany fix — osobno, zanim plan. Trzy zasady: **nie część głównego workflow** (koło ratunkowe; MVP zwykle `research → plan → implement`), **nie zastępuje researchu** (frame = złe nazwanie problemu; czasem frame → research → plan), **nie każdy problem to misframing** (mały rozjazd w sesji vs wielokrotne niepowodzenia).

### Deep dive: Agent-friendly docs

> **Deep dive** — Link do HTML z JS, reklamami i cookie-bannerami utrudnia agentowi nawigację. Trend: markdown dla agentów. **`llms.txt`** — krótki indeks „zacznij stąd" (nie robots.txt). **Cloudflare `Accept: text/markdown`** — treść bez chrome, nagłówek `x-markdown-tokens` do budżetowania. **`.md` w URL** (np. Vercel: `/docs/deployments.md`). Agent na HTML marnuje tokeny na szum.

### Deep dive: Widoczność usługi w internecie agentów

> **Deep dive** — W 2026 produkty często wybiera agent, nie programista (Lavanya Shukla). Sygnały: Resend (63% propozycji Claude Code przy „dodaj mail"), Supabase (markdown docs + SDK), **First Successful Execution** — pierwsze udane wywołanie bez człowieka wzmacnia pętlę. „Your product is no longer your UI. It's your API." Exa i Context7 filtrują też projekty chcące być znalezione przez agenta.
