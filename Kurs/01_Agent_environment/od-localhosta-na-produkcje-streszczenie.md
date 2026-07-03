### Wprowadzenie

Moduł 1 domyka się pierwszym deploymentem produkcyjnym. Lokalna aplikacja dla użytkowników nie istnieje. Wybór platformy to de-risking na cały cykl projektu — dopasowany do stacku i agentów, nie „pierwszy poradnik z Google".

### Łańcuch artefaktów modułu 1

PRD (co, dla kogo), tech-stack (z czego), scaffold, AGENTS.md (konwencje). Brakuje: **gdzie i jak działa** — wypełnia `infrastructure.md`.

### Wybór platformy jako świadoma decyzja

Prework 4.1 rekomenduje Cloudflare pod kurs 1:1, ale pytanie brzmi: dlaczego dobry wybór dla _tego_ projektu? Trzy pytania: pasuje do runtime i buildu (SSR vs static, edge vs Node)? sensowny tryb z agentem (CLI, MCP, logi przez API)? odpowiedź na rollback, skalowanie, koszt przy 10× ruchu bez paniki?

### 10x-infra-research: research z agentem AI

```text
/10x-infra-research
```

Wsad: `context/foundation/tech-stack.md`. Proces: (1) wywiad (procesy długotrwałe, region EU, open source, budżet — `nie wiem` jest OK); (2) web research (dokumentacje, status feature'ów, pricing, ograniczenia); (3) scored comparison — Pass/Partial/Fail po pięciu kryteriach agent-friendly (CLI, managed/serverless, dokumentacja dla agenta, deployment API, MCP/CLI). Skill to procedura + narzędzia (`WebFetch`, `AskUserQuestion`); nie wybiera za ciebie — zostawia tabelę.

### Anti-Confirmation Bias prompting

Modele dostosowują się do twoich przekonań ([sycophancy](https://openai.com/index/sycophancy-in-gpt-4o/)). Zamiast „czy X nadaje się pod Y?" → „dlaczego X to zły wybór pod Y?". Trzy testy wbudowane w skill: **Devil's advocate** (brutalna lista słabości zwycięzcy), **Pre-mortem** (załóż porażkę za 3 miesiące — jakie założenia były błędne), **Unknown unknowns** (compliance, backupy, vendor lock-in, status GA vs beta). Decyzje: akceptuję ryzyko / dopisuję do `infrastructure.md` / zmieniam wybór.

Dodatkowe techniki (prompt):

```text
Rozważam {{decyzję X}}.

Zamiast oceniać mój wybór, przedstaw mi trzy realne alternatywy. Dla każdej zrób
wiersz w tabeli Markdown z kolumnami: koszt wdrożenia, długoterminowy koszt
utrzymania, krzywa uczenia, kluczowe ograniczenie.
```

```text
Rozważam {{decyzję X}}.

Wciel się po kolei w trzy role z mojego zespołu - frontend developera, osoby
odpowiedzialnej za bezpieczeństwo i osoby odpowiedzialnej za koszty. Dla każdej
napisz, jak ta decyzja wpłynie na ich codzienną pracę i priorytety. Jeśli choć
jedna z tych osób zareaguje negatywnie, zaproponuj realną alternatywę.
```

### infrastructure.md - nowy kontrakt

```text
context/foundation/infrastructure.md
```

Trzeci kontrakt w łańcuchu. Szkielet: wybór i uzasadnienie, dopasowanie stacku, CLI/MCP/API fit, preview deploymenty, sekrety, rollback (minuty i kroki), uprawnienia (człowiek vs agent), ryzyka z anti-bias, decyzje techniczne, sygnał zmiany przy wyjściu z MVP. Wsad do Plan Mode w module 2; zapis decyzji na przyszłość (odpowiednik ADR). Treść jest twoja — skill to silnik, nie redaktor.

### Deployment z Plan Mode

Wsad: `infrastructure.md` + `tech-stack.md`. **Plan Mode** — agent read-only: czyta, dopytuje, nie mutuje; efekt = plan krok po kroku (Claude Code: `Shift+Tab`). Workflow: (1) inicjalizacja Plan Mode; (2) ocena planu — poprawki wprost; (3) zatwierdzenie → wykonanie; (4) plan zostaje w repo (`context/deployment/deploy-plan.md`). Plan powinien zawierać: kroki agenta, ręczną konfigurację kont, sekrety, komendy (`wrangler pages deploy` vs `wrangler deploy`). Obserwuj, czy agent nie wychodzi poza plan. Przy problemach z planem → `/10x-lesson`.

### Przygotowania do wdrożenia

Dla 10xCards: konto Cloudflare (darmowy plan), `npx wrangler` + `wrangler login`, Supabase (Postgres — URL i klucz publiczny), GitHub CLI (`gh auth login`).

### Realizacja wdrożenia

Po deployu: infrastruktura stoi, agent ma `wrangler` i `gh`, aplikacja pod publicznym URL. Sprint zero domknięty — w module 2 funkcjonalność MVP.

### Deep dive: CLI vs MCP

> **Deep dive** — **CLI** (wrangler, gh, npm): jawne, audytowalne, zero kosztu kontekstu do wywołania; bezpieczne defaulty (np. `netlify deploy` = draft). **MCP**: auto-discovery, strukturalny JSON, sens w cloud agent / dziesiątkach kontekstowych zapytań; koszt — definicje narzędzi w kontekście. Code Mode Cloudflare (`mcp.cloudflare.com/mcp`): `search()` + `execute()`, ~1000 tokenów vs ~1.17M przy pełnym API. Konfiguracja `.mcp.json`. Wzorzec MVP: zacznij od CLI, dołóż MCP przy powtarzalnych wzorach.

### Deep dive: Wrangler i gh

> **Deep dive** — Wrangler: `whoami`, `pages deployment list`, `pages deployment tail`, `pages secret list`. gh: `auth status`, `pr list`, `pr view`, `pr create --fill`.

### Deep dive: granica dostępu agenta do produkcji

> **Deep dive** — Czy dostęp jest konieczny? MVP: tak do diagnostyki/logów, nie do drop bazy ani rotacji DNS. **Minimalne uprawnienia, ludzka autoryzacja na nieodwracalnym.** Token API zawężony do jednego projektu; w env, nie w repo; destrukcja manualnie w panelu. Ewolucja: staging z pełnym dostępem agenta, produkcja read-only.
