### Wprowadzenie

Hooki operują na kodzie — nie widzą, że deck znika po odśkieżeniu (auth → API → baza). Potrzebne testy **end-to-end** + agent z dostępem do przeglądarki. Antywzorzec: E2E, który przechodzi, ale nie chroni ryzyka. Kontrola: **seed test** + **reguły** ograniczające output agenta.

### Jak agent widzi aplikację

Agent patrzy na **drzewo dostępności** (role, nazwa, stan) — snapshot YAML z referencjami `e15`, deterministyczny, bez VLM. Naturalnie powinien używać `getByRole`, nie CSS.

### Playwright CLI jako interfejs agenta

```bash
npm install -g @playwright/cli@latest
playwright-cli open http://localhost:3000 --headed
```

Komendy: `click`, `fill`, `screenshot` — snapshoty na dysku. **CLI ~4× mniej tokenów niż MCP** przy tym samym scenariuszu — domyślny wybór przy dużym repo.

### Sesja bez logowania

Antywzorzec: każdy test przez formularz logowania. **storageState** — loguj raz, wstrzykuj ciasteczka:

```bash
playwright-cli state-save auth.json
```

`playwright.config.ts`: project `setup` + `dependencies`, `storageState: 'playwright/.auth/user.json'`. `auth.json` → `.gitignore`. Osobne scenariusze login/register, nie zależność dla reszty.

### Od ryzyk do testów E2E

Wejście z `test-plan.md`: 1–3 najwyższe ryzyka wymagające przeglądarki. Heurystyka E2E: wiele granic (auth, routing, API, baza) lub tylko w UI; izolowana funkcja → unit (M3L2). E2E ≠ zero mocków — wewnętrzne granice prawdziwe; drogie zewnętrzne (LLM, płatności) mock na HTTP. E2E wolne i kruche — ostrożnie z liczbą. Bez frontendu: scenariusze API request→baza.

### `/10x-e2e`

Skill świadomy `/10x-implement` i `/10x-tdd` — ten sam `plan.md`, `## Progress`. Pętla fazy:

```text
PLAN → GENERATE → REVIEW → VERIFY
```

```bash
npx @przeprogramowani/10x-cli@latest get m3l4
```

Bramka przed generowaniem: ryzyko na poziomie UI? feature istnieje? brak duplikatu E2E? Inaczej → `/10x-tdd`/`/10x-implement`. Mieszanie faz w jednym planie OK. PLAN: planner→generator (Playwright 1.56: Planner, Generator, Healer) albo szablon promptu.

### Seed test i reguły jakości

`seed.spec.ts` — wzorzec dla Generatora („co pokażesz, to dostaniesz"). Cztery wzorce: **getByRole**; **niezależność** (setup→akcja→asercja→cleanup w jednym teście); **czekanie na stan** (`toBeVisible`, `waitForURL`, nie `waitForTimeout`); **nazwa powiązana z ryzykiem**. Reguły i 5 antywzorców w referencjach skilla (`e2e-quality-rules.md`, `e2e-anti-patterns.md`).

```typescript
// seed.spec.ts
import { test, expect } from "@playwright/test";

test("created deck persists after page reload", async ({ page }) => {
  const deckName = `Test Deck ${Date.now()}`;
  await page.goto("/");

  await page.getByRole("button", { name: "New deck" }).click();
  await page.getByRole("textbox", { name: "Deck name" }).fill(deckName);
  await page.getByRole("button", { name: "Create" }).click();

  await expect(page.getByRole("heading", { name: deckName })).toBeVisible();

  await page.reload();
  await expect(page.getByRole("heading", { name: deckName })).toBeVisible();

  // Cleanup
  await page.getByRole("button", { name: "Delete deck" }).click();
  await page.getByRole("button", { name: "Confirm" }).click();
});
```

`Date.now()` w nazwie = unikalny identyfikator (izolacja przy równoległym uruchamianiu). Prompt E2E (`e2e-prompt-template.md`) uzupełnia skill czterema polami, których seed nie zna: **ryzyko** z `test-plan.md`, **research anchor**, **scenariusz biznesowy** (jedno obserwowalne zachowanie → asercja), **granice** prawdziwe kontra mockowane. Seed i reguły kształtują formę testu; prompt dodaje kontekst ryzyka i ścieżki.

### Pięć antywzorców E2E

1. **Naiwna asercja** — sprawdza tytuł strony zamiast przetrwania danych. Pytanie: _czy asercja padnie, gdy ryzyko z test-plan się zmaterializuje?_
2. **Kruchy selektor** — `nth-child` vs `getByRole`.
3. **Współdzielony stan** — test B zakłada test A → flaky przy równoległości.
4. **`waitForTimeout`** — pada w wolniejszym CI.
5. **Brak cleanup** — `unique constraint` przy drugim przebiegu.

Agent optymalizuje „zielony teraz", nie stabilność jutro.

### Re-prompting

Nie „popraw test" — nazwij antywzorzec, wyjaśnij brak ochrony ryzyka, podaj wzorzec. Przykład naiwnej asercji:

```text
The final assertion checks the page title instead of verifying that
flashcard data survived the reload. This test passes even when
Risk #1 materializes (data loss after refresh).

Replace it with assertions on the actual business outcome: deck heading
and card content must be visible after page.reload(). The test must fail
if the data is lost.
```

### VERIFY: zielony to za mało

Po zielonym: **celowe psucie** chronionego zachowania — test ma paść; jeśli nie, wróć do GENERATE. Skill cofa zepsucie, nie commituje. Izolacja: unikalne ID (`Date.now()`) + cleanup/`afterEach`; Supabase RLS — teardown na tym samym koncie lub service role.

### MCP i tryb wizyjny

MCP: ~114K tokenów vs CLI ~27K; bogatsze narzędzia (mock route, trace). Heurystyka: agent koduje + testuje → **CLI**; dedykowana sesja tylko przeglądarka → MCP. `--caps=vision,network,storage`. Vision: współrzędne, layout — uzupełnienie DOM; w testach preferuj `toMatchSnapshot`/Argos. Otwarte pytania diagnostyczne → M3L5 debug, nie E2E.

### E2E w pipeline jakości

E2E w **CI** (minuty), nie per-edit. **Healer** naprawia selektory po refaktorze UI; przy zmianie logiki biznesowej maskuje bugi — granica między auto-fix a debugowaniem (M3L5).

### Deep dive: browser.bind() i współdzielone sesje

> **Deep dive** — Playwright 1.59: jedna przeglądarka dla CLI, MCP i test runner; `page.screencast` jako dowód scenariusza.

### Deep dive: Composable fixtures vs POM

> **Deep dive** — Fixtures zamiast Page Object Model (~30% mniej kodu); POM przy 200+ testach i wielu inżynierach. Seed popycha agenta ku fixture setup/teardown.

### Deep dive: Workflow E2E jako skill

> **Deep dive** — `/10x-e2e` = kontrakt + progresywne ujawnianie (references/). Skill orkiestruje seed i reguły, nie zastępuje. Dostosuj referencje pod Cypress/WebdriverIO/API-only.
